---
title: "Linux Nfs Server安装"
date: 2023-02-12T19:15:49+08:00
draft: false
tags: []
categories: ["Linux使用"]
---

本文记录了在 Ubuntu20 Server 上创建部署 nfs 服务端的过程。

<!--more-->

## 简介

nfs 即网络文件系统，让你可以通过网络访问远程的文件系统，和本地磁盘操作方式几乎没有差异。

为了防止文件泄露，请尽量在本地部署、安装防火墙并设置密码。如部署在公网，一定要设置足够复杂的密码，避免泄露。


## 安装 nfs 软件包

安装 `nfs-kernel-server`。相较于 `nfs-utils`，`nfs-kernel-server` 运行在内核空间，会减少一些状态切换，能够提升性能。
```shell
sudo apt update
sudo apt install nfs-kernel-server
```

## 配置

nfs 服务器配置选项在 `/etc/default/nfs-kernel-server` 和 `/etc/default/nfs-common` 文件。基本不用修改

### 配置导出目录

所有的导出目录配置在 `/etc/exports`。配置格式为 `目录 host(选项)`，表示将 目录 导出到 host，host 可以为 ip 或者域名。简单示例如下:

```plain
/data 192.168.1.0/24(rw,sync,root_squash)
```

常用选项解释：
+ rw：可读写
+ ro：只读
+ sync：数据同步写入磁盘。
+ async：数据异步写入磁盘，会有部分缓存在内存中。
+ root_squash：防止远程 root 用户具有 root 权限，root 用户会被映射成 nobody，可以降低远程用户权限。建议使用。
+ no_root_squash：远程 root 用户可以具有 root 权限。

### 启动 nfs server

配置完导出目录后，可以使用以下命令启动。

```shell
sudo exportfs -ra
```

使用 `exportfs -v` 命令可以查看当前挂载详情。

### 设置防火墙

ubuntu 可以使用 `ufw` 工具来配置防火墙。

```shell
sudo ufw allow from <ip> to any port nfs
sudo ufw reload
```
如果访问不同，可以先尝试将防火墙放开，然后再配置。


## 客户端挂载

### Windows

以下以 windows11 为例，配置挂载 nfs 远端目录。

#### 安装相关工具

打开 `启动或关闭 Windows 功能`，找到 `NFS服务`，全部勾选，然后确定。


#### 连接远端

在 我的电脑 下点击右键 -> 映射网络驱动器，然后配置远端 nfs server 地址并选择挂载磁盘，格式如下：
```plain
\\<host>\<目录>\
```
如：`\\192.168.1.2\data\`


#### 中文乱码问题解决

按下 `Win + r`，输入 `intl.cpl`，选择 `管理`，然后选择 `更改系统区域设置`，勾选 `Beta版：使用Unicode UTF-8提供全球语言支持`，最后确定并重启电脑。