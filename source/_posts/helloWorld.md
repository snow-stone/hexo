---
title: helloWorld
date: 2018-07-04 21:20:12
tags:
---

# Hello world

在OpenFOAM的使用中，通常的hello world版本用的是`fvCFD.H`这个头文件,但实在包含了太多的内容
```cpp
#include "fvCFD.H"

int main( int argc, char *argv[] ) {
    Info << "Hello FOAM !\n";
    return 0;
}
```
其实`messageStream.H`就够了:
```cpp
#include "messageStream.H" // 包含了"IOstream.H"等declaration和Info的定义

int main( int argc, char *argv[] ) {
    Foam::Info << "Hello FOAM!\n"; // 注意一定要被namespace-qualified "Foam::"
    return 0;
}
```

# std::cout 和 Foam::Info 混用

如果想要加入`std`的标准输出，不再需要`#include <iostream>`:
```cpp
//#include <iostream>   // it seems std is included now through messageStream.H
                        // my guess is through forward declaration of class IOstream of OpenFOAM
						// IOstream.H 里面有 using std::cin; using std::cout; using std::cerr;
#include "messageStream.H"

int main(int argc, char * argv[]){
    int i;
    for(i = 0; i < argc; i++){
        std::cout << "Argument "<< i << " = " << argv[i] << std::endl;
    }

    for(i = 0; i < argc; i++){
        Foam::Info << "Argument "<< i << " = " << argv[i] << Foam::endl;
    }
    return 0;
}
```
这里的main函数可以传入任意个参数，程序输出结果
```bash
$ ./a.out 01 10 haha
Argument 0 = ./argumentToMain_OF
Argument 1 = 01
Argument 2 = 10
Argument 3 = haha
Argument 0 = ./argumentToMain_OF   // 输出结果很一致，都没有引号
Argument 1 = 01
Argument 2 = 10
Argument 3 = haha
```
## 引入argList类

如果改用OpenFOAM中常用的构造`args`的思路：
```cpp
#include "messageStream.H"
#include "argList.H"

int main(int argc, char * argv[]){
    Foam::argList args(argc, argv);

    int i;
    for(i = 0; i < argc; i++){
        std::cout << "Argument "<< i << " = " << args[i] << std::endl;  // access by args
    }

    for(i = 0; i < argc; i++){
        Foam::Info << "Argument "<< i << " = " << args[i] << Foam::endl;
    }
    return 0;
}
```
这个main函数不再能够接收任意个参数，实际上一个也不行，由于默认参数是可执行文件目录：
```bash
$ ./a.out
Argument 0 = ./argumentToMain_OF   // 此处没有引号，为cpp标准库里的输出
Argument 0 = "./argumentToMain_OF" // 为何多了引号？OpenFOAM封装后的输出
```
如果试图传1个参数会遇到
```bash
--> FOAM FATAL ERROR: 
Wrong number of arguments, expected 0 found 1
```
这是因为`args(argc, argv)`在构造函数的末尾有函数`parse调用了check(checkArgs, checkOpts)`对传入参数个数有检查，因此多了少了都是`FatalError.exit()，main函数（或者更准确地说args）能接收的参数个数由argList类静态成员validArgs来确定，具体实现如下：

```cpp
#include "messageStream.H"
#include "argList.H"

int main(int argc, char * argv[]){
	Foam::argList::validArgs.append("someFloat"); // 此处名字可以乱起，因为后文想要转换类型为float，所以起名为someFloat
    Foam::argList args(argc, argv);

    int i;
    for(i = 0; i < argc; i++){
        std::cout << "Argument "<< i << " = " << args[i] << std::endl;  // access by args
    }

    for(i = 0; i < argc; i++){
        Foam::Info << "Argument "<< i << " = " << args[i] << Foam::endl;
    }

	Foam::Info << "Argument "<< 1 << " = " << atof(args[1].c_str()) << Foam::endl;

    return 0;
}
```
也就是在构造args之前把函数能接收参数个数的上限+1（默认为1，现在args这个container长度为2）,所以如果要再增加参数个数继续`append`就好。   
来看执行结果：
```bash
$ ./a.out 01
Argument 0 = ./a.out
Argument 1 = 01
Argument 0 = "./a.out"
Argument 1 = "01"
Argument 1 = 1          # 看来输出float是不带引号的
```

# 小结
1. 如果只考虑OpenFOAM封装的标准输出`messageStream.H`即可   
2. 要向OpenFOAM常用程序中传入参数需要用到`Foam::argList::validArgs.append`

# 问题
为什么有时输出有引号而有时却没有引号？
类型不同，输出的封装不同
