---
title: "go-musicfox打开报错ALSA lib pcm_dmix.c_1000_(snd_pcm_dmix_open) unable to open slave, 且声音会出现莫名其妙的卡顿、声音行为奇怪的解决方案"
date: 2024-09-14T00:23:55+08:00
draft: false
description: ""
---
很有可能你同时装了Hyprland和KDE plasma两种桌面环境，其中一个用pipewire作为音频管理器，另一个用pulseaudio作为音频管理器，这两者会发生冲突，导致奇怪的音频问题出现。

解决方案：
安装pipewire-pulse以替换掉pulseaudio, 解决音频管理器的冲突即可。