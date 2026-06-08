---
title: "Arch系linux安装英伟达显卡驱动小问题（nvidia-smi failed）"
date: 2024-06-10T02:24:05+08:00
draft: false
description: ""
categories: ["技术折腾"]
tags: ["arch linux", "linux", "nvidia", "驱动"]
---
{{< katex >}}


可能你装的是dkms版本的驱动，这种一般要安装linux内核对应的headers，然后会自动安装模块。

比如，如果你用的是linux-zen，那么只要

```bash
sudo pacman -S linux-zen-headers
```

即可。
