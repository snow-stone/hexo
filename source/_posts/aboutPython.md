---
title: Python相关
date: 2018-07-15 14:55:23
tags:
---

# 环境
都说python是很棒的语言，入门学习曲线可以说是相当亲民，但各种环境的“崎岖”让人望而却步，我到底应该用pip呢还是anaconda这样的问题让人挠头 
![](pythonEnv.png)

## pip
实验室哥们推荐我就用pip，他给我的使用建议：   
**安装**scipy，`pip install --user scipy`，所安装的包就在`$HOME/.local/lib/python2.7/site-packages`   
**更新**scipy，`pip install scipy --user -U`，所安装的包就在`$HOME/.local/lib/python2.7/site-packages`   
**移除**所安装的包直接删除对应位置的文件即可，如果想用另一个版本直接将当前的包删除即可，这样干净整洁不容易出错，此例中`rm -rf $HOME/.local/lib/python2.7/site-packages/scipy*`      
**查看包的版本**   
```bash
$ pip install scipy==
Collecting scipy==
  Could not find a version that satisfies the requirement scipy== (from versions: 0.8.0, 0.9.0, 0.10.0, 0.10.1, 0.11.0, 0.12.0, 0.12.1, 0.13.0, 0.13.1, 0.13.2, 0.13.3, 0.14.0, 0.14.1, 0.15.0, 0.15.1, 0.16.0, 0.16.1, 0.17.0, 0.17.1, 0.18.0rc2, 0.18.0, 0.18.1, 0.19.0, 0.19.1, 1.0.0b1, 1.0.0rc1, 1.0.0rc2, 1.0.0, 1.0.1, 1.1.0rc1, 1.1.0)
No matching distribution found for scipy==
You are using pip version 9.0.3, however version 10.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```
如果是python3的库，那么就改用`pip3`，库函数就会放在`$HOME/.local/lib/python3.4/site-packages`（这里是3.4）
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

# IDE
我在windows和linux下也都有anaconda，主要用在包里的spyder，可以实时查看数组变量（函数内部的变量在IDE里面查看不了，因为没有暴露在当前执行脚本的环境中），这在python学习初期很有用，在初期学习中可以有意识地不写函数，这样所有的变量都在可追溯的`Variable explorer`里面便于实时查看。想要查看某变量的类型，在里面集成的ipython console通过`type(variable)`输出即可。   
我选spyder的主要原因：   
1. 用matplotlib画图的时候就在ipython console即时输出图片，避免另弹出画图的新窗口+手动关闭窗口，看到图片就可以调整legend的位置还有字体大小什么的，很直观；如果要拷贝图片另存为png格式直接右键`save image as`就可以，所见即所得（为何这样说？如果legend在图片box外面，`savefig`不会将legend包括进去，也就说savefig最终不会包含box外的legend）。
2. 库函数自动补全
3. 注释和去注释方便，缩进和去缩进方便，高亮易辨识
4. 鼠标点击函数，在不同的库之间跳转

**福利**：spyder -> File -> Print to File (PDF) 有语法高亮

**局限**: 在3d plot里面有些图（比如3D云图）是可以鼠标拖着旋转的，但在spyder里面就是个静止的图（可能原因在于就其本源可能还不如货真价实的ipython console）；因此animation当然也在spyder里面看不了,这个时候就都得回到无IDE下的python解释器环境

# 基本数据类型
**Sequence Types** — str, unicode, list, tuple, bytearray, buffer, xrange  
1. str : string, written in single or double quotes
2. unicode : unicode string, much like string
3. list : constructed by square brackets, its element seperated by comma
4. tuple : constructed by the comma operator (not within square brackets), with or without enclosing parentheses, but an empty tuple must have the enclosing parentheses, such as a, b, c or (). A single item tuple must have a trailing comma, such as (d,).
5. bytearray : created with the built-in function bytearray()
6. buffer : Buffer objects are not directly supported by Python syntax, but can be created by calling the built-in function buffer()
7. xrange : similar to buffers in that there is no specific syntax to create them, but they are created using the xrange() function

**Operators on list**  
```python
>>> myList = ["The", "earth", "revolves", "around", "sun"] # initialize a list with double quotes
>>> myList
 ['The', 'earth', 'revolves', 'around', 'sun'] # output are single quoted
>>> myList = myList + ["sure"]  # need a list to perform operator+ on list objects
>>> myList
 ['The', 'earth', 'revolves', 'around', 'sun', 'sure'] # list + list = list
```

**Sequence Types** -- dict  
A mapping object maps hashable values to arbitrary objects. Mappings are mutable objects (in c++ there are also mutable objects, what are they?). There is currently **only one** standard mapping type, the dictionary.
Dictionaries can be created by placing a comma-separated list of key: value pairs within braces, for example: {'jack': 4098, 'sjoerd': 4127} or {4098: 'jack', 4127: 'sjoerd'}, or by the dict constructor.

以上是抄写，这里是个人使用心得：list里面可以放不同类型的变量，很有用；tuple可以作为函数return多个值的时候的选项例如`fig, ax = plt.subplots()`，当然dictionary也可以；我在OpenFOAM后处理脚本里面用到dict，因为一个dictionary就可以包含算例的绝对路径、后处理时间区间，后处理数据shape...更重要的是在程序读取并处理数据时，主程序里`暴露`出从key到value的过程，比如如果我画平均值就取名为`mean`，如果画均方根就取名为`rms`，这样后处理程序可读性会大大提高；numpy里面array当然用得很多，list可以转化为array

# 库的使用
python强大的地方当然是有各种很棒的库的支持(numpy, scipy, matplotlib等)，使用库用关键字`import`就行。如何查询所用库的路径？进入ipython
```bash
In [1]: import numpy as np
In [2]: np.__                   # 自动补全
np.__
np.__NUMPY_SETUP__   np.__delattr__       np.__getattribute__  np.__new__           np.__repr__          np.__version__
np.__all__           np.__dict__          np.__git_revision__  np.__package__       np.__setattr__       
np.__builtins__      np.__doc__           np.__hash__          np.__path__          np.__sizeof__        
np.__class__         np.__file__          np.__init__          np.__reduce__        np.__str__           
np.__config__        np.__format__        np.__name__          np.__reduce_ex__     np.__subclasshook__
In [2]: np.__path__
Out[2]: ['/home/hluo/.local/lib/python2.7/site-packages/numpy'] # 可以看到输出的是一个list，里面路径变量是str
```
## 自定义库
自定义库也很简单，写了一个脚本叫`Dai_thesis.py`，里面有个函数叫`Fig4p8a`，那么在主程序（同一目录下）只需`import Dai_thesis`，然后就可以`Dai_thesis.Fig4p8a()`，在执行主程序的时候`Dai_thesis.pyc`将会自动生成，这是个二进制文件，表明`Dai_thesis.py`以库的形式被使用，这样就使用了Dai_theis这篇论文里面Fig4.8(a)里面的数据。当然库的使用也有嵌套，例如在上述基础上加一个层级:
```python
# 高层级的库 reference_database.py ： 我画图的时候作为对照的的数据来源汇总：杂志文章、博士论文、解析表达式

import Niewstadt_article_1995

import Dai_thesis

import analytic_functions

import Eggels_thesis

import Eggels_article_1994



# 低一个层级的 Dai_thesis.py : 功用是去特定的地方读特定的文件，返回numpy array组成的一个tuple

import numpy as np

def Fig4p8a(fluid):
    string='/home/hluo/Pictures/Dai_T/meanProfile_velocity/XEqM4_debitMin/profile_XEqM4_debitMin.csv'
    data=np.genfromtxt(string,skip_header=1,delimiter=',')
    
    switcher={
        'EAU':1,  # 选择流体类型
        'PAA':2,
        'XG':3
    }
    
    x = data[:,0]
    y = data[:,switcher[fluid]]
    
    return x, y

def Fig4p9a(fluid):
    string='/home/hluo/Pictures/Dai_T/meanProfile_velocity/XEqP12_debitMin/profile_XEqP12_debitMin.csv'
    data=np.genfromtxt(string,skip_header=1,delimiter=',')
    
    switcher={
        'EAU':1,
        'PAA':2,
        'XG':3
    }
    
    x = data[:,0]
    y = data[:,switcher[fluid]]
    
    return x, y
   


# parameters_T_RES1d_MethodMapped_subMethod_NearestFace.py ： 用dict存储一个OpenFOAM算例后处理里的复合数据类型

# physical parameters

physics={
        'R':0.004,
        'nu':1.0e-6,
        'uTau':0.0473
        }

# output of sample utility

sampling={
        'raw_sample_size':160,
#        'dataShape':(199,4)     # uniform 
#        'dataShape':(188,4)     # face 3 
        'dataShape':(195,4)     # face 2     
#        'dataShape':(195,4)     # face 1    

# 这里展示了这个后处理过程的复杂性：dataShape有多个值
# 因为OpenFOAM sample sets操作后的数组用不同mode输出的数据长度不同
# 例如: uniform 和 face 两个mode下输出的长度不同，在同一mode下不同的地方进行sample操作也可能得(对于
# 复杂的几何外形的算例，网格在空间不均匀，换个sample的位置，几乎一定会变)的不同长度的数据，例如这里的
# face 1,2,3

# 为何要确定长度？因为如果做空间统计（这个例子是时间，只要网格不变，sets不变那数据长度就不变，因为取
# 样的labelList不变），比如在周期条件的圆管流动里，里面沿着半径方向取一系列数学上有对称性的sets，然后
# 想求对于sets的平均（即空间平均），每个sets在这里对应一个数据文件。这时候即使是相当规则的网格（butterfly
# 分成5块），我的测试结果是:这一系列看起来数学上完全对称的sets有时候也会出现个别的特例（要么输出数据
# 长度为0，要么输出数据跟大部分的shape不同，OpenFOAM当然不会报错警告你，所以我自己建立了一个检查机制，
# 这样呢就需要一个目标shape和读取实际数据shape的对照的过程，这样每次后处理我就知道是否有exception。

# 于是在这里，通过这样一个dictionary完成了参数parameters['sampling']['dataShape']的传递）
        }

# data entry parameters

dataEntry={
        'startTime':5.5,  # KinecticEnergy stationary
        'endTime':7.2, 'chunkStep':50,
        'NbOfFiles':171,
        'path':"/store/caseByGeometry/T/new-mesh/pointwise/postProcessing/1d_mapped_NearestFace",
        }

parameters={
        'physics':physics,
        'sampling':sampling,
        'dataEntry':dataEntry,
        }

```

最后看看主程序，`rdb.Dai_thesis.Fig4p9a()`完成了对两个自定义库的嵌套，可读性强;函数传入一个dict比传入长串多个参数可读性强，比如`tsR.pre_check(ps_map.parameters,...)`;在主程序中显式操作数据（无量纲化，坐标轴变换），有迹可循。可读性是强了，可维护性呢？

基于这个主程序，如果要加上另一个算例(例如空间分辨率不同)的结果画在同一个图中，该怎么做呢？编辑另一个算例相应的dict（路径，时间区间等）,然后在主程序开头处import即可(注意，实际上编辑的不是dict，而是库，我这里通过库的形式传入dict)；如果要对同一个算例，画出在两个时间区间的统计结果，怎么办？我的做法是`深度拷贝ps_map.parameters这个dict`，在**主程序**里面显式地修改时间区间（为什么是显式的？因为key是'startTime','endTime'，改的变量是什么对于使用者来说很直观），这样通过这个修改后的dict传入参数，做同样的pre_check和process**两行操作**就可以，主程序仍旧保持着相当一致的格式(较为固定的**两行操作**)。如果要做一个以上两个变种的融合不外乎多几个**两行操作**，易于维护。

加上pre_check会对读入数据是否valid进行判断，因此也易于debug。

用spyder来操作，实时得图，实时跳转至库文件编辑，实时在主程序调整画图的参数。

```python

# 主程序
import matplotlib.pyplot as plt    # python 画图基本库
import reference_database as rdb   # 自定义数据库

def main():
    import timeSeriesReader_ReturnOuterVariables as tsR                    #  自定义库，做时间平均
    import parameters_T_RES1d_MethodMapped_subMethod_NearestFace as ps_map #  自定义库，算例的parameter文件(通过库的方式传递到主程序)，其实就是一个dictionary

    fig1,ax1 = plt.subplots()      # 画图初始化 fig1和ax1

#   plot my data
#   ...
#   这里通过ps_map这个库，去相应路径找待处理的数据(例如OpenFOAM sample的结果)
    l_1d_map2 = tsR.pre_check(ps_map.parameters,"Dai_lines_typeFace_cell-2")       # precheck检查数据是否valid （上面在dataShape处有解释）
    db_1d_map2 = tsR.process(ps_map.parameters,validDataList=l_1d_map2,colonNb=1)  # 仅读取valid数据，做相应后处理，返回一个dictionary

    Ux_bulk_Dai=0.3
    ax1.plot(db_1d_map2['rByD'],db_1d_map2['mean']/Ux_bulk_Dai,label='mapped-2',linewidth=2) # 画图，x轴为rByD，y轴为平均值，无量纲化显式操作
#   ...

#   reference plot
    x1,y1 = rdb.Dai_thesis.Fig4p9a('EAU')  # 嵌套关系表示这里画的是Dai_thesis里面图Fig4.9(a)里面EAU这条线
    ax1.plot(-x1+0.5, y1, label='Dai-2', marker='s', markerfacecolor='none', linewidth=2) # 在同一幅图里面画上对照数据，原始数据需要做变换，显式操作
#	...
#	...

main() # 执行main函数

```

小结：这里主要借我自己在OpenFOAM中后处理的用法（库，库的嵌套，字典，深度拷贝，函数返回值可以是多个[tuple或者dict]）来介绍**python的基本数据类型**，会在另一篇博客中给出针对`sample`这个重要的OpenFOAM utility的全部流程。


## Debug
文本编辑器 vim > gedit 主要小心空格和tab混用，很难找出为啥来
```bash
$ python -t script.py
# This will warn you if you have mixed tabs and spaces

$ cat -A script.py
# On *nix systems, you can see where the tabs
```
### spyder
有次遇到了`TypeError: 'str' object is not callable`，找了半天都没有头绪，结果重启spyder就好了。说明有些错，真不简单，试试重启大法。
