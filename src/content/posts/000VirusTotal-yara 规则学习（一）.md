---
title: VirusTotal-yara 规则学习（一）
pubDate: 2025-12-12
categories: ['Yara']
description: 'todo'
slug:
draft: true
---

VirusTotal 支持使用 yara 规则筛选样本，共有两种方式：

1. 写 yara 上传到 Livehunt，匹配未来上传的样本
2. 使用 Retrohunt 对 VT 历史样本进行离线批量匹配

在 VT 中使用 yara 的步骤：

点开侧边栏，在 IoC Investigation 中，下图黄框按需要点击即可：

<img src ="/vt-yara/1.IoC Investigation.png">

Yara 基本规则：https://yara.readthedocs.io/en/stable/writingrules.html

这里为了更好地测试代码，我使用了 vscode+YARA(插件)+yara64.exe(和yara文件放在同一目录)+rule.yara 的配置，可以快速在本地对目标样本进行测试。

yara64.exe 可以从 https://github.com/VirusTotal/yara 下载。

这里先讲一讲一些比较常用的内容：

第一个是 yara 中常见的关键字：

| all      | and       | any        | ascii    | at          | base64   | base64wide | condition |
| -------- | --------- | ---------- | -------- | ----------- | -------- | ---------- | --------- |
| contains | endswith  | entrypoint | false    | filesize    | for      | fullword   | global    |
| import   | icontains | iendswith  | iequals  | in          | include  | int16      | int16be   |
| int32    | int32be   | int8       | int8be   | istartswith | matches  | meta       | nocase    |
| none     | not       | of         | or       | private     | rule     | startswith | strings   |
| them     | true      | uint16     | uint16be | uint32      | uint32be | uint8      | uint8be   |
| wide     | xor       | defined    |          |             |          |            |           |

yara 规则通常由两部分组成：字符串定义（strings）和条件（condition）。字符串有三种类型，十六进制，文本和正则表达式。

所以第二个讲一下 yara 中的字符串，yara 中的字符串有三种类型：

- 十六进制字符串
- 文本字符串
- 正则表达式

十六进制字符串最基本的写法如下：

```yara
rule test
{
    meta:
        author = "tsglz"
    strings:
        $hex = { 4D 5A 90 00 }
    condition:
        $hex
}
```

这段规则会在样本中查找 $hex 匹配的 16 进制字符串。

除此之外，十六进制字符串有 4 种特殊结构：通配符，非运算符，跳转和替代。

通配符可以占位，表示某个位置的字节位置，但是存在，可以匹配任何内容，用法如下所示：

```yara
rule test
{
    meta:
        author = "tsglz"
    strings:
        $a = { 4D 5A ?? ?? }    // MZ 头
    condition:
        $a
}
```

这样写相较于前一种能够更好地匹配 MZ 头，避免遗漏。另外，通配符是按半字节定义的，所以可以只定义字节的一半，而另一半设置为未知。

从 4.3.0 版本开始，可以使用非运算符，指定某个字节不是特定值。原文档中有如下用法：

```yara
rule test
{
    strings:
        $hex_string = { F4 23 ~00 62 B4 }
        $hex_string2 = { F4 23 ~?0 62 B4 }
    condition:
        $hex_string and $hex_string2
}
```

当出现可变长度时，需要使用跳转运算符，如下所示：

```yara
rule test
{
    strings:
        $hex_string = { F4 23 [4-6] 62 B4 }

    condition:
        $hex_string
}
```

其中 [4-6] 表示 4 到 6 字节的任意序列都可以占据这个位置。但是注意，下限不能大于上限。

有时，可能需要为某个片段提供不同的替代方案，文档中举例如下：

```yara
rule test
{
    strings:
        $hex_string = { F4 23 ( 62 B4 | 56 ) 45 }

    condition:
        $hex_string
}
```

然后是文本字符串，最基本的用法如下：

```yara
rule test
{
    strings:
        $text_string = "shellcode"

    condition:
        $text_string
}
```

文本字符串支持如下转义字符：

|     | `\"`   | Double quote 双引号                                         |
| --- | ------ | :---------------------------------------------------------- |
|     | `\\`   | Backslash 反斜杠                                            |
|     | `\r`   | Carriage return 回车                                        |
|     | `\t`   | Horizontal tab 水平标签                                     |
|     | `\n`   | New line 换行符                                             |
|     | `\xdd` | Any byte in hexadecimal notation 十六进制表示法中的任何字节 |

`nocase` 修饰符可以将字符串转换为不区分大小写模式，此修饰符可以与除 `base64` 、 `base64wide` 和 `xor` 之外的任何修饰符一起使用。用法如下：

```yara
rule test
{
    meta:
        author = "tsglz"
    strings:
        $a = "Nocase" nocase
    condition:
        $a
}
```

宽字符字符串可用于搜索每个字符使用两个字节编码的字符串，例如，如果字符串“Borland”以每个字符两个字节（即 `B\x00o\x00r\x00l\x00a\x00n\x00d\x00` ）的形式编码，则以下规则将匹配：

```yara
rule test
{
    strings:
        $wide_string = "Borland" wide

    condition:
        $wide_string
}
```

注：这里的修饰符只是将字符串中字符的 ASCII 码与零交错排列，它并不支持包含非英文字符的真正 UTF-16 字符串。

xor 修饰符可用于搜索应用了单字节 xor 运算的字符串。也就是说，但你怀疑某段文本在样本中存在异或版本，可以使用这个修饰符，用法如下：

```yara
rule test
{
    strings:
        $xor_string = "This program cannot" xor

    condition:
        $xor_string
}
```

一旦使用了 xor 修饰符，该文本会被尝试从 0x00 ~ 0xFF 的密钥进行异或，有一个命中即判定为命中。

文档中提到将 xor 修饰符 与 wide 和 ascii 结合使用。xor 修饰符会应用在所有其他修饰符之后。这意味着同时使用 xor 和 wide (WIDE) 会导致异或运算应用于交错的零字节。

此外，yara 3.11 起，可以对 xor 的字节范围进行限定，写法如下：

```yara
rule test
{
    strings:
        $xor_string = "This program cannot" xor(0x01-0xaa)
    condition:
        $xor_string
}
```

# xor 写完了，后面写 base64

这里笔者尝试写了一个通用样本的匹配规则，通过 Retrohunt 来查找符合条件的样本：

```yara
import "pe"

rule test_match
{
    meta:
        author = "tsglz"
    strings:
        $a = "Software\\Microsoft\\Windows\\CurrentVersion\\Run" //持久化
        $b = { 4D 5A ?? ?? }    // MZ 头
        $rwx = {41b940000000 41b8 [4 - 24] ff15}    //申请RWX内存
    condition:
        $a and $b and $rwx
}
```

这段规则匹配了三部分内容，首先是 $a 检测常见的注册表持久化路径；然后通过 $b 检测 PE 文件头 MZ；最后通过 $rwx 十六进制匹配典型的 mov rax, ...; mov r8, ...; call [func] 形式，尝试匹配 RWX 分配。

这段规则需要三点均满足才匹配（PE 恶意样本 + 持久化 + RWX），限制较为严格，容易产生误报和漏报：

- 首先是注册表路径，可能出现在一些合法启动的程序中，造成误报；
- 其次，RWX 内存申请 通配符可能过于严格，造成漏报，可以针对性地放宽部分条件。

在下一篇中笔者将讲一讲 yara 中的条件，同时讲一讲怎么提高匹配的针对性，对单一系列的样本进行高准确度的匹配。
