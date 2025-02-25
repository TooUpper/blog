---
title: "JL环境搭建"
date: 2024-12-01 17:30:00
categories: "杰理"
tags: 
- "AC791N"
---

SDK工程默认支持在 Windows 系统上，用 CodeBlocks 编译。

配置环境主要分为三步：

1. 下载并安装 CodeBlocks，下载链接： http://pkgman.jieliapp.com/s/codeblocks
2. 打开安装好的 CodeBlocks 后，关闭 CodeBlocks
3. 下载并安装工具链，下载链接：http://pkgman.jieliapp.com/s/win-toolchain
4. 下载并安装软件包管理器，下载链接：http://pkgman.jieliapp.com/s/pkgmanoffline

到现在，你已经可以打开 CodeBlocks 工程进行编译开发了。

## 官方文档

[杰里官方文档地址](https://doc.zh-jieli.com/AC79/zh-cn/master/index.html)

在官方文档中要选择我们对应的芯片型号与版本，比如：AC791，在选择版本时候建议不要选择最新的 master 选择 release_v 1.2.0 即可；

## 开发环境

杰里官方推荐的集成开发工具为[CodeBlocks官方网站](https://www.codeblocks.org/downloads/)

## 安装工具链

- 在确保当前机器已经安装好 CodeBlocks 之后，就可以开始安装杰理工具链，点击[链接](http://pkgman.jieliapp.com/s/win-toolchain)下载工具链（Windows版本）
- 下载好的杰理工具链安装包名称为 2.5.0.exe，双击后按提示进行安装即可。

## 更新工具链

在确保当前PC已经安装杰理的工具链后，当杰理发布新版本的工具链时，可以按如下步骤检查更新：点击“开始菜单“ ，选择”检查更新“ 。

![jcgx](/public/image/嵌入式/MCU/JL/AC791N/jcgx.png)

如果未更新到杰理工具链最新版本，则会提示下图。

![sjjc](/public/image/嵌入式/MCU/JL/AC791N/sjjc.png)

点击上图界面"确定"按钮后，进入如图所示下载界面。

![sjgz](/public/image/嵌入式/MCU/JL/AC791N/sjgz.png)

点击上图的"下载升级包"后，开始下载新版本工具链的安装包，下载完毕后，提示升级包文件下载成功界面。

![sjwc](/public/image/嵌入式/MCU/JL/AC791N/sjwc.png)

点击"打开升级包文件"后，开始安装下载好的安装包。

![dksjb](/public/image/嵌入式/MCU/JL/AC791N/dksjb.png)

更新到最新的版本后，再次"检查更新"就会提示如下信息。

![zxbb](/public/image/嵌入式/MCU/JL/AC791N/zxbb.png)

## 安装杰理软件包管理器

- 为了能正常使用 AC791N_配置工具 (AC791N_config_tool)，需要事先安装杰理软件包管理器，点击链接 http://pkgman.jieliapp.com/s/pkgmanoffline 下载。
- 双击后按提示进行安装即可。
