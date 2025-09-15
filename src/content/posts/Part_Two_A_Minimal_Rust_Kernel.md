---
title: Part_Two_A_Minimal_Rust_Kernel
pubDate: 2024-11-05
categories: ['rustOS']
description: ''
slug: 
---

## 目的

编译为一个特定的目标系统

## 新增 `FlowOS.json`

```json
{
    "llvm-target": "x86_64-unknown-none",
    "data-layout": "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-i128:128-f80:128-n8:16:32:64-S128",
    "arch": "x86_64",
    "target-endian": "little",
    "target-pointer-width": "64",
    "target-c-int-width": "32",
    "os": "none",
    "executables": true,
    "linker-flavor": "ld.lld",
    "linker": "rust-lld",
    "panic-strategy": "abort",
    "disable-redzone": true,
    "features": "-mmx,-sse,+soft-float"
}
```

## *build-std* 特性

允许按照自己的需要重编译 core 等标准 crate，而不需要使用 Rust 安装程序内置的预编译版本

但是该特性是全新的功能，到目前为止尚未完全完成，所以它被标记为 “unstable” 且仅被允许在 *Nightly* 环境下调用

要启用该特性，在 `.cargo/config.toml` 写入以下语句：

```toml
[unstable]
build-std = ["core", "compiler_builtins"]
```

该配置会告知cargo需要重新编译 `core` 和 `compiler_builtins` 这两个crate，其中 `compiler_builtins` 是 `core` 的必要依赖

重编译需要提供源码，使用如下命令来下载它们

```rustup
rustup component add rust-src
```

编译命令

```cargo
cargo build --target .\src\FlowOS.json
```

## *build-std-features*

为了使用内存相关函数，在 `.cargo/config.toml`  的 \[`unstable`\] 写入以下语句：

```toml
build-std-features = ["compiler-builtins-mem"]
```

设置编译目标：在 `.cargo/config.toml` 加入如下选项，接下来，==使用 cargo build 即可进行编译==

这里因为一些原因，我使用了绝对路径

```toml
[build]
target = "F:\\vscode_repo\\Rust\\FlowOS\\src\\FlowOS.json"
```

## 向屏幕打印字符

写入 `VGA 字符缓冲区`，这段缓冲区的地址是 `0xb8000`，且每个字符单元包含一个 ASCII 码字节和一个颜色字节

代码修改

```rust
static HELLO: &[u8] = b"Hello World!";

#[no_mangle]
pub extern "C" fn _start() -> ! {
    let vga_buffer = 0xb8000 as *mut u8;

    for (i, &byte) in HELLO.iter().enumerate() {
        unsafe {
            *vga_buffer.offset(i as isize * 2) = byte;
            *vga_buffer.offset(i as isize * 2 + 1) = 0xb;
        }
    }

    loop {}
}
```

## 启动内核

### 创建引导映像

```cargo
cargo add bootloader
```

我这里使用的是

```toml
bootloader = "0.9"
```

### 将内核和引导程序组合

```bash
cargo install bootimage
```

### 安装  llvm-tools-preview

```rust
rustup component add llvm-tools-preview
```

### 创建一个可引导的磁盘映像

```bash
cargo bootimage
```

编译好后能在文件夹中找到 `bootimage-FlowOS.bin`

## 运行

在 QEMU 中输入如下命令运行

```bash
qemu-system-x86_64 -drive format=raw,file=target\FlowOS\debug\bootimage-FlowOS.bin
```

也可以使用 dd 工具把内核写入 U 盘，以便在真机上启动，这里不多赘述

原代码作者补注：`bootloader` 包暂时不支持 UEFI，所以不能在 UEFI 机器上启动（现在不知道，没试）

### 使用 `cargo run`

在 `.cargo/config.toml`  中设置 `runner` 配置项：

```toml
[target.'cfg(target_os = "none")']
runner = "bootimage runner"
```

这样，就可以直接使用 `cargo run` 直接运行