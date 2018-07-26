---
title: refineMesh局部加密
date: 2018-07-24 17:54:43
tags:
---

# 流程
从算例的层面来看应该分三步走才算完成了网格加密和初始条件设置:
1. 对于网格整体或者部分(topoSet)进行加密
2. 写入新网格
3. 新网格下的流场初始化(mapFields)

## 使用topoSet确定cellSet
可以用`cylinderToCell`或者其他的`source`，输入为几何外形，输出为cellSet，最好将对应的topoSetDict的名字改为和输出set名字相同
```bash
#!/bin/bash

topoSetDict=system/topoSetDict                    # 环境变量
setName=branch_Port2_5D_1

topoSet -dict $topoSetDict/$setName > log.topoSet # 写入到constant/polyMesh/sets/{branch_Port2_5D_1  nonOrthoFaces  refinedCells}

foamToVTK -cellSet $setName -latestTime           # 将OpenFOAM数据格式转换为vtk格式，此步骤需有时间目录，生成文件在VTK目录下
```
![](topoSet_T.png)
图里彩色部分就是topoSet选定的区域，透明部分是网格整体

## 网格加密
```bash
#!/bin/bash

checkMesh > log.checkOrgMesh
setToRefine=branch_Port2_5D_1

refineMeshDict=system/refineMeshDict

refineMesh -dict $refineMeshDict/$setToRefine -overwrite > log.refineMesh # this will write "polyMesh/cellMap" to startTime dir

checkMesh > log.checkRefinedMesh
```
如果不加overwrite，新的网格会写入到以startTime为基准的下一个时间目录里（包括cellMap），加了overwrite新的网格就改写constant/polyMesh，这样新的case网格仍然在标准的constant位置

## 初始条件（场）设置
```bash
#!/bin/bash

sourcedir=some/source/case/with/original/mesh/and/field/data/we/are/mapping

mapFields $sourcedir -case . -noFunctionObjects -fields '(U p)' -sourceTime '7' -targetRegion region0 > log.mapFields_7 2>&1
```
在target case里面一定要设置好mapFieldsDict，因为这里我选择不用consistent（虽然按照OpenFOAM说明geoemetry和BC都对应一模一样，按理可以尝试）。由于这个操作串行费内存且费时间而且可能在最后时刻幺蛾子，所以建议是先用网格较少的source和target来尝试，排除bug，然后根据checkList来保障每一步的可靠性。有尝试过在实验室机群上的单节点大内存单核上跑，但因为我本地的工作站cpu还更好，测试的结果是在内存允许的情况下使用工作站来进行这个操作。当然，手头刚拿到[occigen](https://www.top500.org/list/2018/06/?page=1)的计算资源，直接在登陆节点(interactively)对一个一千万网格算例做240个核的decomposePar效率非常高，三分钟就完成，希望mapFields也很快！
