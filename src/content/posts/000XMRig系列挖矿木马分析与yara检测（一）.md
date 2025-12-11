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
