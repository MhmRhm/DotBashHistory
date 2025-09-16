# DotBashHistory
For tutorials, please take a look at the [DotBashHistory](https://mhmrhm.github.io/tutorials/).

The following contains some useful commands and configs as a day-to-day reference.

- [General](#general)
- [Monitoring](#monitoring)
- [Cryptography](#cryptography)
- [Disks and Partitions](#disks-and-partitions)
- [File Sharing](#file-sharing)
- [Email Client](#email-client)
- [Systemd](#systemd)
- [Networking](#networking)
- [Version Controlling](#version-controlling)
- [Virtualization](#virtualization)
- [Debugging](#debugging)
- [CMake](#cmake)
- [Development](#development)
- [Kernel Bootup](#kernel-bootup)
- [Kernel Internals](#kernel-internals)
- [Sample Drivers](#sample-drivers)
- [Builds](#builds)
- [Scripts](#scripts)

## General

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
PATH=first:$PATH:last
```
to add directories to the search path.

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

# the script
# ...
```
to use the unofficial [Bash Strict Mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/).

```bash
ps ax 2>&1 1>> stdout.txt
```
to send outputs to a files.

```bash
/usr/bin/time command
time { tar -xf repo.tar.xz && rm -rf repo; }
```
to show CPU time and memory faults after running command.

```bash
# To make temp file
FILE=$(mktemp)
echo $FILE #/tmp/tmp.rvUW4WNN5H
# To make temp folder
DIR=$(mktemp -d)
cd $DIR
```
to create a unique temp file or folder.

```bash
ls -alhF .
```
to list all files and directories with inodes and readable size.

```bash
ls .[^.]*
```
to list all hidden files and directories not including `.` and `..`.

```bash
sudo usermod -aG <group_name> $USER
```
to add user to a group.

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
find / -regex .*initrd.img.* -print
```
to find all paths containing `initrd.img`.

```bash
# To search all files in a path for an expression
grep -Rn path/to/search -e 'expression'
# To show line numbers and sorounding lines
grep -i 'expression' -Hn -B3 -A3 file
```
to find in files.

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
tar cvf - src | (cd dst; tar xvf -)
```
to move a directory with a lot of files.

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
sudo startx /usr/bin/gedit
```
to start a graphical application without a desktop manager.

```bash
firefox localhost:631
```
to open CUPS setup page.

## Monitoring

```bash
lsb_release -a
cat /etc/os-release
virt-what
```
to get information about the host system.

```bash
pidstat -ru -p <PID> 1
```
to show memory and CPU usage of `<PID>` every second.

```bash
netstat -ntl
```
to list all TCP connections in use.

```bash
watch "ps aux | grep ssh | grep -Eo '^[^ ]+' | sort | uniq"
```
to list online users.

```bash
nload -U M -t 3000 eth0
```
to monitor network traffic.

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

## Cryptography

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

## Disks and Partitions

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

## File Sharing

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

## Email Client

```bash
# Choose /usr/bin/vim.basic
sudo update-alternatives --config editor
```
to set vim as editor.

```bash
sudo apt-get install neomutt
```
to install the email client.

```bash
# .muttrc
set imap_user = 'yourname@gmail.com'
set imap_pass = 'your_app_password'
set spoolfile = imaps://imap.gmail.com/INBOX
set folder = imaps://imap.gmail.com/
set record="imaps://imap.gmail.com/[Gmail]/Sent Mail"
set postponed="imaps://imap.gmail.com/[Gmail]/Drafts"
set trash="imaps://imap.gmail.com/[Gmail]/Trash"
set mbox="imaps://imap.gmail.com/[Gmail]/All Mail"
mailboxes =INBOX =[Gmail]/Sent\ Mail =[Gmail]/Drafts =[Gmail]/Spam =[Gmail]/Trash =[Gmail]/All\ Mail
set smtp_url = "smtp://yourname@smtp.gmail.com:587/"
set smtp_pass = $imap_pass
set ssl_force_tls = yes
set edit_headers = yes
set charset = UTF-8
unset use_domain
set realname = "Your Name"
set from = "yourname@gmail.com"
set use_from = yes
set envelope_from=yes
source colors.muttrc
```
to config Mutt. See [here](https://gist.githubusercontent.com/LukeSmithxyz/de94948264649a9264193e96f5610c44/raw/d274199d3ed1bcded2039afe33a771643451a9d5/colors.muttrc) for colors.

```bash
neomutt
```
to run the email client.

## Systemd

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

## Networking

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

## Version Controlling

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
git config --global color.diff.meta 'BrightMagenta'

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
git format-patch origin/main -- 'optional/path/to/include/specific/files'
git format-patch origin/main -- ':!optional/path/to/exclude/specific/files'
git request-pull origin/main fork
```
to prepare your work for integration and create patch files or a pull request.

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
# Commit to feature branch
git commit
# Sign the commit
git commit --amend -s
# Create patch
git format-patch origin/master -o ../
# Check patch for issues
./scripts/checkpatch.pl ../0*
# Find maintainers
./scripts/get_maintainer.pl ../0*
# Email last commit to maintainers
git send-email -1 --annotate --cover-letter --to=maintainer1@linux.com --cc=maintainer2@linux.com --cc=maintainer3@linux.com
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

## Virtualization

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

## Debugging

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

## CMake

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

## Development

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
# Add `deb-src` for each `deb`
sudo nano /etc/apt/sources.list.d/ubuntu.sources
sudo apt-get update
sudo apt-get build-dep git
```
to install all the build dependencies for a program.

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
# To start the LTTng session daemon
lttng-sessiond --daemonize
# To list available userspace tracepoints (requires the program to be running)
lttng list --userspace
# To create a new tracing session
lttng create user-space-session
# To enable a specific userspace tracepoint
lttng enable-event --userspace provider:event
# To start tracing
lttng start
# To destroy the session (clears session data)
lttng destroy
# To display recorded events
babeltrace2 ~/lttng-traces/user-space-session*
```
to use [LTTng](https://lttng.org/docs/v2.13/#doc-tracing-your-own-user-application) for recording traces.

## Kernel Bootup

```bash
cat /proc/cmdline
```
to show parameters passed to the kernel by the boot loader.

```bash
sudo vim /etc/default/grub
# edit for example:
# GRUB_TIMEOUT_STYLE=menu
# GRUB_TIMEOUT=3
# GRUB_SAVEDEFAULT=true
# GRUB_DEFAULT=saved
# GRUB_CMDLINE_LINUX=""
sudo update-grub
```
to update grub settings.

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

## Kernel Internals

```bash
objdump -d -j .modinfo <mod-name>
modinfo <mod-name>
systool -m <loaded-mod-name> -v
```
to show general information about a module.

```bash
lsmod
# To load a single module
sudo insmod <module>
sudo rmmod <module>
# To load with dependencies
sudo modprobe <module>
# To unload with unused dependencies
sudo modprobe -r <module>
```
to list all loaded modules, load a module and remove it.

```bash
cat /sys/module/<module-name>/parameters/<param-name>
echo <value> > /sys/module/<module-name>/parameters/<param-name>
```
to query or change modules parameters.

```bash
# To see module dependencies
cat /lib/modules/$(uname -r)/modules.dep
# To see associated devices
cat /lib/modules/$(uname -r)/modules.alias
```
to list module dependencies and product and vendor ids associated with each driver.

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
cat /proc/softirqs
```
to see system stat on soft interrupt requests.

```bash
cat /proc/<pid>/maps
cat /proc/<pid>/smaps
```
to look up a process’s VAS.

```bash
cat /proc/vmallocinfo
```
to see memory allocations in Kernel Virtual Address space.

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
perf trace -e dma:dma_map_page -e dma:dma_unmap_page insmod dma_m2m_module.ko
```
to show trace events hit while running a process.

```bash
watch -n 0.3 cat /proc/<PID>/sched
```
to watch scheduling statistics on a process.

```bash
# -t, --timestamp
# -y, --no-first
# -S, --unit character
vmstat -tyS M 1
```
to see number of interrupts and context switches per second.

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
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

scripts/config --enable CONFIG_IKCONFIG
scripts/config --enable CONFIG_IKCONFIG_PROC
scripts/config --enable CONFIG_KALLSYMS
scripts/config --enable CONFIG_PRINTK_TIME
scripts/config --enable CONFIG_DEBUG_KERNEL
scripts/config --enable CONFIG_DEBUG_INFO
scripts/config --enable CONFIG_DEBUG_MISC
scripts/config --enable CONFIG_MAGIC_SYSRQ
scripts/config --enable CONFIG_DEBUG_FS
scripts/config --enable CONFIG_KGDB
scripts/config --enable CONFIG_UBSAN
scripts/config --enable CONFIG_KCSAN
scripts/config --enable CONFIG_DEBUG_PAGEALLOC
scripts/config --enable CONFIG_DEBUG_PAGEALLOC_ENABLE_DEFAULT
scripts/config --enable CONFIG_SLUB_DEBUG
scripts/config --enable CONFIG_DEBUG_MEMORY_INIT
scripts/config --enable CONFIG_KASAN
scripts/config --enable CONFIG_DEBUG_SHIRQ
scripts/config --enable CONFIG_SCHED_STACK_END_CHECK
scripts/config --enable CONFIG_DEBUG_PREEMPT
scripts/config --enable CONFIG_PROVE_LOCKING
scripts/config --enable CONFIG_RCU_EXPERT
scripts/config --enable CONFIG_LOCK_STAT
scripts/config --enable CONFIG_DEBUG_RT_MUTEXES
scripts/config --enable CONFIG_DEBUG_MUTEXES
scripts/config --enable CONFIG_DEBUG_SPINLOCK
scripts/config --enable CONFIG_DEBUG_RWSEMS
scripts/config --enable CONFIG_DEBUG_LOCK_ALLOC
scripts/config --enable CONFIG_DEBUG_ATOMIC_SLEEP
scripts/config --enable CONFIG_PROVE_RCU_LIST
scripts/config --enable CONFIG_DEBUG_OBJECTS_RCU_HEAD
scripts/config --enable CONFIG_BUG_ON_DATA_CORRUPTION
scripts/config --enable CONFIG_STACKTRACE
scripts/config --enable CONFIG_DEBUG_BUGVERBOSE
scripts/config --enable CONFIG_FTRACE
scripts/config --enable CONFIG_FUNCTION_TRACER
scripts/config --enable CONFIG_FUNCTION_GRAPH_TRACER
scripts/config --enable CONFIG_IRQSOFF_TRACER
scripts/config --enable CONFIG_PREEMPT_TRACER
scripts/config --enable CONFIG_SCHED_TRACER
scripts/config --enable CONFIG_FRAME_POINTER
scripts/config --enable CONFIG_STACK_VALIDATION
```
to add debug configs to kernel

```bash
# Must build with CONFIG_IKCONFIG
zcat /proc/config.gz
```
to see booted kernel build configs.

```
/dts-v1/;
/plugin/;

/ {
	compatible = "brcm,bcm2835";

	fragment@0 {
		target = <&i2c_arm>;
		__overlay__ {
			#address-cells = <1>;
			#size-cells = <0>;
			status = "okay";

			eeprom@50 {
				compatible = "atmel,24c256";
				reg = <0x50>;
				pagesize = <64>;
			};
		};
	};
};
```
to add a node to I2C bus for an eeprom device.

```
/dts-v1/;
/plugin/;

/ {
	fragment@0 {
		target = <&spidev0>;
		__overlay__ {
			compatible = "bosch,bmp280";
			status = "okay";
			reg = <0x0>;
			spi-max-frequency = <10000000>;
		};
	};
};
```
to add a node to SPI bus for a sensor.

```
/dts-v1/;
/plugin/;

/ {
	compatible = "brcm,bcm2835";

	fragment@0 {
		target = <&gpio>;
		__overlay__ {
			light_controller_pins: light_controller_pins {
				brcm,pins = <20 21 16>;  // GPIO20 = LED, GPIO21 = Button
				brcm,function = <1 0 0>; // 1 = output (LED), 0 = input (Button)
				brcm,pull = <0 2 2>;     // 0 = none (LED), 2 = pull-up (Button)
			};
		};
	};

	fragment@1 {
		target-path = "/";
		__overlay__ {
			light_controller: light_controller@0 {
				compatible = "custom,light-controller";
				pinctrl-names = "default";
				pinctrl-0 = <&light_controller_pins>;
				status = "okay";

				led-gpios = <&gpio 20 0>; // 0 = GPIO_ACTIVE_HIGH
				btn-gpios = <&gpio 21 1>; // 1 = GPIO_ACTIVE_LOW
				bck-gpios = <&gpio 16 1>; // 1 = GPIO_ACTIVE_LOW
			};
		};
	};
};
```
to add a platform device node for a push button and an LED. Take a look at gpio-key-overlay.dts and gpio-led-overlay.dts in Raspberry Pi project.

```
/dts-v1/;
/plugin/;

/ {
	compatible = "brcm,bcm2835";

	fragment@0 {
		target = <&gpio>;
		__overlay__ {
			light_controller_pins: light_controller_pins {
				brcm,pins = <20 21 16>;  // GPIO20 = LED, GPIO21 = Button
				brcm,function = <1 0 0>; // 1 = output (LED), 0 = input (Button)
				brcm,pull = <0 2 2>;     // 0 = none (LED), 2 = pull-up (Button)
			};
		};
	};

	fragment@1 {
		target-path = "/";
		__overlay__ {
			light_controller: light_controller@0 {
				compatible = "custom,light-controller";
				pinctrl-names = "default";
				pinctrl-0 = <&light_controller_pins>;
				status = "okay";

				gpios = <&gpio 20 0>,
					<&gpio 21 1>,
					<&gpio 16 1>;
				gpio-line-names = "led", "btn", "bck";
			};
		};
	};
};
```
to aggregate GPIOs.

```bash
echo gpio-aggregator > /sys/bus/platform/devices/light_controller\@0/driver_override
echo light_controller@0 > /sys/bus/platform/drivers/gpio-aggregator/bind
gpiodetect
# gpiochip0 [pinctrl-bcm2835] (54 lines)
# gpiochip1 [light_controller@0] (3 lines)
gpioinfo gpiochip1
# gpiochip1 - 3 lines:
#         line   0:        "led"       unused  output  active-high 
#         line   1:        "btn"       unused   input  active-high 
#         line   2:        "bck"       unused   input  active-high
gpioset gpiochip1 0=1
```
to control aggregated GPIOs.

```bash
dtc -@ -I dts -O dtb -o i2c-eeprom.dtbo i2c-eeprom.dtso 
mkdir /sys/kernel/config/device-tree/overlays/i2c-eeprom
cat i2c-eeprom.dtbo > /sys/kernel/config/device-tree/overlays/i2c-eeprom/dtbo
```
to compile and insert the device tree overlay at runtime.

```
enable_uart=1
dtoverlay=at24c256
dtoverlay=bmp280
dtoverlay=light-controller
```
to load overlays at boot time add above to config.txt in Raspberry Pi's /boot directory.

## Sample Drivers

```c
#include <linux/delay.h>
#include <linux/init.h>
#include <linux/interrupt.h>
#include <linux/kernel.h>
#include <linux/module.h>

#define pr_fmt(fmt) KBUILD_MODNAME ": %s: " fmt, __func__

// Tasklet function declaration
void tasklet_function(struct tasklet_struct *t);

// Declare a tasklet
DECLARE_TASKLET(my_tasklet, tasklet_function);

static int counter = 0;
static const int max_recursions = 10; // Limit the number of recursive calls

// Tasklet callback function
void tasklet_function(struct tasklet_struct *t) {
  pr_info("Running tasklet callback for the %d time(s)\n", ++counter);

  // Schedule the tasklet again if the recursion limit is not reached
  if (counter < max_recursions) {
    // BUG: sleep in interrupt context.
    // msleep(500);
    tasklet_hi_schedule(&my_tasklet);
  } else {
    pr_info("Reached maximum recursion limit of %d\n", max_recursions);
  }
}

// Module initialization function
static int __init my_init(void) {
  pr_info("Initializing tasklet example module\n");
  // Schedule the tasklet
  tasklet_hi_schedule(&my_tasklet);
  return 0;
}

// Module cleanup function
static void __exit my_exit(void) {
  // Ensure the tasklet is killed
  tasklet_kill(&my_tasklet);
  pr_info("Tasklet example module cleanup\n");
}

module_init(my_init);
module_exit(my_exit);

MODULE_AUTHOR("Mohammad Rahimi <rahimi.mhmmd@gmail.com>");
MODULE_DESCRIPTION("Tasklet example with controlled recursion");
MODULE_LICENSE("GPL");
```
to use tasklets in a kernel module.

```c
#include <linux/delay.h>
#include <linux/init.h>
#include <linux/module.h>
#include <linux/sched.h>
#include <linux/time.h>
#include <linux/workqueue.h>

#define pr_fmt(fmt) KBUILD_MODNAME ": %s: " fmt, __func__

static DECLARE_WAIT_QUEUE_HEAD(my_wait_queue);
static struct workqueue_struct* my_work_queue;
static struct work_struct deferred_wake;
static int condition = 0;

// Workqueue handler function
static void work_handler(struct work_struct *work) {
  // OK: sleep in process context
  msleep(5000);
  condition = 1;
  pr_info("Work finished\n");
  wake_up_interruptible(&my_wait_queue);
}

// Module initialization function
static int __init my_init(void) {
  pr_info("Initializing\n");
  my_work_queue = create_singlethread_workqueue("my_work_queue");
  INIT_WORK(&deferred_wake, work_handler);
  queue_work(my_work_queue, &deferred_wake);

  pr_info("Waiting...\n");
  if (wait_event_interruptible(my_wait_queue, condition != 0)) {
    pr_info("Wait interrupted\n");
    return 0;
  }
  pr_info("Wait finished\n");
  return 0;
}

// Module cleanup function
static void __exit my_exit(void) {
  pr_info("Cleaning up\n");
  flush_workqueue(my_work_queue);
  destroy_workqueue(my_work_queue);
}

module_init(my_init);
module_exit(my_exit);

MODULE_AUTHOR("Mohammad Rahimi <rahimi.mhmmd@gmail.com>");
MODULE_DESCRIPTION("Wait queue example with workqueue and interruptible sleep");
MODULE_LICENSE("GPL");
```
to wait for events and defer work with workqueue in a kernel moule.

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/platform_device.h>

#define DEV_BASE 0x02008000
#define DEV_IRQ 31

static struct resource device_resources[] = {
    [0] =
        {
            .start = DEV_BASE,
            .end = DEV_BASE + 0x4000,
            .flags = IORESOURCE_MEM,
        },
    [1] =
        {
            .start = DEV_IRQ,
            .end = DEV_IRQ,
            .flags = IORESOURCE_IRQ,
        },
};

static struct platform_device my_device = {
    .name = "pdev",
    .id = 0,
    .resource = device_resources,
    .num_resources = ARRAY_SIZE(device_resources),
};

static int __init my_device_init(void) {
  int ret;
  ret = platform_device_register(&my_device);
  if (ret) {
    pr_err("Failed to register platform device\n");
    return ret;
  }
  pr_info("Platform device registered successfully\n");
  return 0;
}

static void __exit my_device_exit(void) {
  platform_device_unregister(&my_device);
  pr_info("Platform device unregistered successfully\n");
}

module_init(my_device_init);
module_exit(my_device_exit);

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Platform Device Example");
MODULE_AUTHOR("Mohammad Rahimi<rahimi.mhmmd@gmail.com>");
```
to statically define a platform device.

```c
#include <linux/init.h>
#include <linux/interrupt.h>
#include <linux/module.h>
#include <linux/of_platform.h>
#include <linux/platform_device.h>

static const struct of_device_id of_device_ids[] = {{
                                                        .compatible = "pdev",
                                                    },
                                                    {}};
MODULE_DEVICE_TABLE(of, of_device_ids);

static const struct platform_device_id platform_device_ids[] = {
    {
        .name = "pdev",
    },
    {}};
MODULE_DEVICE_TABLE(platform, platform_device_ids);

static irqreturn_t irq_handler(int irq, void *dev_id) {
  struct platform_device *pdev = dev_id;
  pr_info("IRQ %d handled for device %s\n", irq, pdev->name);
  return IRQ_HANDLED;
}

static u32 *reg_base;
static struct resource *res_irq;

static int pdrv_probe(struct platform_device *pdev) {
  struct resource *regs;
  int ret;

  /* Get the memory resource */
  regs = platform_get_resource(pdev, IORESOURCE_MEM, 0);
  if (!regs) {
    dev_err(&pdev->dev, "Failed to get memory resource\n");
    ret = -ENXIO;
    goto err_out;
  }

  /* Map the memory resource */
  reg_base = devm_ioremap(&pdev->dev, regs->start, resource_size(regs));
  if (IS_ERR(reg_base)) {
    dev_err(&pdev->dev, "Failed to map memory resource\n");
    ret = PTR_ERR(reg_base);
    goto err_out;
  }

  /* Get the IRQ resource */
  res_irq = platform_get_resource(pdev, IORESOURCE_IRQ, 0);
  if (!res_irq) {
    dev_err(&pdev->dev, "Failed to get IRQ resource\n");
    ret = -ENODEV;
    goto err_unmap;
  }

  /* Request the IRQ */
  ret = devm_request_irq(&pdev->dev, res_irq->start, irq_handler, IRQF_SHARED,
                         pdev->name, pdev);
  if (ret) {
    dev_err(&pdev->dev, "Failed to request IRQ\n");
    goto err_unmap;
  }

  dev_info(&pdev->dev, "Platform driver probed successfully\n");
  return 0;

err_unmap:
  devm_iounmap(&pdev->dev, reg_base);
err_out:
  return ret;
}

static void pdrv_remove(struct platform_device *pdev) {
  free_irq(res_irq->start, pdev);
  devm_iounmap(&pdev->dev, reg_base);
  dev_info(&pdev->dev, "Platform driver removed\n");
}

static struct platform_driver my_platform_driver = {
    .probe = pdrv_probe,
    .remove = pdrv_remove,
    .driver =
        {
            .name = "pdev",
            .owner = THIS_MODULE,
        },
    .id_table = platform_device_ids,
};
module_platform_driver(my_platform_driver);

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Platform Driver Example");
MODULE_AUTHOR("Mohammad Rahimi<rahimi.mhmmd@gmail.com>");
```
to create a platform device driver.

```c
#include <linux/cdev.h>
#include <linux/delay.h>
#include <linux/fs.h>
#include <linux/i2c.h>
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/mutex.h>
#include <linux/nvmem-provider.h>

#define MEM_SIZE 256

static struct class *class;
static dev_t devt;
struct cdev cdev;
struct i2c_client *eeprom;

int open(struct inode *inode, struct file *filp);
int release(struct inode *inode, struct file *filp);
ssize_t read(struct file *file, char __user *buf, size_t count, loff_t *offset);
ssize_t write(struct file *file, const char __user *buf, size_t count,
              loff_t *offset);
static loff_t llseek(struct file *file, loff_t offset, int whence);

int open(struct inode *inode, struct file *filp) { return 0; }

int release(struct inode *inode, struct file *filp) { return 0; }

ssize_t read(struct file *file, char __user *buf, size_t count,
             loff_t *offset) {
  struct device *dev = &eeprom->dev;
  dev_info(dev, "read %lu bytes at %llu", count, *offset);

  uint8_t addr_buf[2] = {(*offset >> 8) & 0xFF, *offset & 0xFF};
  uint8_t data_buf[MEM_SIZE] = {};

  dev_info(dev, "addr_buf[0] = 0x%x, addr_buf[1] = 0x%x", addr_buf[0],
           addr_buf[1]);

  if (count >= MEM_SIZE)
    return -EINVAL;

  int ret = i2c_master_send(eeprom, addr_buf, sizeof(addr_buf));
  if (ret < 0)
    return ret;

  ret = i2c_master_recv(eeprom, data_buf, count);
  if (copy_to_user(buf, data_buf, count))
    return -EFAULT;
  return count;
}

ssize_t write(struct file *file, const char __user *buf, size_t count,
              loff_t *offset) {
  struct device *dev = &eeprom->dev;
  dev_info(dev, "write %lu bytes at %llu", count, *offset);

  uint8_t data[MEM_SIZE + 2] = {(*offset >> 8) & 0xFF, *offset & 0xFF};
  if (count >= MEM_SIZE)
    return -EINVAL;

  if (copy_from_user(&data[2], buf, count))
    return -EFAULT;
  i2c_master_send(eeprom, data, count + 2);
  return count;
}

static loff_t llseek(struct file *file, loff_t offset, int whence) {
  loff_t new_pos;

  switch (whence) {
  case SEEK_SET:
    new_pos = offset;
    break;
  case SEEK_CUR:
    new_pos = file->f_pos + offset;
    break;
  case SEEK_END:
    new_pos = MEM_SIZE + offset;
    break;
  default:
    return -EINVAL;
  }

  if (new_pos < 0 || new_pos >= MEM_SIZE)
    return -EINVAL;

  struct device *dev = &eeprom->dev;
  dev_info(dev, "seeking %llu", new_pos);

  file->f_pos = new_pos;
  return new_pos;
}

static struct file_operations fops = {
  open : open,
  release : release,
  read : read,
  write : write,
  llseek : llseek
};

static const struct i2c_device_id eeprom_device_id[] = {{"24c256,custom", 0},
                                                        {}};
MODULE_DEVICE_TABLE(i2c, eeprom_device_id);

static const struct of_device_id eeprom_of_match_table[] = {
    {.compatible = "atmel,24c256,custom"}, {}};
MODULE_DEVICE_TABLE(of, eeprom_of_match_table);

static int probe(struct i2c_client *client) {
  eeprom = client;

  struct device *dev = &client->dev;
  dev_info(dev, "eeprom driver probed");
  return 0;
}

static void remove(struct i2c_client *client) {
  struct device *dev = &client->dev;
  dev_info(dev, "eeprom driver removed");
}

static struct i2c_driver eeprom_driver = {
    .driver = {.name = "eeprom", .of_match_table = eeprom_of_match_table},
    .probe = probe,
    .remove = remove,
    .id_table = eeprom_device_id};

static int __init eeprom_driver_init(void) {
  int error;

  error = alloc_chrdev_region(&devt, 0, 1, "cdev");
  if (error < 0) {
    pr_err("Failed to allocate chrdev region\n");
    return error;
  }

  cdev_init(&cdev, &fops);
  cdev.owner = THIS_MODULE;
  error = cdev_add(&cdev, devt, 1);
  if (error < 0) {
    pr_err("Failed to add cdev\n");
    goto unregister_region;
  }

  class = class_create("class");
  if (IS_ERR(class)) {
    pr_err("Failed to create class\n");
    error = PTR_ERR(class);
    goto del_cdev;
  }

  struct device *device = device_create(class, NULL, devt, NULL, "eeprom0");
  if (IS_ERR(device)) {
    pr_err("Failed to create device\n");
    error = PTR_ERR(device);
    goto destroy_class;
  }

  pr_info("char device added with major = %d and minor = %d\n", MAJOR(devt),
          MINOR(devt));

  return i2c_add_driver(&eeprom_driver);

destroy_class:
  class_destroy(class);
del_cdev:
  cdev_del(&cdev);
unregister_region:
  unregister_chrdev_region(devt, 1);
  return error;
}
module_init(eeprom_driver_init);

static void __exit eeprom_driver_exit(void) {
  i2c_del_driver(&eeprom_driver);

  device_destroy(class, devt);
  class_destroy(class);
  cdev_del(&cdev);
  unregister_chrdev_region(devt, 1);
}
module_exit(eeprom_driver_exit);

MODULE_DESCRIPTION("Dummy Driver for Atmel I2C EEPROM");
MODULE_AUTHOR("Mohammad Rahimi <rahimi.mhmmd@gmail.com>");
MODULE_LICENSE("GPL");
```
to create a character device driver and link it to EEPROM I2C device driver.

```c
#include <linux/cdev.h>
#include <linux/delay.h>
#include <linux/fs.h>
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/mutex.h>
#include <linux/spi/spi.h>

static struct class *class;
static dev_t devt;
struct cdev cdev;
struct spi_device *sensor;

#define s32 int32_t
#define u32 uint32_t
#define s16 int16_t
#define u16 uint16_t

// Global fine temperature value
s32 t_fine;

s32 bmp280_compensate_T(s32 adc_T, u16 dig_T1, s16 dig_T2, s16 dig_T3) {
  s32 var1, var2, T;
  var1 = ((((adc_T >> 3) - ((s32)dig_T1 << 1))) * ((s32)dig_T2)) >> 11;
  var2 = (((((adc_T >> 4) - ((s32)dig_T1)) * ((adc_T >> 4) - ((s32)dig_T1))) >>
           12) *
          ((s32)dig_T3)) >>
         14;

  t_fine = var1 + var2;
  T = (t_fine * 5 + 128) >> 8; // Temperature in 0.01 °C
  return T;
}

int open(struct inode *inode, struct file *filp);
int release(struct inode *inode, struct file *filp);
ssize_t read(struct file *file, char __user *buf, size_t count, loff_t *offset);

int open(struct inode *inode, struct file *filp) { return 0; }
int release(struct inode *inode, struct file *filp) { return 0; }
ssize_t read(struct file *file, char __user *buf, size_t count,
             loff_t *offset) {
  u8 temp_buf[3] = {};
  u8 calib_buf[26] = {};
  u8 address[1] = {};

  address[0] = 0xFA;
  spi_write_then_read(sensor, address, 1, temp_buf, 3);
  address[0] = 0x88;
  spi_write_then_read(sensor, address, 1, calib_buf, 26);

  print_hex_dump(KERN_INFO, "temp: ", DUMP_PREFIX_OFFSET, 16, 1, temp_buf,
                 sizeof(temp_buf), true);
  print_hex_dump(KERN_INFO, "calib: ", DUMP_PREFIX_OFFSET, 16, 1, calib_buf,
                 sizeof(calib_buf), true);
  s32 temp = bmp280_compensate_T(
      temp_buf[0] << 12 | temp_buf[1] << 4 | temp_buf[2] >> 4,
      calib_buf[1] << 8 | calib_buf[0], calib_buf[3] << 8 | calib_buf[2],
      calib_buf[5] << 8 | calib_buf[4]);

  struct device *dev = &sensor->dev;
  dev_info(dev, "Temp = %d.%d", temp / 100, temp % 100);

  return sprintf(buf, "%d.%d", temp / 100, temp % 100);
}

static struct file_operations fops = {
  open : open,
  release : release,
  read : read,
};

static const struct spi_device_id sensor_device_id[] = {{"bmp280", 0}, {}};
MODULE_DEVICE_TABLE(spi, sensor_device_id);

static const struct of_device_id sensor_of_match_table[] = {
    {.compatible = "bosch,bmp280,custom"}, {}};
MODULE_DEVICE_TABLE(of, sensor_of_match_table);

static int probe(struct spi_device *spi) {
  sensor = spi;

  struct device *dev = &spi->dev;
  dev_info(dev, "sensor driver probed");
  return 0;
}

static struct spi_driver sensor_driver = {
    .driver = {.name = "sensor", .of_match_table = sensor_of_match_table},
    .probe = probe,
    .id_table = sensor_device_id};

static int __init sensor_driver_init(void) {
  int error;

  error = alloc_chrdev_region(&devt, 0, 1, "cdev");
  if (error < 0) {
    pr_err("Failed to allocate chrdev region\n");
    return error;
  }

  cdev_init(&cdev, &fops);
  cdev.owner = THIS_MODULE;
  error = cdev_add(&cdev, devt, 1);
  if (error < 0) {
    pr_err("Failed to add cdev\n");
    goto unregister_region;
  }

  class = class_create("class");
  if (IS_ERR(class)) {
    pr_err("Failed to create class\n");
    error = PTR_ERR(class);
    goto del_cdev;
  }

  struct device *device = device_create(class, NULL, devt, NULL, "sensor");
  if (IS_ERR(device)) {
    pr_err("Failed to create device\n");
    error = PTR_ERR(device);
    goto destroy_class;
  }

  pr_info("char device added with major = %d and minor = %d\n", MAJOR(devt),
          MINOR(devt));

  return spi_register_driver(&sensor_driver);

destroy_class:
  class_destroy(class);
del_cdev:
  cdev_del(&cdev);
unregister_region:
  unregister_chrdev_region(devt, 1);
  return error;
}
module_init(sensor_driver_init);

static void __exit sensor_driver_exit(void) {
  spi_unregister_driver(&sensor_driver);

  device_destroy(class, devt);
  class_destroy(class);
  cdev_del(&cdev);
  unregister_chrdev_region(devt, 1);
}
module_exit(sensor_driver_exit);

MODULE_DESCRIPTION("Dummy Driver for BOSCH SPI BMP280");
MODULE_AUTHOR("Mohammad Rahimi <rahimi.mhmmd@gmail.com>");
MODULE_LICENSE("GPL");
```
to create a character device driver and link it to BMP280 sensor through SPI interface.

```c
// sharedmem.c
#include <linux/cdev.h>
#include <linux/device.h>
#include <linux/fs.h>
#include <linux/init.h>
#include <linux/mm.h>
#include <linux/module.h>
#include <linux/slab.h>
#include <linux/uaccess.h>

#define DEVICE_NAME "sharedmem"
#define CLASS_NAME "sharedmem_class"

static int major;
static struct class *sharedmem_class;
static struct cdev sharedmem_cdev;
static void *kernel_mem;

static int sharedmem_open(struct inode *inode, struct file *file) { return 0; }

static int sharedmem_release(struct inode *inode, struct file *file) {
  return 0;
}

static ssize_t sharedmem_write(struct file *file, const char __user *buf,
                               size_t count, loff_t *ppos) {
  memset(kernel_mem, 0, PAGE_SIZE);
  printk(KERN_INFO "sharedmem: memory cleared by write\n");
  return count;
}

static int sharedmem_mmap(struct file *file, struct vm_area_struct *vma) {
  unsigned long pfn = page_to_pfn(virt_to_page(kernel_mem));
  unsigned long size = vma->vm_end - vma->vm_start;

  if (size > PAGE_SIZE)
    return -EINVAL;

  if (remap_pfn_range(vma, vma->vm_start, pfn, size, vma->vm_page_prot))
    return -EAGAIN;

  return 0;
}

static struct file_operations fops = {
    .owner = THIS_MODULE,
    .open = sharedmem_open,
    .release = sharedmem_release,
    .mmap = sharedmem_mmap,
    .write = sharedmem_write,
};

static int __init sharedmem_init(void) {
  dev_t dev;
  int ret;

  ret = alloc_chrdev_region(&dev, 0, 1, DEVICE_NAME);
  if (ret < 0)
    return ret;

  major = MAJOR(dev);

  cdev_init(&sharedmem_cdev, &fops);
  ret = cdev_add(&sharedmem_cdev, dev, 1);
  if (ret < 0)
    goto err_unregister_chrdev;

  sharedmem_class = class_create(CLASS_NAME);
  if (IS_ERR(sharedmem_class)) {
    ret = PTR_ERR(sharedmem_class);
    goto err_cdev_del;
  }

  if (IS_ERR(device_create(sharedmem_class, NULL, dev, NULL, DEVICE_NAME))) {
    ret = -ENOMEM;
    goto err_class_destroy;
  }

  kernel_mem = (void *)get_zeroed_page(GFP_KERNEL);
  if (!kernel_mem) {
    ret = -ENOMEM;
    goto err_device_destroy;
  }

  printk(KERN_INFO "sharedmem: module loaded\n");
  printk(KERN_INFO "sharedmem: kernel_mem masked virtual address: %p\n",
         kernel_mem);
  printk(KERN_INFO "sharedmem: kernel_mem pfn: %lu\n",
         page_to_pfn(virt_to_page(kernel_mem)));

  return 0;

err_device_destroy:
  device_destroy(sharedmem_class, dev);
err_class_destroy:
  class_destroy(sharedmem_class);
err_cdev_del:
  cdev_del(&sharedmem_cdev);
err_unregister_chrdev:
  unregister_chrdev_region(dev, 1);
  return ret;
}

static void __exit sharedmem_exit(void) {
  dev_t dev = MKDEV(major, 0);

  free_page((unsigned long)kernel_mem);
  device_destroy(sharedmem_class, dev);
  class_destroy(sharedmem_class);
  cdev_del(&sharedmem_cdev);
  unregister_chrdev_region(dev, 1);

  printk(KERN_INFO "sharedmem: module unloaded\n");
}

module_init(sharedmem_init);
module_exit(sharedmem_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Mohammad Rahimi <rahimi.mhmmd@gmail.com>");
MODULE_DESCRIPTION("Shared memory mmap example");
```

```c
// read.c
#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>
#include <unistd.h>

#define DEVICE_PATH "/dev/sharedmem"
#define PAGE_SIZE 4096

int main() {
  int fd = open(DEVICE_PATH, O_RDWR);
  if (fd < 0) {
    perror("Failed to open device");
    return 1;
  }
  void *addr = mmap(NULL, PAGE_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
  if (addr == MAP_FAILED) {
    perror("mmap failed");
    close(fd);
    return 1;
  }
  printf("Mapped memory address: %p\n", addr);

  // Print first 16 bytes of shared memory
  printf("Initial contents of shared memory (first 16 bytes):\n");
  for (int i = 0; i < 16; ++i) {
    printf("%02x ", ((unsigned char *)addr)[i]);
  }
  printf("\n");

  // Wait for user to press Enter
  printf("Press Enter to read shared memory again...");
  getchar();

  // Print first 16 bytes again
  printf("Contents of shared memory after modification (first 16 bytes):\n");
  for (int i = 0; i < 16; ++i) {
    printf("%02x ", ((unsigned char *)addr)[i]);
  }
  printf("\n");

  munmap(addr, PAGE_SIZE);
  write(fd, "clear", 5);
  close(fd);
  return 0;
}
```

```c
// write.c
#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>
#include <unistd.h>

#define DEVICE_PATH "/dev/sharedmem"
#define PAGE_SIZE 4096

int main() {
  int fd = open(DEVICE_PATH, O_RDWR);
  if (fd < 0) {
    perror("Failed to open device");
    return 1;
  }
  void *addr = mmap(NULL, PAGE_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
  if (addr == MAP_FAILED) {
    perror("mmap failed");
    close(fd);
    return 1;
  }
  printf("Mapped memory address: %p\n", addr);

  // Write binary values to shared memory
  unsigned char values[16] = {0xDE, 0xAD, 0xBE, 0xEF, 0x12, 0x34, 0x56, 0x78,
                              0x9A, 0xBC, 0xDE, 0xF0, 0x11, 0x22, 0x33, 0x44};
  memcpy(addr, values, sizeof(values));
  printf("Wrote binary values to shared memory.\n");

  munmap(addr, PAGE_SIZE);
  close(fd);
  return 0;
}
```
to allocate memory and share it between userspace process plus userspace programs to test.

```c
#include <linux/completion.h>
#include <linux/delay.h>
#include <linux/dma-mapping.h>
#include <linux/dmaengine.h>
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/mutex.h>
#include <linux/slab.h>

#define pr_fmt(fmt) "m2m module: " fmt

#define BUF_SIZE 2 * PAGE_SIZE

static void m2m_callback(void *completion) {
  pr_info("DMA transfer completed\n");
  complete(completion);
}

static int __init test_module_init(void) {
  struct dma_async_tx_descriptor *m2m_desc;
  struct completion m2m_completion;
  struct dma_chan *m2m_chan;
  dma_cap_mask_t mask;
  dma_cookie_t cookie;
  struct device *dev;
  dma_addr_t m2m_src;
  dma_addr_t m2m_dst;
  u32 *write_buf;
  u32 *read_buf;
  int ret = 0;

  read_buf = kzalloc(BUF_SIZE, GFP_KERNEL | GFP_DMA);
  if (!read_buf) {
    pr_err("failed to allocate read buffer\n");
    ret = -ENOMEM;
    goto err_out;
  }

  for (int i = 0; i < BUF_SIZE / sizeof(u32); i++) {
    read_buf[i] = i;
  }

  write_buf = kzalloc(BUF_SIZE, GFP_KERNEL | GFP_DMA);
  if (!write_buf) {
    pr_err("failed to allocate write buffer\n");
    ret = -ENOMEM;
    goto err_free_read_buf;
  }

  dma_cap_zero(mask);
  dma_cap_set(DMA_MEMCPY, mask);

  m2m_chan = dma_request_channel(mask, NULL, NULL);
  if (!m2m_chan) {
    pr_err("failed to acquire DMA channel\n");
    ret = -ENODEV;
    goto err_free_write_buf;
  }
  dev = m2m_chan->device->dev;

  pr_info("channel acquired: %d\n", m2m_chan->chan_id);
  init_completion(&m2m_completion);

  m2m_src = dma_map_single(dev, read_buf, BUF_SIZE, DMA_TO_DEVICE);
  if (dma_mapping_error(dev, m2m_src)) {
    pr_err("failed to map src buffer\n");
    ret = -EIO;
    goto err_release_chan;
  }
  pr_info("read_buf mapped:  vir (%px) bus (%pad)\n", read_buf, &m2m_src);

  m2m_dst = dma_map_single(dev, write_buf, BUF_SIZE, DMA_FROM_DEVICE);
  if (dma_mapping_error(dev, m2m_dst)) {
    pr_err("failed to map dst buffer\n");
    ret = -EIO;
    goto err_unmap_src_buf;
  }
  pr_info("write_buf mapped: vir (%px) bus (%pad)\n", write_buf, &m2m_dst);

  m2m_desc = dmaengine_prep_dma_memcpy(m2m_chan, m2m_dst, m2m_src, BUF_SIZE, 0);
  if (!m2m_desc) {
    pr_err("failed to prepare DMA memcpy descriptor\n");
    ret = -ENOMEM;
    goto err_unmap_dst_buf;
  }
  pr_info("DMA memcpy descriptor prepared\n");

  m2m_desc->callback = m2m_callback;
  m2m_desc->callback_param = &m2m_completion;

  cookie = dmaengine_submit(m2m_desc);
  if (dma_submit_error(cookie)) {
    pr_err("failed to submit DMA descriptor\n");
    ret = -EIO;
    goto err_unmap_dst_buf;
  }
  pr_info("DMA descriptor submitted with cookie: %d\n", cookie);

  dma_async_issue_pending(m2m_chan);
  pr_info("DMA transfer started\n");

  if (!wait_for_completion_timeout(&m2m_completion, msecs_to_jiffies(5000))) {
    pr_err("DMA transfer timed out\n");
    ret = -ETIMEDOUT;
    goto err_unmap_dst_buf;
  }

  dma_sync_single_for_cpu(dev, m2m_dst, BUF_SIZE, DMA_FROM_DEVICE);

  if (memcmp(read_buf, write_buf, BUF_SIZE)) {
    pr_err("data mismatch after DMA transfer\n");
    ret = -EIO;
  } else {
    pr_info("data verified successfully\n");
  }

err_unmap_dst_buf:
  pr_info("unmapping destination buffer\n");
  dma_unmap_single(dev, m2m_dst, BUF_SIZE, DMA_FROM_DEVICE);
err_unmap_src_buf:
  pr_info("unmapping source buffer\n");
  dma_unmap_single(dev, m2m_src, BUF_SIZE, DMA_TO_DEVICE);
err_release_chan:
  pr_info("terminating DMA channel\n");
  dmaengine_terminate_sync(m2m_chan);
  pr_info("releasing DMA channel\n");
  dma_release_channel(m2m_chan);
err_free_write_buf:
  pr_info("freeing write buffer\n");
  kfree(write_buf);
err_free_read_buf:
  pr_info("freeing read buffer\n");
  kfree(read_buf);
err_out:
  return ret;
}
module_init(test_module_init);

static void __exit test_module_exit(void) {
  pr_info("test module exited cleanly\n");
}
module_exit(test_module_exit);

MODULE_DESCRIPTION("Test module for DMA memory-to-memory transfer");
MODULE_AUTHOR("Mohammad Rahimi <rahimi.mhmmd@gmail.com>");
MODULE_LICENSE("GPL");
```
to use DMA for memory to memory data transfer.

```c
#include <linux/completion.h>
#include <linux/delay.h>
#include <linux/dma-mapping.h>
#include <linux/dmaengine.h>
#include <linux/init.h>
#include <linux/io.h>
#include <linux/ioport.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/mutex.h>
#include <linux/slab.h>

#define pr_fmt(fmt) "gpio_dma: " fmt

// See "bcm2837.dtsi"
#define GPIO_BASE_PHYS 0x3F200000
#define GPIO_BASE_BUS 0x7E200000
// Chapter 6 of BCM2835 ARM Peripherals
#define GPIO_SIZE 0xA0
#define GPFSEL2 0x08
#define GPSET0 0x1C
#define GPCLR0 0x28

#define BIT29 (1u << 29)

#define BUF_SIZE PAGE_SIZE

#define DW_COUNT 6
#define ON_CYCLES 100
#define TOTAL_CYCLES PAGE_SIZE / sizeof(uint32_t) / DW_COUNT

static void on_complete(void *completion) {
  pr_info("DMA transfer completed\n");
  complete(completion);
}

static int __init gpio_dma_init(void) {
  const int act_led_bit = (29 - 20) * 3;
  void __iomem *gpio_base;
  u32 gpfsel2;
  int ret = 0;

  gpio_base = ioremap(GPIO_BASE_PHYS, GPIO_SIZE);
  if (!gpio_base) {
    pr_err("Failed to ioremap GPIO memory\n");
    ret = -ENOMEM;
    goto err_out;
  }
  pr_info("Mapped GPIO memory at %px\n", gpio_base);

  // Read, modify and write Act LED GPIO register 29
  gpfsel2 = ioread32(gpio_base + GPFSEL2);
  pr_info("GPFSEL2 before: 0x%08x\n", gpfsel2);

  gpfsel2 &= ~(0b111 << act_led_bit); // Clear the bits
  gpfsel2 |= (0b001 << act_led_bit);  // Set to output
  iowrite32(gpfsel2, gpio_base + GPFSEL2);

  gpfsel2 = ioread32(gpio_base + GPFSEL2);
  pr_info("GPFSEL2 after:  0x%08x\n", gpfsel2);

  // Turn the Act LED on
  iowrite32(BIT29, gpio_base + GPCLR0);
  pr_info("Act LED turned on\n");
  msleep(1000);

  struct dma_async_tx_descriptor *mem2dev_desc;
  struct dma_slave_config channel_cfg = {0};
  struct completion mem2dev_completion;
  struct dma_chan *mem2dev_chan;
  dma_cap_mask_t mask;
  dma_cookie_t cookie;
  struct device *dev;
  dma_addr_t mem_src;
  u32 *read_buf;

  read_buf = kzalloc(BUF_SIZE, GFP_KERNEL | GFP_DMA);
  if (!read_buf) {
    pr_err("failed to allocate read buffer\n");
    ret = -ENOMEM;
    goto err_out;
  }

  uint8_t *ptr = (uint8_t *)read_buf;
  uint32_t on_array[DW_COUNT] = {0, 0, 0, BIT29, 0, 0};
  uint32_t off_array[DW_COUNT] = {BIT29, 0, 0, 0, 0, 0};
  for (int i = 0; i < ON_CYCLES; ++i, ptr += sizeof(on_array)) {
    memcpy(ptr, on_array, sizeof(on_array));
  }
  for (int i = ON_CYCLES; i < TOTAL_CYCLES; ++i, ptr += sizeof(off_array)) {
    memcpy(ptr, off_array, sizeof(off_array));
  }
  pr_info("blinking LED initialized with %d on and %d off cycles\n", ON_CYCLES,
          TOTAL_CYCLES - ON_CYCLES);
  print_hex_dump(KERN_INFO, "Read Buffer: ", DUMP_PREFIX_OFFSET, 16, 4,
                 read_buf, BUF_SIZE, false);

  dma_cap_zero(mask);
  dma_cap_set(DMA_SLAVE, mask);
  mem2dev_chan = dma_request_channel(mask, NULL, NULL);
  if (!mem2dev_chan) {
    pr_err("failed to acquire DMA channel\n");
    ret = -ENODEV;
    goto err_free_read_buf;
  }
  dev = mem2dev_chan->device->dev;
  pr_info("channel acquired: %d\n", mem2dev_chan->chan_id);

  channel_cfg.direction = DMA_MEM_TO_DEV;
  channel_cfg.dst_addr = GPIO_BASE_PHYS + GPSET0;
  channel_cfg.dst_addr_width = DMA_SLAVE_BUSWIDTH_4_BYTES;
  dmaengine_slave_config(mem2dev_chan, &channel_cfg);
  pr_info("DMA channel configured for GPIO\n");

  mem_src = dma_map_single(dev, read_buf, BUF_SIZE, DMA_TO_DEVICE);
  if (dma_mapping_error(dev, mem_src)) {
    pr_err("failed to map src buffer\n");
    ret = -EIO;
    goto err_release_chan;
  }
  pr_info("read_buf mapped:  vir (%px) bus (%pad)\n", read_buf, &mem_src);

  // mem2dev_desc = dmaengine_prep_dma_cyclic(mem2dev_chan, mem_src,
  //                                          TOTAL_CYCLES * sizeof(on_array),
  //                                          sizeof(on_array), DMA_MEM_TO_DEV,
  //                                          0);
  mem2dev_desc = dmaengine_prep_slave_single(mem2dev_chan, mem_src,
                                             sizeof(off_array), DMA_MEM_TO_DEV,
                                             DMA_PREP_INTERRUPT | DMA_CTRL_ACK);
  if (!mem2dev_desc) {
    pr_err("failed to prepare DMA memcpy descriptor\n");
    ret = -ENOMEM;
    goto err_unmap_src_buf;
  }
  pr_info("DMA memcpy descriptor prepared\n");

  init_completion(&mem2dev_completion);
  mem2dev_desc->callback = on_complete;
  mem2dev_desc->callback_param = &mem2dev_completion;

  cookie = dmaengine_submit(mem2dev_desc);
  if (dma_submit_error(cookie)) {
    pr_err("failed to submit DMA descriptor\n");
    ret = -EIO;
    goto err_unmap_src_buf;
  }
  pr_info("DMA descriptor submitted with cookie: %d\n", cookie);

  dma_async_issue_pending(mem2dev_chan);
  pr_info("DMA transfer started\n");

  if (!wait_for_completion_timeout(&mem2dev_completion,
                                   msecs_to_jiffies(5000))) {
    pr_err("DMA transfer timed out\n");
    ret = -ETIMEDOUT;
    goto err_unmap_src_buf;
  }

err_unmap_src_buf:
  pr_info("unmapping source buffer\n");
  dma_unmap_single(dev, mem_src, BUF_SIZE, DMA_TO_DEVICE);
err_release_chan:
  pr_info("terminating DMA channel\n");
  dmaengine_terminate_sync(mem2dev_chan);
  pr_info("releasing DMA channel\n");
  dma_release_channel(mem2dev_chan);
err_free_read_buf:
  pr_info("freeing read buffer\n");
  kfree(read_buf);
err_iounmap:
  iounmap(gpio_base);
  pr_info("Unmapped GPIO memory\n");
err_out:
  return ret;
}

static void __exit gpio_dma_exit(void) { pr_info("Exiting GPIO DMA module\n"); }

module_init(gpio_dma_init);
module_exit(gpio_dma_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Mohammad Rahimi");
MODULE_DESCRIPTION("Direct memory access to GPIO registers");
```
to use DMA for memory to GPIO transfer.

```c
#include <linux/completion.h>
#include <linux/ctype.h>
#include <linux/delay.h>
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/module.h>
#include <linux/slab.h>
#include <linux/sysfs.h>
#include <linux/types.h>

#define pr_fmt(fmt) "sysfs-demo: " fmt

char *bin_buf;
int value = 0;
struct kobject *demo_kobj;

static ssize_t demo_bin_read(struct file *filp, struct kobject *kobj,
                             struct bin_attribute *attr, char *buf, loff_t off,
                             size_t count) {
  if (off >= attr->size)
    return 0;
  if (off + count > attr->size)
    count = attr->size - off;

  memcpy(buf, bin_buf + off, count);
  pr_info("Read %zu bytes from offset %lld\n", count, off);
  return count;
}
static ssize_t demo_bin_write(struct file *filp, struct kobject *kobj,
                              struct bin_attribute *attr, char *buf, loff_t off,
                              size_t count) {
  if (off >= attr->size)
    return -EINVAL;
  if (off + count > attr->size)
    count = attr->size - off;

  memcpy(bin_buf + off, buf, count);
  pr_info("Wrote %zu bytes to offset %lld\n", count, off);
  return count;
}
static ssize_t demo_ascii_show(struct kobject *kobj,
                               struct kobj_attribute *attr, char *buf) {
  pr_info("ASCII show called\n");
  return sprintf(buf, "%u\n", value);
}
static ssize_t demo_ascii_store(struct kobject *kobj,
                                struct kobj_attribute *attr, const char *buf,
                                size_t count) {
  pr_info("ASCII store called\n");
  int rc;
  rc = kstrtoint(buf, 0, &value);
  if (rc) {
    pr_err("Invalid input\n");
    return rc;
  }
  return count;
}

static struct bin_attribute demo_bin_attr = {
    .attr =
        {
            .name = "bin_attr",
            .mode = S_IRUGO | S_IWUSR,
        },
    .size = PAGE_SIZE,
    .read = demo_bin_read,
    .write = demo_bin_write,
};
static struct kobj_attribute demo_ascii_attr = {
    .attr =
        {
            .name = "ascii_attr",
            .mode = S_IRUGO | S_IWUSR,
        },
    .show = demo_ascii_show,
    .store = demo_ascii_store,
};

static int __init demo_init(void) {
  pr_info("Initializing module\n");
  int ret;

  bin_buf = kzalloc(PAGE_SIZE, GFP_KERNEL);
  if (!bin_buf) {
    pr_err("Failed to allocate memory\n");
    ret = -ENOMEM;
    goto out;
  }

  demo_kobj = kobject_create_and_add("demo_sysfs", kernel_kobj);
  if (!demo_kobj) {
    pr_err("Failed to create kobject\n");
    ret = -ENOMEM;
    goto out_bin_buf;
  }

  ret = sysfs_create_bin_file(demo_kobj, &demo_bin_attr);
  if (ret) {
    pr_err("Failed to create the sysfs binary file\n");
    goto out_demo_kobj;
  }

  ret = sysfs_create_file(demo_kobj, &demo_ascii_attr.attr);
  if (ret) {
    pr_err("Failed to create the sysfs ascii file\n");
    goto out_demo_kobj;
  }

  return 0;

out_demo_kobj:
  kobject_put(demo_kobj);
out_bin_buf:
  kfree(bin_buf);
out:
  return ret;
}

static void __exit demo_exit(void) {
  pr_info("Exiting module\n");
  kobject_put(demo_kobj);
  kfree(bin_buf);
}

module_init(demo_init);
module_exit(demo_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Mohammad Rahimi");
MODULE_DESCRIPTION("Creates sysfs entries with binary and ASCII attributes");
```
to create a sysfs entry with an ascii and a binary attribute.

```c
#include <linux/ctype.h>
#include <linux/delay.h>
#include <linux/gpio/consumer.h>
#include <linux/interrupt.h>
#include <linux/module.h>
#include <linux/of.h>
#include <linux/platform_device.h>
#include <linux/time.h>
#include <linux/workqueue.h>

#define pr_fmt(fmt) "light-controller: " fmt

static struct gpio_desc *led_gpio;
static struct gpio_desc *btn_gpio;
static struct gpio_desc *bck_gpio;
static bool led_on = false;
static struct delayed_work work;

static void toggle_led(struct work_struct *work) {
  led_on = !led_on;
  pr_info("Button pressed, LED is now %s\n", led_on ? "ON" : "OFF");
  gpiod_set_value(led_gpio, led_on);
}

static irqreturn_t btn_irq_handler(int irq, void *dev_id) {
  pr_info("IRQ triggered for btn\n");
  mod_delayed_work(system_wq, &work,
                   msecs_to_jiffies(200)); // Software debounce delay
  return IRQ_HANDLED;
}
static irqreturn_t bck_irq_handler(int irq, void *dev_id) {
  pr_info("IRQ triggered for bck\n");
  led_on = 0;
  gpiod_set_value(led_gpio, led_on);
  cancel_delayed_work_sync(&work);
  return IRQ_HANDLED;
}

static int light_controller_probe(struct platform_device *pdev) {
  int ret, irq_number;

  led_gpio = devm_gpiod_get(&pdev->dev, "led", GPIOD_OUT_LOW);
  if (IS_ERR(led_gpio)) {
    pr_err("Failed to get LED GPIO\n");
    ret = PTR_ERR(led_gpio);
    goto err_out;
  }
  btn_gpio = devm_gpiod_get(&pdev->dev, "btn", GPIOD_IN);
  if (IS_ERR(btn_gpio)) {
    pr_err("Failed to get btn GPIO\n");
    ret = PTR_ERR(btn_gpio);
    goto err_out;
  }
  bck_gpio = devm_gpiod_get(&pdev->dev, "bck", GPIOD_IN);
  if (IS_ERR(bck_gpio)) {
    pr_err("Failed to get bck GPIO\n");
    ret = PTR_ERR(bck_gpio);
    goto err_out;
  }

  INIT_DELAYED_WORK(&work, toggle_led);
  irq_number = gpiod_to_irq(btn_gpio);
  pr_info("IRQ number for btn: %d\n", irq_number);
  ret = devm_request_threaded_irq(&pdev->dev, irq_number, NULL, btn_irq_handler,
                                  IRQF_TRIGGER_RISING | IRQF_ONESHOT, "btn_irq",
                                  NULL);
  if (ret) {
    pr_err("Failed to request IRQ\n");
    goto err_out;
  }

  irq_number = gpiod_to_irq(bck_gpio);
  pr_info("IRQ number for bck: %d\n", irq_number);
  ret = devm_request_threaded_irq(&pdev->dev, irq_number, NULL, bck_irq_handler,
                                  IRQF_TRIGGER_RISING | IRQF_ONESHOT, "bck_irq",
                                  NULL);
  if (ret) {
    pr_err("Failed to request IRQ\n");
    goto err_out;
  }

  ret = 0;
err_out:
  return ret;
}

static void light_controller_remove(struct platform_device *pdev) {
  gpiod_set_value(led_gpio, 0); // Turn off LED
  pr_info("Light controller module removed\n");
  return;
}

static const struct of_device_id light_controller_of_match[] = {
    {.compatible = "custom,light-controller"},
    {},
};
MODULE_DEVICE_TABLE(of, light_controller_of_match);

static struct platform_driver light_controller_driver = {
    .probe = light_controller_probe,
    .remove = light_controller_remove,
    .driver =
        {
            .name = "light-controller",
            .of_match_table = light_controller_of_match,
            .owner = THIS_MODULE,
        },
};
module_platform_driver(light_controller_driver);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Mohammad Rahimi");
MODULE_DESCRIPTION("Manipulate GPIOs from a kernel module");
```
to monitor and control GPIOs for a platform device.

## Builds

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

# Copy distribution config file to source tree
cp /boot/config-$(uname -r) .config
make olddefconfig
# To avoid error regarding CONFIG_SYSTEM_REVOCATION_KEYS="debian/canonical-revoked-certs.pem"
scripts/config --disable SYSTEM_REVOCATION_KEYS
make menuconfig

make -j$(nproc) all | tee ../config_$(TZ='Asia/Singapore' date +%Y-%m-%dT%H.%M.%S%Z).log
sudo make modules_install
sudo make install

# Old kernel version
uname --kernel-release
sudo reboot
# New kernel version
uname --kernel-release
```
to update kernel from source.

```bash
# On host
KERNEL=kernel8
# Copy config file to source tree
scp <user>@<address>:/boot/<config-file> .config
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make olddefconfig
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make menuconfig
# To save log use | tee $(TZ='Asia/Singapore' date +%Y-%m-%dT%H.%M.%S%Z).log
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j$(nproc) all
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make -j$(nproc) deb-pkg

# Copy new kernel to SoC
cd ..
scp *.deb <user>@<address>:/home/<user>

# On SoC
sudo dpkg -i *.deb
```
to cross-compile and install kernel for an SoC.

```bash
# Add this to drivers/char/Kconfig
source "drivers/char/mydrv/Kconfig"

# Add this to drivers/char/Makefile
obj-$(CONFIG_MYDRV) += mydrv/

# Create drivers/char/mydrv/Kconfig with the following
config MYDRV
        tristate "My test driver"
        default m
        help
          This creates a test driver.
          Will add /dev/mydrv char device.

# Create drivers/char/mydrv/Makefile with the following
obj-$(CONFIG_MYDRV) += mydrv.o

# Create drivers/char/mydrv/mydrv.c with your code
```
to add in-tree Linux driver.

```make
# Create Makefile with the following
obj-m := mydrv.o
mydrv-y := srcfile1.c srcfile2.c
KERNEL_SRC ?= /lib/modules/$(shell uname -r)/build
all default: modules
install: modules_install
modules modules_install help clean:
        $(MAKE) -C $(KERNEL_SRC) M=$(shell pwd) $@
```
to compile an out-of-tree Linux driver. Run `make` afterwards.

```bash
# on host
M=$(pwd) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- \
  make -C <kernel-source-tree> modules
M=$(pwd) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- \
  make -C <kernel-source-tree> INSTALL_MOD_PATH=<temp-directory> modules_install
tar zcvf - <temp-directory> | ssh <user-name>@<target-address> tar zxvf -

# on target
cd ~/<temp-directory>
sudo su
cp -r lib /lib
echo <module-name> >> /etc/modules-load.d/modules.conf
depmod
reboot
```
to cross-compile a linux module and install it on the remote target.

## Scripts

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
