---
title: 'Computers Can Learn from the Heuristic Designs and Master Internet Congestion Control 论文阅读'
date: 2023-09-25 14:35:38
categories: 'Networking'
tags:
  - 'Congestion Control'
  - '笔记'
description: ' '
---

论文地址在[这里](https://dl.acm.org/doi/abs/10.1145/3603269.3604838)。

这篇文章是纯输出观点，没有原文部分。原文部分也不想写了，直接开喷吧。

就是说这个用 ML 去做 CC 可不可行的问题，皮鞋哥有一篇[文章](https://zhuanlan.zhihu.com/p/657936466)来阐述这个事情，就是说它可能是可以借鉴的，毕竟现在的 CC 并不能被公理化证明它就是最优的，反而有很多模糊地带需要做 tuning。我和学弟有一次交流也讨论到这一点了，但是我们觉得 CC 里面有些决策并不影响拥塞水平（这里需要参考 Bolt 那篇文章，DC 内有些 CC 决策失误会很大程度影响拥塞水平），而 ML 要做的是一次决策失误可能出现严重影响的场景。当时的结论是 ML 没必要，现在看来可以重新考虑一下是否可以借鉴某些思路来做 CC。我其实感觉 CC 和控制理论是强相关的，但是由于网络的复杂性，很容易导致内部混沌，并不是一个简单的反馈系统，这又是另一个点了，不是很懂，但是感觉很难。

其实也有一些用 ML 来做 CC 的 paper，比如 [QTCP](https://ieeexplore.ieee.org/document/8357943)，TNSE 2019 的，就是用强化学习做 CC。论文没太细看，但是这个论文说不用在线，用离线的强化学习，他可能有自己的想法。上来来个 Disclaimer，Note that mastering a task does not necessarily mean achieving the optimal performance. 那不达到 optimal performance 要你干啥，现在的 CC 如果说是 mastering a task，那这个工作就和以前的 CC 一样，如果说自己是唯一一个 mastering a task，那后面也没证明，话说的一股 AI 味。中间那图画得比我 PPT 好看多了，可能是在这个领域我看的论文里图画的最牛的一个了，公式又列一堆，整损失函数呗，然后实验搞了个和其他 CC 的 Avg. Normalized 吞吐量和时延对比，说你吗呢，绝对的多大吞吐，多少时延都没说，一平均高其他 CC 挺多，高你吗呢，绝对数值高他吗几 M 平均下来都能高 0.1 左右，最后来 10 个附录，服啦，request 一个 review 意见就多个附录呗。

总之是 AI 味非常非常重的一篇文章，谁他吗审的稿能让这种玩意上的，不是很懂。ML 做 CC 确实是要走一段路，但是纯 AI 做感觉真不行，你至少公布一下绝对吞吐量和时延吧。
