---
layout: post
title: "使用xrdp进行服务器远程GUI访问"
date: 2025-08-04 10:00:00 +0800
categories: [服务器]
tags: [服务器, GUI]
---

![整体样貌总览](/assets/img/xrdp-preview.png)

## 配置方法
1. 安装xrdp

```terminal
sudo apt update
sudo apt install xrdp gnome-session -y
```

2. 配置SSL证书

为了让 xrdp 有权访问系统的图形服务，需要将它加入 ssl-cert 用户组：
```terminal
sudo adduser xrdp ssl-cert
```

3. 配置.xsessionrc文件

参考下面的文件

## 同一个服务器上其他用户需要做的事情
仅需要创建~/.xsessionrc
```~/.xsessionrc
export GNOME_SHELL_SESSION_MODE=ubuntu
export XDG_CURRENT_DESKTOP=ubuntu:GNOME
export XDG_SESSION_TYPE=x11
export XDG_DATA_DIRS=/usr/share/ubuntu:/usr/local/share:/usr/share
# 解决 3D 加速引起的黑屏
export LIBGL_ALWAYS_SOFTWARE=1
# 修复 DBUS 权限冲突
dbus-update-activation-environment --all
```



## 错误及解决方法
### 1. 很卡，几乎没法用
- 降低显示的像素点、以及色彩位数
![修改xrdp配置](/assets/img/xrdp-config.png)


- 修改缓存区的大小
sudo vim /etc/xrdp/xrdp.ini
```/etc/xrdp/xrdp.ini
tcp_send_buffer_bytes=8388608
tcp_recv_buffer_bytes=8388608
```

sudo vim /etc/sysctl.conf
```
net.core.wmem_max=16777216
net.core.rmem_max=16777216
```

执行`sudo sysctl -p`让配置立即生效。

重启xrdp `sudo systemctl restart xrdp`


### 2. 登录后黑屏
原因未知，可能是非正常退出（没有使用Log out），原因同3

解决方法：
```shell
# 彻底杀死该用户的所有进程
sudo pkill -u administrator
```

### 3. 登录后闪退
xrdp不能同时和其他GUI共存，需要使用Log out之后才能再次尝试连接。

> 物理机登录限制：同一时间，新用户如果想远程登录，他绝对不能在机房的物理屏幕上处于登录状态。

> 操作：如果新用户之前在机房用过电脑，请确保他点击了 Log Out（注销）。


## 参考文档
[github-issue_xrdp很卡怎么解决](https://github.com/neutrinolabs/xrdp/issues/1483)