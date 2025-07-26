---
title: "WSL2环境配置"
date: 2024-03-28 19:16:00
categories: "Linux"
tags: 
- "Linux"
---

## WSL2安装配置

**1.勾选适用于Linux的Windows子系统**后**重启**
打开“**控制面板**”->点击"**程序**"->在“**程序和功能**”中->点击“**启用或关闭 Windows 功能**”-> 选中“**适用于 Linux 的 Windows 子系统**”->选择"**立即重新启动**"；

**2.启用虚拟机功能**
安装 WSL 2 之前，必须启用“虚拟机平台”可选功能。 计算机需要[虚拟化功能](https://learn.microsoft.com/zh-cn/windows/wsl/troubleshooting#error-0x80370102-the-virtual-machine-could-not-be-started-because-a-required-feature-is-not-installed)才能使用此功能。
以管理员身份打开 PowerShell 并运行：
**点击“开始”->搜索“Power shell”->右键“Power shell”->已管理员方式运行**

```powershell
> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

此处最好重启一下不然后续启动arch Linux时会报错**error: 0x8004032d(null)**

> 如何没有安装 PowerShell 建议直接从 Microsoft Store 中搜索并下载

**3.下载Linux内核更新包**
从[官网](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)下载最新的Linux内核更新包并**安装**；
**管理员**身份打开PowerShell：

```powershell
> wsl.exe --update
正在检查更新。
已安装最新版本的适用于 Linux 的 Windows 子系统。
```

这里如果安装进度一直为0.0%可以试下开启科学上网；

**4.将WSL2设置为默认版本**
打开 PowerShell运行以下命令，将 WSL 2 设置为默认版本：

```powershell
> wsl --set-default-version 2
有关与 WSL 2 关键区别的信息，请访问 https://aka.ms/wsl2
操作成功完成。
```

**5.安装所选的Linux分发**
打开 [Microsoft Store](https://aka.ms/wslstore)，并选择你偏好的 Linux 分发版。（我这里使用的是arch Linux）
如果没有安装 Microsoft Store 可以试着通过如下命令进行安装：

```c
// 查看可用发行版列表
wsl --list --online
// 安装指定发行版
wsl --install -d <DistroName> 
// wsl --install -d Ubuntu-24.04    
```

首次启动新安装的 Linux 分发版时，将打开一个控制台窗口，系统会要求你等待一分钟或两分钟，以便文件解压缩并存储到电脑上。

然后，需要[为新的 Linux 分发版创建用户帐户和密码](https://learn.microsoft.com/zh-cn/windows/wsl/setup/environment#set-up-your-linux-username-and-password)。

```powershell
Installing,this may take a few minutes...
Iainstallation successful!
Please create a default UNIX user account, The username does not needor to match your windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username:
```

这里要关掉科学上网，不然**Microsoft Store**可能会打不开；

**6.查看当前环境的wsl版本和对应子系统**

```powershell
// 在Windows终端中键入
> wsl -l -v
  NAME    STATE           VERSION
* Arch    Running         2
```

**7.注销安装的Linux子系统账户**

```powershell
> wsl --unregister Arch
```

（名称要与wsl -l -v 命令中NAME一致）

**8.删除安装的Linux子系统**

系统 -> 应用 -> 安装的应用 删除Arch WSL。
