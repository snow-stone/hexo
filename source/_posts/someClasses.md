---
title: 几个类
date: 2018-10-09 22:41:48
tags:
---

# VectorSpace, Vector, Tensor

## mag

VectorSpace 显然是这里最具有一般性最底层的模板类了，Vector和Tensor在它眼里仅是3个元素和9个元素的差别，求`mag`都一视同仁。

`mag`在VectorSpace/Vector里面就是所有元素平方和`magSqr`然后开平方，在Tensor变成9个元素平方和然后开平方。因此一般的Tensor不同重写`magSqr`，SymmTensor因为其形式特别就重写了`magSqr`，如此替换掉最一般的`magSqr`,对SymmTensor仍然可以进行`mag`操作，为什么呢？因为SymmTensor是Tensor是VectorSpace里面的元素，因此一定就有`mag`，此时母类`mag`调用的是SymmTensor版本的`magSqr`。

## 外积

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
