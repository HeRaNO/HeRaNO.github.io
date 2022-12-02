---
title: Polygon, CI/CD 与 Workflow Platform 的一些想法
date: 2022-12-02 17:00:00
categories: 'DevOps'
tags:
	- '开发'
description: ' '
---

这篇文章会写得很杂，只是一个思路整理。

总之 cdoj-execution-worker 并没有达到预期效果，其中有很多东西应该更泛化一些，但是感觉没人搞过，很难。

在某些程度上，这种 Workflow 分解法还是很有效的，我们可以提供一个比 FaaS 稍微重一点的服务，比如叫做 WaaS（Workflow as a Service），把一系列 FaaS 组织起来，或者说这个 FaaS 就是为了执行某个 Workflow 的，但是它依赖于一些 FaaS 的工作。我们只需要把 Workflow 的配置文件丢过去，就可以自动进行一系列操作了。这里面每一步都会涉及一个 FaaS，也就是我们通过一个什么东西来调度 FaaS。

参考网络的设计，可以有一个控制面和一个数据面。控制面用来协调和配置整个过程，数据面用于连通各个 FaaS，相当于我们有个简单的路由器和交换机来引导任务中数据的传输和加工，上层用一个 SDN 来管理。这样我们把这个思路直接做到 SDN 里，就可以对外提供一个三层的网络完成所有 CI/CD 过程了。服务伸缩就变成了一种网络伸缩，并且可以做网络切片。

说实话我没想清楚这么做的具体优势，我感觉在处理报文的速度上应该会有所提升……DC 内部的网络协议和 SDN 部分是解耦的，可能是一个优势。

然后是 Polygon，Polygon 主要自动化的部分就是 CI 过程，造题大家还是正常写题面什么的，但是需要写 generator，validator，checker 等等，其中一点是对拍检查是一个明显的 CI 过程，而实际上每次 commit 都会形成一次 CI，比如每次修改题面就需要重新生成一份题面，改了测试点就要重新生成测试点，更复杂的比如改了 validator 之后需要重新对题面和测试点进行检查，以确保 validator，题面和测试点三者的对应性。现在 Polygon 用的版本控制不太清楚是不是 git，但是整套逻辑是和 Workflow 一样的，分别对不同的部分采取不同的 CI 策略，这点还是比 Github Actions 粒度更细的，因为 Actions 只支持对于 repo 层面的 CI。

然后是 execution-worker，这个其实涉及到了一个运行基础，所有 CI 过程都是 dockerized 的，就需要构建一个支持这种 Workflow 的沙箱环境。但是给每个 Workflow 分配的资源份额是不尽相同的，这样构建沙箱的时候就比较困难。还有 OJ 运行需要的是 Benchmark 功能，但是一般的 checker 运行实际上只需要 Test 功能（Benchmark 和 Test 都是 Go 里面的测试用语），Benchmark 需要一个稳定的基准环境但是 Test 不需要。实际上 SDN 可以做一些同网络不同机器的隔离，甚至不需要规划 IP 端，大概记一下分类就行了。

返回结果也可以做到 FaaS 里，有个专门的消息队列收集结果，或者是一种异步缓冲区。

Web API 也可以看成一种 Workflow，只不过十分简单。
