---
title: OpenFOAM入门
date: 2018-07-05 14:15:39
tags:
---

# 工作环境
1. Linux
2. vim: 有[OpenFOAM关键词自动补全插件](https://bitbucket.org/shor-ty/vimextensionopenfoam)
3. eclipse: 虽然代码自动跳转不全，但看代码的时候用用鼠标也能省点力气
4. meld: 对比代码非常好用，相对vim优势在有GUI，diff算法加上颜色高亮
5. python: OpenFOAM后处理有时会用到字符串的处理，python本身还有很棒的库，用字典可以将一个case的不同属性参数组合并存储{比如路径，粘性，雷诺数，后处理相关(utility sample后得到postProcessing路径下的数据shape是多大[检查读入数据shape以防溢出或者意外情况]，后处理的时间区间与步长[用于在不同时间目录下读取相应后处理数据])}
6. spyder: python IDE，用matplotlib画图便于在同一窗口下实时查看结果，鼠标点击module直接跳转，实时的变量查询。缺点是不能在窗口下生成动画，或者是可以转动的3D图，这种时候改用命令行纯的python解释器就好

# 手动编译源码
## 在win10上bash子系统ubuntu编译流程
1. install libraries including `bison` required by `scotch` as indicated in standard intallation guide. Don't need realy install paraview so leave those extra libaries of openGL aside.
2. if the first step is finished, a `SYSTEMOPENMPI` is then installed (as root in defaut env). Change the `etc/bashrc` to `SYSTEMOPENMPI`
3. as `CGAL` is only [optional](https://www.cfd-online.com/Forums/openfoam-installation/178509-openfoam-4-0-install-without-foamyhexmesh.html) comment the whole file `etc/config/CGAL.sh` and in `ThirdParty/Allwmake` comment out the block for this library
4. run Allwmake in ThirdParty
5. run Allwmake in OpenFOAM-2.3.x
6. `twophaseEulerFoam yyFlexLexer` Error led to [flex version error](https://www.cfd-online.com/Forums/openfoam-installation/187303-installing-openfoam-2-3-x.html). Act as wyldckat said and solve the problem.
7. After compiling all the solvers (last step). Run `./Allwmake > log2.summary 2>&1` to get a full compilation summary.   

## Debug build
什么时候用到Debug build？当出现比如除以零的情况，Debug模式会给出对应函数的行号。   
怎么编译？把`etc/bashrc`里面默认Opt改为Debug，重复上诉流程，编译通常会更慢，编译生成的库不再位于`platforms/xxxOpt`而是`platforms/xxxDebug`，两build相互独立。   
怎么使用？在使用OpenFOAM之前都需要`source etc/bashrc`，这时候载入flag是Debug的版本就好啦。

如果要改安装位置到`$HOME/LocalSoftware`，我把Debug的和原版diff一下：
```bash
$ diff bashrc.Debug bashrc.org 
45c45
< foamInstall=$HOME/LocalSoftware/$WM_PROJECT
---
> foamInstall=$HOME/$WM_PROJECT
79c79
< export WM_COMPILE_OPTION=Debug
---
> export WM_COMPILE_OPTION=Opt
132c132
< export WM_PROJECT_USER_DIR=$HOME/LocalSoftware/$WM_PROJECT/$USER-$WM_PROJECT_VERSION
---
> export WM_PROJECT_USER_DIR=$HOME/$WM_PROJECT/$USER-$WM_PROJECT_VERSION
```
   
## 通过EasyBuild安装
用module更集成的方式配置环境。   
优点是像公用并行集群上一样统一规范，且依赖关系已经配置好。   
缺点是个别OpenFOAM版本安装不上。   
折衷：可以用机群上的MPI来做自己`$HOME`下的编译，不用等着管理员来安装OpenFOAM。
   
## 小结
1. 最重要的ThirdParty里面的库有openmpi(这个流程里面用了系统安装的mpi即SYSTEMOPENMPI)，还有就是scotch和ptscotch（对应并行求解前的decomposePar分解求解域到进程，其中最常用的method叫scotch）
2. CGAL库可以注释掉至少我用不到foamyhexMesh
3. flex的问题可能会碰到，按照帖子指出的解决就好
4. 编译流程到最后才是编译求解器，因为是最顶层
   
教训：编译时间成本较大，想把其中大部分的库的依赖关系搞清楚未必是个有意义的工作，所以尽量先按照流程用最简单（apt-get）的方式来获得依赖的库。比如我就栽过一次在机群的老gcc版本上，编译OpenFOAM-2.3.x用的gcc我推荐至少4.9.2。事先很难把信息汇总全面，没法子，就是得积累经验。   
手动编译的好处：安装的位置在`$HOME`下，在用户权限内运作，干净、标准、规范且修改方便。

# 学习资源
待汇总

# 初学有用的技巧
0. 在计算过程中（提交算例后有个申请的计算时长），发现CFL比较大时间步`deltaT`可以实时改小，可以修改输出间隔，包括粘性`nu`(这个是因为createFields.H里面`MUST_READ_IF_MODIFIED`)都属于runTime-modifiable。湍流模型、边界条件（不确定）、求解器设置(`fvSolution`和`fvScheme`)属于runTime-selectable。
1. changeDictionary
2. topoSet : topoSet 不仅可以得到一个labelList，而且这个labelList还可以用paraview可视化。假如`cellSet`名为`cylTurbGenerator`，`foamToVTK -cellSet cylTurbGenerator -latestTime` 这样就会有`*.vtk`生成，注意到这里面patch和internalMesh似乎被分开了。
3. m4 - blockMeshDict
