# DotBashHistory
This file contains some useful Linux commands and configs.
## General
```bash
chsh
```
to change shell. 

```bash
ls -alih .
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
