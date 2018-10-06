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
-   容积小[2TB]   
`/home/`   
--  容积小[170G]

visu/newton :   
++ 用于读写各种操作的内存很足，当zaurak内存不够用则考虑开始启用   
-- 硬盘IO太慢
`$STORE`   
++ 硬盘存储空间足   
++ 跟zaurak之间传输速度（待估计）比较快   
`$HOME`   
-  空间大小[2TB]

occigen :   
-- `$STORE`[1.5TB]只支持`*.tar.gz`，只能暂时存储   
-- `$SCRATCH`[4TB && fileNb 600000]   
   `$HOME`基本不用


## 推公式哪本书好？
答：   
Turbulence en mecanique des fluides - P. Chassaing 这本里面的公式灰常好，还带有很多物理解释
