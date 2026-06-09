---
title: Part_One_First_Sight
pubDate: 2024-11-05
categories: ['RustOS']
description: '本文介绍了如何创建独立式可执行程序，参考了Writing an OS in Rust资源。内容包括样例代码、cargo.toml配置，以及Windows平台的不同编译命令。可在rust目录下创建config文件进行配置，但注意config和config.toml不能同时存在。之后编译采用裸机方式，并建议使用nightly版本。'
slug:
---

参考：[独立式可执行程序 | Writing an OS in Rust](https://os.phil-opp.com/zh-CN/freestanding-rust-binary/)

样例代码

```rust
#![no_std] // 不链接 Rust 标准库
#![no_main] // 禁用所有 Rust 层级的入口点

use core::panic::PanicInfo;

/// 这个函数将在 panic 时被调用
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}

#[no_mangle] // 不重整函数名
pub extern "C" fn _start() -> ! {
    // 因为链接器会寻找一个名为 `_start` 的函数，所以这个函数就是入口点
    // 默认命名为 `_start`
    loop {}
}
```

cargo.toml

```rust
[package]
name = "FlowOS"
version = "0.1.0"
edition = "2021"

[dependencies]

[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```

Windows 平台使用的编译命令不同

```bash
cargo rustc -- -C link-args="/ENTRY:_start /SUBSYSTEM:console"
```

也可以在 rust 目录下的 `.cargo` 文件夹下创建 `config.toml` 文件，输入

==注意；如果同时包含 config 和 config.toml 请二选一，如果同时存在，无后缀的会被加载==

```toml
[target.'cfg(target_os = "linux")']
rustflags = [
  "-C",
  "link-arg=-nostartfiles"
]

[target.'cfg(target_os = "windows")']
rustflags = [
  "-C",
  "link-args=/ENTRY:_start /SUBSYSTEM:console"
]

[target.'cfg(target_os = "macos")']
rustflags = [
  "-C",
  "link-args=-e __start -static -nostartfiles"
]
```

上述只是拓展的方法，之后的编译都使用裸机，输入如下代码

```
rustup target add thumbv7em-none-eabihf
```

编译命令

```
cargo build --target thumbv7em-none-eabihf
```

另附：使用 nightly 版本
