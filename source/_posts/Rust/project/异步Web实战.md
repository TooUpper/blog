---
title: "异步Web实战"
date: 2024-05-22 18:00:00
categories: "Rust"
tags: 
- "Rust"
- "Rust-实战"
---

# 项目

## 创建项目

## rust编码规范

新建`.rustfmt.toml`文件用于帮助你写出地道的rust代码。

Rustfmt 的设计非常易于配置。您可以创建一个名为 `rustfmt.toml`或 的TOML 文件`.rustfmt.toml`，将其放在项目或任何其他父目录中，它将应用该文件中的选项。

这需要将rust设置为`nightly`时才可使用。

```shell
rustup default nightly
```

在当前工作目录中的 Cargo 项目上运行：

```
cargo +nightly fmt

# rustfmt lib.rs main.rs 将就只格式化“lib.rs”和“main.rs”
# rustfmt lib.rs --check 检查 lib.rs 文件的格式，并给出修改意见，不进行格式化
```

## 学习axum

## 接口设计





# 工具

**1.cargo-edit工具**

```powershell
cargo install cargo-edit
```

**2. Toml文件插件**

`TOML Language Support`

**3. cratescha插件**

`crates`

