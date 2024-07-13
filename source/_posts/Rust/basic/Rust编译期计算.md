---
title: "Rust编译期计算"
date: 2024-06-20 21:51:02
categories: "Rust"
tags: 
- "Rust"
- "Rust-基础"
---

## Rust 编译期计算

*CTFE*(compile time function evaluation)：是指在编译阶段由编译器进行的运算，这种运算不占用程序运行时的时间。

两种方式：

- 过程宏 + Build脚本(build.rs)

- 常量表达式求值
  - 常量函数(const fn)
  - 常量泛型(const generic)

**常量传播**：常量传播是编译器的一种优化方式，例如将3 + 4 优化为 7 ，避免运行时再次计算。

```rust
const X: u32 = 3 + 4; // CTEF
let x: i32 = 3 + 4; // 不是 CTEF,但可能会被常量传播优化，因为他不在常量上下文。
```

### 常量表达式求值

**常量函数**

```rust
const fn fib(n: u128) -> u128 {
    const fn helper(n: u128, a: u128, b: u128, i: u128) -> u128 {
        if i <= n {
            helper(n, b, a + b, i + 1)
        } else {
            b
        }
    }
    helper(n, 1, 1, 1, 2)
}

const X: u128 = filb(10); // X 会在编译器完成求值

fn main() {
    println("{}", X); 
}
```

> 编译期计算通过 MIR(中级中间语言) 中内置的 MIri(编译器内置 MIR 解释器) 实现

**常量泛型**

为什么要有常量泛型？

因为在 Rust 中定义相同的静态数组是不同的元素：

```rust
// 二者是不同的类型
let arr: [3; i32] = {1, 2, 3};
let arr: [5; i32] = {1, 2, 3, 4, 5};
```

为了在使用过程中可以使用泛型统一个的不同长度的数组，官方引入了常量泛型的概念。

```rust
use std::mem::MaybeUninit;

pub struct ArrayVec<T, const N: usize> {
    items: [MaybeUninit<T>; N],
    length: usize,
}
```

### 过程宏 + Build脚本(build.rs)

