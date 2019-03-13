---
title: CheckList
date: 2018-07-26 09:23:58
tags:
---

# 检查表制度
比较深入认识到“检查表”(check-list)是源于[傅盛](https://mp.weixin.qq.com/s?__biz=MjM5NjgzMzkwMQ==&mid=2653646406&idx=1&sn=9fc89b68138045f882a1f5dd04afbef4&chksm=bd3cf1448a4b78527c0ba97e914fcbad89d429696d61fb087b18a87a266ef9cd076df86c1428&mpshare=1&scene=1&srcid=0728p02JYofiPhU6NCYy4tnr#rd)的文章，他提到让工作变得可靠高效，检查表是个保障时效的好东西，看似机械但是可靠。随后是在《黑匣子思维》里面读到，进一步了解到纽约哈德孙河上迫降成功的背后（美当局还是对机组进行了详细地调查）起到积极作用的有无数次飞行的经验，过硬的心理素质，还有副机长分担责任在机长思考的同时协力沟通，副机长在短时间内把空客给的发动机停车的check-list走完。检查表制度已经完全融入了这些航空工业优秀从业者的工作中，如作者所说：人的因素可能是正面的也可能是负面的，然而可靠性的保证，需要仰仗人训练自己“黑匣子”一般的思维。回到傅盛，自己定下的check-list就一定要**严格执行**，要不然你将看不到当前check-list的缺陷，改进也就无从谈起，低级错误/已知错误还是会犯。check-list的存在意义在于，当你头脑不是特别清醒，状态不好的时候，也能给你一个完整严谨的流程，最大程度规避已知的一系列“低级错误”，不单纯依靠记忆和头脑，复杂系统尤为需要如此作业。

# 为什么我要用在OpenFOAM上?
同fluent或其他CFD软件，OpenFOAM算得上复杂系统，初学者容易犯错的系统：不像fluent通过按钮和已经编程的GUI来构建组织求解一个问题的逻辑链，OpenFOAM从边界条件、初始条件、网格文件、流体物性、离散格式、求解器（U求解器，p求解器）还有utility到用到的例如`sampleDict`，`changeDictionaryDict`，`refineMeshDict`，`topoSetDict`等，里面的逻辑关系通过一系列的文本文件来完成。信息的碎片化很可怕，想想你用vim去打开一个个文本文件来检查到底是哪里出了错，这样的过程必然是艰辛的。

为何容易犯错？
1. 文本文件有特定格式打错字也很常见
2. OpenFOAM在tutorial里面给的范本参照并不是MVP（[Minimum Viable Product](https://book.douban.com/subject/27077719/)），也就是说通常都有**不必要**的部分，初学者者并不清楚哪些信息必要，哪些重要，哪些完全没用。

总之细节很多，细节一旦繁多而又难以从中抽离就难有大局观。而check-list就是提醒自己“从哪里出发”，“现在在哪里”，“还要去往哪里”。举例：像飞机起飞的check-list一样，一个check-list可以是对于某算例从无到有的按步骤一步步的配置。“从哪里出发”包括最基本的`0`，`constant`和`system`的文本文件结构，包括边界条件、初始条件，那么“项目的出发点”就是自己剔除上面提到的**不必要**部分后得到的MVP case，基于MVP case的基础可以做加密网格或者其他的变式。顾名思义，“现在在哪里”是check-list当前步骤，“还要去往哪里”取决于自己的目标。

记录“已知在某个步骤上会出现的问题”，然后据此设计check-list来规避已知问题，这是高于平时“工作的自己”层次的思维方式。工作的时候我要debug，要进入多层嵌套的代码，是细节的把控，而check-list让我跳出来，让我看到自己在地图的哪个地方，在某某步骤上的经验和教训是什么，接下来可能的问题是什么，问自己是主要矛盾吗，时间成本会是多少，应该如何决策，这是**做决策**的思维层次([原则](https://book.douban.com/subject/27608239/))。为何需要决策而不是单纯地使用？因为没有“机器”，或者说没有包装好的GUI会告诉我“现在到了哪一步，下一步需要做什么”。

总结一下：相较于fluent，OpenFOAM的劣势是文本文件之间的自然逻辑没有GUI来完成，优势是对一个功能如果充分了解，就可以有很大自由度进行操作（可以按照上面说到的check-list来完成步骤，实现步骤的“自动化”）,因而劣势可以弥补; OpenFOAM最大的优势还是完全开源，只有不懂的地方才是黑箱，经手的所有步骤都在掌握，所有的变量都在掌握

要知道，检查表可繁可简，为节约时间成本阿波罗13号出事之后检查表都被大规模简化，也就是“跳步”，而明白跳步的风险是简化的必须。所以检查表到底应该多繁多简，视情况而定。我根据自己的经验制作了一系列检查表，有的是配置算例流程，有的是关于某一个utility，还有NotToDo-list，总之对我怎么有用怎么来，还不止OpenFOAM，因为尤其跟CAE相关的软件都很大型，按钮特别多，记忆是不可靠的。加上如果在不同的计算机群上面计算，队列是不一样的，对用户的要求是不一样的（比如occigen上面对文件个数要求特别苛刻），用户的需求是不一样的（比如考虑到priority的问题，我又时候可能会选择提交一个计算时间较短的算例），情形很多，未必都需要检查表化，但按照自己的逻辑记录下要点，比起每次决策的时候再去网站上找零散的信息来得更高效。

# 我的检查表
## Pointwise 

### 网格导出流程

- [ ] Select Solver : OpenFOAM 3D
- [x] Set BC types : make sure "all boundaries" are set or you may encounter -> `--> FOAM FATAL ERROR: Continuity error cannot be removed by adjusting the outflow. Please check the velocity boundary conditions and/or run potentialFoam to initialise the outflow.`
- [x] Select blocks
- [ ] Export CAE to some "$CASE/constant/polyMesh"
- [x] Rescale to meter : transformPoints -scale ’(0.001 0.001 0.001)’
- [ ] checkMesh
- [ ] serial run to check

### 配置openfoam周期条件

**语境**   
Pointwise里面网格为domain，OpenFOAM里面网格为patch   

曾经天真的以为按照pointwise里面BC设定时选用`cyclic`，在网格文件生成完成后在`constant/polyMesh/boundary`里加上`neighbourPatch`就好，至少我在18.2R1这个版本不行。按照[Maddalenna](https://www.cfd-online.com/Forums/openfoam-meshing/61596-cyclic-bcs-pointwiseopenfoam-export.html)的提示。行得通的流程如下：   

1. pointwise中要生成完全一样的domain，复制平移都不够（不够是指精度，重要的是点对点的对应关系），要达到这样的标准：create + periodic 这样生成的domain就是原先的twin，在domain list里面也可以看到domain之间的对应关系。这里就有了`neighbourPatch`的基础，没有这一步，极有可能会遇到`face 1 area does not match neighbour *** by ***% -- possible face ordering problem.`

2. 按照pointwise基本流程导出网格文件，但在设置BC的时候设置成`patch`而不是`cyclic`

3. 设置`createPatchDict`来完成`patch`到`cyclic`的转换. 默认createPatch 将生成新的网格文件，`-overwrite`可以改写原先网格文件，副产物还有一系列`*.obj`文件. 正确的dict配置如下：


```cpp
//  File system/createPatchDict
//  姑且在这个语境中用“面”代替patch

    {
        // Name of new patch
        name front_cyclic;     // 操作完成后会是cyclic类的面1的name

        // Type of new patch
        patchInfo
        {
            type            cyclic;      // 新面1，类型
            matchTolerance  0.001;
            neighbourPatch  back_cyclic; // 对应的新面2
        }

        // How to construct: either from patches or set
        constructFrom patches;

        // If constructFrom = patches : names of patches. Wildcards allowed.
        patches ( front );    // 旧面1
    }

    {
        name back_cyclic;      // 新生成的会是cyclic类的面2

        patchInfo
        {
            type            cyclic;
            matchTolerance  0.001;
            neighbourPatch  front_cyclic;
        }

        constructFrom patches;

        patches ( back );     // 旧面2
    }

```
这里需要解释一下：新面1,2是由旧1,2生成，生成的条件有“类型，matchTolerance,neighbourPatch”，当新2还没有生成的时候怎么将新1与新2对应呢？当然，这里**如果**把第一个大括号当成了新1 patch的constructor，它将找不到新2的定义。不过回想一下，其实旧12和新12都互为twin的关系，互相依赖，这样写并无道理，只是...程序具体怎么实现的就不知道了.   
**注：为了保证无误，新旧1和新旧2的名字没有取一样；想改成新旧一样？在这个流程的最后把constant/polyMesh/boundary里面改过来就好**   
注：createPatch在这里是由原先pointwise的twin domains/patches由类型`patch`变成了类型`cyclic`，据utility的功能介绍还可以将patches变成一个patch，或者将faceSet变一个patch


## OpenFOAM
### Simulation 完整流程

0.  env                : purge modules ; set Foam environment
1.  mesh               : transformPoints -scale '(0.001 0.001 0.001)', checkMesh[need a time dir (empty dir is OK)]；格式上mesh变成binary后paraview读入会更快(`foamFormatConvert -constant`)，但OF-2.3.1和OF-3.0.1在binary上不兼容，得通过ascii转换
1.  compile BC         : 如果要用一个新的BC
2.  initial condition (mapFields) and BC (changeDictionary)
3.  topoSet
4.  system/controlDict : check `startTime` 对不对, 保证startTime和endTime不相同，如果相同的话openfoam还不会报错，log的末尾仍旧是`Finalising parallel run` ; 如果有自己编译的模块，加入`libs ("*.so");`
5.  serial run check
6.  decomposePar -time 'startTime' : 清除`startTime目录`里面'uniform', 需要double check流场是否正确地被decompose. 首先写入硬盘的是constant/polyMesh的划分，后`field transfer`是流场的划分
7.  BC                 : double check 一下BC有没有被正确写入`processor*`(value可能被改写，但原则上`member`一定要都在，在BC编写时`write(Ostream&)`写对了就没有问题)
8.  job slurm file     : 
a)  `--job-name` 8个字符   
b)  队列, 节点数，每个节点task数，总task数（可以小于`节点数*单个节点task数`），预估计算长   
c)  可以含有 bash variable : np, nos   
d)  (仅occigen)为避免module相关输出(竟然module purge也会被认为是error我也是醉了)到`*_mpi.%j.err`，slurm file里就不再加入任何的环境配置，通过`0`里面实现slurm任务提交时正确的环境配置   
e)  注意 `--exclusive` 是否必要   
f)  注意 `--mem` 是否足够   
g)  永远不要在executable后面加`&`幻想成后台运行，然后放在其后的command会被继续执行。这样做的结果是`&`之后就没有然后了，而且还可能error message都没有   
9.  logFile            : Dont overwrite. Make sure system/controlDict* correspond to the nos   
10. run on test if possible   
11. run job chain via python   
a) checkList python laundary
b) checkList python auto submit
12. paraview           : 优先decomposed case，internalField没有影响，但reconstruct之后可能`boundaryField`的value会被篡改


### preProcessing
#### mapFields

此为大坑，尤其是OF-2.x的版本，存在一些bug(不是fatal，但会让mapFields运行得无比慢，比fatal还可恶。按照[帖子](https://www.cfd-online.com/Forums/openfoam-bugs/194353-mapfields-major-bug.html)改了还是不行)，但用同样的mapFieldsDict试一试OpenFOAM/4.0-foss-2016b或者OpenFOAM-5.x不仅速度快而且不会有莫名其妙的报错


```bash
#!/bin/bash
source=/some/case/to/be/mapped
#taget 这里target就是"."

mapFields $sourcedir -case . -noFunctionObjects -fields '(U p T)' -sourceTime '8' -targetRegion region0 > log.mapFields_8 2>&1
```

0. 回避  
a) $source里`empty`的BC（有试过，会有报错）  
b) 如果$source里面constant/polyMesh/boundary里面有`mappedPatch`，且如果$target里面没有相应的BC配置，可能会在mapFields最后写入数据的时候报错后果是例如U文件的写入遇到错误而被跳过].在$source有`mappedPatch`的情况下，$target里面constant/polyMesh/boundary也得改成相应BC  
1. 如果不是consistent，目标case里面要编辑好文件mapFieldsDict
2. 确认$source里面startTime，它会是mapFields完成后的时间目录，输出格式改为ascii
3. mapFields (等待时间可能很长)
4. 手动检查mapFields是否无误地完成：检查目标case里面是否有'-nan'
5. 把映射后的场的BC由`calculated`改成相应的物理BC，这样才可以续算，这个步骤可以通过changeDictionary来完成
6. 串行试运行

```bash
# 一个长度为5D的圆管映射到一个长度为10D的圆管，inlet*完全对应
/*--------------------------------*- C++ -*----------------------------------*\
| =========                 |                                                 |
| \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox           |
|  \\    /   O peration     | Version:  2.3.1                                 |
|   \\  /    A nd           | Web:      www.OpenFOAM.org                      |
|    \\/     M anipulation  |                                                 |
\*---------------------------------------------------------------------------*/
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    location    "system";
    object      mapFieldsDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

patchMap        ( inlet0 inlet0         // Source 和 Target 都有的 patch
                  inlet1 inlet1         // 且一一对应
                  inlet2 inlet2
                  inlet3 inlet3
                  inlet4 inlet4 );

cuttingPatches  ( wall                  // 剩下 wall patch 和 outlet*
                  outlet0               // 此处为空->(U p)在 wall 和 oulet* 会出现异常大数甚至nan!!
                  outlet1               // 只填 wall， wall上面插值没有异常大数出现
                  outlet2               // 填上 wall patch 和 outlet*, outlet*都乖乖地保持了“最简单状态” (虽然U在 
                  outlet3               // outlet*上面不应该是 uniform (0 0 0)，但毕竟是 type calculated
                  outlet4 );

// ************************************************************************* //

# 两个T型圆管，几何完全一样，但网格不同
/*--------------------------------*- C++ -*----------------------------------*\
| =========                 |                                                 |
| \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox           |
|  \\    /   O peration     | Version:  2.3.1                                 |
|   \\  /    A nd           | Web:      www.OpenFOAM.org                      |
|    \\/     M anipulation  |                                                 |
\*---------------------------------------------------------------------------*/
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    location    "system";
    object      mapFieldsDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

patchMap        ( Port1 Port1        
                  Port2 Port2
				  Port3 Port3
				  wall  wall  );

cuttingPatches
();
// ************************************************************************* //

```

##### 旋转data然后mapFields
想要rotate data，transformPoints声称可以rotate polyMesh里面的points(也就是网格)，也可以rotate vector field.试过了，确实可以，paraview上就看出来转了90度，但是!将rotate过后的再mapFields就不成功了，经过反复测试始终还是rotate之前的data被map过去了的感觉

### postProcessing

#### userProbeByLabel_noMean
or userProbeByLabelVector_noMean its version for vectors. `$sliceStore` is where you put your `slice`+`number` file (class labelList). userProbeByLabel_noMean dont work on slices but on `labelGroup` which is itself a `labelList` containing all cell id of probes of interest (its defaut location is `$sliceStore`.

0. checkList userFindClosestInLabelList
1. userProbeByLabel_noMean T $sliceStore -time '0:xxxx' 


#### userFindClosestInLabelList
前提是在input argument里面`sliceStore`处已经有`sliceNumberList`和`refVectors`，这个程序找的是遍历在`sliceNumberList`里面所有的slices找到离`refVectors`（**注**：这里仅有一个vector，但仍旧用vectors，为得是日后可以拓展）最近的cell，并将每个slice上面最近的cellID记下来写入到一个文件`labelGroup`.这个程序应当在一个OpenFOAM case中执行，其实就是提取网格信息，因此对于相同网格下的算例仅仅需要运行一次，于是就有了最后一个步骤将`labelGroup`移动到对应同一个网格的公共的目录下共享.详细checkList如下:

0. 为了最终写出的labelList人类可读:`system/controlDikt` change format to ascii
1. userFindClosestInLabelList xx xx
2. cd constant
3. mv file to a shared place (for all cases with the same mesh structure)

#### reconstructPar

在reconstructPar -fields '(U p)'之后，reconstructPar -fields '(phi)'会将phi添加到对应的时间目录里面.


#### sample

##### 基本流程

```bash
#!/bin/bash
sampleDict=system/sampleDict

sample -dict $sampleDict/sets/Dai_lines_5cutsBetween2and3_typeFace_cell -time '8:10' > log.sample-Dai_lines_5cutsBetween2and3_typeFace_cell 2>&1
```

1. 编辑sampleDict：可能需要用python脚本来写入一系列sets的描述，例如160条线就不能全部手写  
a) 检查`fields`，如果写得不对，OpenFOAM并不会报错  
b) 检查是否有header（没有header会有报错），`object`填`sampleDict`  
c) sets或者surfaces的输出文件名都可以个性化编辑
2. 重命名sampleDict，是sets放目录$sampleDict/sets，是surfaces放目录$sampleDict/surfaces
3. sample

注：检查不出sample是否有结果？`rm -rf postProcessing ; sample` 这样会比较明确

##### 用python写sampleDict

```python
# file writeSampleDict_2Diagonals.py

import numpy as np
import os

def writeStartEnd(whichLineGroup,f,z,nPoints):
    if whichLineGroup == 0:
        f.write("\t\t"+ "type\t"  + "uniform;" +"\r\n")
        f.write("\t\t"+ "axis\t"  + "distance;" +"\r\n")
        f.write("\t\t"+ "start\t" + "("+z+ " 0.004 0.004);" +"\r\n")
        f.write("\t\t"+ "end\t" + "("+z+ " -0.004 -0.004);" +"\r\n")
        f.write("\t\t"+ "nPoints\t" + str(nPoints) + ";" +"\r\n")
    elif whichLineGroup == 1:
        f.write("\t\t"+ "type\t"  + "uniform;" +"\r\n")
        f.write("\t\t"+ "axis\t"  + "distance;" +"\r\n")
        f.write("\t\t"+ "start\t" + "("+z+ " -0.004 0.004);" +"\r\n")
        f.write("\t\t"+ "end\t" + "("+z+ " 0.004 -0.004);" +"\r\n")
        f.write("\t\t"+ "nPoints\t" + str(nPoints) + ";" +"\r\n")
    else:
        print "big problem"

def writeLines(f,z_value_List,N,Nb_lineGroup,resolution,dictName):
    for i in range(Nb_lineGroup*N):
        whichLineGroup = i // N  # quotient  0, 1 ,2... Nb_lineGroup
        j = i % N                # remainder 0, 1, 2... Ns
        
        f.write("\t"+dictName+'-'+str(i)+"\r\n")
        f.write("\t{\r\n")
        writeStartEnd(whichLineGroup,f,z_value_List[j],resolution)
        f.write("\t}\r\n")
        f.write("\r\n")        
    
def write_LineSet(f,z_value_List,Nb_slices_along_z,Nb_lineGroup,resolution,dictName):
    f.write("sets\r\n")
    f.write("(\r\n")
    writeLines(f,z_value_List,Nb_slices_along_z,Nb_lineGroup,resolution,dictName)
    f.write(");\r\n")

def write_headerSetting_sampleLines(f,fieldName,interpolationScheme):
    f.write("setFormat raw;\r\n")
    f.write("\r\n")
    f.write("interpolationScheme "+interpolationScheme+";\r\n")
    f.write("\r\n")
    f.write("fields\r\n")
    f.write("(\r\n")
    f.write("\t"+fieldName+"\r\n")
    f.write(");\r\n")

def write_sampleLines(dictName,fieldName,interpolationScheme,z_value_List,Nb_lineDisplacement,Nb_lineGroup,resolution):
    f= open(dictName,"w")
    write_headerSetting_sampleLines(f,fieldName,interpolationScheme)
    write_LineSet(f,z_value_List,Nb_lineDisplacement,Nb_lineGroup,resolution,dictName)
    f.close()
    
def NLines2Dict(Nb_lineDisplacement):
    l = np.linspace(-0.08,-0.04,num=Nb_lineDisplacement)
    listStr = [str(x) for x in l]
    print "A list of all line displacements :\n"
    print listStr
    
    dictName="lineOn2Diagonals"
    fieldName="U_mean"
    interpolationScheme="cell"
    write_sampleLines(dictName,fieldName,interpolationScheme,listStr,Nb_lineDisplacement,2,resolution=200)
    
    os.system('dos2unix '+dictName)
    print "\n"
    print "sampleDict written. Think to add a file header!"

NLines2Dict(64)

```

1. configure python script for sampleDict : dictName, fieldName, interpolationScheme... Sampled data will be named as `postProcessing/sets/time/dictName-n_fieldName. "dictName" may also includes information like "interpolationScheme".

2. run python script

3. add header to sampleDict

4. run sample utility

5. verify generated files in `postProcessing`

6. begin plotting using python

7. edit plot parameters : a spatial statistic will require "time", "number of samples" etc. This file is the footprint of the statistics.

8. configure plot script : ajust "dataShape" or likewise. In other word, not all data are eligable because not all data are of the same shape. We can interpolate but I d rather not.

9. run plot script

10. ajust plot and save fig to `figure`

11. write plot data (x,y) to txt file in foler `data` for reporducing use or later ajustement

## python
### laundary
为了防止occigen `$SCRATCH`关于文件数量的quota溢出，用python在后台`nohup`长时间地监控，对`userDefinedLog`里面`removedTimes`与`dataWritingHistory`的补集的时间步做`reconstructPar`和`rm -rf processor*/timeStep`   

1. 保证`userDefinedLog/removedTimes`存在
2. 设置python脚本的时间参数（预估simu时长，留出余量）和processor个数的参数
3. 检查上一个laundary是否结束:`ls -l log.clean_*`看时间
4. 判断`reconstructPar`的进程:`ls processor0 | xargs ls -l > log.ls`；如果有的目录没有顺利reconstructPar会有内容(非std output)输出到终端，至于目录对应上了但场是否完全(比如`U p nu phi`) reconstructPar 只能看`log.ls`
a) reconstructPar有很多时间步积压，手动完成laundary，然后进行b)
b) reconstructPar运行顺利，没有太多时间步积压在`processor*`里，那么准备续算：`rm userDefinedLog/dataWritingHistory`并清空`userDefinedLog/removedTimes`里面的内容
5. `nohup python laundary.py OF301 > log.clean_*`
6. 记录是在哪个登陆节点，需在**对应的节点**用`ps -eaf | grep $USER`才能找到`nohup`的任务

## cluster
### occigen 
#### 算例提交

0.  openfoam env : **一句话**如果用python来做slurm多个连续任务提交，必须在提交前做好这一步(即checkList第`0`步).具体来说如果忘记了提交slurm会在第一个job报错，然后你发现slurm找不到`mpirun`，这时候补上环境变量再重新提交，新的提交第一个job能通过，但第二个job仍然会报错`找不到mpirun`；似乎slurm的默认环境是你登陆某login节点后第一次提交任务的环境，也就是说：login之后得首先做这个事情再提交任何算例(虽然直接使用`sbatch **`倒是不影响，但对通过python来提交的job chain会被迫终断）
1.  nohup python submit.py > log.submit : 提交job，通过`squeue`的返回值**离散地**监控job状态，通过返回值来检验一个算例完成后(这里还有个bug)修改`system/controlDict`里面`startTime`和`endTime`用于自动提交下一个算例(限制：simuLog不停地被改写，应该把每个job的log都留下来才好)；log.submit取好名字以方便查看
2.  nohup python watchDog.py > log.watchDog : 设置一个运行最大时长，在这个时间内用`reconstructPar`来保留计算输出数据，删除`processor*`里面已经`reconstructPar`完成地时间步大幅度削减文件个数，保证不超过scratch的限额；log取好名字以方便查看
G3.  笔记本上面记录下来是哪个login，什么算例，连续提交多少个job，每个job预计计算的物理的间隔是多少(用于更新`startTime`和`endTime`)：因为`ps -eaf | grep $USER`只能找出相应登陆节点上面的job
4.  检查`log.submit`，查看jobs的情况

#### 数据同步

千万千万要注意`para0`，在`dir`里面并没有用到，一定要double check !!!!
```bash
#!/bin/bash

para0=inlet_0p3
para1=a_0p08
para2=setT_St_1
log=sync_log.$para1
dir=/store/lmfa/fct/hluo/occigen/caseByGeometry/T/shape_square/2a_3_T/BirdCarreau/inlet_0p3/$para1rsync -av occigen:/scratch/cnt0028/mfa0464/hluo/caseByGeometry/T/shape_square/2a_3_T/BirdCarreau/$para0/$para1/$para2/* DATA --exclude processor* &&
echo "DATA sync BirdCarreau : $para0 $para1 $para2 ended with success" >> $dir/$log
```
### newton
1. No `#SBATCH --exclusive` (if not necessary)
2. Naming of the file with exactly 8 chars with `-2` indicates for example 2nd run. Ex : `p10D_gP5-2`
3. consider remove `#SBATCH --mem-per-cpu=4000` because there are nodes which bigger memory available

#### error
1. `error while loading shared libraries: libpsm_infinipath.so.1: cannot open shared object file: No such file or directory` : check `#SBATCH --mem`, this is an error saying you are maybe asking for more memory than the machine can offer
2. `slurmstepd: error: Detected 1 oom-kill event(s) in step...` : check ``#SBATCH --mem`, this is an errory saying that the memory you are asking for is not enough for the program

#### old note

```bash
#!/bin/bash
# FILE : p10D_gP5-2

#BATCH --job-name=MeshConvergence                    # Is this even useful ? SABTCH no?
#SBATCH --output=job.%j.out
#SBATCH --error=job.%j.err 
#SBATCH --mail-user=haining.luo@doctorant.ec-lyon.fr
#SBATCH --mail-type=ALL
#
#SBATCH --partition=mononode
#SBATCH --mem-per-cpu=4000
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --cpus-per-task=1
#SBATCH -t 1-00:00:00

# number of simulation
nos=2
logFile=logSimulation$nos

# 在newton上module purge load部分可以保留
module purge
# Opt/Debug mode on
module load OpenMPI/1.6.5-GCC-4.8.3 ; \
. /home/lmfa/hluo/LocalSoftware/OpenFOAM/OpenFOAM-2.3.x/etc/bashrc.Opt

echo "NodeName = " $SLURMD_NODENAME >> $logFile
echo "JobID = " $SLURM_JOB_ID >> $logFile

mpirun -np 16 pimpleFoam -parallel >> $logFile
```

3. run mapFields need a big memory. Theres now a template for big-memory job. Especially, dont put `&` at the end of commands or mapFields will only last for 2s and theres no error message returned so could be confusing.

```bash
#!/bin/bash
#
##SBATCH --job-name=T-1b_mirrorMerge_mapped
#SBATCH --output=job.%j.out
#SBATCH --error=job.%j.err 
#SBATCH --mail-user=haining.luo@doctorant.ec-lyon.fr
#SBATCH --mail-type=ALL
#
#SBATCH --partition=test
#SBATCH --mem=32000                  # asking for 32000Mb ~ 30Gb of memory 
#SBATCH --nodes=1                    # 1 node
#SBATCH --ntasks-per-node=1          # one task for one node
##SBATCH --cpus-per-task=1           # one cpu per task
##SBATCH -t 1:00:00

# number of simulation
nos=1
logFile=logSimulation$nos

module purge
# debug mode on
module load OpenMPI/1.6.5-GCC-4.8.3 ; \
. /home/lmfa/hluo/LocalSoftware/OpenFOAM/OpenFOAM-2.3.x/etc/bashrc.Opt

# run solver 
#decomposePar  # Do this manually

echo "NodeName = " $SLURMD_NODENAME >> $logFile
echo "JobID = " $SLURM_JOB_ID >> $logFile

#mpirun -np 64 icoFoam -parallel >> $logFile

sourcedir=/home/lmfa/hluo/LocalSoftware/OpenFOAM/hluo-2.3.x/run/1b_mirrorMerge_mapped_NearestFace
mapFields $sourcedir -case . -noFunctionObjects -fields '(U p)' -sourceTime '6.01' -targetRegion region0 > logMapFields_new 2>&1  # No "&" here
```

Here maybe a reason that zaurak well perform likely 2 times better than cpus on newton when the constraint on memory is not there.

```bash
hluo@zaurak $ cat /proc/cpuinfo | grep model
#...
model name	: Intel(R) Xeon(R) CPU E5-1620 v4 @ 3.50GHz

hluo@compil-haswell $ cat /proc/cpuinfo | grep model
#...
model name	: Intel(R) Xeon(R) CPU E5-2660 v3 @ 2.60GHz

hluo@visu $ cat /proc/cpuinfo | grep model
#...
model name	: Intel(R) Xeon(R) CPU E5-2640 v4 @ 2.40GHz
```

## Inkscape 
### 导出

- [ ] draw sketch
- [x] Edit-> Resize page to selection
- [ ] Export PNG image -> Export as 
### clip
1. import image
2. F4 (select "rectangular tool") draw a rectangular (marked by red)
3. F1 (select "select tool"), ctrl+A select all
4. Object -> clip -> set

## gimp
这个软件可以得到图片里面的坐标

## paraview
### export screen shot
export 有什么好说的？嗯......如果要对一系列算例做同样的图一般会选用`load state`来加载`*.pvsm`，似乎注意改一下`*.foam`对应的`path`就好，但是如果要让export screen shot输出同样像素的图的话，记得一定要全屏之后再export！！

### do not skip time 0
1. paraview data.foam
2. remain defaut "Skip Zero Time" then apply : geometry will be visualized but no "Cell Array" (**field data**) is there. Make sense.
3. uncheck "Skip Zero Time" -> apply : cell array will then appear

### color map
- 如果是对成的数据`[-a,a]`，用红白蓝`diverging`挺好，能分辨出`0`对应白
- 如果是`[0,a]`，用黑白灰最好，但paraview好像默认可以从`Edit Color Map`选项卡中带桃心的小按钮`Choose Preset`里面有`X-ray`和`GrayScale`，选择后apply（默认就会变到RGB color Space）；如果想要恢复diverging，最下面有个恢复默认按钮；如果想要自定义，参见[RBG自定义](https://www.cfd-online.com/Forums/openfoam-paraview/105630-paraview-gray-scale-white-black.html)

### 用load state来复现camera视角   
target 目标视角：想要复制的视角，对应的case叫目标case   
working 工作视角：想要在工作case下复现目标视角   
0. 目标视角的存储通过目标case(Visu)里`save state`来实现(默认读取了一个绝对路径的但其实为空白的`target.foam`文件)，得到`target.pvsm`
1. 复制目标case下的`target.pvsm`到工作case(Visu)里，编辑查找关键字`target.foam`并替换成`工作路径/working.foam`
2. 工作路径下创建`working.foam`
3. 工作路径下打开paraview
4. load state 选择编辑后的`target.pvsm`-> "Load State Data File Options" 选`Use File Names From State`
5. 等待复现

注意：
1. 还涉及一个working case里面时间步是不是和state里面一致的事情，我的测试刚好target和working case有相对应的同一时刻的data
2. 在paraview-5.4.1测试成功

### 根据x坐标截取一个cell slice的label
cell slice : 这里的网格是正交的六面体，所以严格意义上一个x坐标对应“一层网格”，通过在paraview里面`find Data`中选取`xcoord between xx and xx`就可以选定这一系列cell   
label      : 一系列的cell的label合起来就是个labelList，而且是全局的，因此可以取出`U`中例如`U[labelList_slice0]`子集进行操作   

下为流程：
0. paraview 可视化 case
1. calculator : xcoord (defaut : point data)
2. Filters -> Point data to cell data 这里我们只想对xcoord操作 但**注意** 这一步会将U(如果有的话)也插值一遍，值会跟先前有略微差别
3. Edit -> fidn Data -> (cells) (Pointdatatocelldata) (criteria : xcoord between [xmin, xmax] -> run selection
4. close "find Data" (selection is still effective)
5. split horozontal (very smalll icon on the right above render view) -> spread sheet -> show only selected elements -> attribute : cells **注意**一定要选`cells`要不然得到的spread sheet行数不是cell number -> check 行数是否等于画网格时定下的个数
6. Toggle cell visibility (eliminate other colons : left with only labelIDs) -> export spread sheet *.csv -> optional depending on paraview version (filter colons by visibility)
7. modify *.csv : add OpenFOAM header (shown below), add `(` and `)` for list
8. use OpenFOAM to read modified file via class `IOList<label>`

```cpp
// example of OpenFOAM header for file named "slice16"
$ head -10 slice16
FoamFile
{
    version     2.0;
    format      ascii;
    class       labelList; // IMPORTANT
    object      slice0;    // not important
}

16900
(
/*
 ...
*/
)
```

### scripting

#### Base
利用`Tools->Start Trace`选上`all properties` `Fully Trace Supplemental Proxies` `Show incremental Trace`   
0. Open -> some `*.foam` file (no need for the real file when running script afterwards, but necessary here for GUI use, to generate the script to run) 
1. clip... slice...
2. export as
3. Tools -> End trace
4. `pvpython script.py`

#### Note
1. 值得注意的是legend，利用以上python trace的结果不加改动的情况下，会默认变得巨大，所以得加上
2. 加上auto-rescale是个必须得有的好习惯
3. 以上1和2的改动放在`renderView1`之前，经测试是有效的

```python
"""
some operations : read some certain array
"""
# Properties modified on t_meanLUTColorBar    调整legend nu_mean** 需要根据实际object名字替换
nu_meanLUTColorBar.TitleFontSize = 4
nu_meanLUTColorBar.LabelFontSize = 4
nu_meanLUTColorBar.ScalarBarThickness = 5
nu_meanLUTColorBar.ScalarBarLength = 0.5

# rescale color and/or opacity maps used to exactly fit the current data range  等同于auto-rescale
slice1Display.RescaleTransferFunctionToDataRange(False, True)

# current camera placement for renderView1    
renderView1.CameraPosition = [0.013999999035149813, -0.08839503035519257, 0.0]
renderView1.CameraFocalPoint = [0.013999999035149813, -0.0020000000949949026, 0.0]
renderView1.CameraViewUp = [0.0, 0.0, 1.0]
renderView1.CameraParallelScale = 0.022360679233547745

# save screenshot
```
4. customize range, logscale, use another color mapping `它们之间`按照以下顺序排列是有效的

```python
# Rescale transfer function
nu_meanPWF.RescaleTransferFunction(2e-06, 0.0003)

# convert to log space
nu_meanLUT.MapControlPointsToLogSpace()

# Properties modified on nu_meanLUT
nu_meanLUT.UseLogScale = 1

# Apply a preset using its name. Note this may not work as expected when presets have duplicate names.
nu_meanLUT.ApplyPreset('Yellow - Gray - Blue', True)

```

5. 如果有多个不同位置的slice，那么一定要记得加上下面这一行(在save screenshot前就行)以保证图"有效部分"的大小不会改变

```python
# reset view to fit data
renderView1.ResetCamera()

# save screenshot
```

#### possible bug
```python
raise ValueError("%s is not a valid value for attribute %s." % (value, name))
ValueError:  is not a valid value for attribute ScaleArrayComponent.
```
可能是最初没有读入后面要用到的数组，也有可能后面取slice什么的超出了计算域，感觉报的是个找不到`目标数据`的错

## svn

### branching
video reference [createBranch](https://www.youtube.com/watch?v=Y9enCuIhwY8),[workWithBranches](https://www.youtube.com/watch?v=1LS-jHQbRXY),[resolvingConflicts](https://www.youtube.com/watch?v=hubWjFgnjvI),[svn_resolve_tree_conflict_in_merge](https://stackoverflow.com/questions/19451800/svn-resolve-tree-conflict-in-merge)

```bash
# Note a good pratice : make sure trunk is updated to the lastest version
# create branch by copying trunk. 
$ svn copy https://subversion.renater.fr/jnnf/trunk https://subversion.renater.fr/jnnf/branches/haining -m 'create branch haining'

# confirm existance of branch/haining in svn server
$ svn list https://subversion.renater.fr/jnnf/branches/ 

# get branch data from svn server
[branches] $ svn update # better than : svn checkout https://subversion.renater.fr/jnnf/branches/haining. This will give problems when doing svn delete at the end of checkList

# make changes on branch/haining, running tests
# if new file added : svn add
[haining] $ svn commit -m 'information on modification'

# make sure branch itself is updated 
[haining] $ svn update 

# now we are going to push changes to trunk. BUT
# firstly we have to merge from trunk to this branch (necessary step):
# possibly we may get some conflicts for THIS merge.
[haining] $ svn merge ^/trunk

# verify merge by running tests
# commit merge
[haining] $ svn commit -m 'merge from trunk to branch/haining is sucessful. ready to reintegrate into trunk'

# go to trunk
[trunk] $ svn update

# now re-integrate from branch/haining to trunk then managing possible conflicts (usually choose postpone and vim)
[trunk] $ svn merge --reintegrate ^/branches/haining

# verify again changes by running tests
# If all is well, we commit
[trunk] $ svn commit -m 'merged trunk from branches/haining'

# branches/haining is now useless
# thus remove them from version control meaning on the svn server. 
[branches] $ svn rm haining 
[branches] $ svn commit 'removing branches/haining'

# wont be able to see 'haining' anymore
$ svn list https://subversion.renater.fr/jnnf/branches/ 

# checking status :
[branches] $ svn status
?       haining        # ? means no longer in version control
```

### resolving conflict

`!M` : `svn rm file --force` (if it is `rm *` not by `svn remove`)


### svn和git之间相互转换
**一定要非常注意！**   
1. `cp -r`拷贝整个文件夹到目的地文件夹   
2. 如果是svn拷贝到git，一定要删掉`.svn` ; 如果反之，一定记得删掉 `.git` (.gitignore倒是无所谓啦)   
3. 然后才**开始**进行其他操作
