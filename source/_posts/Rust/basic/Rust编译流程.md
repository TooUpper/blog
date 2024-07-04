---
title: "Rust编译过程"
date: 2024-06-18 21:41:02
categories: "Rust"
tags: 
- "Rust"
- "Rust-基础"
---

**Rust 编译过程**

![rust-complie-process](/public/image/Rust/basic/rust-complie-process.png)

直接使用源代码非常不方便且容易出错。因此在我们做任何其他事情之前，我们将原始源代码转换为 AST。即使这样做也涉及大量工作，包括词法分析、解析、 宏扩展、名称解析、条件编译、功能门控 AST的检查和验证。

值得注意的是，这些任务之间并不总是有明确的顺序。例如，宏扩展依赖于名称解析来解析宏和 imports。解析需要宏扩展，而这反过来又可能需要分析宏的输出。

**步骤说明**：

1. Rust 源码(Unicode字符)作为 *UTF-8* 编码序列输入到编译器，编译器的 Lexing 会获取字符编码并将他们转换为`token Steam`，然后解析 token Stream 将它们转换为结构化的编译器更容易使用的表单，通常称为`抽象语法树`(AST)  。

   请注意，在解析为 AST 时 ，Rust编译器会对其进行*词法分析*包括但不限于宏扩展、名称解析、#[test]实现、panic实现、AST验证、Feature 检查、检索 Lang 项目等。

2. AST 降低到  *HIR*(高级中间表示) 

   HIR 是抽象语法树 (AST) 对编译器更友好的表示形式，很多 Rust 语法糖在这一阶段，已经被脱糖（desugared）处理。比如 `for` 循环和`while(let)`在这个阶段会被转为`loop`，`if let` 被转为`match`，普遍`impl Trait`转换为泛型参数，存在`impl Trait`转换为虚拟声明`existential type`等等。

   对于编译器来说，所有的版次也就是Edition版本，在到达中间语言层次的时候已经消除了版次差异。

3. HIR 降低到 *THIR*(类型化高级中间表示)

   THIR 是 rustc 在类型检查后生成的另一种IR，它用于MIR建设、详尽性检查和不安全检查。是一个在在 HIR 的基础上进一步添加了类型信息的版本。

4. THIR 降级为 *MIR*(中级中间表示)

   它是 Rust 的一种彻底简化的形式，用于 某些对流量敏感的安全检查——特别是借用检查器！– 也用于优化和代码生成。

> Rust 源码(Unicode字符)作为 *UTF-8* 编码序列输入到编译器，通过分词把词法结构处理为词条流，词条流经过语法解析形成`抽象语法树`，抽象语法树降级（简化）为`高级中间语言`（*HIR*），高级中间语言被用于编译器对代码进行类型检查方法查找等工作，高级中间语言继续降级（简化）为`中级中间语言`（*MIR*），中级中间语言被用于借用检查、优化、代码生成（宏、泛型、单态化）等工作，中级中间语言（MIR）优化为`LLVM中间语言`，最后交给LLVM编译器生成机器码。

[Rust 编译器开发指南](https://rustc-dev-guide.rust-lang.org/the-parser.html)

[Rust编译器专题 | 图解 Rust 编译器与语言设计 Part 1 - Rust精选](https://rustmagazine.github.io/rust_magazine_2021/chapter_1/rustc_part1.html)
