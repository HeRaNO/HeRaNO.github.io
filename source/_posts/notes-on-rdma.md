---
title: RDMA 相关梳理
date: 2024-03-14 15:15:38
categories: 'Networking'
tags:
	- '笔记'
description: ' '
---

总之就是 RDMA 相关的一些梳理，但是还是比较乱。

~~随手整理的，之后讨论发现基本没什么用，还是太屑了~~

## 定义问题

1. 拥塞：一般来说交换机 buffer 无法容纳网络流量造成丢包叫拥塞，但是现在更关注 QoS 了，交换机排队过长导致时延过高也叫拥塞了（更广义）。一般使用后面的定义，理解起来类似 BBR 最佳操作点的那张图。如果使用前面的定义，在 RDMA 拥塞控制上就会理解偏差了。
2. 无损：在网络运行中不拥塞叫无损，这里的拥塞采用前面的定义，就是不丢包。
3. 我们希望的：高吞吐，低时延
4. 为什么做不到：拥塞（后面的定义）

## IB vs RoCE

IB 包括从物理层到网络层的标准和实现，无损网络。不兼容以太网。

RoCE 协议可以跑在以太网上。

IB 没啥好写的，大概就是 credit base 流控什么的，闭源，技术细节不太公开，Switch to RoCE context

## Lossless Ethernet vs Lossy Ethernet

这里的 `loss` 指丢包，但是实际上指的是因为交换机拥塞而丢包。因包损坏而丢包不算在其中。

本身以太网就是 Lossy 的，Lossless 指不会因为交换机拥塞而丢包。

Lossless Ethernet 一定不丢包吗？否：包损坏，但以太网环境下概率很小，[WiFi](https://ieeexplore.ieee.org/document/1404595) 的时候常发生。交换机配置错误，物理链路损坏也可能导致包损坏，所以在设计时也应该考虑进行重传等操作，只不过 hit 的情况少而已。

Lossless Ethernet 一定不因为拥塞丢包吗？一定不丢包，要不然不叫 lossless 了。

Lossless Ethernet 理论上可以用于一切 payload。但是值不值得用是另一个问题，感觉现在只有 RDMA 在谈无损以太网。

### How to Lossless: Flow Control

加入流控算法，[IEEE 802.1Qbb](https://1.ieee802.org/dcb/802-1qbb/)。其实就是 PFC，感觉是先有 PFC 然后被 RoCE 用上了，后续什么 payload 都直接在无损网络上传了。

和小老板讨论的结果是 RoCE 适用于以太网，并且没办法用 IB 专属硬件，如果用 credit base 还需要大量硬件修改，所以采用了 PFC。

但是为了达到 Lossless，还可以 GBN，但是 hurts performance，GBN 不行还可以 SACK，但是硬件实现不好实现，hurts performance。

### 那 Lossy 行不行

[IRN](https://dl.acm.org/doi/10.1145/3230543.3230557)，[LDCP](https://datatracker.ietf.org/meeting/108/materials/slides-108-tsvwg-sessb-82-pfc-free-low-delay-control-protocol-00)

## CC

为什么已经有了流控，不丢包了，还要引入拥塞控制？因为 PFC 导致 buffer 太大了，效果不好。所以 [DCQCN](https://dl.acm.org/doi/10.1145/2785956.2787484) 是优化这个的，感觉可以理解为优化 QoS？

主要目的应转为对尾延迟和吞吐量的优化上。以前有人说拥塞控制一词不好，应该换成尾延迟优化等等，大概就是这个意思。

## DCN vs WAN

数据中心内部网络十分正则 - Clos 架构

- Spin - Agg - ToR，每条相邻级的网线长度几乎相同

导致拥塞的根本原因是大量数据**同时**到达某节点，而不是流量过大。而同时到达某节点的根本原因是网络结构正则。

- 无论据，只有一个论点
- 改进方案
  - 引入随机，消除一部分正则性
  - 不要出现多打一，多打一对于终端类似加锁，但是会把中间链路拉爆

PFC 标准说只能 DCN 内部用，DCBX 协议域下。

跨数据中心流量十分复杂，其中包含数据中心内的流和跨数据中心的流，一般分别控制。

- [Gemini](https://ieeexplore.ieee.org/document/9749785/)
- [Annulus](https://dl.acm.org/doi/abs/10.1145/3387514.3405899)
- [GTCP](https://ieeexplore.ieee.org/document/9546440)
- [IDCC](https://ieeexplore.ieee.org/document/10188700)

## 一些观察和猜想

用 RDMA 主要应该是为了降 core，带宽和时延属于附加收益，因为跑 TCP 也一样跑，只不过 core 多。从理论上长距离 RDMA 可能不支持多中心 AI workload 的模型训练，物理时延是主要瓶颈，除非优化模型训练的通讯过程，否则没法突破物理瓶颈，关注点应该在数据同步上。写论文可能要 argue 为什么它一定要 RDMA，TCP 不行吗？

难点在于无损以太网如何用于跨数据中心网络上，或者在跨数据中心网络上高效实现一个有损以太网。

一个潜在的跨数据中心 RDMA 的使用场景：TailWind (ATC '18) RDMA 做 replica。
