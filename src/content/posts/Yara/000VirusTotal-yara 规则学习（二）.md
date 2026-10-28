---
title: VirusTotal-yara 规则学习（二）
pubDate: 2025-12-14
categories: ['Yara']
description: 'todo'
slug:
draft: true
---

检测行为和家族聚类的区别
换了 YARA 4.5.8 进行检测，后续语法可能与之前的老版本有出入，以最新的为准。（新版本的 yara 区别与原来的 release 包，是需要编译的，与老版本的 yara 在不同的仓库中）

借助 mcp 辅助样本分析，一些其他的平台和工具捕获家族样本，并提取了相应的一些规则用于检测同家族样本。

