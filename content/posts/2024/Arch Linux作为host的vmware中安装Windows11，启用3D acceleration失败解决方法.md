---
title: "Arch Linux作为host的vmware中安装Windows11，启用3D acceleration失败解决方法"
date: 2024-07-02T18:39:44+08:00
draft: false
description: ""
categories: ["技术折腾"]
tags: ["vmware"]
---
{{< katex >}}


解决方案来自<https://gist.github.com/plembo/f0767e4fbcd42c6c98f8271c15ee785d?ref=techhut.tv>

首先确保宿主机开启了3d加速，并且客户机安装了vmware tools。

编辑`~/.vmware/preferences`

在最后加上一行
```bash
mks.gl.allowBlacklistedDrivers = "TRUE"
```

重启虚拟机即可。
