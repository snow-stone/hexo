---
title: Linux Cheat Sheet
date: 2018-09-26 16:26:59
tags:
---

# bash

```bash
# kill一系列进程，ps 可能搞出很多其他不相干的进程号，一定要注意，而且前三行一定不是，怎样通过awk滤过还不清楚
$ ps | awk '{print $1}' | xargs kill


# check for all jobs including "nohup" (at the same login noed)
$ ps -eaf | grep $USER


# check for graphic card status
# both Xorg and gnome-shell stores for cache (computer has already run for many days). After rebooting, there will be much less.
$ nvidia-smi


# screen
# name a screen session
$ screen -S someName
# detache from screen
Ctrl + A then D
# see running screen
$ screen -ls
# re-attach to a screen
$ screen -r ** 

# rsync

# --dry-run
$ rsync --dry-run -av --progress sourcefolder /destinationfolder --exclude thefoldertoexclude
# show progress
$ rsync -av --progress sourcefolder /destinationfolder --exclude thefoldertoexclude
# rsync multiple files at one line, passwd input only once
$ rsync -av remote:dirToCase/{0.165,0.18,0.195,0.21,0.225} /some/local/dir/


# mkdir / rm / regular expressions / using {} for multiple-file usage

# create multiple dirs
$ mkdir processor{0..11}
# rm directories under processor0, processor1, processor2, processor3 which is named 0.1
$ rm -r processor{0..3}/0.1
# rm directories under processor* named '2.8*' '2.9*' using "regular expression"
$ rm -r processor*/2.[89]*
# this is cool, delete files in any directory named as XEq*-cellPoint_p.xy or XEq*-cellPoint_U.xy
$ rm */XEq*-cellPoint_{p,U}.xy   


# sort

$ sort file.csv -n -t "," -k 22 -k 23 -o output 
# sort file *.csv by colon 22 then by colon 23


# find

# counts for the number of directories including it self "." This also counts the hidden directories prefixed by '.'. This is actually a count for "newlines" because wc -l counts for newlines and -print produces output separated by newlines. Changing -print to -print0 which will separates output by null, then there will be no newline and wc -l will always give 0 as result.
$ find . -maxdepth 1 -type d -print | wc -l 

# find directories in . and grep the str (**fast**)
$ find . -type d | grep "2" 
# (**slower**) but only will not produce results of sub directory of '2' as the previous one did.
$ find . -type d -name '2' -print  # dry run 
$ find . -type d -name '2' -delete # this wont work !! instead : xxx | xargs rm -r 是有效的

# find file matching pattern (dry run) then delete them.
$ find . -type f -name 'plotoverline.*.csv' -print 
$ find . -type f -name 'plotoverline.*.csv' -delete 
# find in somedir all subdirectories and chmod to defaut 775
$ find somedir -type d -exec chmod 775 {} \;  


# chmod

# this will change subdirectories and files ! Which is normally not GOOD. You dont want to have files all green/executable.
chmod -R 755 somedir  


# file

$ file *.so        shared object
$ file executable  dynamically/statically linked
$ file *.a         ar archive
```


# vim

```bash

# search 
:/some_string\c    # case insensitive
:/some_string\C    # case sensitive

# git rid of ^M
# 有时候用vim打开文件会发现"^M"，如何去掉？
$ dos2unix fileName

# 多行选中
ctrl + v (visual block) -> 
use cursor (h j k l) to move and select a multi-line-block -> 
s (insert) -> 
input "#" -> 
Esc -> 
all select block is then replaced by "#"

# 1到10行，有caption关键词的行首加上#
1,10s/.*caption/#\0/
```
