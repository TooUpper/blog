---
title: "trait"
date: 2024-07-12 21:58:02
categories: "Rust"
tags: 
- "Rust"
- "Rust-基础"
---

# Copy

```rust
#![allow(unused)]
#[derive(Copy, clone)]
struct A(i8，i32);
fn main() {
    let a=A(1，2);
    let b =a; // (Bit-wise Copy)按位复制，复制后，b和a完全相同，包括内存对齐填充的padding部分。
    let c=A(a.0，a.1); // 逐成员复制，非按位复制，c和a的padding部分不一定相同。
}    
```

**按位复制他们的内存布局是完全相同的，成员复制则不一定相同。**

 ```rust
 #[derive(Debug, Copy, Clone)]
 struct A {
     a: u16,
     b: u8,
     c: bool,
 }
 
 fn main() {
     let a = unsound_a();
     // 尝试将 Some(a) 改为 a 可以发现 Some 本身带有一个检查的效果
     let some_a = Some(a);
 
     println!("a: {:#?}", a);
     println!("some_a: {:#?}", some_a);
 }
 
 
 fn unsound_a() -> A {
     #[derive(Debug, Copy, Clone)]
     struct B {
         a: u16,
         b: u8,
         c: u8,
     }
     // 依次修改 c 的值为 0，1，2 打印输出结果
     let b = B { a: 1, b: 1, c: 1 };
     unsafe {*(&b as *const B as *const A) }
 }
 ```

输出：

```shell
// let b = B { a: 1, b: 1, c: 1 };
a: A {
    a: 1,
    b: 1,
    c: true,
}
some_a: Some(
    A {
        a: 1,
        b: 1,
        c: true,
    },
)

// let b = B { a: 1, b: 1, c: 0 };
a: A {
    a: 1,
    b: 1,
    c: false,
}
some_a: Some( 
    A {
        a: 1,
        b: 1,
        c: false,
    },
)

// let b = B { a: 1, b: 1, c: 2 };
a: A {
    a: 1,
    b: 1,
    c: true,
}
some_a: Some( // let some_a = Some(a); 输出False; let some_a = a; 输出 None
    A {
        a: 1,
        b: 1,
        c: true,
    },
)
```

**可以发现 Some 本身带有一个检查的效果**

 ```rust
#![allow(unused_variables)]

use std::{ptr, mem};
use std::men::needs_drop;

fn main() {
    let mut d = String::from("cccc");
    let d_len = d.len();
    {
        let mut c = String::with_capacity(d_len);

        unsafe {
            // ptr::copy：从原(&d)中copy一份(1 usize)大小的内容到目标区域(&mut c)中
            // 此处 d 是栈上的一个指针
            // 相当于将 d 指向 "cccc" 的地址复制了一份到 c 中
            // 此时就存在了一个双重引用; d 和 c 都指向了"cccc"这块堆内存的区域
            ptr::copy(&d, &mut c, 1);
        };
        println!("{:?}", c.as_ptr());
        // unsafe {
        //     assert_eq!(needs_drop::<*mut u8>, false);  //成立
        //     ptr::drop_in_place(c.as_mut_ptr());
        // }
        // 注掉 drop，会产生double free，
        // 但是不注掉 drop，会产生无效指针
        mem::drop(c);
    }

    println!("{:?}", d.as_ptr());
    d.push_str("c");
    println!("{}", d);
}

// 输出：
// 0x5630bd63f9b0
// 0x5630bd63f9b0
// c  
// 之所以输出一个 C 是因为在上面的作用域中 C 在离开作用域时将 d 指向的那部分内存drop掉了。
// 但是因为有 d 这个引用指向"cccc"区域，所以暂时内存不会被清理，但是会被标识为不可用。
// 所以此时给 d 追加一个"c",d 中的内容应该是,一部分不可用的内容加上"c"
 ```

 ```rust
// Copy 不一定只在栈上进行
use std::cell::RefCell;

    fn main() {
        let a = Box::new(RefCell::new(1));
        let b = Box::new(RefCell::new(2));
        *b.borrow_mut() = *a.borrow();
        println!("b = {}", b.borrow());
    }
 ```

# Move

```rust
#![allow(unused)]
fn main() {
    let mut a = "42".to_string();
    let b = a;    
    // println!("{:?}", a) // Error
    
    a = "32".to_string();
    println!("{:?}", a)  // 32
    // a 会在此处释放 相当于在此处进行  drop(a)
}
```

**上述可以看出 Move 的本质是 Rust 编译期把这个 a 变量重新进行了一个未初始化的标记，并不是立刻进行 drop，它会将 drop 延后到函数末尾释放**

# Drop

析构函数时，按照栈先进后出的顺序进行析构，变量按照内存布局进行析构；
