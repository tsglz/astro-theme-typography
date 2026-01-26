---
title: XMRig系列挖矿木马分析与yara检测（一）
pubDate: 2026-1-26
categories: ['样本分析','Yara']
description: 'XMRig是专为门罗币设计的高性能CPU挖矿软件，可高效利用资源，优化容器环境使用，支持参数调整。本文将从开源代码和样本库分析XMRig系列挖矿木马，找出通用检测规则。首个样本xmrig.exe实为启动器而非矿工程序，它构造命令行参数，包含关键IOC"xmrigMiner.exe"和后台运行参数"- -daemonized"。作者已编写简单匹配规则检测此类投递型Miner Wrapper。'
slug:
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

接下来使用 IDA 查看样本的运行逻辑：

整体看下来，很明显，这是一个 xmrig.exe 的启动器。首先样本获取自身路径：

<img src ="/XMRig/1.xmrig.exe-IDA1.png">

然后创建字符串用于构造命令，这里能找到一个关键 IOC："xmrigMiner.exe"

<img src ="/XMRig/1.xmrig-IOC1.png">

后续是参数的拼接，最后跟随了一个重要参数：“ - -daemonized"，代表脱离控制台，在后台静默挖矿。

<img src ="/XMRig/1.xmrig-IOC2.png">

构造结构如下：

```
"xmrigMiner.exe" [argv[1] argv[2] ... argv[n]] " --daemonized"
```

经过逆向分析，该样本只是一个 投递型 Miner Wrapper，而非实际挖矿的矿工程序，所以这里写了一个简单的匹配规则，如下所示：

```
import "pe"
rule pe_xmrig_coinminer_match
{
    meta:
        author = "tsglz"
        description = "投递型 Miner Wrapper"
    strings:
        $s1 = "xmrigMiner.exe" nocase
        $s2 = "--daemonized"
    condition:
        pe.is_pe and $s1 and $s2
}
```