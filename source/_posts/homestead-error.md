---
title: "homestead 错误处理集合"
categories:
- php
tags:
- php
- homestead
date: 2022-11-10
---
# 1、启动时报错 Stderr: VBoxManage.exe: error: Not in a hypervisor partition (HVP=0) (VERR_NEM_NOT_AVAILABLE).
解决步骤:

1、先检查虚拟化是否启用，如果已经启用则该解决方案不适用。
> 打开任务管理器，选择性能选项则可以看到虚拟化是否启用

2、确认好虚拟化未启用后重启电脑进入bios开启虚拟化，不同的品牌的主板进入bios和开启虚拟化的方法请自行百度
> 铭瑄（maxsun）：https://www.jb51.net/hardware/MotherBoard/816200.html

3、开启虚拟化之后重启电脑，再执行homestead up就可以了