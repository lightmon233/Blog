---
title: "PVE英特尔核显直通给Windows输出视频和音频"
date: 2025-05-18T23:16:06+08:00
draft: false
description: ""
---
{{< katex >}}


> 本方案在四代英特尔平台实测可行，理论上适用于4~10代英特尔cpu

首先确保你成功直通了核显，通过远程桌面可以在任务管理器中看到你的核显（需要核显支持WDDM驱动）

其次你的虚拟机型号得是`q35`，~~实测在我这里选`i440fx`会导致没声音~~（
后续发现选`q35`可以通过hdmi输出声音，选`i440fx`可以通过机箱前面板输出声音（需要调一下虚拟机内部音频驱动的设置）
）

将核显视频和音频设备添加到直通组（如我这里核显音频设备id是`8086:0c0c`，视频设备是`8086:040a`，其中`8086:8c20`是我的板载音频设备id）

![image](/img/cnblogs/3349274-20250518230141382-1807368755.png)

重启PVE节点

从[这里](https://www.123pan.com/s/20P0Vv-d2A6H.html?notoken=1)下载魔改的英特尔核显vbios，或者[自行修改自己的vbios](https://github.com/patmagauran/i915ovmfPkg)，并使用`scp`之类的工具将其放到pve节点的`/usr/share/kvm/`路径下

修改Windows虚拟机的参数如下：

![image](/img/cnblogs/3349274-20250518231342592-990074404.png)

主要是添加/修改了以下三行：
```bash
args: -set device.hostpci0.addr=02.0 -set device.hostpci0.x-igd-gms=0x2 -set device.hostpci0.x-igd-opregion=on
hostpci0: 0000:00:02.0,romfile=intel.rom
vga: none
```
其中`intel.rom`的路径就是`/usr/share/kvm/intel.rom`

至此，直通核显的视频输出问题就已经解决，
接下来只需要正常在web界面添加音频的pci设备给虚拟机即可解决音频输出问题：

![image](/img/cnblogs/3349274-20250518231746815-2029369867.png)

- 如果你后续测试发现虚拟机容易死机并且带着宿主机一起死机，可以尝试以下配置：
```bash
agent: 1
args: -set device.hostpci0.addr=02.0 -set device.hostpci0.x-igd-gms=0x2 -set device.hostpci0.x-igd-opregion=on
bios: ovmf
boot: order=scsi0;ide2;net0
cores: 8
cpu: host
efidisk0: local:104/vm-104-disk-0.qcow2,efitype=4m,pre-enrolled-keys=1,size=528K
hostpci0: 0000:00:02.0,romfile=intel.rom,legacy-igd=on
hostpci1: 0000:00:03.0,rombar=0
hostpci2: 0000:00:1b.0,rombar=0
ide2: local:iso/virtio-win-0.1.271.iso,media=cdrom,size=709474K
machine: pc-i440fx-8.1
memory: 4096
meta: creation-qemu=8.1.5,ctime=1729582691
name: Windows
net0: virtio=BC:24:11:54:4F:AB,bridge=vmbr0,firewall=1
numa: 1
onboot: 1
ostype: win10
scsi0: local:104/vm-104-disk-1.qcow2,iothread=1,size=64G,ssd=1
scsihw: virtio-scsi-single
smbios1: uuid=eaf316a2-c59f-4622-863a-31abb96329cd
sockets: 1
usb0: host=1a2c:9b03
usb1: host=04b3:310c
vga: none
vmgenid: 31789ab5-0b47-449f-b4f3-13833e5baa7b
```
即把机器类型改为`i440fx`，同时给显卡加上`legacy-igd=on`

- 如果发现音频无输出，可以尝试把直通音频设备的`ROM-bar`的勾给去掉：
![image](/img/cnblogs/3349274-20250522195803821-1235365957.png)
