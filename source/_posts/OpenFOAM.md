---
title: OpenFOAM
date: 2019-01-31 10:02:19
tags:
---

# IO
IO的基本特征是硬盘读写，耗时间。

## 后处理里面的IO
目前我见过的OF版本里面后处理的编程范式，仍旧是读一个时间步进行操作再读下一个时间步，这样一段IO之后一点操作操作完了输出一个时间步的结果，然后下一个时间步再一段IO......可能会效率很低：也许来得不如先集中读入所有数据（内存足够大），然后再处理后集中输出。不过要能够这样处理的前提除去内存，对OpenFOAM里field类读写操作的类需要在时间维度上变成动态数组。

我的认识：读入field的时候确实是动态的数组，但在时间维度不是。

## Serial or parallel ?

### 早期OpenFOAM
进行并行计算时要`decomposePar`将计算域分为多个部分，网格文件实际上也分开了。在写入的时候也是写入到每个`processor`文件中，这是早期openfoam版本的做法，弊端是产生的文件量特别大。有些机器上面，比如occigen就有文件个数的限制，只允许200000个文件。如果500个processor，一个processor里面最多只能400个文件，而一个时间步里面至少有`U, p, phi`，所以这个限制还挺严格。

如何解决？要么定期`reconstructPar`然后把那些processor里面对应的`U, p, phi`删掉，需要注意的有两点：
1.第一个操作完成，第二步删除再进行 
2.不要对openfoam正在读写的文件进行操作
第2点比较好规避，选择所有时间步，避开倒数2个，剩下的一般都写好了（因为一般输出数据间隔都不会很小）；而第1点，就需要保证第一步的返回值为0，经由判断后再进行相应删除

### 较新的OpenFOAM

[openfoam-plus_1712和openfoam5](https://www.openfoam.com/releases/openfoam-v1712/parallel.php)提供了一种直接输出一个整体流场`U, p, phi`的方式。(吐槽一下：这应该是一个并行CFD的基本功能才对)

经尝试想要在早期版本中编译并行输出的模块很复杂，弃。

## 数据格式
**binary or ascii**   
模拟的数据输出可以选择，各有优劣：
- binary数据存储体积更小，推荐在simu中间使用，而且在paraview读取数据的时候（20M算例，两个时间步）至少比ascii要快7倍
- 在初始场中推荐使用ascii，因为万一要改边界条件binary格式不是那么好下手(vimdiff或者meld都对ascii支持更好)
- 再者openfoam支持一个simu里面两种格式文件共存，utility和solver在读数据的时候都是以文件头上`format`相应格式读取

binary或者ascii，如何在二者中转换？  
复杂解且不完全：比如从binary转为ascii，可以用`decomposePar`然后`reconstructPar`，第二步的时候把输出格式改为ascii，`internalField`的读写应该没问题，但是BC呢还是得check一下
简单解且官方：foamFormatConvert，用system/controlDict里面的格式来写输出（不过这会overwrite原数据）,默认会将constant/polyMesh里面的数据也按照格式重写

## 相关的类
`UListIO.C`里有底层的`operator<<`，判断了是否输出的是ascii后判断是不是`uniform`的UList，如果是uniform就用`{}`，如果不是就用`()`，这就是为什么OpenFOAM里面只要输出List就一定带有大括号或者小括号.

## 典型读写范例

### bug
据测试...读的时候文件可以用绝对路径来且可以较长，但写的时候得注意绝对路径不一定奏效（即使确定了路径存在），为啥要说这一点，因为“不奏效”指无warning且不报错（如果debug不开的话）但就是找不到输出文件，写的时候用`constant`作为输出路径比较保险（当下目录`.`也需质疑）

### 读写范例
通过IOobject读入写场，注意，OpenFOAM里面场是`*Field`而`Field`是一个`List`，所以其实是`List`读写(上面写到还跟UList有关)

#### 读

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

#### 写
此例用runTime而没用用mesh作为objectRegistry，但需要`system/controlDict`

```cpp
// IOobjectWriter.C
// List<label>

#include "Time.H"
#include "argList.H"

#include "IOList.H"    
#include "OFstream.H"

using namespace Foam;

// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

int main( int argc, char *argv[])
{
    #include "setRootCase.H"
    #include "createTime.H"

    IOList<scalar> IOScalarList 
    (
        IOobject
        (
            "IOSList",
            "constant",
            runTime,
            IOobject::NO_READ,
            IOobject::AUTO_WRITE
        )
    );

	List<scalar> sList(8);
    sList[0] = 7.0;
    sList[1] = 9.0;
    sList[2] = 1.0;
    sList[3] = 2.1;
    sList[4] = 4.0;
    sList[5] = 7.0;
    sList[6] = 4.0;
    sList[7] = 0.0;
	IOList<scalar> IOScalarList1
	(
		IOobject
	   	(
			"IOSList1",
		    "constant",
			runTime,
            IOobject::NO_READ,
            IOobject::AUTO_WRITE
		),
		sList
	);

    Info<< "Writing " << IOScalarList.name() << " to " << IOScalarList.objectPath() << endl;

    OFstream os(IOScalarList.objectPath());   // 如果要用writeHeader，需要OFstream
    OFstream os1(IOScalarList1.objectPath());

	//writeHeader only
	IOScalarList.writeHeader(os);
	IOScalarList1.writeHeader(os1);

	//output as list                          // 没有header，就是"( )" 样式的输出
	//os << IOScalarList;
	//os1 << IOScalarList1;

	//output and header                       // 含有header，可以通过OpenFOAM再次读入，如上例volVectorField读入的时候文件必须含有header
	//IOScalarList.write();
	//IOScalarList1.write();
	
    Info<< "\nEnd\n" << endl;
}

// ************************************************************************* //


// Make/files
IOobjectWriter.C

EXE = $(FOAM_USER_APPBIN)/IOobjectWriter

// Make/options
EXE_INC = \
    -I$(LIB_SRC)/OpenFOAM/lnInclude

EXE_LIBS = \
    -lOpenFOAM

```
这里没有用到`fvCFD.H`这个容量超级大的`*.H`文件，也没有用到`class fvMesh`，仅仅是`List`的读入，算是个minimum code.但如果要加入读`volScalarField`呢？下面是上面代码加入`volScalarField`的变种，但如果想要保持尽量少的头文件并不容易.下面这段代码编译错误.
```bash
$ wmake
SOURCE=userProbeByLabel.C ;  OMPI_CXX="g++" mpicxx -Dlinux64 -DWM_DP -Wall -Wextra -Wno-unused-parameter -Wold-style-cast -Wnon-virtual-dtor -O2 -march=native -fuse-ld=bfd  -DNoRepository -ftemplate-depth-100 -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/finiteVolume/lnInclude -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/meshTools/lnInclude -IlnInclude -I. -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OSspecific/POSIX/lnInclude   -fPIC -c $SOURCE -o Make/linux64GccDPOpt/userProbeByLabel.o
userProbeByLabel.C: In function ‘int main(int, char**)’:
userProbeByLabel.C:59:7: error: variable ‘Foam::volScalarField mean’ has initializer but incomplete type
       scalarFieldName+"_mean",
       ^
userProbeByLabel.C:76:14: error: variable ‘Foam::volScalarField scalarField’ has initializer but incomplete type
              scalarFieldName,
              ^
userProbeByLabel.C:84:18: error: variable ‘Foam::volScalarField sPrime’ has initializer but incomplete type
   volScalarField sPrime = scalarField - mean;
                  ^
make: *** [Make/linux64GccDPOpt/userProbeByLabel.o] Error 1
```
说明`volScalarField`并没有正确地被初始化，查`userProbeByLabel.dep`发现没有对应的模板类`GeometricField`的身影，但加入`#include "GeometricField.H"`的最终结果也是莫名其妙一堆错.当然一个肯定可行的解法是头文件加上`fvCFD.H`，基于测试我发现`#include "fvc.H"`或者`#include "fvm.H"`均可通过编译，而且能够正常运行.代码如下，关键行`#include "fvc.H"`被注释.

```cpp
#include "Time.H"
#include "argList.H"
#include "fvMesh.H"
#include "timeSelector.H"

//#include "fvc.H"  关键行

#include <fstream>

using namespace Foam;

// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

int main( int argc, char *argv[])
{
    timeSelector::addOptions();
	argList::validArgs.append("scalarFieldName");
    argList::validArgs.append("timeOfAverageField");
    argList::validArgs.append("position");

    #include "setRootCase.H"
    #include "createTime.H"

	word scalarFieldName(args.additionalArgs()[0]);
	word timeOfAverageField(args.additionalArgs()[1]);
	word position(args.additionalArgs()[2]);

	instantList timeDirs = timeSelector::select0(runTime, args);

    IOList<label> labelGroup
    (
        IOobject
        (
            "labelGroup",
            position,
            runTime,
            IOobject::MUST_READ,
            IOobject::NO_WRITE
        )
    );

	//output and header
	//labelGroup.write();
	Info<< labelGroup << endl;

	mkDir("userDefinedLog");
	std::ofstream fluctuationLog
	(
	   	fileName(string("userDefinedLog")/string("fluctuation_labelGroup_"+scalarFieldName)).c_str(),
		ios_base::app
	);

    #include "createMesh.H"

	volScalarField mean
	(
	    IOobject
	    (
		    scalarFieldName+"_mean",  // 难道是 IOobject naming ???
		    timeOfAverageField,
		    mesh,
		    IOobject::MUST_READ
	    ),
		mesh
	);

	forAll(timeDirs, timeI)
	{
		runTime.setTime(timeDirs[timeI], timeI); 
	    Info<< "Time = " << runTime.timeName() << endl;

		volScalarField scalarField
		(
        	IOobject
        	(
            	scalarFieldName,
           	    runTime.timeName(),
           		mesh,
            	IOobject::MUST_READ
        	),
			mesh
		);

		volScalarField sPrime = scalarField - mean;

        fluctuationLog << runTime.timeName() << " ";
        forAll(labelGroup, i)
        {
            //fluctuationLog << labelGroup[i] << " " ;
            fluctuationLog << sPrime.internalField()[labelGroup[i]] << " " ;
        }
        fluctuationLog << std::endl;
	}
	
    Info<< "\nEnd\n" << endl;
}

// ************************************************************************* //

// Make/files
userProbeByLabel.C

EXE = $(FOAM_USER_APPBIN)/userProbeByLabel

// Make/options
EXE_INC = \
    -I$(LIB_SRC)/OpenFOAM/lnInclude \
    -I$(LIB_SRC)/finiteVolume/lnInclude \
    -I$(LIB_SRC)/meshTools/lnInclude

EXE_LIBS = \
    -lOpenFOAM \
    -lfiniteVolume \
    -lmeshTools
```

#### 计算和写（标准）

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

#### 计算和写（避开量纲）

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

nu.internalField()=userCalcNu(strainRate); // 如果用strainRate.internalField()会有什么区别？编译倒是可以通过

// 输出

nu.write();

```

#### 计算一个跟壁面相关的场（也就是仅在边界上才有值）并输出
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

#### 初始化一个单位张量场

```cpp

volTensorField identity
(
   IOobject
    (
        "identity",
        runTime.timeName(),
        mesh,
        IOobject::NO_READ,
        IOobject::AUTO_WRITE
    ),
    mesh,
    dimensionedTensor // dimensioned<tensor>
    (
        "identity",
        dimless,
        tensor::I
    )
);

```

# Boundary Condition

## fixedValueFvPatchVectorField
Dirichlet边界(可以随时间变化)，通常继承自fixedValueFvPatchVectorField.编写边界条件的时候要注意：

1. data member 申明(`*.H`)和初始化(`*.C`)的顺序要一致
2. 一定要要让所有constructor里面data member都完成初始化(`*.C`)
3. `*.C`里面的`virtual void write(Ostream&) const`务必要包含所有必要的data member，因为在`decomposePar`时写入`processor*`里边界条件用的就是这个`write`
4. `*.C`里面的`makePatchTypeField`似乎是个宏函数，一定要修改成`makePatchTypeField(fvPatchVectorField, SameAsClassName);`
5. 可以写一些non-member function用来做简单的计算

**窍门**   
所在边界的名字：`patch().name()`   
当前时间步的时间参数：`this->db().time().timeOutputValue()`.在这个类`runTime`不可见

# Tips

## 常用的object

**timeSelector** : 通常以 -time 起，后接 `':500,1200:1300,3000:'`

**args** : argList类 —— argument list. 构造函数对应 `#include "setRootCase.H"`

**runTime** : 可以用write()方法，将所有注册的object都写出来（如果不是NO_WRITE），但一般的流场也可以自己用write()，例如`U.write()`. 构造函数对应 `#include "createTime.H"`

**mesh** : fvMesh类，可以用来遍历：`forAll(mesh.C(), celli)`，也可以通过`mesh.V()`来找网格体积

## 几个函数

OpenFOAM里面文件头怎么来的？ : writeHeader

## 几个坑

**scalarExplicitSetValue** : 源`fvOptions/constraints/general/explicitSetValue`，居然连`field`不设置都可以不报错..

**fixedTemperatureConstraint** : 是针对能量方程的constraint，即compressible solvers

## 一些继承关系疏理

**singlePhaseTransportModel** ： nonNewtonianIcoFoam里面通过singlePhaseTransportModel来初始化流变模型，与`viscosityModel`没有继承关系，因此`singlePhaseTransportModel`不能用Foam::tmp<Foam::volScalarField> Foam::viscosityModel::strainRate()这个方法。然而`BirdCarreau`是`viscosityModel`的子类，它是如何被`singlePhaseTransportModel`包装起来的呢？

**Pstream** : 继承自`UPstream`(内有方法：parRun(), master(), procNo()...)，不过通常用的时候这样用`Pstream::master()`，用于比如输出到文件而不想因为mpi多次重复输出

## 常用的功能
### max/min
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

### findPatchID
通过字符串来找到一个patchList里面对应的patchID
```cpp
label patchID = mesh.boundaryMesh().findPatchID("walls"); 
```

### boundary value还是离网格最近的cell value?
`U.boundaryField()[patchi]`这个是boundary value.`U.patchInternalField()`这是离boundary最近的cell些的U的cell value

### faceCells
cell到face是很自然的，因为：cell是什么？多个face闭合起来成为cell。那怎么反过来找呢？
```cpp
const fvPatchList& patches = mesh.boundary();

patches[somePatch].faceCells[facei]; // 返回值就是patches[somePatch][facei]对应的唯一cell的cellID

```

## 相互引用的数据
```cpp
// 通过U找mesh()
const fvMesh & mesh  =  U.mesh();

// 通过runTime(Foam::Time is an objectRegistry)找db()，通过db()找phi
const surfaceScalarField& phi = runTime.db().lookupObject<surfaceScalarField>("phi"); // 能通过编译，但runTime能不能从objectRegistry找到是另一回事

// 在边界条件里面(fvPatchField)找db()
this->db()
// 再通过db()找计算time的值
const scalar t = this->db().time().timeOutputValue()

```
