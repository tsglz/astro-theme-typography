---
title: XMRig系列挖矿木马分析与yara检测（一）
pubDate: 2025-12-11
categories: ['样本分析','Yara']
description: 'todo'
slug:
draft: true
---

XMRig 是一款专为门罗币（XMR）设计的高性能 CPU 挖矿软件，可以高效利用 CPU 资源，确保在不影响系统正常运行的前提下最大化挖矿效率。此外，该软件优化了在容器环境中的使用，便于在服务器等场景下部署和完成挖矿任务，且支持调整参数，实现个性化配置。

该软件可以在 github 上下载：https://github.com/xmrig/xmrig

<img src ="/XMRig/1.xmrig-github.png">

接下来笔者会从 xmrig 原作者的开源代码和样本库中已经确认的 xmrig 系列挖矿木马两方面进行分析，从而找出该系列木马的通用检测规则。

首先开始对第一个样本的分析，对该系列样本有一个初步了解：

> 样本名：xmrig.exe
>
> sha1:e502523c46e12da3006d86442a7d65aa46e490aa
>
> sha256:0336e377528646d711dba3d88782426de25a8c5b4be587707d43ac7afe18f090

使用 DIE 对样本简单查看，发现该样本里附加了一个 ADL 音频文件：

<img src ="/XMRig/1.xmrig.exe-die.png.png">

对节进行查看，并没有发现非常明显的挖矿木马特征：

<img src ="/XMRig/1.xmrig.exe-exeinfo.png">
