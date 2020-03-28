---
title: 局部网格加密
date: 2018-07-12 20:59:41
tags: OpenFOAM
---

基于tutorial里面cavity完全正交的网格(拓展为3D)，通过`topoSet`选定一个`cellSet`，用refineMesh进行局部加密。icoFoam求解器识别并接受这样局部加密的存在但对局部加密后的网格使用`checkMesh`会“检查出”polyhedra，而且non-orthogonality Max也会变差。

# 加密网格checkList

```bash

topoSet > log.topoSet # boxToCell

checkMesh > log.checkOrgMesh

refineMesh -dict system/refineMeshDict -overwrite > log.refineMesh 
# 不加overwrite网格会写在时间步目录里
# refineMesh 默认是一分为二，三个方向一分为二就是八倍网格数

checkMesh > log.checkMesh
# 不再具有网格诊断价值

```

## polyhedra? Non-orthogonal?
因为level0的大网格被分割成了level1的小面，所以曾经level0的六面体，如果有一个面被分割现在就是9面体，如果有两个面被分割现在就是12面体，三个面那就是15面体。从形状上判定仍然还是纯正交网格，但从“面的个数”上就不是。因为面只能被两个网格共享。   
局部加密造成了拓扑的改变，从下面的结果看来checkMesh给出的non-orthogonality不再正确。不正确？因为网格如果仍旧正交应该是`Mesh non-orthogonality Max: 0 average: 0`。

```bash
$ diff log.checkOrgMesh log.checkMesh
<     Minimum face area = 2.5e-05. Maximum face area = 2.5e-05.  Face area magnitudes OK.
<     Min volume = 1.25e-07. Max volume = 1.25e-07.  Total volume = 0.00025.  Cell volumes OK.
<     Mesh non-orthogonality Max: 0 average: 0
---
>     Minimum face area = 6.25e-06. Maximum face area = 2.5e-05.  Face area magnitudes OK.
>     Min volume = 1.5625e-08. Max volume = 1.25e-07.  Total volume = 0.00025.  Cell volumes OK.
>     Mesh non-orthogonality Max: 25.2394 average: 4.00976
75c79
<     Max skewness = 2e-08 OK.
---
>     Max skewness = 0.333333 OK.
```
有试过在复杂网格的局部进行三个方向的加密，`non-orthogonality Max`增加了同样神奇的数值25，至于是不是从来都是25，为什么是25我就不知道了。

## 真的还是正交网格吗？
按照cfd-forum上面所说：是的！还有人讨论关于怎么将网格“正确地”可视化地问题，总结如下：paraview可以直接读OpenFOAM格式的网格，但需要把`Decompose Polyhedra`选项给关掉;`foamToVTK`加上option`-poly`后再用paraview来读。   

前者：`Decompose Polyhedra`需要在读入`*.foam`后（点apply前），点properties里面搜索栏右边小齿轮才会在下面出现，默认值是勾取，这里需要“去勾”。上图
![paraview](check_DecomposePolyhedra.png)
这里是默认值，看到有奇怪的对角线，似乎网格不再正交
![paraview uncheck Decompose Polyhedra后](unCheck_DecomposePolyhedra.png)
去勾后恢复本来面貌

后者：`foamToVTK`会生成体数据和面数据，读入体数据。
![不加-poly](foamToVTK_front.png)
不加option同样会显示不规则的网格，这里可以注意一下vtk文件里面可以选择用`cellID`染色，有意思。
![加上-poly正面](foamToVTK_op-poly_front.png)
加上option后的正面
![加上-poly背面](foamToVTK_op-poly_back.png)
加上option后的背面
![加上-poly另一个角度看侧面](foamToVTK_op-poly_anotherAngle.png)
加上option后，再从另外一个角度来看，能大概看出个`cellID`排序的规律。当然这样“视觉”的规律没有用，算例复杂之后就很难有视觉规律了，毕竟程序是按照“逻辑”规律工作。

## 小结
局部加密虽好，但checkMesh对生成的网格不再有准确的诊断能力，而对于OpenFOAM格式的网格直接的诊断工具仅有checkMesh，所以只能考虑转换网格格式用其他软件诊断（不过还不知道能不能正确识别这些局部加密的网格呢！）。另外一方面要注意的是局部加密后网格数变大了，比如之前算过的统计收敛的流场不能直接续算（数组不一样大了嘛），需要`mapFields`。

# 范例流程

从算例的层面来看应该分三步走才算完成了网格加密和初始条件设置:

1. 对于网格整体或者部分(topoSet)进行加密
2. 写入新网格
3. 新网格下的流场初始化(mapFields)

## 使用topoSet确定cellSet

可以用`cylinderToCell`或者其他的`source`，输入为几何外形，输出为cellSet，最好将对应的topoSetDict的名字改为和输出set名字相同

```bash
#!/bin/bash

topoSetDict=system/topoSetDict                    
setName=branch_Port2_5D_1                         # 一个topoSetDict对象，取名branch_Port2_5D_1

topoSet -dict $topoSetDict/$setName > log.topoSet # 写入到constant/polyMesh/sets/{branch_Port2_5D_1  nonOrthoFaces  refinedCells}

foamToVTK -cellSet $setName -latestTime           # 将OpenFOAM数据格式转换为vtk格式，此步骤需有时间目录，生成文件在VTK目录下
```

![](topoSet_T.png)
图里彩色部分就是topoSet选定的区域，透明部分是网格整体

## 网格加密

```bash
#!/bin/bash

checkMesh > log.checkOrgMesh
setToRefine=branch_Port2_5D_1                     # 一个refineMeshDict对象，取名branch_Port2_5D_1

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

在target case里面一定要设置好mapFieldsDict，因为这里我选择不用consistent（虽然按照OpenFOAM说明geoemetry和BC都对应一模一样，按理可以尝试）。由于这个操作串行费内存且费时间而且可能在最后时刻幺蛾子，所以建议是先用网格较少的source和target来尝试，排除bug，然后根据checkList来保障每一步的可靠性。有尝试过在实验室机群上的单节点大内存单核上跑，但因为我本地的工作站cpu还更好，测试的结果是在内存允许的情况下使用工作站来进行这个操作。当然，手头刚拿到[occigen](https://www.top500.org/list/2018/06/?page=1)的计算资源，直接在登陆节点(interactively)对10M网格算例做240个核的decomposePar效率非常高，三分钟就完成，希望mapFields也很快！自答：事实是occigen做一个从1.5M到10M网格的mapFields也要整整1h。验证了那句话“愿望很美好，现实很骨感“。



更一下：mapFields在2.3-x里面有bug，费时，高版本bug被修复了。