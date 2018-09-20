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
这俩不互为反函数，如果要用这种方式完成binary和ascii的转换，一定double check BC，因为BC可能被改写

一句话：reconstructPar 一定要看好BC

## reconstructed case or decomposed case
paraview中有这个选项，然而，经历reconstructPar或者decomposePar后BC可能被篡改，所以单纯看边界的话有可能会被吓倒在地，怎么可能！同一个时间步decompose情况下用paraview看BC上的值好好的，结果reconstructPar之后paraview一看……面目全非！这种差异在numberOfSubdomains更容易出现(例如120)，而设置成4就有可能没问题，可能可以算作bug

一句话：当你看到reconstruct之后的BC不对的时候，即使processor*已经被清空了，此时也不用慌，再一次decomposePar帮你恢复BC（至于numberOfSubdomains设置为多少，最好跟并行算的时候一样吧）。即使又出了什么幺蛾子BC的值不能恢复，至少internalField的值不会被动到

上图，这个就是我并行120个核算出来之后想都没想就reconstructPar的效果，一脸懵
![reconstructPar之后](paraRun.png)
但把原数据也就是放在processor*里面的数据用paraview来看的时候就变成了，用`numberOfSubdomains=120`再重新decomposePar也是一样的效果
![decomposedCase](serialRun.png)
