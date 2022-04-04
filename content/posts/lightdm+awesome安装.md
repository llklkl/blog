---
title: "Lightdm+awesome安装"
date: 2022-04-04T16:32:52+08:00
draft: false
tags: []
categories: ["Linux使用"]
---

awesome+lightdm 小内存机器必备。本文记录 awesome 和 lightdm 安装和配置的相关命令。

<!--more-->

## 安装 awesome
安装 awesome 比较简单，用 apt 命令即可
```shell
sudo apt install awesome
```


## 安装 lightdm
安装 lightdm 比较简单，用 apt 命令即可

```shell
sudo apt install lightdm lightdm-gtk-greeter lightdm-settings/focal
```

### 配置

ligthdm 有一个默认配置文件，在 `/usr/share/doc/lightdm/lightdm.conf.gz`，将其复制到 `/etc/lightdm/` 作为自己的配置文件
```shell
cd /etc/lightdm/
sudo cp /usr/share/doc/lightdm/lightdm.conf.gz /etc/lightdm/

sudo gunzip lightdm.conf.gz
```

#### 开启 Xdmcp
按以下方式修改 `lightdm.conf` 文件：
```shell
[XDMCPServer]
enabled=true
port=177
```

然后重启 lightdm
```shell
sudo service lightdm restart
```

#### 修改默认会话
按以下方式修改 `lightdm.conf` 文件：
```shell
[Seat:*]
user-session=awesome
```

如果不能生效，尝试修改 `$HOME/.dmrc` 文件
```shell
[Desktop]
Session=awesome
```

修改配置之后，都需要重启 lightdm。