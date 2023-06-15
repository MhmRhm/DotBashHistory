# DotBashHistory
This file contains some useful Linux commands and configs.

# General
```bash
history -d -2
```
to remove the last executed command.
```bash
chsh
```
to change the shell.

```bash
ls -alhF .
```
to list all files and directories with inodes and readable size.

```bash
find / -regex .*initrd.img.* -print
```
to find all paths containing `initrd.img`.

```bash
ls .[^.]*
```
to list all hidden files and directories not including `.` and `..`.

```bash
tar -zcvf arch.tar.gz file dir
tar -ztvf arch.tar.gz
tar -zxpvf arch.tar.gz
# tar xvpf arch.tar.xz # for tar.xz files.
```
to create then show the table of contents for the archive then extract it.

```bash
ps ax 2>&1 1>> stdout.txt && cat stdout.txt
```
to list all processes and send outputs to appropriate files.

```bash
PATH=first:$PATH:last
```
to add directories to the search path.

```bash
mknod mypipe p
tail -f mypipe &
man man > mypipe
unlink mypipe
```
to create a named pipe and connect two processes through it.

```bash
#!/bin/bash

PORT=17356

if [ ! -f /etc/ssh/sshd_config_bak ]; then
        cp /etc/ssh/sshd_config /etc/ssh/sshd_config_bak
fi

sed -i 's/#Port 22/Port '"$PORT"'/' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
sed -i 's/#PermitTunnel no/PermitTunnel yes/' /etc/ssh/sshd_config

if [ -f ./users.txt ]; then
        rm ./users.txt
fi

MYIP=$(curl -s https://checkip.amazonaws.com)

for i in {1..2}; do
        PASS=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 20 | head -n 1)
        deluser "user$i" &> /dev/null
        useradd "user$i" --shell /sbin/nologin
        echo "user$i:$PASS" | chpasswd
        echo "ssh://user$i:$PASS@$MYIP:$PORT#Profile $i" >> users.txt
done
cat users.txt
```
to setup SSH for VPN use and automatically add users. Later import profiles to NetMod Syna.

```bash
watch "ps aux | grep ssh | grep -Eo '^[^ ]+' | sort | uniq"
```
to list online users.

```bash
nload -U M -t 3000 eth0
```
to monitor network traffic.

```bash
ls -al /usr/share/zoneinfo/
```
to list available timezones.

```bash
TZ=Iran date
```
to show time in a timezone.

```bash
/usr/bin/time command
```
to show CPU time and memory faults after running command.

```bash
pidstat -ru -p <PID> 1
```
to show memory and CPU usage of `<PID>` every second.

```bash
netstat -ntl
```
to list all TCP connections in use.

```bash
sudo startx /usr/bin/gedit
```
to start a graphical application without a desktop manager.

# Journal
```bash
journalctl -r SYSLOG_IDENTIFIER=sudo
```
to list the most recent commands ran with `sudo`.

```bash
journalctl -S -1h30m
```
to list all logs since one and a half hours ago.

```bash
journalctl -S 06:00:00 -U 07:00:00
```
to list all logs since 6 AM until 7 AM.

```bash
journalctl -fkp 3..4
```
to show a live feed for kernel messages including only errors and warnings.

```bash
udevadm monitor
```
to monitor events passed between udev and kernel.

```bash
udevadm info /dev/sda
```
to get all information for a device.

```bash
journalctl --unit=ssh.service
```
to list logs related to a service.

# Disks and Partitions
```bash
lsblk
```
to list all block devices.

```bash
parted -l
cat /proc/partitions
```
to list partitions.

```bash
fdisk /dev/sdx
```
to manipulate partitions on a device.

```bash
mkfs -t ext4 /dev/sdx
```
to create a filesystem on a partition.

```bash
mkfs -n /dev/sdx
```
to not create a filesystem; instead, show where superblock backups will be.

```bash
fsck -p /dev/sdx
```
to automatically fix minor issues.

```bash
fsck -b 32768
```
to replace the superblock with backup at 32768.

```bash
mount -t ext4 /dev/sdx mpoint/
umount /dev/sdx
```
to mount and unmount a filesystem.

```bash
blkid
mount UUID=662A-CC93 mpoint/
mount PARTUUID=a0ccdae8-d24c-774c-8d71-3814390703bd mpoint/
```
to list UUIDs and mount using UUIDs.

```bash
mount -t vfat /dev/sdx mpoint/ -o ro,noexec,nosuid,uid=1000
mount -t vfat /dev/sdx mpoint/ -o rw,noexec,nosuid,uid=1000,remount
```
to mount a partition read-only then remount it with writing enabled.

```bash
mount -n -o remount /
```
to remount a read-only root filesystem without changing `/etc/mtab`.

```bash
mount -a
```
to mount all partitions in `/etc/fstab` except `noauto` ones.

```bash
mount -t tmpfs -o size=10M tmpfs mpoint/
```
to create a filesystem on RAM.
```bash
du -sh *
```
to get folder and file sizes in a directory.

```bash
df
```
to get filesystems disk usage.

```bash
pvs && vgs && lvs
pvdisplay && vgdisplay && lvdisplay
```
to list physical volumes, volume groups, and logical volumes.

```bash
vgcreate myvg /dev/sdx
vgextend myvg /dev/sdy
lvcreate --size 10g --type linear -n mylv1 myvg
lvcreate --size 10g --type linear -n mylv2 myvg
mkfs -t ext4 /dev/mapper/myvg-mylv1
lvremove myvg/mylv2
lvresize -r -l +100%FREE myvg/mylv1
```
to create, remove and resize logical volumes.

# Kernel Bootup
```bash
cat /proc/cmdline
```
to show parameters passed to the kernel by the boot loader.

```
systemd.unit=multi-user.target ro quiet splash video=HDMI-A-1:D
```
to boot into text mode with GRUB press `e` and add the above kernel arguments.

```
setenv extraargs ${extraargs} systemd.unit=rescue.target; env print;
```
to add rescue mode to kernel arguments using [uboot](https://u-boot.readthedocs.io/en/v2021.04/index.html) on OrangePi 5 and print environment variables.

```
mmc dev 1; setenv scriptaddr 0x00500000; setenv prefix /; setenv script boot.scr; load ${devtype} ${devnum}:${distro_bootpart} ${scriptaddr} ${prefix}${script}; source ${scriptaddr}
```
to boot into MMC using uboot on OrangePi 5.

```
setenv devtype usb; setenv devnum 0; setenv distro_bootpart 1; setenv scriptaddr 0x00500000; setenv prefix /; setenv script boot.scr; usb start; load ${devtype} ${devnum}:${distro_bootpart} ${scriptaddr} ${prefix}${script}; source ${scriptaddr}
```
to boot into USB using uboot on OrangePi 5.

```
videoinfo
terminal_output console
set gfxmode=1280x1024 # set gfxmode=auto
set # print env
terminal_output gfxterm
```
to list available resolutions in GRUB, set the resolution, and print environment variables.

```
ls ($root)/
```
to list files and directories in the GRUB root directory.

```
linux /($root)/vmlinuz quiet splash systemd.unit=graphical.target
initrd /($root)/initrd
boot
```
to manually boot from the GRUB command line.

# Systemd
```bash
systemd-cgls
```
to list control groups.

```bash
systemctl list-units --all --full
```
to list all systemd units.

```bash
systemd-run --on-calendar='02-14 12:46:00' /bin/wall "Buy Chocolate!" # once a year
systemd-run --on-active=1m /bin/wall "Wish you luck!" # a minute in future
systemctl list-timers
systemctl stop <UNIT>
```
to schedule a task using systemd.

```
[Unit]
Description=My Service
After=network.target

[Service]
Type=idle
WorkingDirectory=<dir>
User=<username>
ExecStart=<dir>/.venv/bin/python3 <dir>/script.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
to add a new service to systemd. Create the service file at `/etc/systemd/system/`. Run `systemctl daemon-reload` afterward.

```bash
systemctl show <unit>
```
to list all properties for a unit.

# File Sharing
```bash
python3 -m http.server
```
to share a directory over a secure network.

```bash
rsync --rsh='ssh -p<port> -i mainkey.pem' -avn --delete mydir <username>@<ipaddress>:/home/<username> | grep deleting
```
to list files that will be deleted after syncing two folders.

```bash
smbclient -U <username> -L <server>
```
to list shared folders on a Windows server where <username> is the user name on the Windows machine and <server> is the IP address or Computer Name.

```bash
sudo mount -t cifs '\\<ip_address>\<share>' <mountpoint> -o user=<username>,pass=<password>
```
to mount a shared Windows folder on Linux.

```bash
sshfs username@address:dir mountpoint -p port
fusermount -u mountpoint
```
to mount another Linux directory on the network using ssh.

# Networking
```bash
ip route show
ip route del default
ip route add default via 192.168.1.1 dev eth0
```
to replace the default route (0.0.0.0/0) with gateway at 192.168.1.1 for device eth0.

```bash
ip route add 10.23.2.0/24 via 192.168.1.100
ip route del 10.23.2.0/24
```
to route traffic from 192.168.1.0/24 subnet to 10.23.2.0/24 subnet through gateway at 192.168.1.100.

```bash
host google.com
host 216.239.38.120
```
to get the hostname or IP address for eighter of them.

```bash
cat /etc/services
```
to list predefined ports.

```bash
cat /var/lib/dhcp/dhclient.leases
```
to list IP address leases.

```bash
iptables -L
```
to list firewall rules.

```bash
ip neigh show
```
to list ARP cache.

```bash
iw dev wlan0 scan
iw wlan0 connect '<network_name>'
```
to list wireless networks and connect to one.

```bash
curl --trace-ascii - https://checkip.amazonaws.com
```
to print all data transferred during page requests.

```bash
tar zcvf - directory | ssh -i mainkey.pem -p 17456 remote_user@remote_address tar zxvf -
```
to copy a directory to a remote host.

```bash
scp -o Port=17456 -o IdentityFile=mainkey.pem remote_user@remote_address:file /local_directory
```
to copy a file from a remote host.

```bash
lsof -iTCP:17456 -n -P
```
to list processes connected to port 17456.

```bash
lsof -U -n -P
```
to list Unix Domain Sockets.

```bash
tcpdump -i wlan0 -nXXSs 0 port not 22 and ip
```
to show all traffic but SSH on IPv4.

```bash
netcat -l 17456
```
```bash
netcat 192.168.1.100 17456
```
to open a communication channel between two computers.

```bash
nmap <ip_address> -p17450-17460,17000
```
to scan a host for open ports.

# Developnemt
```bash
sudo apt-get update
apt-cache policy cmake
```
to check the repository version of a package.

```bash
apt-file search gl.h
```
to find packages that will install `*gl.h*` header file.
