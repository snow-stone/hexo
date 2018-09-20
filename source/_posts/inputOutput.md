---
title: 输入与输出
date: 2018-09-11 17:57:23
tags:
---

# 并行
进行并行计算时要`decomposePar`将计算域分为多个部分，网格文件实际上也分开了。在写入的时候也是写入到每个`processor`文件中，这是早期openfoam版本的做法，弊端是产生的文件量特别大。有些机器上面，比如occigen就有文件个数的限制，只允许200000个文件。如果500个processor，一个processor里面最多只能400个文件，而一个时间步里面至少有`U, p, phi`，所以这个限制还挺严格。

如何解决？要么定期`reconstructPar`然后把那些processor里面对应的`U, p, phi`删掉，需要注意的有两点：
1.第一个操作完成，第二步删除再进行 
2.不要对openfoam正在读写的文件进行操作
第2点比较好规避，选择所有时间步，避开倒数2个，剩下的一般都写好了（一般输出数据间隔都不会很小）；而第1点，就需要保证第一步的返回值为0，经由判断后再进行相应删除

另外[openfoam-plus_1712和openfoam5](https://www.openfoam.com/releases/openfoam-v1712/parallel.php)提供了一种直接输出一个整体流场`U, p, phi`的方式。但具体能否在较早版本中兼容使用待观察

# binary or ascii
模拟的数据输出可以选择，各有优劣：binary数据存储体积更小，推荐在simu中间使用；在初始场中推荐使用ascii，因为万一要改边界条件binary格式不是那么好下手(vimdiff或者meld都对ascii支持更好)，再者openfoam支持一个simu里面两种格式的储存，utility在读数据的时候包括solver在读数据的时候都是要读头文件以相应格式读取。

binary或者ascii，如何在二者中转换？  
复杂解且不完全：比如从binary转为ascii，可以用`decomposePar`然后`reconstructPar`，第二步的时候把输出格式改为ascii，`internalField`的读写应该没问题，但是BC呢还是得check一下
简单解且官方：foamFormatConvert，用system/controlDict里面的格式来写输出（不过这会overwrite原数据）
