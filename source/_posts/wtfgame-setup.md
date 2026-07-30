---
title: WTFGame 开服与运维
categories: 'DevOps'
toc: false
date: 2021-05-19 21:34:56
tags:
	- 运维
---

纯怀旧服：124.70.22.189（已经倒了）

<!-- more -->

WTFGame 是一款经典的网页小游戏，我们一般称之为怼猫，或者猫猫互怼，玩法是用上下左右控制猫猫移动，按空格可以抓人，Q 键可以进行玩家互动。经典模式的时候当屏幕闪烁红光时按向下键趴下，按向上跳跃，控制猫到达红旗地点，瘟疫模式有点类似糖豆人的瘟疫模式，适合多人竞技，也算是挺好玩的。

怎么要素这么像糖豆人草……

这款游戏是高中的时候看飙车队玩的，自己玩得挺少，因为那个时候我不怎么打游戏，跟个逆天一样。但是网络冲浪的时候发现这款游戏竟然是开源的，里面还有源码包和开服包（在[这里](https://github.com/wheatup/wtfgame)）。于是打算自己开个私服玩。

大概的技术是，前端是核心游戏客户端，调用 Qici 引擎实现了所有场景，后端用 Java 写的，使用 Websocket 使得可以多人联机。所以就变得很简单，前端 NGINX 代一下，然后搭一个 Websocket 服务器，把后端 war 包挂上就行了。

现在的大概结构是，搞个 NGINX 把前端挂上，然后搞个支持 Websocket 的服务器把 war 包挂上就完事。准备好开服包和服务器，然后就开始吧。

在 Ubuntu 下搭建 NGINX 参考[这里](https://ubuntu.com/tutorials/install-and-configure-nginx)，就照着样例做就好了，注意搭建端口可以搭在除了 8080 以外的任何端口，因为后端要用。

注意之前 NGINX 搭好后的 Welcome 页是搭在 80 端口的，如果前端也搭在 80 端口的话会冲突，这时候注意修改 Welcome 页的端口，或者直接把它干掉也可。

参考 war 包里的配置，后端大概一定要搭在 Glassfish 上，因为配置文件都写好了。但是 Glassfish 真的感觉是一个已经被废弃的技术，但是还是用 Glassfish 了。其他的服务器比如 Tomcat 都尝试失败了……

Glassfish 部署 war 包使用 `asadmin deploy` 部署即可。

开服说明里还写了要改 index.html，将 `window['address']` 一栏后面的值填写为服务器地址。根据 war 包里的 Glassfish 的配置文件和后端写法，如果没对 war 包里内容进行修改的话，Websocket 应该访问的地址为：

``` plain
ws://<your_domain>:<glassfish_port>/TestServer/server
```

一般来说，默认 Glassfish 是跑在 8080 端口的。根据这个配置即可。

后端搭好后，建议用 Websocket 测试工具先测一下 Websocket 是不是通的，要不然会一直卡在连接服务器。

加载前端时，如果卡 29%，是在加载游戏主场景包，等一段时间就好，如果实在太卡请考虑加服务器带宽……

个人所用的环境是华为云鲲鹏服务器，配置：

- CPU：2 vCPUs，Huawei Kunpeng 920 2.6GHz，AArch64
- RAM：4G
- Network：1Mbps

ARM YES!

前端使用 NGINX，后端使用 Glassfish 5.0.1 搭建成功。

已知问题是游戏介绍页无法滚动……

以上。
