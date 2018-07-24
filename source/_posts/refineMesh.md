---
title: refineMesh局部加密
date: 2018-07-24 17:54:43
tags:
---

# 算例流程
1. 对于网格整体或者部分(topoSet)进行加密
2. 写入新网格
3. 新网格下的流场初始化(mapFields)
这里重点讲讲1和2两点

# 使用topoSet确定cellSet
可以用`cylinderToCell`或者其他的`source`，输入为几何外形，输出为cellSet，最好将对应的topoSetDict的名字改为和输出set名字相同
```bash
topoSetDict=system/topoSetDict
setName=branch_Port2_5D_1

topoSet -dict $topoSetDict/$setName > log.topoSet # 写入到constant/polyMesh/sets/{branch_Port2_5D_1  nonOrthoFaces  refinedCells}

foamToVTK -cellSet $setName -latestTime           # 需有时间目录
```

![](topoSet_T.png)
图里彩色部分就是topoSet选定的区域，透明部分是网格整体
