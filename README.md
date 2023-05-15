# DotBashHistory
This file contains some useful Linux commands and configs.
## General
```bash
chsh
```
to change shell.

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
```
to create then show table of contents for the archive then extract it.

```bash
ps ax 2>&1 1>> stdout.txt && cat stdout.txt
```
to list all processes and send outputs to appropriate files.

```bash
PATH=first:$PATH:last
```
to add directories to search path.

```bash
mknod mypipe p
tail -f mypipe &
man man > mypipe
```
to create a named pipe and connect two processes through it.

```bash
udevadm monitor
```
to monitor events passed between udev and kernel.

```bash
udevadm info /dev/sda
```
to get all information for a device.

# Journal
```bash
journalctl -r SYSLOG_IDENTIFIER=sudo
```
to list most recent commands ran with `sudo`.

```bash
journalctl -S -1h30m
```
to list all logs since one and a half hours ago.

```bash
journalctl -S 06:00:00 -U 07:00:00
```
to list all logs since 6 AM until 7 AM.

```bash
journalctl -fk
```
to show a live feed for kernel messages.

```bash
journalctl -p 3..4
```
to show only errors and warnings.

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
mount -t ext4 /dev/sdx mpoint/
umount /dev/sdx
```
to mount and unmount a filesystem.

```bash
blkid
mount UUID=xx mpoint/
```
to list UUIDs and mount using UUIDs.

