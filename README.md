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
tar -ztvf arch.tar.gz
```
to show table of contents for the archive. Use `zxvf` to extract. Use `zcvf` to create an archive from files.
