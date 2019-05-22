---
title: Python相关
date: 2018-07-15 14:55:23
tags:
---

# 环境
都说python是很棒的语言，入门学习曲线可以说是相当亲民，但各种环境的“崎岖”让人望而却步，我到底应该用pip呢还是anaconda这样的问题让人挠头，见图便知。
![](pythonEnv.png)

## 我的需求
easybuild依赖于python，简单介绍一下用easybuild：它属于Modules工具，可以（自动搜寻依赖关系）安装各种版本的python，安装相应的python库，配置相应的`清晰干净`的python环境，尤其在多用户高性能计算机群上广泛使用。我需要管理手头多个版本的OpenFOAM和与它们依赖的库（其实主要用到的也就mpi和scotch）。用户体验：当你的工作环境很单一不用Modules没问题，不过一旦复杂度上去了，用Modules工具可以做到事半功倍，"模块化的环境"不仅让工作环境清爽整洁、条理清楚，还可以不费劲地将自己熟悉的整个工作环境在另一台机器上复现出来。我手头与python依赖的环境有：
1. Linux CentOS 系统自带 /usr/bin/python
2. easybuild (仅基于1来安装)
3. /usr/bin/ipython
4. /usr/bin/pip
5. anaconda

我要使用easybuild安装的module的时候用这个alias：
```bash
alias Easybuild='EASYBUILD_PREFIX=$HOME/.local/easybuild; \
                 module use $EASYBUILD_PREFIX/modules/all; \
                 module load EasyBuild; \
                 EASYBUILD_MODULES_TOOL=EnvironmentModules'
```
也就是说默认不加载easybuild的环境，为何？
```bash
$ echo $PYTHONPATH

$ Easybuild
$ echo $PYTHONPATH
/home/hluo/.local/easybuild/software/EasyBuild/3.2.1/lib/python2.7/site-packages
```
看到图里面右上角那个至少有三个箭头朝外的环境变量了吗？就是它。而anaconda有它自己一套的环境变量:
```bash
$ Anaconda                  # 加载anaconda环境
$ which python
~/bin/anaconda2/bin/python
$ which ipython
~/bin/anaconda2/bin/ipython
$ which pip
~/bin/anaconda2/bin/pip     # 是不是开始复杂起来了。。

$ ipython
In [1]: import sphinx

In [2]: sphinx.__path__
Out[2]: ['/home/hluo/bin/anaconda2/lib/python2.7/site-packages/Sphinx-1.4.1-py2.7.egg/sphinx']     # 把库放在了$HOME/bin/anaconda2
In [3]: import numpy as np

In [4]: np.__path__
Out[4]: ['/home/hluo/.local/lib/python2.7/site-packages/numpy'] 
```
为啥两个库存放的地方不同？按理说这是pip安装的目录(见下文)，我肯定之前干了什么然后失忆了，促成了此“姻缘”。。总之我在尝试安装h5py的时候遇到了各种问题，最终因为盘根错节根本厘不清！想搞清楚？重装吧。其实每次重新安装都有“做得更好”的可能，前提是厘清楚，然后养成好的习惯在使用的时候用哪一个就哪一个，切忌同时使用pip和anaconda。`一个程序就做一件事情，然后把它做好`，身边的那些计算机牛人们说过的话哈，引申一下受用了。

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
anaconda也可以安装在自己`$HOME`下且包很全，有个conda命令来管理包的历史、更新、安装、卸载。我在初期就用anaconda，试图用conda安装h5py失败了（安装h5py的动机[源自一篇关于OpenFOAM湍流进口条件的学士论文](https://github.com/timofeymukha/eddylicious)，可惜这个人写的库只适用于structured rectilinear grid，也就是进口得是方形，圆管就不行了。对于我来说，直接用用不上，但eddylicious库的编写里得更多的是面向过程，加上只有一个轴的方向`y`是边界层，蛮好读懂；另一方面他改写了timeVaryingMappedFixedValue，把输出格式改成了HDF5，按照[要求](https://bitbucket.org/lesituu/timevaryingmappedhdf5fixedvalue)编译：要注意Make/options里面通过HDF5_DIR而不是CPATH和LD_LIBRARY_PATH来找，且在环境中设置不够，需要写入到Make/options里面`HDF5_DIR=$HOME/.local/easybuild/software/HDF5/1.8.17-foss-2016a`，这里的HDF5用easybuild安装，跟使用的OpenFOAM/2.3.1-foss-2016a匹配），就`h5py`来说，pip网上的资料还是多些。

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
    l_1d_map2 = tsR.pre_check(ps_map.parameters,"Dai_lines_typeFace_cell-2")       # precheck检查数据是否valid （上面在dataShape处有解释），返回valid数据的路径
    db_1d_map2 = tsR.process(ps_map.parameters,validDataList=l_1d_map2,colonNb=1)  # 仅读取valid数据，做相应后处理，返回一个dictionary

    Ux_bulk_Dai=0.3
    ax1.plot(db_1d_map2['rByD'],db_1d_map2['mean']/Ux_bulk_Dai,label='mapped-2',linewidth=2) # 从dictionary中取出相应数据画图，x轴为rByD，y轴为平均值，无量纲化显式操作
#   ...

#   reference plot
    x1,y1 = rdb.Dai_thesis.Fig4p9a('EAU')  # 嵌套关系表示这里画的是Dai_thesis里面图Fig4.9(a)里面EAU这条线
    ax1.plot(-x1+0.5, y1, label='Dai-2', marker='s', markerfacecolor='none', linewidth=2) # 在同一幅图里面画上对照数据，原始数据需要做变换，显式操作
#	...
#	...

main() # 执行main函数

```

小结：这里主要借我自己在OpenFOAM中后处理的用法（库，库的嵌套，字典，深度拷贝，函数返回值可以是多个[tuple或者dict]）来介绍**python的基本数据类型**，会在另一篇博客中给出针对`sample`这个重要的OpenFOAM utility的全部流程。

## matplotlib

### basics
![](hierarchy1.png)

关于matplotlib里面Figure, Axes, Axis, Tick的层级关系，[这个博客](https://dev.to/skotaro/artist-in-matplotlib---something-i-wanted-to-know-before-spending-tremendous-hours-on-googling-how-tos--31oo)很好

![](hierarchy2.png)

通过传递`ax`来画出caseList里面的所有中间图像，利用`cut`来区分取实际数据的哪个slice，用`ax_principle`来画最终的图像

```python
# -*- coding: utf-8 -*-
"""
Created on Wed Dec 12 21:08:57 2018

@author: hluo
"""

import numpy as np
import matplotlib.pyplot as plt

def plotSlice(ax, sliceNumber, dataDir, cut=0.5):
    data = np.genfromtxt("../"+dataDir+"/"+"userDefinedLog/slice"+str(sliceNumber)+"_mean_rms")
    
    time = data[:,0]
    cutSliceIndex = int(cut*len(time))
    
    ax.plot(time[cutSliceIndex:],data[cutSliceIndex:,2])
    ax.plot(time[:cutSliceIndex],data[:cutSliceIndex,2],linestyle=':')
    ax.legend(bbox_to_anchor=(1.3, 1), ncol=1, shadow=True)
    
    return np.mean(data[cutSliceIndex:,2])
    
def plotCaseWithSlices(ax_cases, dataDir, positionList, marker, cut):
    meanOfRMS = np.zeros(len(positionList))
    
    fig, ax_in_case = plt.subplots()
    for i, position in enumerate(positionList):
        meanOfRMS[i] = plotSlice(ax_in_case, position, dataDir, cut)

    positionList = np.asarray(positionList)
    ax_cases.plot(positionList/8.0,meanOfRMS, label=dataDir, marker=marker)
    
def main():
    plt.style.use('seaborn-white')
    caseList=["BirdCarreau/inlet_0p3",
              "BirdCarreau/inlet_0p3-a_0p5-setT_St_1",
              "BirdCarreau/inlet_0p3-a_0p5-setT_St_5"]
    markerList=["s",
                "^",
                "o"]
    positionList = [0,1,2,3,4,5,6,7,8,9,10,11,12,16,24,32,40,48,56,64,72,73,74,75]
    
    fig, ax_principle = plt.subplots()

    for i, caseDir in enumerate(caseList):
        plotCaseWithSlices(ax_principle, caseDir, positionList, markerList[i], cut=0.7)
        
    ax_principle.legend(bbox_to_anchor=(1., 1), ncol=1, shadow=True)
    ax_principle.set_xlabel(r"$x/D$")
    ax_principle.set_ylabel(r"mixing factor")
    ax_principle.set_ylim(0,0.5)
    fig.savefig('../PICTURE_mixingFactor/mixingFactor.png', bbox_inches='tight', dpi=300)
    
main()

```
### backend
在非图形化界面（非anacoda+spyder）情况下，有可能遇到`ImportError: No module named Tkinter`而卡住在backend上画不出任何图，需要在文件头加上

```python
import matplotlib                                                                                             
matplotlib.use('agg')
```

这样至少可以savefig成功

### rcParams

```python
# 这段代码修改的是全局变量，但是
# 赋过初值之后就不能再修改
# 也就是说如果有两个module，前一个font赋了值20，后一个怎么改都无效

rc('font',**{'family':'serif','serif':['Palatino']})
rc('text', usetex=True)    
style.use('seaborn-white') # from defaut
plt.rcParams.update({'font.size': 20})
plt.rcParams['savefig.dpi'] = 100
```

### 图片尺寸大小
```python
fig, ax = plt.subplots(figsize=(16,10))    # 16 inches * 10 inches

# things plotted ...

fig.savefig("*.png", bbox_inches='tight')  # "tight"会自适应边角，裁掉后不再是(16,10)；

#用bbox_inches的初衷是把跑到画图框之外的legend强制包括到fig内，缺陷除了把figsize改写了还有就是
#可能会把坐标轴ylabel不完全save到fig中去，如果出现了这种情况，去掉tight就好
```

### add image to a plot

```python
fig, ax = plt.subplots()

# ...

im = plt.imread('someFigure.png')
rect=[0.1, 0.8, 0.3, 0.3]
ax_new = fig.add_axes(rect, anchor='NE', zorder=-1) # 原Figure上add_axes
ax_new.imshow(im)
ax_new.axis('off') # 新axe不显示axis

# fig.savefig() # 原Figure已经加上了im
```

## doctest库
这个库挺酷，不过不知道用处大不大，一言以蔽之：这是一个自我检测跨行注释里面内容是否通过测试的库。   

```python
#!/usr/bin/env python2
# -*- coding: utf-8 -*-

"""
>>> a = Interval(30, 40)
>>> b = Interval(35, 50)

>>> a
Interval(30, 40)

>>> a == b
False
>>> a in b
True

>>> a[0], a[1]
(30, 40)

>>> a < b
False
>>> a < Interval(100, 200)
True

>>> a("some", "args")
('called with ', ('some', 'args'))

>>> len(a)
10
"""


class Interval(object):
    __slots__ = ('start', 'end')

    def __init__(self, start, end):
        self.start = start
        self.end   = end

    def __lt__(self, other): # a completely leftOf b ?
        """
        >>> a = Interval(30, 32)
        >>> b = Interval(35, 50)
        >>> a < b
        True
        """
        return self.end < other.start

    def __eq__(self, other):
        return self.start == other.start and self.end == other.end

    def __getitem__(self, i): # a[0], a[1]
        if not 0 <= i <= 1: raise IndexError
        if i == 0: return self.start
        if i == 1: return self.end

    def __repr__(self): # representation of the object:
        return "Interval(%i, %i)" % (self.start, self.end)

    def __str__(self): # string of the object:
        return "%i\t%i" % (self.start, self.end)

    def __call__(self, *args): # a(*args)
        return "called with ", args

    def __len__(self): # len(a)
        return self.end - self.start

    def __add__(self, other):
        return Interval(min(self.start, other.start), max(self.end, other.end))

    def __contains__(self, other): # a in b
        return other.start < self.end and other.end >= self.start

if __name__ == "__main__":
    import doctest
    doctest.testmod()

```

测试`python demo1.py`没有任何输出；如果加上`-v`

```bash
$ python demo1.py -v
Trying:
    a = Interval(30, 40)
Expecting nothing
ok
Trying:
    b = Interval(35, 50)
Expecting nothing
ok
Trying:
    a
Expecting:
    Interval(30, 40)
ok
Trying:
    a == b
Expecting:
    False
ok
Trying:
    a in b
Expecting:
    True
ok
Trying:
    a[0], a[1]
Expecting:
    (30, 40)
ok
Trying:
    a < b
Expecting:
    False
ok
Trying:
    a < Interval(100, 200)
Expecting:
    True
ok
Trying:
    a("some", "args")
Expecting:
    ('called with ', ('some', 'args'))
ok
Trying:
    len(a)
Expecting:
    10
ok
Trying:
    a = Interval(30, 32)
Expecting nothing
ok
Trying:
    b = Interval(35, 50)
Expecting nothing
ok
Trying:
    a < b
Expecting:
    True
ok
10 items had no tests:
    __main__.Interval
    __main__.Interval.__add__
    __main__.Interval.__call__
    __main__.Interval.__contains__
    __main__.Interval.__eq__
    __main__.Interval.__getitem__
    __main__.Interval.__init__
    __main__.Interval.__len__
    __main__.Interval.__repr__
    __main__.Interval.__str__
2 items passed all tests:
  10 tests in __main__
   3 tests in __main__.Interval.__lt__  # 函数里面的专属
13 tests in 12 items.
13 passed and 0 failed.
Test passed.
```
不过这里要注意的是貌似python自带一个Interval的库，所以...有些干扰：比如这里只有`__lt__`和`__eq__`，没有`__gt__`，但可以有操作：

```python
>>> a > b
False
```
[关于操作符号与函数的mapping映射关系](https://docs.python.org/2/library/operator.html#mapping-operators-to-functions)

总结一下，这个库可以用于检测函数的基本功能是否完好。但需要满足前提条件：必须是三个引号那种注释，注释得是`>>>`这种interactive(也就是ipython)里面的格式且不能随意换行；可以在函数里面单独加入一个comment block；但是在头上加入一个随意其他的换行comment好像有可能导致功能失效

# Debug
## 缩进
文本编辑器 vim > gedit 主要小心空格和tab混用，很难找出为啥来
```bash
$ python -t script.py
# This will warn you if you have mixed tabs and spaces

$ cat -A script.py
# On *nix systems, you can see where the tabs
```
## spyder
有次遇到了`TypeError: 'str' object is not callable`，找了半天都没有头绪，结果重启spyder就好了。说明有些错，真不简单，试试重启大法。


# python提交算例到机群
而你又没有pySlurm的时候.....

# 实用手册

## iterable
`iterable`顾名思义就是“能堪循环者”，python里面循环可以有简单写法`for item in some_list:`，这样可以省去指明循环指标变量`i`。不专业的（我）使用的时候有时候还是想要加上`i`，但似乎不是所有的`iterable`都可以加`i`，例如`f = open(fileName,'r')`这里返回值`f`是`type file`，它就不能用`i`来引用，想要对它循环输出行号，我没有成功.[待贴代码验证]

当iterable在循环中被改变(比如`list.remove()`)，要**特别注意**！！可能会直接跳出循环，因为iterable被改变了

## str are immutable
python里面`str`可以slice，但不能被修改，`string.replace`也只是对其拷贝进行的`replace`操作.[参考stackoverflow](https://stackoverflow.com/questions/46850850/python-function-to-modify-string)

