---
title: "使用Systemd创建开机登录前自启动脚本服务并自动登录"
date: 2024-10-10T15:57:58+08:00
draft: false
description: ""
categories: ["工具脚本"]
tags: ["systemd"]
---
{{< katex >}}


# 开机登录前自启动脚本服务

首先确定你的系统是否使用`systemd`来管理系统服务，在shell中输入`systemctl`命令来判断，有输出则为`systemd`系统。

进入`/etc/systemd/system`目录，创建`myservice.service`，其中`myservice`是你要自定义的服务名。

编辑`myservice.service`文件，修改其内容为：

```bash
[Unit]
Description=my service unit # 填写你的service描述
After=docker.service # 表示本服务依赖于 docker.service，即 docker.service 必须先启动，本服务才能启动。
Before=getty@tty1.service # 表示本服务必须在 getty@tty1.service 启动之前启动，即在登录界面显示之前启动

[Service]
Type=forking # "forking"表示脚本执行后父进程退出，子进程在后台运行，适合daemon脚本
             # "simple"表示普通脚本
             # "oneshot"表示脚本执行后会立即退出，适合纯命令类型的脚本
ExecStart=/path/to/your_sript # 你要执行的脚本的路径
# User=root # 如果你想要脚本在登录后再执行，可以加上这一项
StandardOutput=tty # 在tty中打印脚本的标准输出
StandardError=tty # 在tty中打印脚本的error信息

[Install]
WantedBy=getty.target # 表示本服务希望被 getty.target 所依赖。getty.target 是一个目标单元，表示系统进入多用户状态
```

接着，执行
```bash
sudo systemctl enable myservice.service
```
重启后，即可生效配置。

或者
```
sudo sytemctl daemon-reload && sudo systemctl enable myservice.service --now
```
以立即使配置生效。

# 自动登录脚本

修改`/etc/systemd/system/getty.target.wants/getty@tty1.service`或者
`/etc/systemd/system/getty@tty1.service.d/autologin.conf`

更改`ExecStart=...`这一行为：
```bash
ExecStart=-/sbin/agetty -o '-p -f -- \\u' --noclear --autologin username - $TERM
```
即新增`-f --autologin username`这三个参数，注意这三个参数的位置，`-f`参数在`-o ''`内部。
-/sbin/agetty：指定 agetty 命令的路径。这里的 - 表示使用绝对路径，确保命令被正确找到。
-o '-p -f -- \u'：设置 agetty 的选项。
-p：在终端显示提示符。
-f：强制终端进入原始模式。
--：表示命令行选项结束。
\u：显示用户名。
--noclear：在登录后不清除屏幕，保留之前的输出。
--autologin username：自动登录指定用户 username。
-$TERM：使用当前终端类型的设置。
重启以使得修改生效。

## 为什么已经创建了登录前自启动任务还要设置自动登录？

实测如果不设置自动登录，虽然脚本会显示成功运行，但是实际上似乎有问题，猜测可能是有部分网络相关的服务会在登录后才会启用。
