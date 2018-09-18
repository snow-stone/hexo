---
title: 检查表
date: 2018-07-26 09:23:58
tags:
---

# 检查表制度
比较深入认识到“检查表”(check-list)是源于[傅盛](https://mp.weixin.qq.com/s?__biz=MjM5NjgzMzkwMQ==&mid=2653646406&idx=1&sn=9fc89b68138045f882a1f5dd04afbef4&chksm=bd3cf1448a4b78527c0ba97e914fcbad89d429696d61fb087b18a87a266ef9cd076df86c1428&mpshare=1&scene=1&srcid=0728p02JYofiPhU6NCYy4tnr#rd)的文章，他提到让工作变得可靠高效，检查表是个保障时效的好东西，看似机械但是可靠。随后是在《黑匣子思维》里面读到，进一步了解到纽约哈德孙河上迫降成功的背后（美当局还是对机组进行了详细地调查）起到积极作用的有无数次飞行的经验，过硬的心理素质，还有副机长分担责任在机长思考的同时协力沟通，副机长在短时间内把空客给的发动机停车的check-list走完。检查表制度已经完全融入了这些航空工业优秀从业者的工作中，如作者所说：人的因素可能是正面的也可能是负面的，然而可靠性的保证，需要仰仗人训练自己“黑匣子”一般的思维。回到傅盛，自己定下的check-list就一定要**严格执行**，要不然你将看不到当前check-list的缺陷，改进也就无从谈起，低级错误/已知错误还是会犯。check-list的存在意义在于，当你头脑不是特别清醒，状态不好的时候，也能给你一个完整严谨的流程，最大程度规避已知的一系列“低级错误”，不单纯依靠记忆和头脑，复杂系统尤为需要如此作业。

同fluent或其他CFD软件，OpenFOAM算得上复杂系统，初学者容易犯错的系统：不像fluent通过按钮和已经编程的GUI来构建组织求解一个问题的逻辑链，OpenFOAM从边界条件、初始条件、网格文件、流体物性、离散格式、求解器（U求解器，p求解器）还有utility到用到的例如`sampleDict`，`changeDictionaryDict`，`refineMeshDict`，`topoSetDict`等，里面的逻辑关系通过一系列的文本文件来完成。信息的碎片化很可怕，想想你用vim去打开一个个文本文件来检查到底是哪里出了错，这样的过程必然是艰辛的。

为何容易犯错？
1. 文本文件有特定格式打错字也很常见
2. OpenFOAM在tutorial里面给的范本参照并不是MVP（[Minimum Viable Product](https://book.douban.com/subject/27077719/)），也就是说通常都有**不必要**的部分，初学者者并不清楚哪些信息必要，哪些重要，哪些完全没用。

总之细节很多，细节一旦繁多而又难以从中抽离就难有大局观。而check-list就是提醒自己“从哪里出发”，“现在在哪里”，“还要去往哪里”。举例：像飞机起飞的check-list一样，一个check-list可以是对于某算例从无到有的按步骤一步步的配置。“从哪里出发”包括最基本的`0`，`constant`和`system`的文本文件结构，包括边界条件、初始条件，那么“项目的出发点”就是自己剔除上面提到的**不必要**部分后得到的MVP case，基于MVP case的基础可以做加密网格或者其他的变式。顾名思义，“现在在哪里”是check-list当前步骤，“还要去往哪里”取决于自己的目标。

记录“已知在某个步骤上会出现的问题”，然后据此设计check-list来规避已知问题，这是高于平时“工作的自己”层次的思维方式。工作的时候我要debug，要进入多层嵌套的代码，是细节的把控，而check-list让我跳出来，让我看到自己在地图的哪个地方，在某某步骤上的经验和教训是什么，接下来可能的问题是什么，问自己是主要矛盾吗，时间成本会是多少，应该如何决策，这是**做决策**的思维层次([原则](https://book.douban.com/subject/27608239/))。

总结一下：相较于fluent，OpenFOAM的劣势是文本文件之间的自然逻辑没有GUI来完成，优势是对一个功能如果充分了解，就可以有很大自由度进行操作（可以按照上面说到的check-list来完成步骤，实现步骤的“自动化”）,因而劣势可以弥补；当然OpenFOAM最大的优势还是完全开源，只有不懂的地方才是黑箱，经手的所有步骤都在掌握，所有的变量都在掌握。

要知道，检查表可繁可简，为节约时间成本阿波罗13号出事之后检查表都被大规模简化，也就是“跳步”，而明白跳步的风险是简化的必须。所以检查表到底应该多繁多简，视情况而定。我根据自己的经验制作了一系列检查表，有的是配置算例流程，有的是关于某一个utility，还有NotToDo-list，总之对我怎么有用怎么来，还不止OpenFOAM，因为尤其跟CAE相关的软件都很大型，按钮特别多，记忆是不可靠的。加上如果在不同的计算机群上面计算，队列是不一样的，对用户的要求是不一样的（比如occigen上面对文件个数要求特别苛刻），用户的需求是不一样的（比如考虑到priority的问题，我又时候可能会选择提交一个计算时间较短的算例），情形很多，未必都需要检查表化，但按照自己的逻辑记录下要点，比起每次决策的时候再去网站上找零散的信息来得更高效。

## Inkscape export

- [ ] draw sketch
- [x] Edit-> Resize page to selection
- [ ] Export PNG image -> Export as 

## Pointwise export

- [x] Select blocks
- [ ] Export CAE
- [x] Rescale to meter : transformPoints -scale ’(0.001 0.001 0.001)’
- [ ] checkMesh
- [ ] serial run to check

## Complete Setting up for OpenFOAM

env                : purge modules ; set Foam environment
mesh               : transformPoints -scale '(0.001 0.001 0.001)', checkMesh[need a time dir (empty dir is OK)]
initial condition (mapFields) and BC (changeDictionary)
topoSet
system/controlDict : if the startTime corresponds to initial time dir
serial run check
decomposePar       : need to verify if the time dir is well decomposed
job slurm file     : name it right ; check node, cpu, time(estimation) ; variables : np, nos ; (仅occigen)为避免有相关输出到`*_mpi.%j.err`前面已经做过这里就不再需要module load
logFile            : Dont overwrite. Make sure system/controlDict* correspond to the nos
run on test if possible
run simu
if any monitoring script, launch

## inletBC mapped from class "mappedFixedValueFvPatchField"

Generate mesh and checkMesh
Modify constant/polyMesh/boundary
Modify BCs in startTime/U (如果startTime 非空)


## mapFields

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

## reconstructPar

在reconstructPar -fields '(U p)'之后，reconstructPar -fields '(phi)'会将phi添加到对应的时间目录里面.


## sample

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
