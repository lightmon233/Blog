---
title: "ArchLinux长时间不更新永远会有的unknown trust报错解决方法汇总"
date: 2024-09-04T22:40:13+08:00
draft: false
description: ""
categories: ["技术折腾"]
tags: ["arch linux", "linux"]
---
{{< katex >}}


```bash
sudo pacman-key --refresh-keys

# enable ntp and ensure the time correct
timedatectl set-ntp 1
timedatectl status

rm -fr /etc/pacman.d/gnupg
# create pacman master key
pacman-key --init
# reload keys from keyring resources
pacman-key --populate

sudo rm /var/cache/pacman/pkg -r

pacman -Sy archlinux-keyring && pacman -Syu
```
