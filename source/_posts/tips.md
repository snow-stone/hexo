---
title: tips
date: 2018-09-12 19:08:49
tags:
---

# 常用的object

**timeSelector** : 通常以 -time 起，后接 `':500,1200:1300,3000:'`

**args** : argList类 —— argument list. 构造函数对应 `#include "setRootCase.H"`

**runTime** : 可以用write()方法，将所有注册的object都写出来（如果不是NO_WRITE），但一般的流场也可以自己用write()，例如`U.write()`. 构造函数对应 `#include "createTime.H"`

**mesh** : fvMesh类，可以用来遍历：`forAll(mesh.C(), celli)`，也可以通过`mesh.V()`来找网格体积

# 几个函数

OpenFOAM里面文件头怎么来的？ : writeHeader

# 几个坑

**scalarExplicitSetValue** : 源`fvOptions/constraints/general/explicitSetValue`，居然连`field`不设置都可以不报错..

**fixedTemperatureConstraint** : 是针对能量方程的constraint，即compressible solvers

# 一些继承关系疏理

**singlePhaseTransportModel** ： nonNewtonianIcoFoam里面通过singlePhaseTransportModel来初始化流变模型，与`viscosityModel`没有继承关系，因此`singlePhaseTransportModel`不能用Foam::tmp<Foam::volScalarField> Foam::viscosityModel::strainRate()这个方法。然而`BirdCarreau`是`viscosityModel`的子类，它是如何被`singlePhaseTransportModel`包装起来的呢？

**Pstream** : 继承自`UPstream`(内有方法：parRun(), master(), procNo()...)，不过通常用的时候这样用`Pstream::master()`，用于比如输出到文件而不想因为mpi多次重复输出

# 常用的功能
## max/min
场(GeometricField，一般是某volScalarField)的最大值，最小值:（虽然我不确认并行的时候对不对）
```cpp
max(someField); //这个给出的是internalField()和boundaryField()中数值上最大的值
max(someField.internalField()); // 这是网格内的体积元volume中的最大值
max(someField.boundaryField()); // 这个你会阴沟里翻船，因为这里相当于max(allBoundaryGeometricFields)，意味着这首先是“边界_i/patch_i”的list，虽然我不知道findMax会用什么方法来比较这几个list的大小，但从输出的index可以判断对应的就是边界的指标

//因此要求得boundaryField()上的最大值应当循环
forAll(mesh.boundaryMesh(), patchI)
{
	max(someField.boundaryField()[patchI]) // 这里才是对某一个边界上的所有面积元face上的值构成的list求最大值
}
//并且在求得每个边界上最大值后再从从中求最大值
```
## findMax/findMin
这个紧接前面，要注意的是findMax返回的index是所操作数组的index而非全局index
面积元的[全局index](https://www.cfd-online.com/Forums/openfoam-programming-development/129723-how-get-face-ids-boundary.html)在哪里呢？
```cpp
//  通过start()来找到全局faces中，所在patch第一个face的全局index
forAll(mesh.boundaryMesh().names(), patchName) 
{
    polyPatch cPatch = mesh.boundaryMesh()[patchName]; //虽然觉得这样有点啰嗦，为啥要用name嘛
    forAll(cPatch, faceI) 
    {
        label faceId = cPatch.start() + faceI;
	    Info << faceId << " -> "  << endl;
	    Info << "  cPatch[faceI]        = " << cPatch[faceI] << endl;   // 此处输出的就是face本身，face是什么？四个点的label就确定一个面
	    Info << "  mesh.faces()[faceId] = " << mesh.faces()[faceId] << endl; // 全局输出看看是不是一样，mesh.faces()就是存所有faces的地方
    }
}

//输出
/*
...
1633 -> 
  cPatch[faceI]        = 4(733 734 755 754)
  mesh.faces()[faceId] = 4(733 734 755 754)
1634 -> 
  cPatch[faceI]        = 4(754 755 776 775)
  mesh.faces()[faceId] = 4(754 755 776 775)
...
*/
```
