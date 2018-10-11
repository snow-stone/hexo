---
title: 几个类
date: 2018-10-09 22:41:48
tags:
---

# VectorSpace, Vector, Tensor

VectorSpace 显然是这里最具有一般性最底层的模板类了，Vector和Tensor在它眼里仅是3个元素和9个元素的差别，求`mag`都一视同仁。

`mag`在VectorSpace/Vector里面就是所有元素平方和`magSqr`然后开平方，在Tensor变成9个元素平方和然后开平方。因此一般的Tensor不同重写`magSqr`，SymmTensor因为其形式特别就重写了`magSqr`，如此替换掉最一般的`magSqr`,对SymmTensor仍然可以进行`mag`操作，为什么呢？因为SymmTensor是Tensor是VectorSpace里面的元素，因此一定就有`mag`，此时母类`mag`调用的是SymmTensor版本的`magSqr`。
