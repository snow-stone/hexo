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
