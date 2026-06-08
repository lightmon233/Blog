---
title: "pve安装后删除local-lvm并把其空间全部分给local"
date: 2024-10-18T19:19:54+08:00
draft: false
description: ""
---
{{< katex >}}


在安装pve的时候，系统默认分配给local的空间非常小，我们可以通过以下方法把local-lvm删除，并将其空间还给local。

注意！

在webui的`pve`节点的`磁盘`选项中找到`LVM-Thin`，删除`data`卷。

![image](/img/cnblogs/3349274-20241018191708327-1281376262.png)

删除后此处为空。

接着打开终端执行以下命令：

```bash
lvresize --extents +100%FREE --resizefs pve/root
```

此命令会将剩余空间全部分配给local。

执行完后也可以继续执行下述命令以重新4k对齐。

```bash
resize2fs /dev/mapper/pve-root
```

调整后的空间：

![image](/img/cnblogs/3349274-20241018191939942-1819696149.png)

点击数据中心-存储，编辑local，把所有内容都勾选。