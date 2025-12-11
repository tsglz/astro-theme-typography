---
title: 脱壳中的附加数据（overlay）
pubDate: 2025-12-8
categories: ['样本分析']
description: 'todo'
slug:
draft: true
---

一般程序加载的过程中，区段会被映射到内存中。而 overlay 即为程序中不被映射到内存的数据，它将被程序以打开自己的方式读取。不是区段中包括的文件中断的数据，都是 overlay。

所以在脱壳过程中可能遇到这样的问题，脱完壳还是发现程序加载失败了。这是因为从内存中 dump 下来的文件是没有 overlay 的，需要手动把内容粘贴到 dump 下来的数据后面。
