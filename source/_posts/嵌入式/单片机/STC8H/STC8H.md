---
title: "STC8H"
date: 2024-09-02 08:41:00
categories: "嵌入式"
tags: 
- "STC8H"
---

## STC8H



Keil C51程序自动加载了一个名为”STARTUP.A51”的文件，在这个文件里面进行了一系列的初始化操作后进入用户编写的C语言程序入口main函数中，main函数执行完毕后，STARTUP.A51文件后有一句跳转到程序入口main函数的语句，所以会再次进入C语言主程序main函数中执行相关内容。
