---
title: Linux使用上踩过的坑
date: 2019-10-04
tags:
---

# Linux使用上踩过的坑

## 前言

记录我在linux使用上遇到的一些坑，帮助其他人能够更快的发现问题(ps: ctrl+f进行关键词搜素)

## 主要内容

### 开机优化及问题排查

- 使用 systemd-analyze blame 查看各个服务开机耗时，可disable一些不必要的服务

- dmesg查看启动的系统日志，对存在的问题优化

- 设置grub/etc/default/grub文件的GRUB_CMDLINE_LINUX_DEFAULT参数loglevel=0，开机时显示最详细的日志信息

- 如果gdm启动过慢，或许可以通过修改/etc/gdm/custom.conf文件添加
  
  [daemon]
  
  WaylandEnable=false
  
  禁用Wayland使用Xorg服务

### 一些外设的命令行配置

#### wifi

- 一般系统自带命令wpa,参考 [Arch linux wiki](https://wiki.archlinux.org/index.php/WPA_supplicant)
  
  1.通过`wpa_passphrase *MYSSID* *passphrase* > /etc/wpa_supplicant/example.conf`生成用户名密码配置文件
  
  2.通过`wpa_supplicant -B -i wlp1s0 -c /etc/wpa_supplicant/example.conf`连接wifi

- 通过安装[netctl](https://wiki.archlinux.org/index.php/Netctl)进行连接
  
  1.通过仿照目录/etc/netctl/examples/下配置生成配置文件到目录/etc/netctl/下
  
  2.通过命令`sudo netctl start home`连接wifi

#### 蓝牙

1.通过`sudo systemctl start bluetooth.service`启动蓝牙服务

2.安装[Bluetooth](https://en.wikipedia.org/wiki/Bluetooth)进行蓝牙控制，参考Bluetooth文档连接蓝牙

#### 硬盘

1.修改[/etc/fstab](https://en.wikipedia.org/wiki/Fstab)，对硬盘进行开机时挂载行为控制,一些常见的配置

- 对于外置硬盘如果插入开机自动挂载`UUID=******** /mnt/disk   ext4    defaults,nofail 0 2 `

2.通过`sudo mount -a`，安装让系统按照/etc/fstab进行挂载

#### 显示器

通过[Xrandr](https://wiki.archlinux.org/index.php/Xrandr)命令进行显示器的连接，一些常用的操作:

1.`xrandr --output HDMI-1 --mode 1920x1080 --pos 1920x0`在显示器右侧添加另一个显示器

2.`xrandr --output HDMI-1 --off`关闭HDMI-1接口的显示器

ps:可安装[Arandr](https://christian.amsuess.com/tools/arandr/)进行图像界面的操作

### 音频

1.安装alsa-utils,`sudo pacman install alsa-utils`

2.使用amixer命令控制

- `amixer`显示声卡信息

- `amixer sset Master unmute`解除静音

- `amixer sset Master 80%`调整Master声卡音量为80%

### 其他的一些有用的命令

#### 图片优化

1.安装[ImageMagick](https://wiki.archlinux.org/index.php/ImageMagick)

2.一些常用的命令

- `convert -sample 50%x50%  xxx.jpg  xxx-opt.jpg` 图片压缩,减小图片体积

- `convert  xxx.jpg  xxx.png` 图片转化
