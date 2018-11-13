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

## probes
OpenFOAM本身的probes如果刚好碰上网格边上会有warning   
属于functionObjects，在simu的同时运行，如果想要postProcssing，用execFlow...，但是：需要在constrolDict里面加入,自己写的BC会有奇怪报错...   
对策：用wyldckat写的，摘干净的最通用的ExecFunctionObjects，把所有的functionObject都放在system/system/controlDict.functions，这样就可以分开了

## reconstructPar与decomposePar
这俩不互为反函数，如果要用这种方式完成binary和ascii的转换，一定double check BC，因为BC可能被改写

一句话：reconstructPar/decomposePar 一定要看好BC

## reconstructed case or decomposed case
paraview中有这个选项，然而，经历reconstructPar或者decomposePar后BC可能被篡改，所以单纯看边界的话有可能会被吓倒在地，怎么可能！同一个时间步decompose情况下用paraview看BC上的值好好的，结果reconstructPar之后paraview一看……面目全非！这种差异在numberOfSubdomains更容易出现(例如120)，而设置成4就有可能没问题，可能可以算作bug。当然也许是我写出来的bug，毕竟有的边界value要生成随机数，我随机种子又没变。

一句话：当你看到reconstruct之后的BC不对的时候，即使processor*已经被清空了，此时也不用慌，再一次decomposePar帮你恢复BC（至于numberOfSubdomains设置为多少，最好跟并行算的时候一样吧）。即使又出了什么幺蛾子BC的值不能恢复，至少internalField的值不会被动到

上图，这个就是我并行120个核算出来之后想都没想就reconstructPar的效果，一脸懵
![reconstructPar之后](paraRun.png)
但把原数据也就是放在processor*里面的数据用paraview来看的时候就变成了下图，用`numberOfSubdomains=120`再重新decomposePar也是一样的效果
![decomposedCase](serialRun.png)

## 同一个类的不同变种编译为不同名字的lib
如上，如果想要将多个lib混用在一个case里面，在system/controlDict里面都加入lib是必须的，但其实这样不行，做不到混用。如何做到呢？需要修改类的名字，并与makePatchTypeField(fvPatchVectorField, pVFvPatchVectorField2Dpf_Port1)这个macro function对应，这样不仅lib名字不同其实里面的类也不同，这样就可以混用了。


## cyclic
`U,p`设置周期性条件后，在生成的数据里type为`cyclic`你会发现`U,p`都没有value，仅有一个type (`phi`却有，也是`cyclic`类型).那为啥我用`patchIntegrate U inlet`又不是零呢？经过仔细地看源码和比对
```cpp
// 读入U，找到cyclic对应的patchLabel

Info << U.boundaryField()[patchLabel] << endl;
/*
你猜输出什么？
type            cyclic;
仅有type！没有value？不过，读入的就是这样的数据(volVectorField.boundaryField()[cyclic patch])，输出这样的数据也在情理之中
*/

vectorField& Ub_cyclic = U.boundaryField()[patchLabel]
Info << Ub_cyclic << endl;
/*
看输出...就有了，不过vectorField有type，这个似乎也不是特别有道理，毕竟vectorField仅仅是List<vector>
type            cyclic;


16900
(
...
...
...
)
*/

// 参照patchIntegrate.C，里面没有输出整个List而仅仅输出sum；于是我在同样的patch上面求sum输出，计算结果是一样的
Info << sum(U.boundaryField()[patchLabel]) << endl;
Info << sum(Ub_cyclic) << endl;
/*
(11560.70065 80.71560542 3.732255509)
(11560.70065 80.71560542 3.732255509)
*/

// 所以可能是GeometricBoundaryField类对cyclic type输出的问题？？
```
这里得出的结论是：`U`和`p`在数据文件里面，boundaryField项的其中一个patch上有可能仅有type（例如cyclic），但没有value，这不意味着`U.boundaryField()[cyclic patch]`上面没有值。简单地来说：当你看数据文件里面边界上没有值，并不是真的没有值，边界上还可以算通量呢！   
PS : `phi`同为`cyclic`类型，但边界上却有值，且value写在数据文件里
