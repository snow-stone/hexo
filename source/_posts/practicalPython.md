---
title: 实用python
date: 2018-09-25 15:51:01
tags:
---

# iterable
`iterable`顾名思义就是“能堪循环者”，python里面循环可以有简单写法`for item in some_list:`，这样可以省去指明循环指标变量`i`。不专业的（我）使用的时候有时候还是想要加上`i`，但似乎不是所有的`iterable`都可以加`i`，例如`f = open(fileName,'r')`这里返回值`f`是`type file`，它就不能用`i`来引用，想要对它循环输出行号，我没有成功.[待贴代码验证]

当iterable在循环中被改变(比如`list.remove()`)，要**特别注意**！！可能会直接跳出循环，因为iterable被改变了

# str are immutable
python里面`str`可以slice，但不能被修改，`string.replace`也只是对其拷贝进行的`replace`操作.[参考stackoverflow](https://stackoverflow.com/questions/46850850/python-function-to-modify-string)
