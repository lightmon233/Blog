---
title: "Debian 13 (PVE内核) 下 Intel e1000e 网卡间歇性 “Hardware Unit Hang” 断网问题原因与解决"
date: 2026-04-26T16:52:59+08:00
draft: false
description: ""
---
{{< katex >}}


> :warning: 本文大量引用自鱼丸粗面的[这篇文章](https://zhangmin.vip/index.php/2025/12/05/%E3%80%90%E5%AE%9E%E6%88%98%E8%AE%B0%E5%BD%95%E3%80%91%E8%A7%A3%E5%86%B3-debian-13-pve%E5%86%85%E6%A0%B8-%E4%B8%8B-intel-e1000e-%E7%BD%91%E5%8D%A1%E9%97%B4%E6%AD%87%E6%80%A7-hardware-unit-hang/)

我的网卡是I217-LM，用的是e1000e的驱动，故障日志为：

```bash
Apr 26 16:03:44 pve kernel: e1000e 0000:00:19.0 nic0: NIC Link is Down
Apr 26 16:03:44 pve kernel: vmbr0: port 1(nic0) entered disabled state
Apr 26 16:03:47 pve kernel: e1000e 0000:00:19.0 nic0: NIC Link is Up 1000 Mbps Full Duplex, Flow Control: Rx/Tx
Apr 26 16:03:47 pve kernel: vmbr0: port 1(nic0) entered blocking state
Apr 26 16:03:47 pve kernel: vmbr0: port 1(nic0) entered forwarding state
Apr 26 16:04:07 pve kernel: e1000e 0000:00:19.0 nic0: Detected Hardware Unit Hang:
TDH <0>
TDT <3>
next_to_use <3>
next_to_clean <0>
buffer_info[next_to_clean]:
time_stamp <105cdabd7>
next_to_watch <0>
jiffies <105cdb6c0>
next_to_watch.status <0>
MAC Status <80083>
PHY Status <796d>
PHY 1000BASE-T Status <3800>
PHY Extended Status <3000>
PCI Status <10>
Apr 26 16:04:08 pve kernel: e1000e 0000:00:19.0 nic0: NIC Link is Down
```

## 故障原因

> 这是 Linux 内核中 e1000e 驱动的一个经典 Bug，在较新的内核版本（尤其是 5.15+ 到 6.x）配合特定批次的 Intel I219 系列网卡时极易触发。

> 网卡的 TCP 分段卸载 (TSO, TCP Segmentation Offload) 和通用分段卸载 (GSO) 功能在处理高并发或特定数据包时，可能导致网卡的环形缓冲区 (Ring Buffer) 指针计算错误或死锁，导致网卡硬件挂起并尝试重置。

## 解决方案

以我的网卡名`nic0`为例

### 临时

```bash
sudo ethtool -K nic0 tso off gso off gro off
```

### 持久化

编辑`/etc/network/interfaces`：

```bash
auto lo
iface lo inet loopback

iface nic0 inet manual
        post-up /usr/sbin/ethtool -K nic0 tso off gso off gro off #在你的网卡模块下添加此行，注意tab。

iface nic1 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.10.200/24
        gateway 192.168.10.1
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0

source /etc/network/interfaces.d/*
```

## 补充

> 如果关闭 Offload 后仍偶发断网，可能是电源管理 (ASPM) 问题。可修改 GRUB 配置：

1. 编辑 `/etc/default/grub`，在 `GRUB_CMDLINE_LINUX_DEFAULT` 中追加：`pcie_aspm=off`
2. 更新 GRUB：`update-grub`。