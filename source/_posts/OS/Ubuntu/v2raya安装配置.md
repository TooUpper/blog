---
title: "v2raya安装配置"
date: 2024-07-26 19:16:00
categories: "OS"
tags: 
- "Linux"
---

## 安装 v2ray-core

下载：

```shell
https://github.com/v2fly/v2ray-core
```

解压：

```shell
sudo unzip v2ray-linux-64.zip -d /usr/local/v2ray-core
```

拷贝geoip.dat和geosite.dat到/usr/local/share/v2ray/：

```shell
sudo mkdir -p /usr/local/share/v2ray/
sudo mv /usr/local/v2ray-core/*dat  /usr/local/share/v2ray/
```

## 安装 v2rayA

下载：

```shell
https://github.com/v2rayA/v2rayA
```

安装：

```shell
sudo dpkg -i installer_debian_x64_*.deb
```

## 配置 v2rayA

```shell
vim /etc/default/v2raya

# 添加如下配置

V2RAYA_V2RAY_BIN=/usr/local/v2ray-core/v2ray
V2RAYA_V2RAY_CONFDIR=/usr/local/v2ray-core

# 输入":wq!"后可保存更改
```

设置开机自启：

```shell
sudo systemctl enable --now v2raya
sudo systemctl status v2raya 
```

打开v2rayA：`v2rayA Web panel`
