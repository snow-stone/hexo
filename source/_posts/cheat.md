---
title: cheat
date: 2018-09-19 22:20:33
tags:
---

# 如何骗OpenFOAM

## timeDir篇
如果小心输出时间步全部刚好差了0.01怎么办，例如[5,7]变成了[5.01,7.01]，timeDir里面的`U,p`都有个冠冕堂皇的`location`的值对应时间。但其实可以把目录`5.01`改成`5`，`7.01`改成`7`，这时比如要续算，solver便认为对应的时间步其实是`5`和`7`。似乎sample这些utility也可以这样骗[待确认]。

一句话：外面的目录名是个“货真价实”的幌子，可以欺骗solver和utility


# 如何被OpenFOAM骗

## reconstructPar与decomposePar
这俩不互为反函数，如果要用这种方式完成binary和ascii的转换，一定double check，尤其BC
