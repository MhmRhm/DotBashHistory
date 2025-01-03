# DotBashHistory
For tutorials, please take a look at my [DotBashHistory](https://mhmrhm.github.io/tutorials/) webpage.

The following contains some useful commands and configs as a day-to-day reference.

- [General](#general)
- [Cryptography](#cryptography)
- [Journal](#journal)
- [Disks and Partitions](#disks-and-partitions)
- [Kernel Bootup](#kernel-bootup)
- [Systemd](#systemd)
- [File Sharing](#file-sharing)
- [Networking](#networking)
- [Version Controlling](#version-controlling)
- [Docker](#docker)
- [CMake](#cmake)
- [Development](#development)
- [Debugging](#debugging)
- [Kernel Internals](#kernel-internals)

# General

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

# the script
# ...
```
to use the unofficial [Bash Strict Mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/).

```bash
man 5 passwd
```
to search man pages for `passwd` in section 5.

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
grep -Rn <path/to/search> -e '<text_to_search>'
```
to search all files in a path for an expression.

```bash
ls .[^.]*
```
to list all hidden files and directories not including `.` and `..`.

```bash
# to create
tar -zcvf arch.tar.gz file dir
# to list TOC
tar -ztvf arch.tar.gz
# to extract .tar.gz
tar -zxpvf arch.tar.gz -C ./outdir
# to extract .tar.xz
tar -xpvf arch.tar.xz
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
sed -i.bak 's/#VERBOSE_FSCK=no/VERBOSE_FSCK=yes/' /etc/sysconfig/rc.site
```
to change a line in a file and keep a backup.

```bash
mknod mypipe p
tail -f mypipe &
man man > mypipe
unlink mypipe
```
to create a named pipe and connect two processes through it.

```bash
sudo usermod -aG <group_name> $USER
```
to add user to a group.

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
cat /etc/locale.gen
locale -a
sudo locale-gen ms_MY.UTF-8
```
to list all available or generated locales and generate one.

```bash
/usr/bin/time command
time { tar -xf repo.tar.xz && rm -rf repo; }
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
        echo "ssh://user$i:$PASS@$MYIP:$PORT#Profile$i" >> users.txt
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
gpg --list-keys
gpg --list-secret-keys
gpg --list-public-keys
gpg --export --armor 'user@mail.com' > public.key
gpg --export-secret-keys --armor 'user@mail.com' > private.key
```
to generate a set of keys and export the public and private keys.

```bash
gpg --import public.key
```
to import someone else's public key or your private key.

```bash
# export GPG_TTY=$(tty)
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

```bash
dmesg --follow --human --level debug
```
to see debug messages by drivers.

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
du -sh $(ls -A) | sort -hr
```
to get sorted folder and file sizes in a directory.

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

```bash
# To enable and start
timedatectl set-ntp true
# To check the service status
timedatectl status
# To see verbose service information
timedatectl timesync-status
# To view the last 24 hours of logged events
journalctl -u systemd-timesyncd --no-hostname --since "1 day ago"
```
to enable time synchronization.

# File Sharing

```bash
python3 -m http.server 8127 -d /home/user/
```
to share a directory on a host over a secured network on port 8127.

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
git config --global merge.conflictstyle diff3
git config --global rerere.enabled true
git config --local submodule.recurse true
git config --global credential.helper 'cache --timeout 900'
git config --global core.autocrlf true
git config --local core.whitespace indent-with-non-tab # tab-in-indent to treat tabs as error

git config --global gpg.program gpg2
git config --local user.signingkey <key_id>
git config --local commit.gpgsign true
```
to list all configs and set some.

```bash
git log -1 --patch
git log -S "expression" # commits that changed number of 'expression'
git log -L :function:file # commits that changed inside 'function'
git log --no-merges -- filename
git log --name-only --oneline
git log --oneline --stat filename
git log feat --not main # same as main..feat or feat ^main
git log --left-right --oneline main...feat # on main and feat but not both
git log --left-right --oneline HEAD...MERGE_HEAD # while in merge conflict
git log --pretty=format:"%C(Red)%d %C(Yellow)%h %C(Cyan)%ch %C(Green)%an %C(White)%<(80,trunc)%s" --graph --all --since="one week ago"
git log --grep 'text' --grep 'word' --all-match # commits mentioning 'text' and 'word'
git reflog --pretty=format:"%C(White)%gd %C(Yellow)%h %C(Cyan)%ch %C(Green)%an %C(magenta)%gs %C(White)%<(80,trunc)%s"
git shortlog -sne --all # to get a list of contributors on all branches
```
to see commit history in different ways.

```bash
git diff # to see unstaged chanes
git diff --staged # to see staged changes # same as --cached
git diff --name-only
git diff HEAD # to see all changes
git diff --check # for white spaces
git difftool abc123 def456 -- filename
git diff $(git merge-base feat main)
git diff main..feat  # head to head
git diff main...feat # head to common ancestor
git diff --ignore-space-change # same as -b
git rev-list HEAD # to get all parents hashes
```
to get the uncommitted changes or compare revisions.

```bash
git clone --depth 10 https://github.com/MhmRhm/DotBashHistory.git
git pull --unshallow
```
to shallow clone a repository then unshallow it if needed.

```bash
# see all branches and trackings
git branch --all -vv

# checkout remote into local and set tracking
git checkout -b new_branch origin/main
# to remove tracking
git branch --unset-upstream

# checkout in detached head
git checkout origin/main
# create branch from detached head
git switch -c new_branch
# set tracking
git branch --set-upstream-to=origin/main
```
to show branches and bring from remote to local.

```bash
git log --oneline --left-right --merge --patch

git show :1:file # base
git show :2:file # ours
git show :3:file # theirs
git diff --ours
git diff --base
git diff --theirs

vim file
git checkout --conflict=merge file # --conflict=diff3
git checkout --ours file
git checkout --theirs file
```
to list commits causing the conflict and compare with different stages while in merge conflict mode. Edit the file then restore the conflict markers or take one side.

```bash
git merge --strategy=recursive -Xours feat
```
to merge with recursive strategy but favouring ours for conflicts.

```bash
git rerere status
git rerere diff
git rerere
```
to see what rerere recorded and apply it.

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
git reset --soft  abc123 # move ref
git reset --mixed abc123 # move ref, reset index
git reset --hard  abc123 # move ref, reset index, discard working tree
git reset --hard  abc123 file # have an old revision in your working tree
```
to reset three trees to different extent.

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

git add -A
git commit --amend --no-edit
```
to change last commit message or just the content.

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
git merge --ff-only feat
```
to rebase instead of merge.

```bash
git rebase -i HEAD~3
git rebase --continue
git rebase --abort
```
to interactively modify past three commits. Continue after making changes or abort.

```bash
git apply --whitespace=fix patch
git rebase --whitespace=fix HEAD~
```
to fix whitespaces.

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

```bash
# to add a submodule to a repo
git submodule add https://project.git
git diff --cached --submodule

# to update a submodule
cd module_name && git pull # or change and commit
cd .. && git add module_name # then commit

# to push a repo without missing work in submodules
git push --recurse-submodules=check

# to clone a repo with submodule(s)
git clone https://project.with.submodule.git
git submodule update --init --recursive # --remote --merge if WIP

# to switch branches
git checkout --recurse-submodules feat
```
to add, update, push and clone with submodules.

```bash
git filter-branch --tree-filter 'rm -f big_file.bin' HEAD
git filter-branch --index-filter 'git rm --ignore-unmatch --cached file.big' -- abc123^.. # faster than --tree-filter
git filter-branch --subdirectory-filter module HEAD
git filter-branch --commit-filter '
        GIT_AUTHOR_NAME="New User";
        GIT_AUTHOR_EMAIL="new.user@mail.com";
        git commit-tree "$@";' HEAD
```
to remove a file from every commit or a specific commit, export a sub directory as separate repository or change commiter name and email.

```bash
git grep --column --line-number --show-function '^\s*throw;$' abc123
git grep --count 'throw' feat
```
to search for an expressions in a tree.

```bash
git bisect start HEAD v1.1 # bad then good commit

git bisect bad
git bisect good
git show
git bisect reset

# or

git bisect run tests # tests returns 0 on success
```
to search for when a bug was first introduced.

```bash
git blame -L:function file --color-by-age
git blame -C -C -C -L 20,35 file # -C to detect moved code -C from commit that create file -C from every commit
```
to see which commit changed each line.

```bash
git tag -s v1.0 -m 'a signed tag' # to sign tag
git tag -v v1.0 # verify signed tag
git commit -a -s -m 'Signed commit' # sign commit
git merge --verify-signatures -S signed-branch # verify and sign merge commit
```
to include gpg signing in workflow.

```bash
git switch feat
git format-patch origin/main #/path/to/specific/files/optionally
git request-pull origin/main fork
git send-email /tmp/0001-feat.patch --to=maintainer@mail.com --cc=reviewer@mail.com # use --compose to add a cover letter
```
to prepare your work for integration and create a pull request or send patch to maintainer.

```bash
git am -i 0001-feat.patch
git cherry-pick abc123
```
to apply patch or cherry-pick.

```bash
git log --oneline --graph
# choose commit
git commit-tree -m 'fake init comit' abc123^{tree}
# def456
git rebase --onto=def456 abc123
git log --oneline --graph
```
to trim your history.

```bash
git archive main --prefix='project/' | gzip > $(git describe main).tar.gz
git archive main --prefix='project/' --format=zip > $(git describe main).zip
```
to export latest code in an archive.

```bash
# producer
git bundle create ../repo.bundle HEAD main # all commits
git bundle create ../my_commits.bundle main --not origin/main # local commits

# consumer
git bundle verify ../repo.bundle
git pull ../repo.bundle main
git fetch ../repo.bundle feat:other-feat
```
to share work when offline.

```bash
git commit # commit your changes in a feature branch
git commit --amend -s # sign your commit
git format-patch origin/master -o /tmp/ # create patch file relative to master
./scripts/checkpatch.pl /tmp/0001-Fix.patch # check the patch file for issues
./scripts/get_maintainer.pl /tmp/0001-Fix.patch # find maintainers responsible for the fix
git send-email /tmp/0001-Fix.patch --to=user1@mail.com --cc=user2@mail.com # email them the patch file
```
to contribute to linux source code.

```bash
sudo apt-get install lighttpd libcgi-pm-perl gamin
cd repository
git instaweb
git instaweb --stop
```
to serve git repository in a simple web interface.

```bash
git cat-file -p HEAD # to show commit object
git ls-tree HEAD # to show commit tree object
git rev-list --objects --all # to list every referenced object

git hash-object file # to get the sha1 of file, use -w to write to git
git cat-file -p abc123 # to get content of object, use -t to get type

git write-tree # to write index into a tree object
git read-tree --prefix=bak abc123 # to write tree in bak directory

git commit-tree def456 -p abc123 -m 'message' # to create a commit object

git update-ref refs/heads/feat abc123 # to create a branch

git count-objects -vH # to get stats on objects

git prune -vn --expire=now # to see loose objects that will be removed

GIT_TRACE_PACK_ACCESS=true GIT_TRACE_PACKET=true GIT_TRACE_SETUP=true git fetch # to debug git
```
to work with git internals.

# Docker

```bash
nano Dockerfile
#FROM alpine:latest
#RUN apk add bash
#CMD ["/bin/bash"]

sudo docker build --no-cache -t <image> .
sudo docker images

sudo docker run -it <image>
# go back to host's terminal without
# exiting container with Ctrl+PQ
sudo docker ps -a
sudo docker stop <container>

sudo docker rm <container>
sudo docker rmi <image>
```
to build an image with a tag name, list images, run a container in interactive mode, list containers, and remove containers and images.

```bash
sudo docker run --rm -it <image>
```
to run and remove the container afterwards.

```bash
docker stop $(docker ps --all --quiet)
docker rm $(docker ps --all --quiet)
docker rmi $(docker images --quiet)
```
to stop and remove containers and images.

```bash
docker system prune --all --volumes

# to clean build cache
docker builder prune --all

# to clean containers and images
docker image prune --all
docker container prune
```
to clean everything not running.

```bash
sudo vim /etc/docker/daemon.json
# {
#   "log-driver": "journald"
#   "debug":true,
#   "log-level":"debug"
# }
docker logs --follow --details <container>

# to see docker engine logs
journalctl --follow --unit docker.service
```
to change logging level and view docker logs.

```bash
docker search <image> --filter is-official=true --no-trunc --limit 100
```
to search docker registries from CLI.

```bash
docker manifest inspect <image>
```
to inspect supported platforms and architectures for an image on registries.

```bash
docker history <image>
docker inspect <image|container>
```
to see how an image is created, layer sizes and manifest.

```bash
# to run interactively
docker run --interactive --tty <image> /bin/sh
exit

# to start interactively only if run interactively
docker start --interactive <container>

# to keep running without terminal interactions
docker start <container>
```
to start a stopped container in interactive mode.

```bash
# create the image
vim Dockerfile
docker build -t <user_name>/<image>:<tag> .

# inspect the image
docker history <image>

# add another tag
docker tag <user_name>/<image>:<tag> <user_name>/<image>:latest

# push the image
cat pass.text | docker login --username mhmrhm --password-stdin
docker push <user_name>/<image>:<tag>
```
to publish an image on docker hub.

```Dockerfile
FROM ubuntu:rolling as dvel_img

RUN apt-get update && apt-get -y upgrade
RUN apt-get -y install --no-install-recommends build-essential git libssl-dev
RUN apt-get -y remove cmake || echo 'cmake not installed'
RUN apt-get -y autoremove

WORKDIR /src
RUN git clone --depth=1 --recurse-submodules https://gitlab.kitware.com/cmake/cmake.git
RUN mkdir cmake-build

WORKDIR /src/cmake-build
RUN ../cmake/bootstrap --parallel=$(nproc) --prefix=/src/usr/local && make && make install

FROM ubuntu:rolling as prod_img
COPY --from=dvel_img /usr/lib/aarch64-linux-gnu/libssl.so /usr/lib/aarch64-linux-gnu/libssl.so
COPY --from=dvel_img /usr/lib/aarch64-linux-gnu/libssl.so.3 /usr/lib/aarch64-linux-gnu/libssl.so.3
COPY --from=dvel_img /usr/lib/aarch64-linux-gnu/libcrypto.so /usr/lib/aarch64-linux-gnu/libcrypto.so
COPY --from=dvel_img /usr/lib/aarch64-linux-gnu/libcrypto.so.3 /usr/lib/aarch64-linux-gnu/libcrypto.so.3
COPY --from=dvel_img /src/usr/local /usr/local

WORKDIR /
ENTRYPOINT ["cmake", "--version"]
```
```bash
docker build -t mhmrhm/dvel_cmake:3.29.2_1d31a00e --target dvel_img .

docker build -t mhmrhm/cmake:3.29.2_1d31a00e . # or
docker build -t mhmrhm/cmake:3.29.2_1d31a00e --target prod_img .
```
to build multiple images from a single Dockerfile.

```bash
docker buildx create --driver docker-container --name container

docker buildx build --builder container \
--platform linux/amd64,linux/arm64/v8,linux/s390x,linux/arm/v7 \
-t mhmrhm/cmake:3.29.2_1d31a00e --target prod_img \
--file Dockerfile --push .

docker buildx prune --builder container
docker buildx rm container
docker system prune -af --volumes 
```
to build for multiple platforms, push and clean after.

```yaml
services:
  nginxproxymanager:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginxproxymanager
    restart: always
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./nginx/data:/data
      - ./nginx/letsencrypt:/etc/letsencrypt

  gitea:
    image: 'gitea/gitea:latest'
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=postgres_gitea:5432
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
      - GITEA__service__DISABLE_REGISTRATION=true
      - GITEA__admin__DISABLE_REGULAR_ORG_CREATION=true
      - GITEA__openid__ENABLE_OPENID_SIGNUP=false
    restart: always
    ports:
      - '2222:22'
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    depends_on:
      postgres_gitea:
        condition: service_healthy

  postgres_gitea:
    image: 'postgres:latest'
    container_name: postgres_gitea
    restart: always
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=gitea
      - POSTGRES_DB=gitea
    volumes:
      - ./gitea/postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 30s
      timeout: 20s
      retries: 3

  runner_gitea:
    image: gitea/act_runner:nightly
    container_name: runner_gitea
    environment:
      GITEA_INSTANCE_URL: "http://<server-ip>:3000"
      GITEA_RUNNER_REGISTRATION_TOKEN: "<from-gitea>"
      GITEA_RUNNER_NAME: "Local Runner"
    volumes:
      - ./gitea/runner:/data
      - /var/run/docker.sock:/var/run/docker.sock

  nextcloud:
    image: 'nextcloud:latest'
    container_name: nextcloud
    restart: always
    volumes:
      - ./nextcloud:/var/www/html:z
    environment:
      - POSTGRES_HOST=postgres_nextcloud
      - REDIS_HOST=redis_nextcloud
      - POSTGRES_USER=nextcloud
      - POSTGRES_PASSWORD=nextcloud
      - POSTGRES_DB=nextcloud
    depends_on:
      redis_nextcloud:
        condition: service_started
      postgres_nextcloud:
        condition: service_healthy

  cron_nextcloud:
    image: 'nextcloud:latest'
    restart: always
    container_name: cron_nextcloud
    volumes:
      - ./nextcloud:/var/www/html:z
    entrypoint: /cron.sh
    depends_on:
      redis_nextcloud:
        condition: service_started
      postgres_nextcloud:
        condition: service_healthy

  redis_nextcloud:
    image: 'redis:latest'
    restart: always
    container_name: redis_nextcloud

  postgres_nextcloud:
    image: 'postgres:latest'
    restart: always
    container_name: postgres_nextcloud
    environment:
      - POSTGRES_USER=nextcloud
      - POSTGRES_PASSWORD=nextcloud
      - POSTGRES_DB=nextcloud
    volumes:
      - ./nextcloud/postgres:/var/lib/postgresql/data:Z
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 30s
      timeout: 20s
      retries: 3

  emulatorjs:
    image: lscr.io/linuxserver/emulatorjs:latest
    container_name: emulatorjs
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Singapore
      - SUBFOLDER=/
    volumes:
      - ./emulatorjs/config:/config
      - ./emulatorjs/data:/data
```
to setup gitea, nexcloud and more behind a reverse proxy. [Use duckdns.org to add SSL Certificates.](https://notthebe.ee/blog/easy-ssl-in-homelab-dns01/). Use `sudo docker compose up -d` for runner to work.

```bash
# to run services
docker compose --file compose.yml up --detach

# to list containers
docker compose --file compose.yml ps --all
# to list processes
docker compose --file compose.yml top

# to stop and restart (faster than down and up)
docker compose --file compose.yml stop
docker compose --file compose.yml restart

# to remove everything
docker compose --file compose.yml down --volumes --rmi all
```
to work with compose.

```bash
# to create a swarm on host_1
docker swarm init --advertise-addr <IP>:2377 --listen-addr <IP>:2377

# to list joining keys
docker swarm join-token manager
docker swarm join-token worker

# to change join tokens
docker swarm join-token --rotate manager 

# to watch nodes on a swarm on a manager
watch --interval 0.5 'docker node ls'

# to remove a worker node from a manager
docker node rm <node>

# to promote or demote nodes
docker node promote <node>
docker node demote <node>

# to prevent managers from rejoining after leaving
docker swarm update --autolock=true
# to get the unlock key from a joined manager
docker swarm unlock-key
# to join a manager after restart
docker swarm unlock

# to prevent node from accepting work and exit existing ones
docker node update --availability drain <node>
```
to config a Swarm.

```bash
# to create 3 containers of an image in a swarm
docker service create --replicas 3 --publish 80:80 <image> <entry_cmd>

# to monitor services in a swarm
docker service ls
docker service ps <service>
docker service inspect <service>
docker service logs <service|node> --follow --details --timestamps

# to add or remove containers
docker service scale <service>=5

# to update a service
docker service update \
--image <new_image> --entrypoint <cmd> --args '<arg1 arg2 arg3>' \
--update-parallelism 2 --update-delay 10s <service>

# to remove the service and containers
docker service rm <service>
```
to manage services in a swarm.

```bash
# backup a swarm with at least 3 managers
# save unlock key
docker swarm unlock-key
# stop docker
sudo service docker stop
# create backup
tar -zcvf swarm_backup_<node>_<ip>.tar.gz /var/lib/docker/swarm
# restart docker
sudo service docker start
# unlock swarm
docker swarm unlock


# restore on same node that was backed up with same IP
sudo service docker stop
sudo rm -rf /var/lib/docker/swarm
tar -zxpvf swarm_backup_<node>_<ip>.tar.gz -C /
sudo service docker start
docker swarm unlock
docker swarm init --force-new-cluster --advertise-addr <IP>:2377 --listen-addr <IP>:2377
```
to backup and restore a swarm from a manager and non-leader node.

```bash
docker system prune -af
sudo service docker stop
sudo vim /etc/docker/daemon.json
# {
#   "bip": "192.168.2.0/8"
# }
sudo service docker restart
```
to change docker default network address.

```bash
# to list all networks
docker network ls
# to list containers using a network
docker inspect bridge --format '{{json .IPAM.Config}}{{json .Containers}}' | jq
# to create a network
docker network create --driver <bridge|overlay|macvlan|...> <name>
# to run an image attached to a network
docker run --tty --interactive --network <name> --publish 80:80 <image> <cmd>
# to remove a network
docker network rm <network>
```
to manage docker networks.

```bash
# to mount existing directory into container
docker run --volume <host_abs_path>:<container_abs_path> <image> <cmd>
# to create a local volume
docker volume create --driver local <name>
# to list volumes
docker volume ls
# to inspect a volume
docker inspect <volume>
# to mount a volume into container
docker run --volume <volume>:<container_abs_path> <image> <cmd>
# to remove a volume
docker volume rm <volume>
# to remove all volumes
docker volume prune -af
```
to mount and work with volumes.

```bash
echo "p@$$w0rd" | docker secret create my_secret1 -
cat | docker secret create my_secret2 - # type secret then press ctrl+d twice
docker secret create my_secret3 ./.env

docker secret ls
docker inspect my_secret3

docker service create --secret my_secret1 <image>
docker service update --secret-add my_secret2 --secret-add my_secret3 <service>

docker exec -it <container> bash
echo "|$(cat /run/secrets/my_secret1)|"
```
to add secrets to swarm and use in services.

# CMake

```bash
cmake --help
cmake -B ./build_dir -S ./source_dir
cmake -B ./build_dir -S ./source_dir -G Ninja
cmake -B ./build_dir -S ./source_dir -L
cmake -B ./build_dir -S ./source_dir -L -D CMAKE_BUILD_TYPE=Release
cmake -B ./build_dir -S ./source_dir -L -U CMAKE_BUILD_TYPE
cmake -B ./build_dir -S ./source_dir -LAH # with advance and help strings
cmake --list-presets
cmake --preset=WinRelease
```
to configure a project.

```bash
# in CMakeLists.txt:
# function(addDependency)
#     list(APPEND CMAKE_MESSAGE_CONTEXT "addDep")
#     message(DEBUG "dependency included")
# endfunction()

cmake -B ./build_dir -S ./source_dir --log-level=DEBUG
cmake -B ./build_dir -S ./source_dir --log-context
cmake -B ./build_dir -S ./source_dir --trace
```
to debug the configuration process.

```bash
cmake --build ./build_dir -j $(nproc)
cmake --build ./build_dir --target clean
cmake --build ./build_dir --clean-first
cmake --build ./build_dir --config release
```
to build a project and targets.

```bash
cmake --install ./build_dir --config release --prefix /opt/prj/
```
to install a project and components.

```bash
cmake -E # to list commands
cmake -E make_directory dir
```
to run some commands in a platform-independent way.

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
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes --verbose ./executable
```
to find memory leaks.

```bash
python3 -m venv .venv
. .venv/bin/activate
deactivate
```
to create a Python virtual environment.

```bash
git clone --depth=1 --recurse-submodules https://gitlab.kitware.com/cmake/cmake.git
mkdir cmake-build
cd cmake-build
../cmake/bootstrap --parallel=$(nproc) && make && sudo make install
ctest --rerun-failed --output-on-failure
cd .. && rm -rf cmake/ cmake-build/
```
to install latest cmake from source.

```bash
wget https://download.qt.io/official_releases/qt/6.5/6.6.1/single/qt-everywhere-src-6.6.1.tar.xz
tar xvf qt-everywhere-src-6.6.1.tar.xz

mkdir build-qt-everywhere-src-6.6.1
cd build-qt-everywhere-src-6.6.1/

sudo apt-get install build-essential cmake ninja-build libgl-dev libegl-dev libfontconfig1-dev libinput-dev libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-cursor-dev libxcb-glx0-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-util-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev libatspi2.0-dev libclang-dev

../qt-everywhere-src-6.6.1/configure -skip qtwayland -skip qtwebengine | tee ../config_$(TZ='Asia/Singapore' date +%Y-%m-%dT%H.%M.%S%Z).log
cmake --build . --parallel $(nproc) | tee ../build_$(TZ='Asia/Singapore' date +%Y-%m-%dT%H.%M.%S%Z).log
sudo cmake --install .
```
to build Qt6 from source.

```bash
git clone --recurse-submodules https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
make mrproper

# for host
# copy distribution config file to source tree
cp /boot/config-release .config
make olddefconfig
# to avoid error regarding CONFIG_SYSTEM_REVOCATION_KEYS="debian/canonical-revoked-certs.pem"
scripts/config --disable SYSTEM_REVOCATION_KEYS
make menuconfig

make -j$(nproc) all | tee ../config_$(TZ='Asia/Singapore' date +%Y-%m-%dT%H.%M.%S%Z).log
sudo make modules_install
sudo make install

# old kernel version
uname --kernel-release
sudo reboot
# new kernel version
uname --kernel-release

# for an SoC
KERNEL=kernel8
# copy config file to source tree
scp <user>@<address>:/boot/config* .config
make ARCH=arm64 olddefconfig
make ARCH=arm64 menuconfig

make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- all | tee ../config_$(TZ='Asia/Singapore' date +%Y-%m-%dT%H.%M.%S%Z).log
make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- deb-pkg
cd ..
scp *.deb <user>@<address>:/home/<user>

# on SoC
sudo dpkg -i *.deb
```
to update kernel from source.

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

scripts/config --enable CONFIG_DEBUG_KERNEL
scripts/config --enable CONFIG_DEBUG_INFO

scripts/config --enable CONFIG_DEBUG_MISC

# Generic kernel debugging instruments
scripts/config --enable CONFIG_MAGIC_SYSRQ
scripts/config --enable CONFIG_DEBUG_FS
scripts/config --enable CONFIG_KGDB
scripts/config --enable CONFIG_UBSAN
scripts/config --enable CONFIG_KCSAN

# Memory debugging
scripts/config --enable CONFIG_SLUB_DEBUG
scripts/config --enable CONFIG_DEBUG_MEMORY_INIT
scripts/config --enable CONFIG_KASAN

scripts/config --enable CONFIG_DEBUG_SHIRQ

scripts/config --enable CONFIG_SCHED_STACK_END_CHECK

scripts/config --enable CONFIG_DEBUG_PREEMPT

# Lock debugging
scripts/config --enable CONFIG_PROVE_LOCKING
scripts/config --enable CONFIG_LOCK_STAT
scripts/config --enable CONFIG_DEBUG_ATOMIC_SLEEP

scripts/config --enable CONFIG_BUG_ON_DATA_CORRUPTION

scripts/config --enable CONFIG_STACKTRACE

scripts/config --enable CONFIG_DEBUG_BUGVERBOSE

scripts/config --enable CONFIG_FTRACE
scripts/config --enable CONFIG_FUNCTION_TRACER
scripts/config --enable CONFIG_FUNCTION_GRAPH_TRACER

# Arch specific
scripts/config --enable CONFIG_FRAME_POINTER
scripts/config --enable CONFIG_STACK_VALIDATION
```
to add debug configs to kernel

```bash
# on host
make -C <kernel-source-tree> M=$(pwd) ARCH=arm64 \
  CROSS_COMPILE=aarch64-linux-gnu- modules
make -C <kernel-source-tree> M=$(pwd) ARCH=arm64 \
  CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=<temp-directory> modules_install
tar zcvf - <temp-directory> | ssh <user-name>@<target-address> tar zxvf -

# on target
cd ~/<temp-directory>
sudo su
cp -r lib /lib
echo <module-name> >> /etc/modules-load.d/modules.conf
depmod
reboot
```
to cross-compile linux modules and install them on remote target.

# Debugging

```bash
# To ask when some expression is ambiguous
(gdb) set multiple-symbols ask
# To pretty print arrays
(gdb) set print array on
# To print array indexes
(gdb) set print array-indexes on
# To limit printing array elements
(gdb) set print elements <num>
# To omit repeated array elements
(gdb) set print repeats <num>
# To omit deeply nested members
(gdb) set print max-depth <num>
# To stop printing strings after null
(gdb) set print null-stop <on|off>
# To print with indentation
(gdb) set print pretty <on|off>
# To print unions
(gdb) set print union <on|off>
# To show the actual drived type
(gdb) set print object <on|off>
# To show v-tables
(gdb) set print vtbl <on|off>
# To split bytes on nibbles
(gdb) set print nibbles on
# To whether show frame arguments
(gdb) set print frame-arguments <all|scalars|presence|none>
# To print arguments at function entry
(gdb) set print entry-values <no|default|both|...>
# To control what `frame` shows
(gdb) set print frame-info <auto|location-and-address|...>
```
to customize GDB.

```bash
# Start GDB with a program but without running it
gdb <executable>

# Attach GDB to an already running process
sudo gdb <executable> <pid>
sudo gdb -p <pid>

# Pass arguments to inferior
gdb --args <executable> <args>

# Start GDB with editor set
EDITOR=/usr/bin/nano gdb

# Start GDB without any program or process
gdb
# Then, load a program or attach to a running process manually:
(gdb) file <executable>
# Execute the program
(gdb) run
# Execute the program and break on first line
(gdb) start
# Attach to a running instance
(gdb) attach <pid>
# To stop inferior debugging
(gdb) kill
(gdb) detach
(gdb) quit
```
to start gdb in different ways or stop it.

```
(gdb) show logging
```
to show the settings for saving the output of gdb commands to a file.

```bash
# Break on a location (line or file:line or function)
(gdb) break <locspec>
# Break on all overloads of a function
(gdb) break method
# Break on next line
(gdb) break
# Break if
(gdb) break <locspec> if <expression>
# To break only in one thread
(gdb) break <locspec> thread <id>
# Print list of all break, watch and catch points
(gdb) info breakpoints
# Ignore next n crossings of a breakpoint.
(gdb) ignore <id> <n>
# Remove all or some of breakpoints
(gdb) delete <ids>
# Enable or disable breakpoints
(gdb) disable <id>
(gdb) enable <id>
 ```
 to work with breakpoints.

```bash
# Break when a variable changes
(gdb) watch <var>
# Break when data at address changes
(gdb) watch -location *<address>
# Break when address is accessed
(gdb) awatch -location *<address>
# Break when variable is read
(gdb) rwatch <var>
# Print list of all watchpoints
(gdb) info watchpoints
```
to work with watchpoints.

```bash
# To break on an exception
(gdb) catch throw <exception>
# To break on a rethrow
(gdb) catch rethrow <exception>
# To break on a catch block
(gdb) catch catch <exception>
# To break on a system call
# List of system calls: /usr/include/asm-generic/unistd.h
(gdb) catch syscal <system-call>
```
to work with catchpoints.

```bash
# Add a conditional breakpoint
(gdb) break <function> if <expression>
# Set a condition on a breakpoint
(gdb) condition <id> <expression>
# Remove the condition on a break point
(gdb) condition <id>
```
to set and clear conditional breakpoints.

```bash
(gdb) commands <id>
  silent
  print <var>
  enable <id>
  continue
end
```
to execute commands after hitting a breakpoint.

```bash
(gdb) save breakpoints file
(gdb) source file
```
to save breakpoints to a file and load from it.

```bash
# To continue to reach a different line with debug info
(gdb) step <count>
# To continue to the next line in current stack
(gdb) next <count>
# To continue to just after function in the stack returns
(gdb) finish
# To continue until after current line or the stack returns
# If called at the end of a loop, continues until loop is finished
(gdb) until
# If called in recursive function, continues until function is over
# <locspec> can be a line number or file:line or function name
(gdb) until <locspec>
# To immediately give a command prompt so you can issue other commands
# (Non-Stop Mode)
(gdb) run&
(gdb) attach&
(gdb) step&
(gdb) next&
(gdb) continue&
(gdb) finish&
(gdb) until&
# To run backward
(gdb) reverse-step
(gdb) reverse-next
(gdb) reverse-continue
(gdb) reverse-finish # where current function was called
# To cancel execution of a function call and return
(gdb) return <return-value-expression>
# To resume execution at <locspec>
# Break once due to temporary breakpoint
(gdb) tbreak <locspec>
(gdb) jump <locspec>
```
to move the control around.

```bash
# To prevent stepping in function containing <locspec>
(gdb) skip function <locspec>
# To prevent stepping in all functions of a file
(gdb) skip file <name>
# To manage skips
(gdb) info skip
(gdb) skip enable <ids>
(gdb) skip disable <ids>
(gdb) skip delete <ids>
```
to disable stepping in functions.

```bash
(gdb) set pagination off
(gdb) set non-stop on
# Set before running or attaching
(gdb) run
(gdb) attach <pid>
# To continue all threads
(gdb) continue -a
# To stop all threads
(gdb) interrupt -a
# To list existing threads
(gdb) info threads
# To switch between threads
(gdb) thread <id>
```
to set non-stop mode in debugging multi-threaded code, only the current thread
will pause while others keep running, unless explicitly stopped.

```bash
# To list all checkpoints
(gdb) info checkpoints
# To create a checkpoint from current state
(gdb) checkpoint
# To restore a checkpoint
(gdb) restart checkpoint-id
# To delete checkpoints
(gdb) delete checkpoint <ids>
```
to save and restore checkpoints.

```bash
# To list all stack frames with local variables
(gdb) backtrace -full -frame-arguments all
# To list innermost count frames
(gdb) backtrace +<count>
# To list outermost count frames
(gdb) backtrace -<count>
# To list frames beyond main
(gdb) backtrace -past-main -past-entry
# To list all frames for all threads
(gdb) thread apply all backtrace
```
to print stack frames.

```bash
# To move between frames
(gdb) frame level <num>
(gdb) frame function <name>
(gdb) up
(gdb) down
# To show information on a frame
(gdb) frame
(gdb) info frame
(gdb) info args # <name> or -t <type-name>
(gdb) info locals # <name> or -t <type-name>
# To list all variables of a certain type in the program
(gdb) thread apply all -s frame apply all -s info locals -q -t <variable-type>
```
to sellect a frame and show information about it.

```bash
# To show a function
list <function>
# To show only fully qualified functions
list -qualified <namespace::function>
# To show a function in a file
list <file>:<function>
# To show a line in a file
list <file>:<line>

# To show current source location
(gdb) list .
# To show lines after current source location
(gdb) list +
# To show lines before current source location
(gdb) list -
# To show lines between two source locations
(gdb) list <from>,<to>
# To show more surrounding lines
(gdb) list -20,+50

# To reset source location
(gdb) frame
```
to show source locations.

```bash
(gdb) list -source <file> -line 0
(gdb) forward-search <expression>
# or search <expression>
(gdb) reverse-search <expression>

(gdb) edit <line>
(gdb) edit <function>

# To show file name in vim
:echo expand('%:p')
```
to search or edit a source location.

```bash
# To show the source path
(gdb) show directories
# To show substitution rules
(gdb) show substitute-path

# To reset source path to default
(gdb) directory
# To reset substitution rules
(gdb) unset substitute-path
# To remove a substitution rule
(gdb) unset substitute-path </from/dir>

# To add directory to source path
(gdb) directory <dir1> <dir2> ...
# To add substitution rule
(gdb) set substitute-path </from/dir> </to/dir>
```
to manipulate source directories that GDB looks inside.

```bash
# For a function in a file
(gdb) disassemble /sb '<file>'::<namespace::class::method>
# For current instruction at program counter
(gdb) disassemble /sb $pc-64,+64
```
to show assembly code.

```bash
(gdb) explore value <expression>
(gdb) explore type <expression>
```
to interactively explore an expression or its type.

```bash
(gdb) print <expression>
(gdb) print <var-name>
(gdb) whatis <var-name>
```
to print an expression or a variable value or type.

```bash
# To print data at <address> as if it where of type <type>
(gdb) print {<type>} <address>
(gdb) print {double} 0xffab

# Artificial Array
# To print data at *&<first_element> as an array of decltype(<first_element>)
(gdb) print <first_element>@<length>
(gdb) set $ptr = &my_vector[0]
(gdb) print *$ptr@3
# $1 = {3, 2, 1}
(gdb) print /x *(unsigned char*)$ptr@(8*3)
# $2 = {0x3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x2, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x1, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0}
```
to use GDB operators in expressions.

```bash
(gdb) set $i = 0
(gdb) print *(array[$i++])
# Hit enter/return key again
```
to print pointed-to elements of an array of pointers.

```bash
(gdb) print /c 10
# $1 = 10 '\n'
(gdb) print /t 10
# $2 = 1010
(gdb) print /x 10
# $3 = 0xa
(gdb) print /z 10
# $4 = 0x0000000a
(gdb) print /r 10 # to bypass pretty-printers
# $5 = 0x0000000a
(gdb) print /d 0xfffffff6
# $6 = -10
(gdb) print /u 0xfffffff6
# $7 = 4294967286
```
to format the print output.

```bash
(gdb) explore object
(gdb) print sizeof(object)
# $9 = 40
(gdb) x /40bx &object # b for bytes
(gdb) x /20hx &object # h for half words  (4 bytes)
(gdb) x /10wx &object # w for words       (4 bytes)
(gdb) x /5gx &object  # g for giant words (8 bytes)
(gdb) x /40bt &object # t for base two
(gdb) x /32wi &object->method # i for instructions
(gdb) print $_   # print the address last examined
(gdb) print $__  # print the content last examined
```
to examine an address.

```bash
# To show last ten values
(gdb) show values
# To show surrounding of <num>
(gdb) show values <num>
```
to show variables history.

```bash
(gdb) info registers
(gdb) p/x $pc # for program counter
(gdb) x/i $pc # for next instruction
(gdb) p/x $sp # for top of the stack
(gdb) p/x $fp # for current stack

(gdb) help function
(gdb) break <function> if $_any_caller_matches("<pattern>", <depth>)

(gdb) show convenience
(gdb) print $_inferior
(gdb) print $_thread
(gdb) print $_gthread
(gdb) print $_exitcode
(gdb) print $_exitsignal
(gdb) print $_shell_exitcode
(gdb) print $_shell_exitsignal
(gdb) print $_gdb_major
(gdb) print $_gdb_minor
```
to work with registers, convenience functions and variables.

```bash
# To create the dump
(gdb) dump binary value dmp.bin object
(gdb) dump binary memory dmp.bin 0xaaaaaabb72e0 0xaaaaaabb7300
# To restore the dump
(gdb) restore dmp.bin binary 0xaaaaaabb72e0
# To view the dump
xxd -g4 dmp.bin
```
to save memory into a file and restore it.

```bash
(gdb) gcore
gdb <executable> core.<pid>
```
to create core dumps and investigate them.

```bash
(gdb) compare-sections
# Use -r for just read-only sections
(gdb) compare-sections -r
```
to compare the sections of an executable file with the same sections in the target machine’s memory.

```bash
# To add to displays
(gdb) display object
(gdb) display /t object.member
(gdb) display /10wx &object
# To show all displays again
(gdb) display
# To list all displays
(gdb) info display
# To delete all displays
(gdb) delete display
# To disable/enable a display
(gdb) disable display <id>
(gdb) enable display <id>
```
to automatically display expressions when program stops.

```bash
# To pretty print arrays
(gdb) print -array -- <array>
# To also print array indexes
(gdb) print -array-indexes -- <array>
# To limit number of elements shown
(gdb) print -elements <num> -- <array>
# To limit repeated array elements
(gdb) print -repeats <num> -- <array>
# To limit nested members
(gdb) print -max-depth <num> -- <variable>
# To stop printing strings after null
(gdb) print -null-stop <on|off> -- <text>
# To print with indentation
(gdb) print -pretty <on|off> -- <text>
# To print unions
(gdb) print -union <on|off> -- <var>
# To show the actual drived type
(gdb) print -object <on|off> -- <obj>
# To show v-tables
(gdb) print -vtbl <on|off> -- <obj>
# To split bytes in 4-bit groups
(gdb) print -nibbles on -- /t <value>
```
to alter print settings for current command.

```bash
(gdb) print -pretty -object -vtbl -- /x *obj
# $1 = (Derived) {
#   <Base2> = {
#     <Base1> = {
#       _vptr.Base1 = 0xaaaaaaabfcc0 <vtable for Derived+16>
#     }, <No data fields>}, <No data fields>}
(gdb) set $vtbl = 0xaaaaaaabfcc0 - 16
(gdb) set $i = 0
(gdb) while($i < 24)
# print ((void **)$vtbl)[$i++]
# end
# $2 = (void *) 0x0
# $3 = (void *) 0xaaaaaaabfd28 <typeinfo for Derived>
# $4 = (void *) 0xaaaaaaaa0d2c <Derived::sayHello() const>
# $5 = (void *) 0xaaaaaaaa0e8c <Derived::~Derived()>
# ...
(gdb) print -object -pretty -- *(std::type_info*)($3)
# $26 = (__cxxabiv1::__si_class_type_info) {
#   <__cxxabiv1::__class_type_info> = {
#     <std::type_info> = {
#       _vptr.type_info = 0xfffff7e77958 <vtable for __cxxabiv1::__si_class_type_info+16>,
#       __name = 0xaaaaaaaa0f48 <typeinfo name for Derived> "7Derived"
#     }, <No data fields>}, 
#   members of __cxxabiv1::__si_class_type_info:
#   __base_type = 0xaaaaaaabfd40 <typeinfo for Base2>
# }
(gdb) x /22wi 0xaaaaaaaa0e8c
# ...
#    0xaaaaaaaa0ebc <_ZN7DerivedD0Ev>:    stp     x29, x30, [sp, #-32]!
#    0xaaaaaaaa0ec0 <_ZN7DerivedD0Ev+4>:  mov     x29, sp
#    0xaaaaaaaa0ec4 <_ZN7DerivedD0Ev+8>:  str     x0, [sp, #24]
#    0xaaaaaaaa0ec8 <_ZN7DerivedD0Ev+12>: ldr     x0, [sp, #24]
#    0xaaaaaaaa0ecc <_ZN7DerivedD0Ev+16>: bl      0xaaaaaaaa0e8c <_ZN7DerivedD2Ev>
#    0xaaaaaaaa0ed0 <_ZN7DerivedD0Ev+20>: mov     x1, #0x8                        // #8
#    0xaaaaaaaa0ed4 <_ZN7DerivedD0Ev+24>: ldr     x0, [sp, #24]
#    0xaaaaaaaa0ed8 <_ZN7DerivedD0Ev+28>: bl      0xaaaaaaaa0af0 <_ZdlPvm@plt>
#    0xaaaaaaaa0edc <_ZN7DerivedD0Ev+32>: ldp     x29, x30, [sp], #32
#    0xaaaaaaaa0ee0 <_ZN7DerivedD0Ev+36>: ret
(gdb) info symbol '_ZdlPvm@plt'
# operator delete(void*, unsigned long)@plt in section .plt of /home/user/main
```
to print v-table of an object.

```bash
(gdb) info auxv
(gdb) info os
```
to get system information.

# Kernel Internals

```bash
sudo vim /etc/default/grub
# edit for example:
# GRUB_TIMEOUT_STYLE=menu
# GRUB_TIMEOUT=3
# GRUB_CMDLINE_LINUX=""
sudo update-grub
```
to update grub.

```bash
# To enable kernel debug messages
debug
# To output loglevels less than n
loglevel=n
# To let kernel to be preempted
preempt=full
# To isolates CPUs from scheduler
isolcpus=0,4-8
# To boot into single-user mode
single
```
useful kernel command line arguments.

```bash
cat /proc/cmdline
```
to query current kernel command line arguments.

```bash
# to list files
lsinitramfs /boot/initrd.img-6.11.3

# to extract
TMPDIR=$(mktemp -d)
unmkinitramfs /boot/initrd.img-6.11.3 ${TMPDIR}
tree ${TMPDIR}

# to create
find ${TMPDIR} | cpio -o -H newc | gzip > initrd.img-mod-6.11.3
```
to extract existing initramfs, modify, then create a new initramfs from it.

```bash
# uInitrd
mkimage -A arm -O linux -T ramdisk -C none -a 0 -e 0 -n uInitrd -d /boot/initrd.img-$version /boot/uInitrd

# boot.src
mkimage -C none -A arm -T script -d /boot/boot.cmd /boot/boot.scr
```
on SoCs booting from U-Boot, to extract existing initramfs, modify, then create a new initramfs from it.

```bash
modinfo <mod-name>
systool -m <loaded-mod-name> -v
```
to show general information about a module.

```bash
lsmod
sudo insmod <module>
sudo rmmod <module>
```
to list all loaded modules, load a module and remove it.

```bash
sysctl -w kernel.dmesg_restrict=1
sudo dmesg
```
to prevent non-privileged users from viewing kernel messages. See [sysctl-explorer](https://sysctl-explorer.net).

```bash
sudo bash -c 'echo "<1>test_script: Hello '"$(whoami)"'!" > /dev/kmsg'
```
to print messages into kernel logs from scripts at KERN_ALERT level.

```bash
cat /proc/zoneinfo
```
to print memory zone watermarks. If number of free pages drop below the min value, kernel will deny page allocation request on that zone. 

```bash
sudo cat /sys/kernel/debug/lru_gen
```
to see Multi-Generational LRU stats.

```bash
sudo su
# Enable the magic SysRq keys
echo 1 > /proc/sys/kernel/sysrq
# Kill the memory hungry process
echo f > /proc/sysrq-trigger
dmesg
```
triggering out-of-memory killer using [Linux Magic System Request](https://www.kernel.org/doc/html/latest/admin-guide/sysrq.html).

```bash
man choom
choom -p $(pidof init)
```
to show OOM killer score and adjustment value.

```bash
taskset -c 1,2,3 yes > /dev/null &
```
to set a task to run on limited CPU cores.

```bash
sudo perf sched record
sudo perf sched map > report.txt
sudo perf timechart -i ./perf.data
```
to record context-switches on CPU cores.

```bash
watch -n 0.3 cat /proc/<PID>/sched
```
to watch scheduling statistics on a process.

```bash
# To start with a policy and priority
sudo chrt <--other|--fifo|--rr|...> <priority> <command> <arguments>
# To change the policy and priority
sudo chrt --fifo -p 99 <pid>
# To see the policy and priority
chrt -p <pis>
```
to set, change and query scheduling policy and priority.

```bash
# To list all cgroups and their processes
systemd-cgls
# To list cgroups
systemctl -t slice --all
# To list cgroups descendants
systemctl -t scope --all
# To list processes and their cgroup
# Add -w for wide output, -f for parent-child
ps -axfeo pid,user,tty,stat,cgroup,cmd
# To monitor cgroups resources
systemd-cgtop
```
to view and monitor cgroups.
