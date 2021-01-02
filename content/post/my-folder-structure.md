---
title: "我的存档文件夹目录结构"
date: 2020-09-17T19:48:59+08:00
draft: false
tags: ["备份","存档","目录结构"]
categories: ["备份"]
---

记录下我目前用来存档或者是备份的文件夹的目录结构。结构设计的目标有两点：

- 所有想要存档的文件都有相应合适的区域存储。
- 能够快速找到某个文件。

<!--more-->

说白了就是找到一个好的分类方法，然而这个目标是不容易达成的。

同时有许多情况不允许文件分门别类的放好，比如学习一门课程或是做了一个工程，期间阅读了一些书籍或是文档，又写了一些代码，这时候这些文件还是保持放在一起比较好。再者，文件夹总是有自发混乱的倾向，要么勤加整理，要么平常拿出一些注意力保持整洁，这也是有些困难。因此有了我的一个建议：

**每个文件夹下写一个README**，表示文件夹创建的时间、缘由、将会保存的文件等。这篇文章便是我根目录的说明文件。

下面是我主要的目录结构：

```plain
|--Archive
|--Book
|  |--ComputerScience
|  |--ComputerTechnology
|  |  |--Programming
|  |  `--Software
|  |--ElectronicsEngineering
|  |--Literature
|  |--Mathematics
|  `--Physics
|--Data
|--Document
|  |--Hardware
|  |  |--ST
|  |  `--Xilinx
|  |--Notice
|  `--Policy
|--KnowledgeBase
|  |--Course
|  `--Paper
|--Media
|  |--Music
|  |--Picture
|  `--Video
|--Misc
|--Project
|  `--00_library
`--Resource
   |--Library
   |--Software
   `--OSImage
```

首先要注意的一点是**分类是具有个人特色的**，比如我有很多电子书，我就把Books单独拿出来做了一个目录，我的文学书籍很少于是也没有继续细分。整个结构应该根据个人的情况调整。之后是目录说明：

- Archive，存档：存储简历、考试信息、通讯录、证件、证件照片、账单、收藏夹、配置文件等。
- Book，书籍：分类建议参考图书馆。
- Data，数据：实验数据、测试数据、原始资料等。
- Document，文档：硬件datasheet、指南、通知、政策文件等，建议首先按来源分类，其次按内容分类。
- KnowledgeBase，知识：学习历程、课程、论文等，课程按照时间分类。
- Media，媒体：音乐、图片、视频，应该区分自己拍的照片和视频。
- Misc，杂项：放杂物。
- Project，工程：用于工作目录，00_libraries存放自己的代码库。
- Resource，资源：外部代码库、软件安装包、系统镜像等。

建议相同资源手动加一个编号，比如Projects中的00_library。当需要单独工作于一个目录时建议使用驱动器映射/快捷方式/文件夹链接等方式。数据的完好保存不在于临时拷贝，在于**及时正确的备份**；版本控制也不在于临时拷贝。

## 一点总结

- 写README，做好记录。
- 按需要设计目录结构。
- 做好备份。

我用的备份软件：

- FreeFileSync (Win/Linux) [官网](https://freefilesync.org)

- syncthing(Win/Linux) [官网](https://syncthing.net/)

