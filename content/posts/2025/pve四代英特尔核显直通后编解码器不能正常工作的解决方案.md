---
title: "pve四代英特尔核显直通后编解码器不能正常工作的解决方案"
date: 2025-05-07T14:01:18+08:00
draft: false
description: ""
---
{{< katex >}}


先上系统配置：
![image](/img/cnblogs/3349274-20250507135420152-747553747.png)
再上参考文章：
[PVE下黑群晖核显直通无法开启硬解的解决方法](https://foxi.buduanwang.vip/virtualization/pve/1582.html/)

使用`lspci -n`查看核显的`dev id`;
比如我的就是`040a`
![image](/img/cnblogs/3349274-20250507135652322-1885844346.png)

接着在要直通的虚拟机的`Hardware`栏中将直通的显卡项删掉；
![image](/img/cnblogs/3349274-20250507135811278-762066007.png)

编辑虚拟机配置文件：
```bash
# /etc/pve/qemu-server/100.conf
# 这里100换成你的虚拟机的id
# 这里x-pci-device-id的值要换成你的核显的id
args: -device vfio-pci,host=00:02.0,addr=0x02,x-pci-device-id=0x040A
# ...
```
![image](/img/cnblogs/3349274-20250507140100619-1720545689.png)
保存后重启虚拟机，核显的编解码器就可以正常调用了
![image](/img/cnblogs/3349274-20250507140212376-43583696.png)

ps:
`x-pci-devide-id`的值可以和宿主的不一样，它指定的是虚拟机中显卡的id，可以填和自己核显同型号的其他id；
如果这样操作解码器还是不工作的话，可以尝试把虚拟机的`Display`改为`Standard VGA(std)`