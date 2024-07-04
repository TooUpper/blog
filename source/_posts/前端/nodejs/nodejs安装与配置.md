---
title: "nodejs安装与配置"
date: 2024-06-01 22:20:00
categories: "Rust"
tags: 
- "Rust"
- "Rust-基础"
---

## 下载

[Node.js](https://nodejs.org)

## 安装

下一步即可

## 配置环境变量

1.在你需要的目录下新建两个文件夹【node_global】和【node_cache】

```shell
npm config set prefix "D:\tools\SDK\nodejs\configuration\node_global"

npm config set cache "D:\tools\SDK\nodejs\configuration\node_cache"
```

【此电脑】-单击右键【属性】-【高级系统设置】-【环境变量】

2.在【系统变量】中点击【新建】

```she
NODE_PATH
D:\tools\SDK\nodejs\configuration\node_global\node_modules
```

3.编辑【用户变量】中的【Path】

将默认C盘下【AppData\Roaming\npm】修改为【node_global】的路径

4.在【系统变量】中选择【Path】添加【%NODE_PATH%】

