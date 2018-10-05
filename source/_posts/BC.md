---
title: 边界条件
date: 2018-10-05 17:40:31
tags:
---

# fixedValueFvPatchVectorField
Dirichlet边界(可以随时间变化)，通常继承自fixedValueFvPatchVectorField.编写边界条件的时候要注意：

1. data member 申明(`*.H`)和初始化(`*.C`)的顺序要一致
2. 一定要要让所有constructor里面data member都完成初始化(`*.C`)
3. `*.C`里面的`write`也要包含所有必要的data member，因为在`decomposePar`时写入`processor*`里边界条件用的就是这个`write`
4. `*.C`里面的`makePatchTypeField`似乎是个宏函数，一定要修改成`makePatchTypeField(fvPatchVectorField, SameAsClassName);`
5. 可以写一些non-member function用来做简单的计算

**窍门**   
所在边界的名字：`patch().name()`   
当前时间步的时间参数：`this->db().time().timeOutputValue()`.在这个类`runTime`不可见
