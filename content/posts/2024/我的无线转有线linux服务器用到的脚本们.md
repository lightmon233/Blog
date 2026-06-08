---
title: "我的无线转有线linux服务器用到的脚本们"
date: 2024-11-28T18:10:17+08:00
draft: false
description: ""
categories: ["工具脚本"]
tags: ["linux", "shell", "软路由"]
---
{{< katex >}}


#### [lnxrouter](https://github.com/garywill/linux-router/blob/master/lnxrouter)
`/home/light/Scripts/lnxrouter`

#### route.sh
`/home/light/Scripts/route.sh`
```bash
#!/bin/bash
sudo /home/light/Scripts/lnxrouter -i enp2s0 -g 0 --daemon
```

#### route.service
`/etc/systemd/system/route.service`
```bash
[Unit]
Description=run lnxrouter
Before=getty@tty1.service

[Service]
Type=forking
ExecStart=/home/light/Scripts/route.sh
StandardOutput=tty
StandardError=tty

[Install]
WantedBy=getty.target
```

#### getty@tty1.service
`/etc/systemd/system/getty.target.wants/getty@tty1.service`
```bash
#  SPDX-License-Identifier: LGPL-2.1-or-later
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Getty on %I
Documentation=man:agetty(8) man:systemd-getty-generator(8)
Documentation=https://0pointer.de/blog/projects/serial-console.html
After=systemd-user-sessions.service plymouth-quit-wait.service getty-pre.target
After=rc-local.service

# If additional gettys are spawned during boot then we should make
# sure that this is synchronized before getty.target, even though
# getty.target didn't actually pull it in.
Before=getty.target
IgnoreOnIsolate=yes

# IgnoreOnIsolate causes issues with sulogin, if someone isolates
# rescue.target or starts rescue.service from multi-user.target or
# graphical.target.
Conflicts=rescue.service
Before=rescue.service

# On systems without virtual consoles, don't start any getty. Note
# that serial gettys are covered by serial-getty@.service, not this
# unit.
ConditionPathExists=/dev/tty0

[Service]
# the VT is cleared by TTYVTDisallocate
# The '-o' option value tells agetty to replace 'login' arguments with an
# option to preserve environment (-p), followed by '--' for safety, and then
# the entered username.
ExecStart=-/sbin/agetty -o '-p -f -- \\u' --noclear --autologin light - $TERM
Type=idle
Restart=always
RestartSec=0
UtmpIdentifier=%I
StandardInput=tty
StandardOutput=tty
TTYPath=/dev/%I
TTYReset=yes
TTYVHangup=yes
TTYVTDisallocate=yes
IgnoreSIGPIPE=no
SendSIGHUP=yes

# Unset locale for the console getty since the console has problems
# displaying some internationalized messages.
UnsetEnvironment=LANG LANGUAGE LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT LC_IDENTIFICATION

[Install]
WantedBy=getty.target
DefaultInstance=tty1
```
