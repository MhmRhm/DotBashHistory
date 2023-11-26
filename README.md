# DotBashHistory
This file contains some useful Linux commands and configs.

- [General](#general)
- [Cryptography](#cryptography)
- [Journal](#journal)
- [Disks and Partitions](#disks-and-partitions)
- [Kernel Bootup](#kernel-bootup)
- [Systemd](#systemd)
- [File Sharing](#file-sharing)
- [Networking](#networking)
- [Version Controlling](#version-controlling)
- [Development](#development)

# General
```bash
!! && !-2
```
to execute last and second-to-last commands.

```bash
history -d -2
```
to remove the last executed command.
```bash
chsh
```
to change the shell.

```bash
nohup <my_command> &
```
to run a command so it can continue after shell is closed.

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

```bash
sudo apt-get install libinput-bin
sudo libinput list-devices
xinput --list
xinput --list-props <id>
```
to list input devices and properties of a device.

```bash
sudo libinput debug-events --show-keycodes
xwininfo
xev
```
to capture and debug UI events.

```bash
dbus-monitor --system
dbus-monitor --session
```
to monitor D-Bus events originated from system and session.

```bash
firefox localhost:631
```
to open CUPS setup page.

```bash
FILE=$(mktemp)
echo $FILE #/tmp/tmp.rvUW4WNN5H
```
to create a unique temp file.

```bash
tar cvf - src | (cd dst; tar xvf -)
```
to move a directory with a lot of files.

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

```python
# pip install python-telegram-bot --upgrade
# pip install requests

import logging, requests
from telegram import (
    Update
)
from telegram.ext import (
    Application,
    CommandHandler,
    ContextTypes
)

bot_father_token = "YOUR_TOKEN"
your_username = "YOUR_ID"

async def getCommand(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    logging.info(f"{update.effective_user}, {update.effective_chat}")
    if update.effective_user.username != your_username:
        return
    await update.message.reply_text("...")
    ip = requests.get('https://checkip.amazonaws.com').text.strip()
    await update.message.reply_text(ip)

def main() -> None:
    logging.basicConfig(filename="ipbot.log", encoding='utf-8', level=logging.INFO)
    application = Application.builder().token(bot_father_token).connect_timeout(30).build()
    application.add_handler(CommandHandler("get", getCommand))
    application.run_polling()

if __name__ == "__main__":
    main()
```
to run a Telegram bot for fetching dynamic IP.

```python
# pip install boto3
# pip install schedule

import logging, time, socket, requests, schedule, boto3
from botocore.exceptions import ClientError

IP = ''

def getCurrentIP() -> str:
    return requests.get('https://checkip.amazonaws.com').text.strip()

def emailNewIP(ip: str) -> bool:
    SENDER = "YOUR_NAME <YOUR_FROM_MAIL>" # keep < and >
    RECIPIENT = "YOUR_TO_MAIL"
    AWS_REGION = "ap-southeast-2" # can find it in server address
    SUBJECT = "New IP Assignment"
    BODY_TEXT = f"{ip} assigned to `{socket.gethostname()}`."
    CHARSET = "UTF-8"
    
    client = boto3.client('ses',
                          region_name=AWS_REGION,
                          aws_access_key_id="YOUR_KEY",
                          aws_secret_access_key="YOUR_SECRET"
                          )
    
    try:
        response = client.send_email(
            Destination = {'ToAddresses': [RECIPIENT]},
            Message = {
                'Body': {'Text': {'Charset': CHARSET,'Data': BODY_TEXT}},
                'Subject': {'Charset': CHARSET,'Data': SUBJECT},
                },
            Source = SENDER
        )
    except ClientError as e:
        logging.critical(e.response['Error']['Message'])
        return False
    else:
        logging.info(f"Email sent to {RECIPIENT} with new IP {ip}.")
        logging.info(response['MessageId'])
        return True

def periodic_task():
    global IP
    if (ip := getCurrentIP()) != IP:
        if emailNewIP(ip):
            IP = ip

schedule.every(60).seconds.do(periodic_task) # every minute

def main() -> None:
    logging.basicConfig(filename="in_mail_ip.log", encoding='utf-8', level=logging.INFO)
    while True:
        schedule.run_pending()
        time.sleep(1)

if __name__ == "__main__":
    main()
```
to run a task for notifying dynamic IP changes.

# Cryptography
```bash
gpg --gen-key
gpg --list-secret-keys
gpg --export --armor 'user@mail.com' > public.key
```
to generate a set of keys and export the public key.

```bash
gpg --import public.key
```
to import someone else's key.

```bash
gpg --default-key 'user@mail.com' --sign file
```
to encrypt and sign contents.

```bash
gpg --verify file.gpg
gpg --output file --decrypt file.gpg
```
to verify and decrypt a signed content.

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
mount
```
to list all mount points.

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
tar zcvf - directory | ssh -i mainkey.pem -p 17456 remote_user@remote_address tar zxvf -
```
to copy a directory to a remote host.

```bash
scp -r src user@remote_host:dst
scp -o Port=17456 -o IdentityFile=mainkey.pem remote_user@remote_address:file /local_directory
```
to copy a file or a directory structure from or to a remote host.

```bash
rsync --rsh='ssh -p<port> -i mainkey.pem' -avn --delete mydir <username>@<ipaddress>:/home/<username> | grep deleting #-a: all, -v: verbose, -n: dry run, -z: zip
```
to list files that will be deleted after syncing two folders.

```bash
smbclient -U <username> -L <server>
```
to list shared folders on a Windows server where `<username>` is the user name on the Windows machine and `<server>` is the IP address or Computer Name.

```bash
[ShareName]
comment = Needs username and password to access
path = /path/to/shared/dir
valid users = user1 @group2
guest ok = no
writable = yes
browsable = yes
```
to setup a SMB share.

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

```bash
#!/bin/bash

# IP range (replace with your desired range)
start_ip="192.168.1.1"
end_ip="192.168.1.10"

# Port to check
port=80

# Iterate through the IP range
IFS='.' read -r -a start <<< "$start_ip"
IFS='.' read -r -a end <<< "$end_ip"

for ((i=${start[0]}; i<=${end[0]}; i++))
do
  for ((j=${start[1]}; j<=${end[1]}; j++))
  do
    for ((k=${start[2]}; k<=${end[2]}; k++))
    do
      for ((l=${start[3]}; l<=${end[3]}; l++))
      do
        ip="${i}.${j}.${k}.${l}"
        echo "Checking $ip"
        
        # Check if the port is open
        nc -z -v -w 1 "$ip" "$port" 2>&1 | grep "succeeded" && echo "Port $port is open on $ip"
      done
    done
  done
done
```
to scan an IP range for a port (AI generated.)

# Version Controlling
```bash
git config --list --show-origin
git config --local user.email "account@mail.com" && git config --local user.name "Your Name"
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global merge.tool vimdiff
```
to list all configs and set some.

```bash
git diff # to see unstaged chanes
git diff --staged # to see staged changes # same as --cached
git diff HEAD # to see all changes
git diff --check # for white spaces
git difftool abc123 def456 -- filename
git diff $(git merge-base feat main)
git diff main..feat  # head to head
git diff main...feat # head to common ancestor
```
to get the uncommitted changes or compare revisions.

```bash
git log -1 --patch
git log -S "piece of code"
git log --no-merges -- filename
git log feat --not main # same as main..feat or feat ^main
git log --left-right --oneline main...feat # on main and feat but not both
git log --pretty=format:"%C(Yellow)%h %C(Cyan)%ch %C(Green)%cn %C(White)%s" --graph --since="one week ago"
```
to see commit history in different ways.

```bash
git reflog
git log -g --oneline
git reflog main
git log -g --oneline main
```
to see where HEAD has been pointing to.

```bash
git show HEAD@{1}     # previously pointing to
git show HEAD^ HEAD^2 # both parent if is a merge commit
git show HEAD~ HEAD~2 # parent and grand parent
```
to show different commits relative to HEAD.

```bash
git restore --staged filename
git restore filename
# or
git checkout filename
```
to discard changes.

```bash
git rm --cached filename
git rm filename
git mv oldfilename newfilename
```
to move or remove files from git.

```git
# save and apply with same staged condition
git stash --keep-index
git stash apply --index # or pop to apply and remove
# save with untracked files
git stash --include-untracked
# save everything including ignored files
git stash --all

git stash list
git stash drop stash@{0}

# create branch from head when stashed and apply stash
git stash branch test_feat
```
to work with stashes.

```git
git clean -ndx
git clean -fdx # use -ff for repos in repos
```
to safely clean a repository from all untracked and ignored file.

```bash
git commit --amend
```
to change last commit message.

```bash
git remote -v
git remote show origin
git remote add remotename https://github.com/MhmRhm/DotBashHistory.git
git remote rename oldname newname
git remote remove oldname
```
to add, rename and remove remotes.

```bash
git tag -a v2.0 abc123 -m "Version 2.0 LTS"
git tag v2.1 456def
git show v2.1
gti push origin --tags
git tag -d v2.1
git push origin --delete v2.1
```
to manage annotated and lightweight tags.

```bash
# create branch
git branch newbranch && git checkout newbranch
git checkout -b newbranch
git switch -c newbranch
# work on branch
# merge branch
git checkout main && git merge newbranch
# delete branch
git branch -d newbranch
```
to create new branch from HEAD and switch to it merge then delete it.

```bash
git branch feat2 123abc
```
to branch-out from a commit.

```bash
git branch --all

git switch -c newname
# or
git branch --move oldname newname

git push --set-upstream origin newname
git push origin --delete oldname
```
to rename or create a local branch, push it to origin and remove the old branch.

```bash
git checkout -b localname origin/remotename
# or
git branch --set-upstream-to origin/remotename
git push origin currentbranch:remotename
```
to checkout a remote branch or push a local branch with different name from or to origin.

```bash
git checkout feat
git rebase main
git checkout main
git merge feat
```
to rebase instead of merge.

```bash
git merge --squash --no-commit feat
```
to merge changes on top of index without creating two parent commits and holding off the single commit.

```bash
git merge --no-ff
```
to prevent fast-forward and create an explicit merge.

```bash
git checkout feat
git rebase --onto main sprint feat
git checkout main
git merge feat
git branch -d feat
```
to take the `feat`, figure out the patches since it diverged from the `sprint`, and replay these patches in the `feat` as if it was based directly off the `main`.

```yaml
version: '3.8'
services:
  nginxproxymanager:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
      - '3000:3000'
    volumes:
      - ./nginxproxymanager/data:/data
      - ./nginxproxymanager/letsencrypt:/etc/letsencrypt
  gitea:
    image: gitea/gitea:1.20.5
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=db:5432
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
      #- GITEA__service__DISABLE_REGISTRATION=true
      #- GITEA__admin__DISABLE_REGULAR_ORG_CREATION=true
      #- GITEA__openid__ENABLE_OPENID_SIGNUP=false
    restart: unless-stopped
    ports:
      - '2222:22'
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    depends_on:
      - db
  db:
    image: postgres:14
    restart: unless-stopped
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=gitea
      - POSTGRES_DB=gitea
    volumes:
      - ./postgres:/var/lib/postgresql/data
```
to setup gitea behind reverse proxy. [Use duckdns.org to add SSL Certificates.](https://notthebe.ee/blog/easy-ssl-in-homelab-dns01/)

```bash
sudo apt-get install lighttpd libcgi-pm-perl gamin
cd repository
git instaweb
git instaweb --stop
```
to serve git repository in a simple web interface.

```bash
git switch feat
git format-patch origin/main
git request-pull origin/main fork
```
to prepare your work for integration.

```bash
git am -i 0001-feat.patch
git cherry-pick abc123
```
to apply patch or cherry-pick.

```bash
git archive main --prefix='project/' | gzip > $(git describe main).tar.gz
git archive main --prefix='project/' --format=zip > $(git describe main).zip
```
to export latest code in an archive.

# Development
```bash
sudo apt-get update
apt-cache policy cmake
```
to check the repository version of a package.

```bash
apt-file search gl.h
```
to find packages that will install `*gl.h*` header file.

```bash
ldconfig -v
```
to rebuild shared libraries cache.

```bash
nano Dockerfile
#FROM alpine:latest
#RUN apk add bash
#CMD ["/bin/bash"]
sudo docker build -t <name> .
sudo docker images
sudo docker run -it <name>
sudo docker ps -a
sudo docker rm <id>
sudo docker rmi <name>
```
to build an image with a tag name, list images, run a container in interactive mode, list containers, and remove containers and images.

```bash
python3 -m venv .venv
. .venv/bin/activate
deactivate
```
to create a Python virtual environment.

```bash
wget https://download.qt.io/official_releases/qt/6.5/6.5.1/single/qt-everywhere-src-6.5.1.tar.xz
tar xvf qt-everywhere-src-6.5.1.tar.xz

mkdir build-qt-everywhere-src-6.5.1
cd build-qt-everywhere-src-6.5.1/

sudo apt-get install ninja-build libgl-dev libegl-dev libfontconfig1-dev libinput-dev libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-cursor-dev libxcb-glx0-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-util-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev libatspi2.0-dev libclang-dev

../qt-everywhere-src-6.5.1/configure -skip qtwayland -skip qtwebengine | tee ../build-qt.log
cmake --build . --parallel 7 | tee -a ../build-qt.log
sudo cmake --install .
```
to build Qt6 from source.
