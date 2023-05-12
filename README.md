# DotBashHistory
This file contains some useful Linux commands and configs.
## General
```bash
chsh
```
to change shell. 

```bash
ls -ahli .
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
tar -zxvf arch.tar.gz
```
to create then show table of contents for the archive then extract it.

```bash
ps ax 2>&1 1>> stdout.txt && cat stdout.txt
```
to list all processes and send outputs to appropriate files.

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
journalctl -f
```
to show a live feed. 

```bash
journalctl -p 3..4
```
to show errors and warnings.
