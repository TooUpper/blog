---
title: "main函数与命令行参数"
date: 2024-10-15 21:11:00
categories: "C语言"
tags: 
- "C语言进阶"
---

**main 函数的本质究竟是什么?**

- main 函数是操作系统调用的函数
- 操作系统总是将 main 函数作为**应用程序的开始**
- 操作系统将 main 函数的**返回值**作为**程序的退出状态**

程序在运行的时候就会产生一个进程，那么操作系统如何去判断这个程序是否是异常退出的呢，有些操作系统会弹出一个对话框，告诉我们应用程序异常退出；

> 在 window 下可以使用 echo %ERRORLEUEL% 指令打印出 main 函数返回的值

**那么操作系统是如何使用 main 函数给他的返回值呢**

```c
// A.c
int main() {
    printf("I'm A");
    return 0;
}

// B.c
int main() {
    printf("I'm B");
    return 99;
}
```

此时我们只在 cmd 中执行 B.exe && A.exe（从 C 语言来看功能就是正常执行完 B.exe 之后在去执行 A.exe）:

这里我们会发现，程序只输出了 I'm B，说明 99 找个返回值对于操作系统来说是异常的，是不正常执行之后返回的状态；

此时我们反过来  A.exe && B.exe 就会发现，既输出了 I'm A 也输出了 I'm B，说明 0 找个返回值对于操作系统来说是个正常执行后所返回的状态；

> 建议在写程序时候利用这种返回值为 int 0 的标准 main 函数；

**main 函数的参数**

```c
int main()
int main(int argc)
int main(int argc, char* argv[])
int main(int argc, char* argv[], char* env[])
// argc - 命令行参数的个数
// argv - 命令行数组
// env  - 环境变量数组    
```

**以 GCC 编译器为例，该如何使用：**

![CGCCliebiaocanshu](/public/image/C/C语言进阶/CGCCliebiaocanshu.png)

```c
#include "stdio.h"


int main(int argc, char* argv[], char* env[]) {

    int i = 0;

    printf("==============Begin argv==============\n");

    for(i = 0; i < argc; i++) {
        printf("%s\n", argv[i]);
    }

    printf("==============End argv==============\n");

    printf("\n");
    printf("\n");
    printf("\n");

    printf("==============Begin env==============\n");

    for(i = 0; env[i] != NULL; i++) {
        printf("%s\n", env [i]);
    }

    printf("==============End env==============\n");

    // 运行 ./a.out 输出：
    /*
    
    ==============Begin argv==============
    ./a.out
    ==============End argv==============



    ==============Begin env==============
    SHELL=/bin/bash
    WSL2_GUI_APPS_ENABLED=1
    WSL_DISTRO_NAME=Ubuntu-22.04
    NAME=ThinkPad
    PWD=/home/kay/workspace/test
    LOGNAME=kay
    MOTD_SHOWN=update-motd
    HOME=/home/kay
    LANG=C.UTF-8
    ...
    ==============End env==============
    */

}
```

> 环境变量数组中，并没有给出具体的长度信息，所以通过空去判断；
>
> env 中打印的是程序在运行过程中，操作系统传递进行来的环境变量的信息；
>
> argv 中的参数就是传递给 main 函数所使用的参数；

**main 函数一定是程序执行的第一个函数吗？**

这类问题要分情况讨论，以 GCC 编译器为例

```c
#include <stdio.h>

// 判断是否是 GCC 编译器，如果不是则定义一个空的宏
#ifndef __GNUC__
#define __attribute__(x) 
#endif

// GCC 编译器的属性关键字，表示在 main 函数执行之前先执行下列的函数也就是 void before_main()
__attribute__((constructor))
void before_main() {
    printf("%s\n",__FUNCTION__);
}

// GCC 编译器的属性关键字，表示在 main 函数执行之后再执行下列的函数也就是 void after_main()
__attribute__((destructor)) 
void after_main() {
    printf("%s\n",__FUNCTION__);
}

int main() {
    printf("%s\n",__FUNCTION__);
    
    return 0;
}
	/*
	before_main
	main
	after_main
	*/
```

**小结**

- 一个 C 程序是从 main 函数开始的
- main 函数是**操作系统**调用的函数
- main 函数有**参数**和**返回值**
- 现代编译器支持再 main 函数前调用其他函数
