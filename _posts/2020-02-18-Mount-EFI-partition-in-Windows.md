---
layout: post
title: "Windows下挂载与卸载efi分区"
subtitle: ''
date: 2020-02-18 00:17:00
author: "突击核"
header-style: text
hidden: false
tags:
  - Windows
  - 黑苹果
  - Hackintosh
  - Diskpart
---
#Windows下挂载与卸载efi分区
调试黑苹果的EFI时翻车时无法使用四叶草恢复配置，这时常常会需要在Windows下挂在EFI分区进行恢复。
##使用管理员运行命令行工具
![](https://gitee.com/Astral/img/raw/master/blog/efi-1.png)
##使用Diskpart工具挂载efi分区
1. `list disk` 列出系统中拥有的磁盘
2. `select disk 0` 根据所要挂载的分区所在的磁盘输入盘符，请根据实际情况选择
3. `list partition`列出所选磁盘拥有的分区
4. `select partition 1`选择EFI引导分区，类型为系统的分区，就是EFI引导分区
5. `assign letter=p:`为所选分区分配盘符，请分配空闲盘符
6. `exit`退出
![](https://gitee.com/Astral/img/raw/master/blog/efi-2.png)
##卸载分区
`remove letter=p` 移除盘符
