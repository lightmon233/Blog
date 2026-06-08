---
title: "当你用neovim的mason插件管理lsp config，并且配置好bash的bashls后，却没有正常工作的解决方式"
date: 2023-12-14T19:16:06+08:00
draft: false
description: ""
categories: ["技术折腾"]
tags: ["neovim", "bash", "linux"]
---
{{< katex >}}


# 刚开始遇到这个情况我百思不得其解，检查了neovim checkhealth，以为是npm包管理的问题，然后删了下删了下
## 不但没有解决还把包管理整乱了……

<font color="red" size="5">
<del>
后来发现是我没仔细看bash-language-server这个包的官方文档。。。
</del>
</font>

## 以下是bash-language-server的官方仓库：
<https://github.com/bash-lsp/bash-language-server>
## 看到了dependency里面的shellcheck没？
我们需要的就是这个包，那么我们只需要
```bash
sudo pacman -S shellcheck
```
就好了
