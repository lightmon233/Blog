---
title: "openwrt路由器lan口莫名其妙断网的补丁式解决方案"
date: 2026-04-25T21:51:35+08:00
draft: false
description: ""
---
{{< katex >}}


> :warning: 本文思路来源于[这篇文章](https://zhuanlan.zhihu.com/p/607551263)，问题原因见[此帖](https://www.cnblogs.com/lightmon5210/p/19933148)。

在宿舍部署的基于PVE虚拟机方案的openwrt软路由一直以来都工作得十分正常。然而，这个学期开始的时候，我尝试开机，主板上的蜂鸣器发出长声嘀（没有间隔）。虽然根据[联想知识库的说明](https://iknow.lenovo.com.cn/detail/097650)来看应该是显卡和显示器未插好导致的，但是经过反复尝试无果后我选择换了张主板（B85->H81）。
- 在H81台式机这个阶段，会出现openwrt时不时死机的bug，而且会顺带着宿主机一起死机，需要强制重启来解决。

后来，扔了台式机，换了HP 800g1 dm这个小主机。
- 在小主机阶段，会出现lan口频繁断网的bug，并不会导致openwrt和宿主机死机，只需重新拔插小主机和ap之间的网线即可解决。

这两个阶段的问题，共同的表现是lan口的服务暂停，上网设备无法获取dhcp服务下发的ipv4地址：

![18016dacb33b5e4bbc550153c6f48c2d](/img/cnblogs/3349274-20260425214957145-1118340392.jpg)

一般来说，遇到问题后应该去查看系统日志，然而我翻看了pve和openwrt的日志，均找不到相关日志，于是，就有了这篇博客的内容：用打补丁的方式临时解决一下这个问题。

## 网络环境

因为我是基于PVE的虚拟机方案，所以有必要先介绍一下我的网络环境，如下图所示：

![image](/img/cnblogs/3349274-20260425223155476-780037649.png)

经过实验，当`ETH0`接口（pve的lan口）掉ip（断网）的时候，需要拔插`ETH0`和`AP`（ap的lan口）之间的网线，才能解决问题；但是由于这个现象十分频繁，一直手动拔插十分不现实，因此决定写脚本使得当pve的lan口断网时，自动重启lan口。

## PVE操作

### 脚本

```bash
#!/bin/bash
#/usr/local/bin/net_watchdog.sh

TARGET="192.168.10.119" # 你的AP ip

while true; do
    # 检测网络
    if ! ping -c 3 -W 2 $TARGET > /dev/null; then
        echo "$(date) - Network down! Restarting..." >> /var/log/net_watchdog.log
        ifdown nic0 && ifup nic0 # 或者你的重启网络命令
        sleep 20        # 重启后等待20秒，防止频繁抖动
    else
        sleep 5         # 每5秒检测一次，降低CPU占用
    fi
done
```

### 添加执行权限

```bash
chmod +x /usr/local/bin/net_watchdog.sh
```

### 创建 Systemd 服务文件

```toml
#/etc/systemd/system/net-watchdog.service

[Unit]
Description=Network Watchdog Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/net_watchdog.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### 启动并启用服务

```bash
systemctl daemon-reload
systemctl start net-watchdog
systemctl enable net-watchdog
```

搞定之后，就可以在`/var/log/net_watchdog.log`文件中查看lan口的重启信息了：

![image](/img/cnblogs/3349274-20260425224626200-151218029.png)

可以看到我这里重启得非常频繁……😭

## 附：openwrt脚本（如果你是openwrt实体机的话）

```bash
#!/bin/bash

TARGET="192.168.10.119" # 替换成你的ap lan口ip

if /bin/ping -c 3 -W 2 $TARGET > /dev/null; then
    exit 0
else
    logger "Network unreachable! Restarting LAN interface..."
    /sbin/ifup lan
    sleep 10
fi
```

注意添加可执行权限。

可以通过openwrt自带的计划任务来启用脚本，比如：

```bash
crontab -e
```

添加以下内容：

```bash
* * * * * /bin/bash /root/check_net.sh >/dev/null 2>&1
```

重启openwrt，脚本激活时系统日志就会有类似如下的输出：

```bash
Sat Apr 25 18:11:00 2026 cron.err crond[10729]: USER root pid 16297 cmd /bin/bash /root/check_net.sh >/dev/null 2>&1
Sat Apr 25 18:11:04 2026 user.notice root: Network unreachable! Restarting LAN interface...
Sat Apr 25 18:11:04 2026 daemon.notice netifd: Interface 'lan' is now down
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Registering new address record for fe80::be24:11ff:fe91:8b90 on br-lan.*.
Sat Apr 25 18:11:04 2026 daemon.notice ttyd[12083]: [2026/04/25 18:11:04:2047] N: rops_handle_POLLIN_netlink: DELADDR
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Withdrawing address record for 192.168.10.1 on br-lan.
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Leaving mDNS multicast group on interface br-lan.IPv4 with address 192.168.10.1.
Sat Apr 25 18:11:04 2026 daemon.err odhcpd[3926]: Failed to send to ff02::1%lan@br-lan (Network unreachable)
Sat Apr 25 18:11:04 2026 daemon.notice ttyd[12083]: [2026/04/25 18:11:04:2064] N: rops_handle_POLLIN_netlink: DELADDR
Sat Apr 25 18:11:04 2026 daemon.notice ttyd[12083]: [2026/04/25 18:11:04:2065] N: rops_handle_POLLIN_netlink: DELADDR
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Interface br-lan.IPv4 no longer relevant for mDNS.
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Interface br-lan.IPv6 no longer relevant for mDNS.
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Leaving mDNS multicast group on interface br-lan.IPv6 with address fc00::1.
Sat Apr 25 18:11:04 2026 kern.info kernel: [   84.202900] br-lan: port 1(eth0) entered disabled state
Sat Apr 25 18:11:04 2026 kern.info kernel: [   84.205774] device eth0 left promiscuous mode
Sat Apr 25 18:11:04 2026 kern.info kernel: [   84.206924] br-lan: port 1(eth0) entered disabled state
Sat Apr 25 18:11:04 2026 user.warn mwan3-hotplug[16327]: hotplug called on lan before mwan3 has been set up
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Withdrawing address record for fe80::be24:11ff:fe91:8b90 on br-lan.
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Withdrawing address record for fc00::1 on br-lan.
Sat Apr 25 18:11:04 2026 daemon.notice netifd: Interface 'lan' is disabled
Sat Apr 25 18:11:04 2026 daemon.notice netifd: bridge 'br-lan' link is down
Sat Apr 25 18:11:04 2026 daemon.notice netifd: Interface 'lan' has link connectivity loss
Sat Apr 25 18:11:04 2026 daemon.notice netifd: Network device 'eth0' link is down
Sat Apr 25 18:11:04 2026 kern.info kernel: [   84.435162] 8021q: adding VLAN 0 to HW filter on device eth0
Sat Apr 25 18:11:04 2026 kern.info kernel: [   84.436686] br-lan: port 1(eth0) entered blocking state
Sat Apr 25 18:11:04 2026 kern.info kernel: [   84.437839] br-lan: port 1(eth0) entered disabled state
Sat Apr 25 18:11:04 2026 kern.info kernel: [   84.439124] device eth0 entered promiscuous mode
Sat Apr 25 18:11:04 2026 kern.info kernel: [   84.441441] br-lan: port 1(eth0) entered blocking state
Sat Apr 25 18:11:04 2026 kern.info kernel: [   84.442787] br-lan: port 1(eth0) entered forwarding state
Sat Apr 25 18:11:04 2026 daemon.notice netifd: Interface 'lan' is enabled
Sat Apr 25 18:11:04 2026 daemon.notice netifd: Interface 'lan' is setting up now
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Joining mDNS multicast group on interface br-lan.IPv4 with address 192.168.10.1.
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: New relevant interface br-lan.IPv4 for mDNS.
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Registering new address record for 192.168.10.1 on br-lan.IPv4.
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Joining mDNS multicast group on interface br-lan.IPv6 with address fc00::1.
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: New relevant interface br-lan.IPv6 for mDNS.
Sat Apr 25 18:11:04 2026 daemon.info avahi-daemon[4398]: Registering new address record for fc00::1 on br-lan.*.
Sat Apr 25 18:11:04 2026 daemon.notice netifd: Interface 'lan' is now up
Sat Apr 25 18:11:04 2026 daemon.notice netifd: Network device 'eth0' link is up
Sat Apr 25 18:11:04 2026 daemon.notice netifd: bridge 'br-lan' link is up
Sat Apr 25 18:11:04 2026 daemon.notice netifd: Interface 'lan' has link connectivity
Sat Apr 25 18:11:04 2026 user.warn mwan3-hotplug[16463]: hotplug called on lan before mwan3 has been set up
Sat Apr 25 18:11:04 2026 user.notice firewall: Reloading firewall due to ifup of lan (br-lan)
Sat Apr 25 18:11:06 2026 daemon.warn odhcpd[3926]: A default route is present but there is no public prefix on lan thus we don't announce a default route by overriding ra_lifetime!
Sat Apr 25 18:11:10 2026 user.notice nlbwmon: Reloading nlbwmon due to ifup of lan (br-lan)
Sat Apr 25 18:11:19 2026 daemon.warn odhcpd[3926]: A default route is present but there is no public prefix on lan thus we don't announce a default route by overriding ra_lifetime!
Sat Apr 25 18:11:22 2026 daemon.warn odhcpd[3926]: A default route is present but there is no public prefix on lan thus we don't announce a default route by overriding ra_lifetime!
Sat Apr 25 18:11:23 2026 daemon.warn odhcpd[3926]: A default route is present but there is no public prefix on lan thus we don't announce a default route by overriding ra_lifetime!
Sat Apr 25 18:11:27 2026 daemon.warn odhcpd[3926]: A default route is present but there is no public prefix on lan thus we don't announce a default route by overriding ra_lifetime!
Sat Apr 25 18:11:30 2026 daemon.err uhttpd[4067]: luci: accepted login on /admin for root from 192.168.10.81
```

> 整理时发现一个有趣的事：PVE下的`ifup`命令和openwrt下的`ifup`命令是不同的实现，前者只有当lan口未正常运行时才会启动lan口，后者几乎是强制重启lan口；因此PVE的重启lan口命令需要在前面加一个`ifdown`。