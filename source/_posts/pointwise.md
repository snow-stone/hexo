---
title: pointwise
date: 2018-10-22 22:21:21
tags:
---

# 语境
Pointwise里面网格为domain，OpenFOAM里面网格为patch

# 导出为openfoam格式网格并配置周期条件cyclic

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
**注**：为了保证无误，新旧1和新旧2的名字没有取一样；想改成新旧一样？在这个流程的最后把constant/polyMesh/boundary里面改过来就好   
注：createPatch在这里是由原先pointwise的twin domains/patches由类型`patch`变成了类型`cyclic`，据utility的功能介绍还可以将patches变成一个patch，或者将faceSet变一个patch
