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
模拟的数据输出可以选择，各有优劣：binary数据存储体积更小，推荐在simu中间使用，而且在paraview读取数据的时候（20M算例，两个时间步）至少比ascii要快7倍；在初始场中推荐使用ascii，因为万一要改边界条件binary格式不是那么好下手(vimdiff或者meld都对ascii支持更好)，再者openfoam支持一个simu里面两种格式的储存，utility在读数据的时候包括solver在读数据的时候都是要读头文件以相应格式读取。

binary或者ascii，如何在二者中转换？  
复杂解且不完全：比如从binary转为ascii，可以用`decomposePar`然后`reconstructPar`，第二步的时候把输出格式改为ascii，`internalField`的读写应该没问题，但是BC呢还是得check一下
简单解且官方：foamFormatConvert，用system/controlDict里面的格式来写输出（不过这会overwrite原数据）,默认会将constant/polyMesh里面的数据也按照格式重写

# 通过IOobject读入写出场

## 读入

```cpp

volVectorField velocityField
(
    IOobject
    (
        velocityFieldName,
        runTime.timeName(),
        mesh,
        IOobject::MUST_READ
    ),
    mesh
);

// 后面引用velocityField即可，注意这里只读了runtime.timeName()里面的，文件名为velocityFieldName这个文件

```

## 计算和输出（标准）

```cpp

volScalarField strainRate
(
   IOobject
    (
        "strainRate_"+velocityFieldName,
        runTime.timeName(),
        mesh,
        IOobject::NO_READ,
        IOobject::NO_WRITE
    ),
    Foam::sqrt(2.0)*mag(symm(fvc::grad(velocityField))) // 这里就继承了量纲
);

```

## 计算和输出（避开量纲）

```cpp

scalarField userCalcNu(scalarField strainRate) // 这里想要偷懒，全部无量纲
{
    scalar nuInf_ = 2e-6;
    scalar nu0_   = 3e-4;
    scalar k_     = 1;
    scalar n_     = 0.326;
    scalar a_     = 2.0;  // defaut BirdCarreau

    return
        nuInf_
      + (nu0_ - nuInf_)
        *pow(scalar(1) + pow(k_*strainRate, a_), (n_ - 1.0)/a_);
}

// 先初始化一个变量nu
volScalarField nu
(
    IOobject
    (
        outputFieldName,
        runTime.timeName(),
        mesh,
        IOobject::NO_READ,
        IOobject::AUTO_WRITE
    ),
	//userCalcNu(strainRate) Not a volScalarField ! 如何由scalarField变成volScalarField??
    mesh,
    dimensionedScalar
    (
        outputFieldName,
        dimless,                               // 续前面偷懒的地方，算个无量纲
        scalar(0.) // 很奇怪如果是vector这里可以用vector::zero，但scalar::zero就不行
    )
);

// 然后再赋值

nu.internalField()=userCalcNu(strainRate);

// 输出

nu.write();

```

## 计算一个跟壁面相关的场（也就是仅在边界上才有值）并输出
可以参照wallShearStress，这样输出的场（yPlus也是类似）可以在paraview可视化，也许需要去掉`internal mesh`的勾选

```cpp

//初始化变量，依旧是volVectorField而不是什么boundaryField

volVectorField wallShearStress
(
    IOobject
    (
        "wallShearStress",
        runTime.timeName(),
        mesh,
        IOobject::NO_READ,
        IOobject::AUTO_WRITE
    ),
    mesh,
    dimensionedVector
    (
        "wallShearStress",
        sqr(dimLength)/sqr(dimTime),
        vector::zero
    )
);

//赋值过程对着wallShearStress.boundaryField()赋值

//输出过程

wallShearStress.write();

```
