---
title: Linux Cheat Sheet
date: 2018-09-26 16:26:59
tags: cheatSheet
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
:1,10s/.*caption/#\0/

# encoding (other than utf8)
:set fileencoding=iso-8859-1

# paste without autoindent
:set paste

# add cursorline
:set cursorline

# reload file
:e
:edit

# 遇到难缠的(可能是硬的tab，也就是当下如果vim配置是以空格来做tab，也能通过/\t，搜索得到，且vim光标摁'j'移动不到行首，会卡住在这些tab的位置)
# IndentationError: unindent does not match any outer indentation level
:retab

```

# git

```bash

# 密码缓存300秒

$ git config credential.helper 'cache --timeout=300'

# 不再输入密码

$ git config --global credential.helper store


```

# latex

建议做个makefile方便操作（见下）
```bash
new=actualVersion
old=olderVersion

all:
    pdflatex ${new}.tex
    bibtex ${new}
    pdflatex ${new}.tex
    pdflatex ${new}.tex

old:
    pdflatex ${old}.tex
    bibtex ${old}
    pdflatex ${old}.tex
    pdflatex ${old}.tex

bibs:
    bibtex ${new}
    bibtex ${old}

diff:
    latexdiff ${old}.tex ${new}.tex > diff.tex
    latexdiff ${old}.bbl ${new}.bbl > diff.bbl
    pdflatex diff.tex
    pdflatex diff.tex

clean:
    rm *.bbl *.aux *.blg *.log
```

完整编译`${new}.tex`：`make`
完整编译`${old}.tex`：`make old`
用`latexdiff`完成对整篇`new`与`old`版本的对校（可以包括reference也可以不包括）: `make diff`

用到reference对校功能的时候要注意：   
1. 比较的是`*.bbl`而不是`*.bib`   
2. 对校reference的时候，用`make bibs ; make diff`
3. 对校成功的话，`old`为红色，`new`为蓝色     
4. 改`new.bib`的时候要注意尽量不要增加新的field：例如`old.bib`里面没有`pages`,`volume`，如果在`new.bib`中加入，有可能让`latexdiff old.bbl new.bbl > diff.bbl` prompt，可以通过`enter`来跳过这一系列warning但最终得到的`diff.pdf`在reference部分会**在增加field的当前项开始后面全篇**都会被蓝色override干扰校对。经测试：增减1~2个field似乎在允许范围内  
5. 让`latexdiff old.bbl new.bbl > diff.bbl` prompt有可能是`Stop`和`NoStop`，这种warning可以忽略

## beamer

overlays : 
```latex
\only<1>{}

\onlyenv{}

\begin{overprint}
    \onslide<1>
\end{overprint}

%...
```
