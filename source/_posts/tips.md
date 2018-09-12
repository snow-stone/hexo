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
