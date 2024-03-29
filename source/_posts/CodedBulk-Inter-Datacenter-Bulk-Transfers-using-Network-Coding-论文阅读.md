---
title: 'CodedBulk: Inter-Datacenter Bulk Transfers using Network Coding 论文阅读'
date: 2023-09-25 13:57:52
categories: 'Networking'
tags:
	- 'Congestion Control'
	- '笔记'
---

[CodedBulk](https://www.usenix.org/conference/nsdi21/presentation/tseng)

<!-- more -->

这篇论文是看 hostCC 作者主页的时候看到的，一打开就一个 bulk transfer，哈哈，还是看看吧。

## Abstract

本文介绍了用于数据中心间高吞吐量批量传输的系统 CodedBulk。CodedBulk 的核心是使用网络编码，这是编码理论界的一项技术，可保证单个 bulk 传输的最佳吞吐量。之前在有线网络中使用网络编码的尝试面临着一些实际和基本的障碍。CodedBulk 利用数据中心间网络的独特属性，并使用定制设计的逐跳流量控制机制，在现有传输协议上高效实现了网络编码，从而解决了这些障碍。与不使用网络编码的机制相比，在地理分布的数据中心间网络上运行的端到端 CodedBulk 实现了将批量传输吞吐量提高了 1.2-2.5 倍。

> 就是说他的主要方法是逐跳拥塞控制和网络编码，没听说过用这个途径解决的。

## Introduction

据预测，目前跨 DC 的 WAN 流量将在未来五年内变为现在的四倍。但是远距离传输的容量是受 [Shannon Limit](https://en.wikipedia.org/wiki/Noisy-channel_coding_theorem) 限制的。

> 这个 limit 是与功率相关的。

以前物理层进步很快，WAN 上运输的效率就随之增加，但是现在要 hit Shannon limit 了，所以增加不了了。

目前跨 DC 的 WAN 流量主要是大文件的跨地域复制（geo-replication）。比如视频，数据库等等。来做容灾，可用性和 CDN 这种。

> 3DC 容灾啥的都是很常见的方案了。字节就是 3DC 跨地理备份的，但是这三个 DC 离得好像不是很远？

研究者回顾了一个经典问题：给定一个网络拓扑，从一个数据源向多个目的 DC 传输数据的吞吐量最高的方法是什么？答案是使用网络编码，这是编码理论界的一项技术，它将经典的最大流量最小割定理推广到（组播）批量传输的情况中。

> 为啥啊.jpg，留下了不会通信的眼泪。

网络编码通过使用在网计算，保证了单批传输的最佳吞吐量。相比之下，使用不进行在网计算的机制来实现最佳吞吐量，则是一个 open problem。

网络编码在无线网络中已被成功应用。其在有线网络中的应用面临着一些实际和根本性的障碍。

> 感觉是硬拉来写论文的。

在现实方面，有三个挑战。首先，网络编码需要网络路由器对数据进行缓冲和计算，这就需要存储和计算资源。其次，在路由器上运行定义计算的「网络代码」不仅需要关于网络拓扑和单次传输的先验知识，而且无法扩展到拥有数百万路由器和链路的网络。最后，网络编码需要一个实体来控制终端主机和网络。

传统 ISP 网络这个可能做不了，但是跨 DC 的 WAN 上可能是可以做的。网络编码可以只在资源丰富的 DC 上做，要么在边缘路由器上，要么在服务器上，反正编码不产生性能降低。数据中心间广域网运营者已经部署了定制的边界路由器，因此在这些路由器上增加计算和存储资源，以利用可用的广域网带宽实现更高的吞吐量（这很昂贵，而且越来越难扩展），是一个很好的 trade off。

第二个和第三个挑战在跨 DC 网络上也可以解决，比如网络规模不大，也就几百个路由器，路由器支持 SDN，可以有先验知识。

> 其实就微软搞的 SDWAN。

传统的网络编码假设的是网络上就一条流，无论是前台流量（如交互式流量）还是后台流量（如其他批量传输）。更一般地说，网络编码假设所有链路的延迟和带宽相同，链路延迟和带宽随时间保持不变。但这一假设在实践中并不成立，例如，零星的高优先级交互式流量，以及数据中心间广域网上的多个并发批量传输会导致假设不成立。我们将此称为非对称链路问题。文章是通过逐跳流控解决的，在路由器上带个大 buffer，然后分给多个活跃流避免 buffer overflow。

> 感觉槽点很多啊，凭啥觉得分了就避免了，而且传个超大文件 buffer 一定能 buff 得住？要多大的 buffer，制造带这么大 buffer 的路由器要多少钱，感觉都是问题。

然而，使用逐跳流控机制进行网络编码会带来额外的限制：所有需要在路由器上进行编码的数据流都必须收敛到相同的速率。与此同时，路由器需要利用数据平面有限的存储和计算资源。研究者实现了期望的性质：

- 对于每条单独的批量传输，所有要编码的流都收敛到相同速率
- 对于共享链路的所有批量传输，通过链路的所有流量的速率都收敛到最大最小公平速率
- 链路无死锁

> 收敛到一个超低速率大伙一起低利用率是吧。

然后 CodedBulk 不需要改变底层，运行在 TCP 和 MPLS 都可以。

## Overview

前面没啥好说的，就在铺垫一些选路的事情，Fig 1 中斯坦纳树是次优的归因为求解斯坦纳树是 NP-H 的，而且一旦构建了所示的斯坦纳树，就无法再以实现更高吞吐量的方式打包其他斯坦纳树。

> 原文感觉说得挺抽象的，他用的术语跟我能理解的术语区别太大，比如那个 packing 就完全没理解是什么 packing。

如果是单源单汇的话，可以使用多路径，这就是一个最大流问题，单源多汇一直是 open problem，2000 年的时候，一篇论文指出对于组播，最大吞吐等于源到每一个汇的最小割的最小值。但是对于一般的有向图，到达一个节点并不是直接转发，而是要做一些计算之后再转发。

> 如果路途中做计算了肯定要损失时延，倒是时延不是很关心，主要是带宽能不能跑满。

比如想传两个包 $A$ 和 $B$，单源多汇，每个汇都收 $A$ 和 $B$，实际上网络编码传输的是 $A$ 和 $A\oplus B$，比如如果有些汇要接收 $B$，它就要同时收到 $A$ 和 $A\oplus B$ 才能解码出 $B$，有些还是往中间节点上传的，中间节点就要负责去解码然后转发。

> 挺扭曲的这个想法，在算力网传输也有个这样的问题，每个节点要带一个缓存，但是那个更复杂，还要加解密，总的来说十分的扭曲。

对于单次批量传输，大概可以分为如下五步：

1. 源节点收到批量传输请求后，会通知控制器进行批量传输。该通知包含源标识符、每个目的地的标识符，以及关于不使用某些链接或中间节点的可选策略（例如，出于安全和隔离目的）。
2. 控制器维护中心间网络的静态拓扑结构。虽然可以通过优化来利用实时流量信息，但目前的 CodedBulk 实现并没有使用这种优化。控制器会为每个目的地计算源和目的地之间的一组边不相交路径，以及每条路径的带宽。利用这些路径，控制器会使用网络编码算法计算批量传输的网络代码。网络代码包括每条流量的路由和转发信息，以及每个中间节点对到达该节点的流量所做的计算。这些编码可表示为四个基本功能的组合，即转发，组播，编码转发和编码组播。
3. 一旦计算出网络代码，控制器就会在参与批量传输的每个节点上安装网络代码。
4. 一旦安装了代码，控制器就会通知源。源将批量传输文件分割为多个子文件（由代码定义），然后使用 CodedBulk 启动批量传输。例如源可以将文件分为两个大小相等的子文件并传输它们。每个中间节点独立执行 CodedBulk 的逐跳流控机制。重要的是，这里的「跳」指的是网络拓扑图上的数据中心。CodedBulk 假定交互流量始终以最高优先级发送，并且需要两个额外的优先级。
5. 批量传输完成后，源会通知控制器。控制器定期从所有网络节点卸载不活跃的代码。

> 感觉思路还是一样的，只不过引入了网络编码，然后网络编码感觉也就那样，对我来说不是很 make sense。文章下面注释了，引用中的网络编码算法需要有向无环图作为输入。然而构造中的多路径集可能会导致有环。他们改了一下，然后引了一个他们自己的 repo，说的是 technical report，但是 repo 里没有明显的 report 全是代码，不知道去哪儿了。而且 controller 之外的文件夹一群 place holder，我不好说。

## Comment

这可能是第二篇我看不下去的论文。反正后面就是一样的，为了解决流不同步问题加缓存，缓存爆了就逐跳反压，用拥塞控制搞。最后在 200Mbps 链路上测，聚合后速率达到 600Mbps，当然测的情况是汇点少，汇点多速率也会变慢。总之感觉也就那样，还是之前提到的缓存效率和价格的问题，方法啥的感觉也没啥创新，虽然多了个网络编码，但是我没感觉到有啥突破性的优化。不用网络编码，单用一个 SD-WAN 加多路径最优选路会不会也能达到一样的效果，我感觉是可以的。
