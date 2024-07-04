---
title: "Rust数据内存布局"
date: 2024-06-21 18:42:02
categories: "Rust"
tags: 
- "Rust"
- "Rust-基础"
---

## 数值类型

**整数类型**

| 类型   | 最小值 | 最大值 | 大小(bytes) | 对齐(bytes) |
| ------ | ------ | ------ | ----------- | ----------- |
| `u8`   | 0      | 28-1   | 1           | 1           |
| `u16`  | 0      | 216-1  | 2           | 2           |
| `u32`  | 0      | 232-1  | 4           | 4           |
| `u64`  | 0      | 264-1  | 8           | 8           |
| `u128` | 0      | 2128-1 | 16          | 16          |

| 类型   | 最小值  | 最大值 | 大小(bytes) | 对齐(bytes)align(bytes) |
| ------ | ------- | ------ | ----------- | ----------------------- |
| `i8`   | -(27)   | 27-1   | 1           | 1                       |
| `i16`  | -(215)  | 215-1  | 2           | 2                       |
| `i32`  | -(231)  | 231-1  | 4           | 4                       |
| `i64`  | -(263)  | 263-1  | 8           | 8                       |
| `i128` | -(2127) | 2127-1 | 16          | 16                      |

**浮点数**

| 类型 | 大小(bytes) | 对齐(bytes) |
| ---- | ----------- | ----------- |
| f32  | 4           | 4           |
| f64  | 8           | 8           |

f64 在 x86 系统上对齐到 4 bytes。

**usized & isized**

usize 无符号整形，isize 有符号整形。 在 64 位系统上，长度为 8 bytes，在 32 位系统上长度为 4 bytes。

**bool**

bool 类型，取值为 true 或 false，长度和对齐长度都是 1 byte。

## array

数组的内存布局为系统类型元组的有序组合。

```rust

fn main() {

    let A: [i32; 3] = [1, 2, 3];
    println!("{:?}", std::mem::size_of::<[i32; 3]>()); // 大小: 12
    println!("{:?}", std::mem::align_of::<[i32; 3]>()); // 对齐: 4
}
```

## str

**char 类型**

char 表示：一个 32 位长度字符，Unicode 标量值 [Unicode Scalar Value](http://www.unicode.org/glossary/#unicode_scalar_value) 范围为 in the 0x0000 - 0xD7FF 或者是 0xE000 - 0x10FFFF。

**str 类型**

str 与 [u8] 一样表示一个 u8 的 slice。Rust 中标准库中对 str 有个假设：符合 UTF-8 编码。内存布局与 [u8] 相同。

> [u8]是一个字节切片的类型，而不是一个固定大小的数组。切片是对数组的引用或数组部分的一个动态视图，允许你访问一个连续的内存区域，就像访问一个数组一样，但切片本身并不拥有其引用的数据。
>
> `[u8; N]`：这是一个固定大小的字节数组，其中 `N` 是数组的长度。例如，`[u8; 4]` 是一个包含四个 `u8` 元素的数组。
>
> 二者要注意不要搞混了

**slice**

slice 是 DST 类型，是类型 T 序列的一种视图。 slice 的使用必须要通过指针，&[T] 是一个胖指针，保存指向`数据的地址`和`元素个数`。 `slice 的内存布局与其指向的 array 部分相同`。

**&str 与 String 的区别**

下面给出 &str String 的内存结构比对：

```rust
// String
let mut my_name = "Pascal".to_string();
my_name.push_str( " Precht");
// str
let last_name = &my_name[7..];
```

String

```shell
                     buffer
                   /   capacity
                 /   /  length
               /   /   /
            +–––+–––+–––+
stack frame │ • │ 8 │ 6 │ <- my_name: String
            +–│–+–––+–––+
              │
            [–│–––––––– capacity –––––––––––]
              │
            +–V–+–––+–––+–––+–––+–––+–––+–––+
       heap │ P │ a │ s │ c │ a │ l │   │   │
            +–––+–––+–––+–––+–––+–––+–––+–––+

            [––––––– length ––––––––]
```

String vs &str

```rust
         my_name: String   last_name: &str
            [––––––––––––]    [–––––––]
            +–––+––––+––––+–––+–––+–––+
stack frame │ • │ 16 │ 13 │   │ • │ 6 │  // &str 没有 capacity
            +–│–+––––+––––+–––+–│–+–––+
              │                 │
              │                 +–––––––––+
              │                           │
              │                           │
              │                         [–│––––––– str –––––––––]
            +–V–+–––+–––+–––+–––+–––+–––+–V–+–––+–––+–––+–––+–––+–––+–––+–––+
       heap │ P │ a │ s │ c │ a │ l │   │ P │ r │ e │ c │ h │ t │   │   │   │
            +–––+–––+–––+–––+–––+–––+–––+–––+–––+–––+–––+–––+–––+–––+–––+–––+

			[––––––––––––––––––––––––––– my_name –––––––––––––––––––––––––––]
										[–––––––––––– last_name ––––––––––––]
```

## struct

结构体是带命名的复合类型，有以下几种

**具名结构体**

```rust
struct A {
    a: u8,
}
```

**元组结构体**

```rust
struct Position(i32, i32, i32);
```

**单元结构体**

```rus
struct Gamma;
```

**内存布局**

> 数据对齐
>
> 数据对齐是指将数据存储在内存中时，按照特定的规则将数据放置在内存地址上的一种方式。
>
> 数据对齐的主要目的是为了提高数据读取效率。当CPU访问正确对齐的数据时，它的运行效率最高。若数据没有对齐，CPU在读取或写入数据时可能需要进行多次操作，这会降低CPU的效率，增加系统的开销。
>
> 编译器优化 -> 字段重排
>
> Rust编译器在优化结构体时，可能会进行字段重排（field reordering），这是为了优化内存访问、提高数据缓存的效率，以及确保数据满足平台的内存对齐要求。
>
> Rust编译器的字段重排是一种优化技术，旨在提高程序的性能和内存使用效率。



Rust 中结构体的对齐属性等于`它所有成员中最大的那个`。Rust 会在必要的位置填充空白数据，以保证每一个成员都正确地对齐，同时`整个类型的尺寸是对齐属性的整数倍`。例如：

```rust
struct A {
    a: u8, // 1B
    b: u32, // 4B
    c: u16, // 2B
}

// 打印下变量地址，可以根据结果看到对齐属性为 4, 结构大小为 8 byte 。
// 1 + 4 + 2 = 7
fn main() {
    let a = A {
        a: 1,
        b: 2,
        c: 3,
    };
    println!("0x{:X} 0x{:X} 0x{:X}", &a.a as *const u8 as usize, &a.b as *const u32 as usize , &a.c as *const u16 as usize );
    println!("{:?}", std::mem::size_of::<A>());
}

0x327EFBF35E 0x327EFBF358 0x327EFBF35C
8

// rust编译器会进行内存重排，并填充所需大小(这里是1B)
struct A {
    b: u32,
    c: u16,
    a: u8,
    _pad: [u8; 1],
}

// foo1字段顺序：data2(0), count(4), data1(6) 
// foo1字段顺序：data1(8), count(c), data2(e) 
// 可以看到编译器会改变 Foo<T, U> 中成员顺序。
// 内存优化原则要求不同的范型可以有不同的成员顺序。 如果不优化的可能会造成如下情况，造成大量内存开销：
struct Foo<u16, u32> {
    count: u16,
    data1: u16,
    data2: u32,
}

struct Foo<u32, u16> {
    count: u16,
    _pad1: u16,
    data1: u32,
    data2: u16,
    _pad2: u16,
}
```

## tuple

元组是匿名的复合类型，有以下几种 tuple：

```rust
() (unit)
(f64, f64)
(String, i32)
(i32, String) (different type from the previous example)
(i32, f64, Vec<String>, Option<bool>)
```

tuple 的结构和 Struct 一致，只是元素是通过 index 进行访问的。

## closure

闭包相当于一个捕获变量的结构体，实现了`FnOnce`或`FnMut`或`Fn`。

```rust
fn f<F : FnOnce() -> String> (g: F) {
    println!("{}", g());
}

let mut s = String::from("foo");
let t = String::from("bar");

f(|| {
    s += &t;
    s
});
// Prints "foobar".
```



## union



## enum



## trait



## Dynamically Sized Types(DST)



## 零大小类型(ZST, Zero Sized Type)



## 空类型(Empty Types)



## 数据布局



## 

