---
title: Q&A
date: 2018-10-06 02:44:18
tags:
---

# 这里显然是自问自答区

## 如何做出成体系好debug的后处理tool chain？
答：先用bash把我的user*系列utility连起来，通过文件夹好好组织；后续的python后处理一定要主程序条理清楚，加上合适的输出，去掉冗余输出，有exception control；一个类别的后处理，尽量用同一个python module去实现通用。往具体的说：先做好牛顿流动的case，在一个个case中成长，在痛苦中净化和进化。


## 如何组织DNS数据？
答：
```bash

  |- raw data      # 原始数据（用于备份） + 文件夹良好组织的小型后处理文件
  |- visu data     # 特征时间步的数据（可能比raws小不了多少），用于生成大型的后处理文件(场），paraview save state 等等
  |- image         # 把图片分开放
  |- pythonScript  # python 都放这里

```

## 海量数据的存放？
答：    
zaurak :   
+++ 固态硬盘IO速度快效率高，包括查看文件大小也是速度奇快   
`/store/`   
--  容积小[2T]   
`/home/`   
--  容积小[170G]

srv-data-1 :   
没有空间了，基本弃用

visu/newton :   
++ 容积大[4.8T / 66T]   
++ 用于读写各种操作的内存很足，当zaurak内存不够用则考虑开始启用   
-- 硬盘IO太慢
`/store/lmfa/fct/`   
++ 硬盘存储空间足   
++ 跟zaurak之间传输速度（待估计）比较快   
`$HOME`   
++  有备份，可放重要数据
--  空间太小[1T / 2T]

occigen :   
-- `$STORE`[1.5T]只支持`*.tar.gz`，只能暂时存储   
-- `$SCRATCH`[4T && fileNb 600000]   
   `$HOME`基本不用


## 推公式哪本书好？
答：   
Turbulence en mecanique des fluides - P. Chassaing 这本里面的公式灰常好，还带有很多物理解释


## 远程可以进行的操作
1. 提交算例
2. zaurak, occigen, newton上都有paraview，可以用script来做一些简单的可视化
