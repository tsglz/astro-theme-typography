---
title: 对TEMPEST代码的研究（一）
pubDate: 2025-03-24
categories: ['C2']
description: ''
slug: 
---

项目地址：[Teach2Breach/Tempest: A command and control framework written in rust.](https://github.com/Teach2Breach/Tempest)

Tempest 是一个完全由 rust 编写的研究性 C2，作者在项目介绍中提到，该项目并未借鉴其它框架，是以该项目生成的 shellcode 或许可以与 cobalt strike 生成的 shellcode 进行对比学习

Tempest 的组件分为如下三部分：服务器，客户端，代理/植入物
顺便一提，由于笔者对于 linux 以及 mac 不甚精通，目前的研究仅限于 Windows 部分的分析

## 基础架构

**Anvil** - 服务器

- 2 个带有 API 的服务器。所有 API 都经过身份验证，并且不支持 unauth-discovery。
- SQLite 本地数据库
- 内部功能（构建 IMPS、生成 shellCode 等）

**Conduit** - 黑客客户端

- 终端用户界面 （TUI）
- “实时”仪表板显示
- 便携，完全在终端中运行

**IMPS** - 信标/代理/植入物
Windows 功能：

- 通过 TLS 的 AES 加密通信
- 专注于 OPSEC，功能丰富（无膨胀）
- 远程进程注入
- Bof 支持
- .DOTNET 可执行文件支持
- WMI
- TEB 步行 “noldr”

## 服务器-anvil

当前配置为 2 个不同端口（443，8443），在 `config.toml` 中进行配置，443 被用于植入物，目前不支持关闭侦听器，处于默认打开状态。此外，初始访问使用的是 HTTPS 协议，其它协议有待拓展

anvil 服务器使用 config.toml 进行配置，用户可在其中定义自己的用户名和密码进行验证或其它操作（如 LITCRYPT 密钥）。服务器还需要本地存储的 TLS 证书才能通过 HTTPS 提供内容

`config.toml` 文件示例

```rust
[[users]]
username = "forge"
password = "forge"

[[users]]
username = "adversary"
password = "pwd1"

[cert]
private_key = "/home/adversary/Tempest/Anvil/cert/key.pem"
certificate = "/home/adversary/Tempest/Anvil/cert/cert.pem"

[crypt]
LITCRYPT_ENCRYPT_KEY = "......"
```

填写完配置文件，即可使用如下命令进行构建

```rust
cargo build --release
```

## 客户端-conduit

跨平台的与 c2 交互的客户端，完全在终端运行，编译后可以移植

其中坏死植入物用红色表示，管理员用橙色表示，存活植入物用白色表示

## 植入物

Windows 植入程序使用 `LdrGetDllHandle` 和 `LdrGetProcedureAddress` 来查找程序中调用的几乎所有其他 API 的函数地址。

而 Windows_noldr 变体使用 TEB->PEB 遍历，首先读取 CPU 寄存器以查找 TEB（基于 `NTCurrentTeb`），然后定位 PEB 并遍历 PEB 以查找 ntdll.dll 或 kernel32.dll 等 dll。这允许植入体查找 API 调用的函数地址，而无需调用 `LdrGetDllHandle` 和 `LdrGetProcedureAddress`

socks 代理，该模块目前为单个植入物设计。出于安全考虑，作者将代理设置为仅在 c2 服务器上本地可用。这意味着为了使用代理，操作者必须通过 ssh 进入运行 anvil c2 服务器的机器，命令如下：

```powershell
ssh -L 1080：localhost：1080 username@anvilserver
```

## differences

在作者的描述中，目前该项目可以生成 exe 和 dll 文件，但由于时间有限，并不支持生成 shellcode

此外，作者采用精简化处理框架，删去了大量冗余功能，加入了 bof 支持、有效的远程进程注入和 WMI 支持，编写了 whoami、ipconfig 和其他更基本的模块（如 ps），通过实际深入研究 Windows 内部，并调用与 Task Manager 等相同的函数，以与 Task Manager 相同的方式检索进程列表，但使用隐蔽的动态定位函数地址

具体如何实现还需要进一步对项目进行分析