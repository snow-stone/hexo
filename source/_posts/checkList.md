---
title: checkList
date: 2018-07-26 09:23:58
tags:
---

# 检查表
比较深入认识到“检查表”(check-list)是源于傅盛的文章，他提到让工作变得可靠高效，检查表是个保障时效的好东西，看似机械但可靠性好。随后是在《黑匣子思维》里面读到，了解到纽约哈德孙河上双发停车机长在短时间内完成决策，选择并顺利完成迫降，这个成功（美当局还是对机组进行了详细地调查）里面当然有无数次飞行的经验，过硬的心理素质，还有副机长分担责任在机长思考的同时协力沟通，副机长负责把空客给的发动机停车的check-list走完，当然最后时刻还有空客津津乐道的自动调整飞行姿态优化迫降角度的系统的功劳。我想，检查表制度已经完全融入了这些航空工业优秀从业者的工作中，如作者所说：人的因素可能是正面的也可能是负面的，然而可靠性的保证，需要仰仗人训练自己“黑匣子”一般的思维。回到傅盛，自己定下的check-list就一定要**严格执行**，要不然你将看不到当前check-list的缺陷，改进也就无从谈起，低级错误/已知错误还是会犯。check-list的存在意义在于，当你头脑不是特别清醒，状态不好的时候，也能给你一个完整严谨的流程，最大程度规避已知的一系列“低级错误”，不单纯依靠记忆和头脑，面对复杂系统尤为需要如此作业。

同fluent或其他CFD软件，OpenFOAM算得上复杂系统，或者说容易犯错的系统：不像fluent一样，没有按钮，从边界条件、初始条件、网格文件、流体物性、离散格式、求解器（U求解器，p求解器）还有utility到用到的例如`sampleDict`，`changeDictionaryDict`，`refineMeshDict`，`topoSetDict`等，这些都是一个算例需要用到的基本文本文件。文本文件不注意格式或者打错字很常见，加上OpenFOAM给的范本参照并不是MVP（Minimum Value Product），也就是说通常都有**不必要**的部分，而使用者并不清楚哪些信息必要，哪些重要，哪些完全没用。总之细节很多，细节一旦繁多就难有大局观，check-list就是提醒自己“在哪里”，“还要去向哪里”，“已知的可能会出现的问题”，是高于平时“工作的自己”那一个层次的思维方式。工作的时候我要debug，要进入多层嵌套的代码，是细节的把控，而check-list让我跳出来，让我看到自己在地图的哪个地方，接下来可能的问题是什么，问自己是主要矛盾吗，这是做决策的思维层次。

总结一下：相较于fluent，OpenFOAM的劣势是文本文件之间的自然逻辑没有GUI来完成，优势是对一个功能如果充分了解，就可以有很大自由度进行操作（可以按照上面说到的check-list来完成步骤，实现步骤的“自动化”）,因而劣势可以弥补，当然OpenFOAM最大的优势还是完全开源。

## Inkscape


draw sketch
Edit-> Resize page to selection
Export PNG image -> Export as (不要覆盖原有png)



## Pointwise


 Select blocks
 Export CAE
 Rescale to meter : transformPoints -scale ’(0.001 0.001 0.001)’
 checkMesh
 serial run to check



## General Setting up


mesh (transformPoints -scale '(0.001 0.001 0.001)', checkMesh[need a time dir (empty is OK)])
initial condition (mapFields) and BC (changeDictionary)
topoSet
system/controlDict : if the startTime corresponds to initial time dir
check serial run : if not passed, turn back
decomposePar : need to verify if the time dir is well decomposed
job submission file : check node, cpu, time(estimation), np, nos
verify logFile. Don't overwrite. Make sure system/controlDict* correspond to the nos

run on test if possible
run simu



## inletBC mapped


Generate mesh and checkMesh
Modify constant/polyMesh/boundary
Modify BCs in startTime/U 



## mapFields


if not consistent, make sure the mapFieldsDict(target case) is good.
In target case, verify startTime==109.9 of target case before doing mapFields $source -case $target -noFunctionObjects -fields '(U p)' -sourceTime '109.9'

if finished successfully (=verify if in fields whether there's -nan) [other possible error can be existence of empty patch or in source case constant/polyMesh/boundary there are mappedPatch if in the target case there's no corresponding mappedPatch in constant/polyMesh/bounday. mapFields could(not 100% sure) throw error at the end of execution and writing U is then ommited->map failed]
change the calculated type : some boundaryField will be calculated which is not good for continous calculation. Proceed with changeDictionary -instance '109.9'

check serial run



## reconstructPar

如果之前做过 reconstructPar -fields '(U p)'，现在用reconstructPar -fields '(phi)'不会将之前的目录删掉，是添加phi到对应的时间目录里面.


## sample


sampleDict (may need python script to write)
if needed convertToCylindrical -fields '(U)' -time '109.9'

sample -dict system/sampleDictName -time '109.9'



## sampleDict


make sure inside fields you have the RIGHT name. (OF won't tell you anything about existence of the fields)
make sure you have the header and inside header you have object sampleDict;

sampling sets or surfaces, output file name of sets or surfaces can both be personalized
name sets or surfaces as lines_complement_typeUniform_cell-155 : -155 is better for recognizing the number. lines_complement_typeUniform_cell works as an input for postProcessing python script to make sampleDict sample result and postProcessing python script consistent.
Not sure of sample results? rm -rf postProcessing and see if there is any ouput.
