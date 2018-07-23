---
title: Python相关
date: 2018-07-15 14:55:23
tags:
---

# Python环境
都说python是很棒的语言，入门学习曲线可以说是相当亲民，但各种环境的“崎岖”让人望而却步，我到底应该用pip呢还是anaconda这样的问题让人挠头 
![aboutPython](aboutPython.png)

## pip
实验室哥们推荐我就用pip，他给我的使用建议：   
**安装**scipy，`pip install --user scipy`，所安装的包就在`$HOME/.local/lib/python2.7/site-packages`   
**移除**所安装的包直接删除对应位置的文件即可，如果想用另一个版本直接将当前的包删除即可，这样干净整洁不容易出错      
**查看包的版本**   
```bash
$ pip install scipy==
Collecting scipy==
  Could not find a version that satisfies the requirement scipy== (from versions: 0.8.0, 0.9.0, 0.10.0, 0.10.1, 0.11.0, 0.12.0, 0.12.1, 0.13.0, 0.13.1, 0.13.2, 0.13.3, 0.14.0, 0.14.1, 0.15.0, 0.15.1, 0.16.0, 0.16.1, 0.17.0, 0.17.1, 0.18.0rc2, 0.18.0, 0.18.1, 0.19.0, 0.19.1, 1.0.0b1, 1.0.0rc1, 1.0.0rc2, 1.0.0, 1.0.1, 1.1.0rc1, 1.1.0)
No matching distribution found for scipy==
You are using pip version 9.0.3, however version 10.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```
### 安装特例
`h5py`的安装需要用到`devel`包（for developper 或者叫 no-binary 包），因为基于`h5py`的特性需要基于已有的`mpi`和`hdf5`重新编译，比如安装的时候报错`Python.H not found`，那就说明python包里面该有`*.H`的地方没有源文件，也许安装的不是开发者安装包（仅仅是pre-compiled后的库函数），当然另一个情况是给的头文件寻找路径不对。那如果报错是找不到`mpi.h`或者`hdf5.h`，那么就是环境中找不到`mpi`或`hdf5`包里的头文件，也对应前面两种可能。

这里说起来有点绕，这是因为`h5py`是python对hdf5库的一个wrapper，也就是基于用C或者C++的hdf5库的接口重新包装的结果，因而需要保证在当前环境中hdf5的库完成配置（`CPATH`指定头文件的路径，`LD_LIBRARY_PATH`指定`*.so`库的搜索路径），hdf5又依赖于mpi，那么mpi也应该完成配置。这样才能保证重编译的基本要求。需要指出的是一般python包按照以上傻瓜安装就行，不需要操心这么多。

在python里面no-binary安装对应
```bash
$ pip install --user h5py # 预编译好的
$ pip install --no-binary=h5py h5py --user # no-binary 版本
```
## anaconda
anaconda也可以安装在自己`$HOME`下且包很全，有个conda命令来管理包的历史、更新、安装、卸载。我在初期就用anaconda，试图用conda安装h5py失败了（安装h5py的动机[源自一篇关于OpenFOAM湍流进口条件的学士论文](https://github.com/timofeymukha/eddylicious)，可惜这个人写的库只适用于structured rectilinear grid，也就是进口得是方形，圆管就不行了。对于我来说，直接用用不上，但eddylicious库的编写里得更多的是面向过程，加上只有一个轴的方向`y`是边界层，蛮好读懂；另一方面他改写了timeVaryingMappedFixedValue，把输出格式改成了HDF5），就`h5py`来说，pip网上的资料还是多些。
