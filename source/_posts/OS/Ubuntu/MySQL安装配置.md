---
title: "MySQL安装配置"
date: 2024-07-28 20:41:00
categories: "OS"
tags: 
- "Linux"
- "Ubuntu"
---

## 安装

**添加 MySQL APT 存储库**

 [MySQL APT 存储库下载页面](https://dev.mysql.com/downloads/repo/apt/)。

安装

```shell
$ sudo dpkg -i mysql-apt-config_0.8.32-1_all.deb 
```

**更新存储库**

```shell
$ sudo apt-get update
```

**安装MySQL**

```shell
$ sudo apt-get install mysql-server
```

## 卸载

**1. 停止MySQL服务**

在卸载之前，首先需要确保MySQL服务已经停止运行，以防止卸载过程中出现潜在问题。可以使用以下命令来停止MySQL服务：

```bash
$ sudo systemctl stop mysql
```

或者，如果你使用的是较旧的Ubuntu版本，可能需要使用以下命令：

```bash
$ sudo service mysql stop
```

**2. 卸载MySQL软件包**

使用`apt-get`命令来卸载MySQL服务器及其相关软件包。通常，除了`mysql-server`外，可能还需要卸载`mysql-client`和`mysql-common`等软件包。使用以下命令进行卸载：

```bash
$ sudo apt-get remove --purge mysql-server mysql-client mysql-common
```

`--purge`选项会卸载软件包并删除其配置文件。

**3. 删除MySQL的配置文件和数据目录**

尽管`apt-get remove --purge`命令会删除大多数配置文件，但为了确保彻底卸载，建议手动删除MySQL的配置文件和数据目录。这些文件通常位于`/etc/mysql`和`/var/lib/mysql`目录下。使用以下命令进行删除：

```bash
$ sudo rm -rf /etc/mysql /var/lib/mysql
```

此外，有些MySQL版本可能会在`/var/log/mysql`目录下生成日志文件，你也可以选择删除这些日志文件：

```bash
$ sudo rm -rf /var/log/mysql
```

**4. 清理残留文件和依赖项**

使用`apt-get autoremove`命令来自动删除不再需要的依赖项。此外，`apt-get autoclean`命令可以清理下载的软件包缓存，但这一步不是必须的，因为它不会直接删除MySQL的残留文件。

```bash
$ sudo apt-get autoremove  
$ sudo apt-get autoclean
```

**5. 验证卸载结果**

为了验证MySQL是否已完全卸载，可以尝试运行`mysql`命令查看其版本。如果MySQL已成功卸载，系统将显示一个错误消息，指出找不到`mysql`命令。

```bash
$ mysql --version
```

如果看到类似“Command 'mysql' not found, but can be installed with:”的错误消息，则表示MySQL已成功卸载。

