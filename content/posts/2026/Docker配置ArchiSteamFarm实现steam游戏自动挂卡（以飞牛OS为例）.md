---
title: "Docker配置ArchiSteamFarm实现steam游戏自动挂卡（以飞牛OS为例）"
date: 2026-04-16T23:50:53+08:00
draft: false
description: ""
categories: ["技术折腾"]
tags: ["steam", "archisteamfarm", "ast", "自动化", "挂机"]
---
{{< katex >}}


❌ **首先，在Docker中运行ArchiSteamFarm是不被官方建议的行为（被视为[进阶安装方式](https://github.com/JustArchiNET/ArchiSteamFarm/wiki/Docker-zh-CN#docker)），因此着这种使用方式相比[普通桌面系统安装](https://github.com/JustArchiNET/ArchiSteamFarm/wiki/Docker-zh-CN#docker)使用会有更多bug和意想不到的行为，请提前做好心理准备，正如官方文档所说的，**

> **在Docker容器中运行ASF通常会带来一些新问题，您必须自己面对并解决。**

叠甲完成，回到正题。

既然是在Docker环境中部署，那首先先了解一下Docker的网络结构：

![image](/img/cnblogs/3349274-20260416235051916-351657116.png)

以飞牛OS为例，其Docker默认使用bridge模式，容器的虚拟接口看到的是`172.17.0.0/16`这个子网，而假设宿主机飞牛OS属于`192.168.18.0/24`这个子网；局域网内设备想要访问容器内开启的服务，就需要设置一层端口映射，把宿主机的端口映射成bridge子网范围内的一组ip+端口号，用宿主机的ip+端口访问容器内的服务。

接着下载容器的镜像，配置相关参数，用镜像创建**ASF**(ArchiSteamFarm)容器，就像每个docker用户都会做的那样：

![image](/img/cnblogs/3349274-20260416235813638-15946678.png)
![image](/img/cnblogs/3349274-20260416235859947-1960290655.png)

端口映射和卷映射都很自然地配置好，接着启动ASF容器，你会发现：
**竟然打不开网页！**

![image](/img/cnblogs/3349274-20260417000327320-129516476.png)


## 坑一 打不开服务页面

别慌，让我们不要太过于相信自己的经验，而是回到`ArchiSteamFarm`的官方文档，来到 Docker - IPC 这一栏，可以看到里面来了句：

> [您必须额外做两件事，才能使 IPC（基于Kestrel HTTP服务端的“ASF Web 接口”，作为最终用户使用的前端或者第三方集成工具使用的后端与ASF进程进行交互） 在 Docker 中正常工作](https://github.com/JustArchiNET/ArchiSteamFarm/wiki/Docker-zh-CN#ipc)，

即：

### 修改掉默认的监听地址`localhost`

文章开头回顾了docker的网络架构，ASF的默认监听地址是`localhost`（包括docker版），而Docker无法将外部流量路由到环回接口，所以我们需要把默认的监听地址`localhost`修改掉，那该怎么修改呢？文档中也给到了这个自定义ipc配置：

```json
## /path/to/your/config/IPC.config
{
    "Kestrel": {
        "Endpoints": {
            "HTTP": {
                "Url": "http://*:1242"
            }
        }
    }
}
```
（这里的`*`不可以换成`192.168.18.102`，因为容器虚拟接口不含这个宿主ip）
修改完把这个`IPC.config`复制到装载路径为`/app/config/`的本地路径下后重启容器即可打开web界面。

可是这时打开web界面会发现下方提示：

![image](/img/cnblogs/3349274-20260417002650263-2140920326.png)

### 设置`IPCPassword`或者在自定义`IPC.config`中修改默认的`KnownNetworks`

因为`IPCPassword`需要在另一个文件里设置，所以为了省事我选择在`IPC.config`中加上了`KnownNetworks`的配置：

```json
{
  "Kestrel": {
    "Endpoints": {
      "HTTP": {
        "Url": "http://*:1242"
      }
    },
    "KnownNetworks": [
       "172.17.0.0/16",
       "192.168.18.0/24"
    ]
  }
}
```

重启容器即可正常打开web页面：

![image](/img/cnblogs/3349274-20260417004231655-967386507.png)

## 坑二 ASF无法登陆steam

![image](/img/cnblogs/3349274-20260417111542676-1475191770.png)

新建`bot`，输入`Name`，`SteamLogin`，`SteamPassword`; `Enabled`设为`√`后，手机steam客户端批准登陆后，`ASF_ui`报错`HandleLoginResult() Unable to login to Steam: TryAnotherCM/Invalid`，如下日志输出所示：

![image](/img/cnblogs/3349274-20260417112154792-776365629.png)

其实这里我们可以注意到在这个报错前还有个warning，让我们确认已在steam手机客户端上确认登陆或直接使用steam令牌验证码。
我们尝试在这里输入Y/N，可以发现没有任何反应，因为这里是日志界面，不是终端。

这里我们就可以发现问题所在了，其实ASF的终端在等待我们的输入来确认下一步干什么，但是我们压根无法输入，所以ASF也就无法确认如何登陆steam了。

> PS：如果这里是在普通桌面客户端环境下的话，不会有任何问题，因为桌面环境会打开一个终端给我们输入。

那么问题来了，在`ASF_ui`的web页面下，我们该如何输入呢？

这个问题其实在官方文档的 [Docker - 高级技巧](https://github.com/JustArchiNET/ArchiSteamFarm/wiki/Docker-zh-CN#%E9%AB%98%E7%BA%A7%E6%8A%80%E5%B7%A7) 一栏中写出了：

> 通常，您应该在全局配置文件中设置Docker容器内的ASF运行于`Headless: true`模式。 这会明确告诉ASF，您无法直接提供它所需的数据，它不应该在命令行中直接询问这些。

于是我们来到`ASF配置`页，将`Headless`启用：

![image](/img/cnblogs/3349274-20260417115129619-757615322.png)

保存配置，等待ASF自动重启。

然后我们可以发现日志中的输出已经出现了变化：

![image](/img/cnblogs/3349274-20260417115328541-1117418419.png)

这时我们启动机器人，就已经出现了让我们输入2FA代码的提示框，从手机steam客户端获取2FA代码后输入即可上线机器人。

机器人已变为在线状态：

![image](/img/cnblogs/3349274-20260417115600027-1547393567.png)

---

之后我们设置下挂卡选项即可启动自动挂卡：

![image](/img/cnblogs/3349274-20260417115701363-110616816.png)

![image](/img/cnblogs/3349274-20260417115822375-779363109.png)
