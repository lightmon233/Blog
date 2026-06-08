---
title: "Thinkpad X230安装Arch Linux修复键盘无法调节屏幕亮度"
date: 2024-09-15T21:38:00+08:00
draft: false
description: ""
---
{{< katex >}}


在`/etc/default/grub`中给`GRUB_CMDLINE_LINUX_DEFAULT`
添加参数：
```bash
acpi_osi='!Windows 2012'
```
然后使用命令重新生成grub配置：
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
重启即可