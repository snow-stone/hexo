---
title: OpenFOAM积累学习0
date: 2019-01-31 10:02:19
tags: OpenFOAM
---

# IO
IO的基本特征是硬盘读写，耗时间。

## 后处理里面的IO
目前我见过的OF版本里面后处理的编程范式，仍旧是读一个时间步进行操作再读下一个时间步，这样一段IO之后一点操作操作完了输出一个时间步的结果，然后下一个时间步再一段IO......可能会效率很低：也许来得不如先集中读入所有数据（内存足够大），然后再处理后集中输出。不过要能够这样处理的前提除去内存，对OpenFOAM里field类读写操作的类需要在时间维度上变成动态数组。

我的认识：读入field的时候确实是动态的数组，但在时间维度不是。

## Serial or parallel ?

### 早期OpenFOAM
进行并行计算时要`decomposePar`将计算域分为多个部分，网格文件实际上也分开了。在写入的时候也是写入到每个`processor`文件中，这是早期OpenFOAM版本的做法，弊端是产生的文件量特别大。有些机器上面，比如occigen就有文件个数的限制，只允许200000个文件。如果500个processor，一个processor里面最多只能400个文件，而一个时间步里面至少有`U, p, phi`，所以这个限制还挺严格。

如何解决？要么定期`reconstructPar`然后把那些processor里面对应的`U, p, phi`删掉，需要注意的有两点：
1.第一个操作完成，第二步删除再进行 
2.不要对OpenFOAM正在读写的文件进行操作
第2点比较好规避，选择所有时间步，避开倒数2个，剩下的一般都写好了（因为一般输出数据间隔都不会很小）；而第1点，就需要保证第一步的返回值为0，经由判断后再进行相应删除

### 较新的OpenFOAM

[openfoam-plus_1712和openfoam5](https://www.openfoam.com/releases/openfoam-v1712/parallel.php)提供了一种直接输出一个整体流场`U, p, phi`的方式。(吐槽一下：这应该是一个并行CFD的基本功能才对)

经尝试想要在早期版本中编译并行输出的模块很复杂，弃。

## 数据格式
**binary or ascii**   
模拟的数据输出可以选择，各有优劣：
- binary数据存储体积更小，推荐在simu中间使用，而且在paraview读取数据的时候（20M算例，两个时间步）至少比ascii要快7倍
- 在初始场中推荐使用ascii，因为万一要改边界条件binary格式不是那么好下手(vimdiff或者meld都对ascii支持更好)
- 再者OpenFOAM支持一个simu里面两种格式文件共存，utility和solver在读数据的时候都是以文件头上`format`相应格式读取

binary或者ascii，如何在二者中转换？  
复杂解且不完全：比如从binary转为ascii，可以用`decomposePar`然后`reconstructPar`，第二步的时候把输出格式改为ascii，`internalField`的读写应该没问题，但是BC呢还是得check一下
简单解且官方：foamFormatConvert，用system/controlDict里面的格式来写输出（不过这会overwrite原数据）,默认会将constant/polyMesh里面的数据也按照格式重写

`foamFormatConvert`也不是一直都行得通：ascii格式的文件是OF-2.3.x/OF-3.0.1兼容的，但binary就不是，遇到以下报错需考虑版本问题(另外一点窍门是转换的时候如果方便，把`constant`和`system`移动到与时间步分离的目录下运行`foamFormatConvert -constant`，这样可以避免时间步的数据被影响)
```bash
--> FOAM FATAL IO ERROR: 
Expected a ')' while reading binaryBlock, found on line 20 an error

file: synthetic_phasedStepFrom_test_from_0/From0p3_3_of3/constant/polyMesh/faces at line 20.

    From function Istream::readEnd(const char*)
    in file db/IOstreams/IOstreams/Istream.C at line 111.

FOAM exiting
```

## 相关的类
`UListIO.C`里有底层的`operator<<`，判断了是否输出的是ascii后判断是不是`uniform`的UList，如果是uniform就用`{}`，如果不是就用`()`，这就是为什么OpenFOAM里面只要输出List就一定带有大括号或者小括号.

## 典型读写范例

### bug
据测试...读的时候文件可以用绝对路径来且可以较长，但写的时候得注意绝对路径不一定奏效（即使确定了路径存在），为啥要说这一点，因为“不奏效”指无warning且不报错（如果debug不开的话）但就是找不到输出文件，写的时候用`constant`作为输出路径比较保险（当下目录`.`也需质疑）

### 读写范例
通过IOobject读入写场，注意，OpenFOAM里面场是`*Field`而`Field`是一个`List`，所以其实是`List`读写(上面写到还跟UList有关)

#### 读

```cpp
// (始) 读一个名为velocityFieldName的向量场
volVectorField velocityField
(
    IOobject
    (
        velocityFieldName,
        runTime.timeName(),   // 所在目录
        mesh,
        IOobject::MUST_READ
    ),
    mesh
);

// (结) 后面引用velocityField即可，注意这里只读了runtime.timeName()里面的，文件名为velocityFieldName这个文件

// （始）读一个名为"labelGroup"的labelList 
IOList<label> labelGroup
(
    IOobject
    (
        "labelGroup",
        position,             // 所在目录
        runTime,
        IOobject::MUST_READ,
        IOobject::NO_WRITE
    )
);

// 附上"labelGroup"文件的样本格式
FoamFile
{
    version     2.0;
    format      ascii;
    class       labelList;
    location    "constant";
    object      labelGroup;
}

21
(
26524615
25273885
24395085
23786685
23330385
22975485
22671285
22417785
22198085
21995285
21826285
21657285
21657285
21031985
20355985
19865885
19494085
19189885
18919485
18699785
18496984
)

// (结) 读一个名为"labelGroup"的labelList
```

**Note** : `topoSet`好像可以做`labelList`的操作，它的功能还很多！包括`{label/zone/face/point/box/rotatedBox/cylinder/sphere/...}toCell`

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

# 编写边界条件

## fixedValueFvPatchVectorField
Dirichlet边界(可以随时间变化)，通常继承自`fixedValueFvPatchVectorField`，编写边界条件的时候要注意：

1. data member 申明(`*.H`)和初始化(`*.C`)的顺序要一致
2. 一定要要让所有constructor里面data member都完成初始化(`*.C`)
3. `*.C`里面的`virtual void write(Ostream&) const`务必要包含所有必要的data member，因为在`decomposePar`时写入`processor*`里边界条件用的就是这个`write`
4. `*.C`里面的`makePatchTypeField`似乎是个宏函数，一定要修改成`makePatchTypeField(fvPatchVectorField, SameAsClassName);`
5. 可以写一些non-member function用来做简单的计算

**窍门**   
所在边界的名字：`patch().name()`   
当前时间步的时间参数：`this->db().time().timeOutputValue()`.在这个类`runTime`不可见

# 常用Tips

## object

**timeSelector** : 通常以 -time 起，后接 `':500,1200:1300,3000:'`

**args** : argList类 —— argument list. 构造函数对应 `#include "setRootCase.H"`

**runTime** : 可以用write()方法，将所有注册的object都写出来（如果不是NO_WRITE），但一般的流场也可以自己用write()，例如`U.write()`. 构造函数对应 `#include "createTime.H"`

**mesh** : fvMesh类，可以用来遍历：`forAll(mesh.C(), celli)`，也可以通过`mesh.V()`来找网格体积

## 函数

### 书写文件头

OpenFOAM里面文件头怎么来的？ : writeHeader

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

### findMax/findMin

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

## 类

### VectorSpace, Vector, Tensor: mag

VectorSpace 显然是这里最具有一般性最底层的模板类了，Vector和Tensor在它眼里仅是3个元素和9个元素的差别，求`mag`都一视同仁。

`mag`在VectorSpace/Vector里面就是所有元素平方和`magSqr`然后开平方，在Tensor变成9个元素平方和然后开平方。因此一般的Tensor不同重写`magSqr`，SymmTensor因为其形式特别就重写了`magSqr`，如此替换掉最一般的`magSqr`,对SymmTensor仍然可以进行`mag`操作，为什么呢？因为SymmTensor是Tensor是VectorSpace里面的元素，因此一定就有`mag`，此时母类`mag`调用的是SymmTensor版本的`magSqr`。

### 外积

```cpp
#include "vector.H"
#include "tensor.H"
#include "IOstreams.H"

using namespace Foam;

int main()
{
/*
    Info<< vector::zero << endl
        << vector::one << endl
        << vector::dim << endl
        << vector::rank << endl;
*/

    vector a(1, 2, 3);
    Info << "a = " << a << endl;
    Info << "a*a = " << a*a << endl;

    return 0;
}

```

如果将`#include "tensor.H"`注释掉会报一个比较长的错：

```bash
$ wmake
Making dependency list for source file Test-vector.C
SOURCE=Test-vector.C ;  OMPI_CXX="g++" mpicxx -Dlinux64 -DWM_DP -Wall -Wextra -Wno-unused-parameter -Wold-style-cast -Wnon-virtual-dtor -O2 -march=native -fuse-ld=bfd  -DNoRepository -ftemplate-depth-100  -IlnInclude -I. -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OSspecific/POSIX/lnInclude   -fPIC -c $SOURCE -o Make/linux64GccDPOpt/Test-vector.o
OMPI_CXX="g++" mpicxx -Dlinux64 -DWM_DP -Wall -Wextra -Wno-unused-parameter -Wold-style-cast -Wnon-virtual-dtor -O2 -march=native -fuse-ld=bfd  -DNoRepository -ftemplate-depth-100  -IlnInclude -I. -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OSspecific/POSIX/lnInclude   -fPIC -Xlinker --add-needed -Xlinker --no-as-needed Make/linux64GccDPOpt/Test-vector.o -L/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/platforms/linux64GccDPOpt/lib \
      -lOpenFOAM -ldl   -lm -o /home/hluo/OpenFOAM/hluo-2.3.1/platforms/linux64GccDPOpt/bin/Test-vector
[hluo@zaurak userVector]$ vim Test-vector.C 
[hluo@zaurak userVector]$ wmake
Making dependency list for source file Test-vector.C
SOURCE=Test-vector.C ;  OMPI_CXX="g++" mpicxx -Dlinux64 -DWM_DP -Wall -Wextra -Wno-unused-parameter -Wold-style-cast -Wnon-virtual-dtor -O2 -march=native -fuse-ld=bfd  -DNoRepository -ftemplate-depth-100  -IlnInclude -I. -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude -I/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OSspecific/POSIX/lnInclude   -fPIC -c $SOURCE -o Make/linux64GccDPOpt/Test-vector.o
Test-vector.C: In function ‘int main()’:
Test-vector.C:18:23: error: no match for ‘operator*’ (operand types are ‘Foam::vector {aka Foam::Vector<double>}’ and ‘Foam::vector {aka Foam::Vector<double>}’)
  Info << "a*a = " << a*a << endl;
                       ^
Test-vector.C:18:23: note: candidates are:
In file included from /home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/VectorSpace.H:168:0,
                 from /home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/Vector.H:44,
                 from /home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/vector.H:39,
                 from Test-vector.C:1:
/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/VectorSpaceI.H:552:13: note: template<class Form, class Cmpt, int nCmpt> Form Foam::operator*(Foam::scalar, const Foam::VectorSpace<Form, Cmpt, nCmpt>&)
 inline Form operator*
             ^
/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/VectorSpaceI.H:552:13: note:   template argument deduction/substitution failed:
Test-vector.C:18:24: note:   cannot convert ‘a’ (type ‘Foam::vector {aka Foam::Vector<double>}’) to type ‘Foam::scalar {aka double}’
  Info << "a*a = " << a*a << endl;
                        ^
In file included from /home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/VectorSpace.H:168:0,
                 from /home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/Vector.H:44,
                 from /home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/vector.H:39,
                 from Test-vector.C:1:
/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/VectorSpaceI.H:565:13: note: template<class Form, class Cmpt, int nCmpt> Form Foam::operator*(const Foam::VectorSpace<Form, Cmpt, nCmpt>&, Foam::scalar)
 inline Form operator*
             ^
/home/hluo/.local/easybuild/software/OpenFOAM/2.3.1-foss-2016a/OpenFOAM-2.3.1/src/OpenFOAM/lnInclude/VectorSpaceI.H:565:13: note:   template argument deduction/substitution failed:
Test-vector.C:18:24: note:   cannot convert ‘a’ (type ‘Foam::vector {aka Foam::Vector<double>}’) to type ‘Foam::scalar {aka double}’
  Info << "a*a = " << a*a << endl;
                        ^
make: *** [Make/linux64GccDPOpt/Test-vector.o] Error 1

```

报错的意思是我们找`Foam::Vector<double>`与`Foam::Vector<double>`之间的`operator*`找不到，只能有后面vector前面scalar的`operator*`（分别在VectorSpaceI.H:552和VectorSpaceI.H:565），但因为vector,tensor和VectorSpace是由一种复杂的方式耦合在一起的，所以并没有报一个找不到俩vector的外积。但如果加上"tensor.H"一切都对了，输出结果：

```bash
$ Test-vector 
a = (1 2 3)
a*a = (1 2 3 2 4 6 3 6 9)
```

### List

`List` and `UList` are different. `List a(10)` then `a.append(something)` will append to the tail of the initialized `a` resulting a `a[11]=something` 

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

# 几个坑

**scalarExplicitSetValue** : 源`fvOptions/constraints/general/explicitSetValue`，居然连`field`不设置都可以不报错..

**fixedTemperatureConstraint** : 是针对能量方程的constraint，即compressible solvers

# 一些继承关系疏理

**singlePhaseTransportModel** ： nonNewtonianIcoFoam里面通过singlePhaseTransportModel来初始化流变模型，与`viscosityModel`没有继承关系，因此`singlePhaseTransportModel`不能用`Foam::tmp<Foam::volScalarField> Foam::viscosityModel::strainRate()`这个方法。然而`BirdCarreau`是`viscosityModel`的子类，它是如何被`singlePhaseTransportModel`包装起来的呢？

**Pstream** : 继承自`UPstream`(内有方法：`parRun(), master(), procNo()`...)，不过通常用的时候这样用`Pstream::master()`，用于比如输出到文件而不想因为mpi多次重复输出

# Pirating

## timeDir

如果小心输出时间步全部刚好差了0.01怎么办，例如[5,7]变成了[5.01,7.01]，timeDir里面的`U,p`都有个冠冕堂皇的`location`的值对应时间。但其实可以把目录`5.01`改成`5`，`7.01`改成`7`，这时比如要续算，solver便认为对应的时间步其实是`5`和`7`。似乎sample这些utility也可以这样骗[待确认]。

一句话：外面的目录名是个“货真价实”的幌子，可以欺骗solver和utility

## setFields

setFields用于在internalField里面写一个值。选择区域最常用的估计是`boxToCell`，但定义`box`有格式

```cpp
box (x1 y1 z1) (x2 y2 z2);

// 这里必须有 x1 < x2, y1 < y2, z1 < z2
```

如果不符合上述规范，不会有报错，但不会在internalField里面有写任何的值

这里给出一个符合规范的setFieldsDict

```cpp
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    location    "system";
    object      setFieldsDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

defaultFieldValues
(
    volScalarFieldValue T 0
);

regions
(
    boxToCell
    {
        box (-0.004 -0.08 -0.004) (0.004 -0.004 0.004);
        fieldValues
        (
            volScalarFieldValue T 1
        );
    }
);
```

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

如上，如果想要将多个lib混用在一个case里面，在`system/controlDict`里面都加入lib是必须的，但其实这样不行，做不到混用。如何做到呢？需要修改类的名字，并与makePatchTypeField(fvPatchVectorField, pVFvPatchVectorField2Dpf_Port1)这个macro function对应，这样不仅lib名字不同其实里面的类也不同，这样就可以混用了。


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

