---
title: VirusTotal-yara 规则学习（一）
pubDate: 2025-12-9
categories: ['Yara']
description: 'VirusTotal支持yara规则筛选样本，有两种方式：Livehunt匹配未来样本，Retrohunt批量匹配历史样本。文章展示了一个匹配PE恶意样本+持久化+RWX内存分配的yara规则，但存在误报和漏报问题。目前匹配到5.0K样本，计划提高匹配针对性。'
slug:
---

VirusTotal 支持使用 yara 规则筛选样本，共有两种方式：

1. 写 yara 上传到 Livehunt，匹配未来上传的样本
2. 使用 Retrohunt 对 VT 历史样本进行离线批量匹配

在 VT 中使用 yara 的步骤：

点开侧边栏，在 IoC Investigation 中，下图黄框按需要点击即可：

<img src ="/VirusTotal-yara 规则学习/1.IoC Investigation">

Yara 基本规则：https://yara.readthedocs.io/en/stable/writingrules.html

首先尝试写一个通用样本的匹配规则，通过 Retrohunt 来查找符合条件的样本：

```yara
import "pe"

rule test_match
{
    meta:
        author = "tsglz"
    strings:
        $a = "Software\\Microsoft\\Windows\\CurrentVersion\\Run" //持久化
        $b = { 4D 5A 90 00 }    // MZ 头
        $rwx = {41b940000000 41b8 [4 - 24] ff15}    //申请RWX内存
    condition:
        $a and $b and $rwx
}
```

这段规则匹配了三部分内容，首先是 $a 检测常见的注册表持久化路径；然后通过 $b 检测 PE 文件头 MZ；最后通过 $rwx 十六进制匹配典型的 mov rax, ...; mov r8, ...; call [func] 形式，尝试匹配 RWX 分配。

这段规则需要三点均满足才匹配（PE 恶意样本 + 持久化 + RWX），限制较为严格，容易产生误报和漏报：

- 首先是注册表路径，可能出现在一些合法启动的程序中，造成误报；
- 其次，PE 文件的 MZ 头可以适当放宽匹配，而非仅仅匹配 { 4D 5A 90 00 }；
- 最后，RWX 内存申请 通配符可能过于严格，造成漏报，可以针对性地放宽部分条件。

截至目前匹配到 5.0K 样本，后续将提高匹配的针对性，对单一系列的样本进行高准确度的匹配。
