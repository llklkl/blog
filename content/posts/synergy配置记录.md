---
title: "Synergy配置记录"
date: 2022-06-16T20:44:43+08:00
draft: false
tags: []
categories: ["Linux使用"]
---

## 简介
Synergy 是一款可以实现多端共享键鼠功能的软件，功能强大，还可以实现剪切板共享。官网：[Symless](https://symless.com/synergy)

使用场景：两台机器，一台 Windows11，一台 Ubuntu22.04，键鼠插在 Windows 机器上。

目标：实现一套键鼠控制两台机器。

<!--more-->

## 配置
Synergy 分为服务端和客户端，可以用键鼠直接控制的一端应该作为服务端，另外一端作为客户端。注意，目前的版本(v1.4.3) 在 Linux 环境下，还不支持 Wayland，只能用 X11，所以配置的时候会禁用掉 Wayland。

### Windows 侧配置
Windows 端直接使用 gui 配置就好了，没有多余的配置。开始调试的时候，可以先进设置禁掉 TLS 加密，后面调试完成了再打开。

### Linux 侧配置

以下配置仅针对 Ubuntu22.04 系统，其他 Linux 可以参考。

#### 禁用 Wayland

修改 `/etc/gdm3/custom.conf`:
```
[daemon]
WaylandEnable=false
```

#### 配置 Synergy 自启动

为了让 Synergy 在登陆界面也能使用，所以需要让 Synergy 在 gnome greeter 界面的时候就启动。这就需要增加一个 synergy.desktop 文件到 `/usr/share/gdm/greeter/autostart` 目录下，内容如下：

```
[Desktop Entry]
Type=Application
Version=1.0
Name=Synergy
Comment=Keyboard and mouse sharing solution Path=/usr/bin
Exec=/usr/bin/synergyc -f --name <client_name> 192.168.1.1:24800
Icon=synergy
Terminal=false
Categories=Utility;
Keywords=keyboard;mouse;sharing;network;share;
```

注意 `<client_name>` 需要替换成自已想要的名称，ip 替换成 Synergy 上显示的能连通的 ip。

然后增加一个脚本，在登陆成功之后 kill 掉 Synergy 进程（这一步实际可以不用，登陆成功 Synergy 父进程退出，Synergy也就退出了），脚本附加到 `/etc/gdm3/PostLogin/Default` 后面，内容如下：

```shell
#!/bin/sh
/usr/bin/killall synergyc
while [ $(pgrep -x synergyc) ]; do
    sleep 0.1
done
```

最后再添加一个 synergy.desktop 到 `~/.config/authstart/` 目录下，内容如下：
```
[Desktop Entry]
Type=Application
Version=1.0
Name=Synergy
Comment=Keyboard and mouse sharing solution Path=/usr/bin
Exec=/usr/bin/synergyc --daemon --name <client_name> 192.168.1.1:24800
Icon=synergy
Terminal=false
Categories=Utility;
Keywords=keyboard;mouse;sharing;network;share;
```

如果需要开启 TLS 加密，那么就在 synergy.desktop 文件中的 Exec 项增加 `--enable-crypto` 参数，然后将**服务端**证书的指纹写到 `~/.synergy/SSL/Fingerprints/TrustedServers.txt` 和 `/var/lib/gdm3/.synergy/SSL/Fingerprints/TrustedServers.txt` 文件内。前面的文件是给登陆后的 Synergy 用的，后面那个是给登陆时的 Synergy 用的。注意，这里不能使用软链接。计算证书指纹的命令如下：
```
openssl x509 -fingerprint -in Synergy.pem -inform pem -sha256
```

如果想要打开日志，增加 `--debug DEBUG --log /path/to/logfile` 就好了，调试的时候可以用用。

### 后记

可以看到，整个配置过程其实不需要键鼠在两台机器之间来回切换，只需要能 ssh 登陆 linux 机器就行配置就行了。

关于为什么不能 systemd 让 synergy 自启，我也不太清楚。systemd 直接启动 synergy 会出现找不到屏幕的情况，即使手动指定了 `--display` 了也还是不行。
