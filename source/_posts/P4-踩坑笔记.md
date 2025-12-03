---
title: P4 踩坑笔记
date: 2025-12-03 15:34:47
categories: 'Networking'
tags:
	- 'P4'
	- '笔记'
---

[P4: programming protocol-independent packet processors](https://dl.acm.org/doi/10.1145/2656877.2656890)

<!-- more -->

又名：学了一个月 P4 找到编译器三个 bug。~~实际编译器 bug 要比想象中要多。~~

由于 P4 实在坑太多所以开篇文章来记一下。全文均使用 P4-16 标准，在 Tofino 和 Tofino2 架构下编程。文章中包含主观臆断和经验方法，不足之处敬请批评指正。

## 安装和配置

看 [README](https://github.com/p4lang/open-p4studio) 就是了，很清楚并且没什么坑。安装过程中下载依赖需要搭梯子。

目前 Ubuntu 24.04 无法编译，必须 Ubuntu 22.04。Debian 或者 Arch 什么的没试过，也别试了，这是个类嵌入式编程的环境。

## P4 学习

### 语法

首先需要学一些 P4 语法，可以看 [P4-16 Language Specification](https://github.com/p4lang/p4-spec)。但是也没必要上来就看，可以根据示例代码学，而且也没必要每条都懂，会写代码就行了。

学习 P4 的一个障碍是 **P4 不是一门通用语言**，P4 是专为交换机设计的语言，所以一些通用语言的功能 P4 是没有的，比如循环和递归，一些通用语言的功能 P4 是受限的，比如函数嵌套。如果想理解代码为啥不能编译，你需要清楚数据面代码要跑在一个 ASIC 上（甚至不是 CPU，控制面才跑在 CPU 上），这个处理器的运算能力不高，但是匹配-执行（Match-Action）能力很强，所以处理器可以提供的工作流是：每个包线性地经过一条流水线进行加工，这个加工过程也必须简单，可以线性地完成。每个工序就是一次匹配-执行，通过自定义匹配-执行的过程来实现交换机的可编程。

匹配-执行通俗理解就是在 KVDB 里做查询，Key 是你定义的一些键值和匹配方式，查询匹配可以有多种方式（精确匹配，掩码匹配，lpm，或者架构定义的匹配方法），其匹配能力是远大于查路由的；Value 是一个 Action，也就是匹配上后要执行什么操作。这个体现在语言中就是 `table` 和 `action`，`table` 定义了 Key **是哪些 field**，用什么匹配方法，怎么查 Key，以及可能有哪些操作，`action` 就是对应的操作逻辑。

第二个问题就出现在 **数据面和控制面是分离的**。数据面就是实际数据经过的平面，这个平面只负责按规则处理数据，这个规则就是 P4 代码里规定的，数据面必须确定这个规则，并且运行的时候这个规则不能变。但是这个规则只定义了 Key 的 field 和有哪些 action，并不是哪些具体的 Key 对应哪些 action。用路由转发举例子就是，数据面会定义说用 IP 包里的目的地址做 lpm，也会定义可能的所有 action，但具体是哪些 IP 匹配上后对应哪些 action，调用对应 action 的时候参数是什么，比如希望 10.1.0.0/16 转发去出口 1，10.2.0.0/16 转发去出口 2，把 10.3.0.0/16 的所有包丢了，这是控制面配的。控制面不负责数据面的任何逻辑，只是给数据面的逻辑填充配置数据用的。当然也有一些对数据面的补充，比如可以配置 App 实现定时生成包，但这些完全不影响数据面的运行。但非常痛苦的一点在于，如果控制面下发的数据对不上数据面的规则，这个问题只能在运行时里看出来，因为分离了数据面和控制面，所以这个问题不会在编译期发现，责任在控制面发的数据不对。但这里发的数据可以是一个 action 名和对应参数，也就是匹配上了之后以什么参数执行什么 action，这里是带一些逻辑的，但是数据面 P4 代码里不会带这个逻辑，它只定义了有哪些 action，所以 **分离了但没完全分离**。

除了匹配-执行外，P4 的另一个特点是有状态转发，也就是在转发中可以记录关于包的某些信息，数据面上可以直接拿来用。但是这个功能也十分受限。数据结构只有 Register，也就是数组，为了去操作这些 Register，还需要配合一个 action。除了 Register，还有 Counter 可以实现包个数和大小的累计统计，但是数据面上不能读取一个 Counter 中的值，这个值只能在控制面上读到。

### Tofino 系列架构

因为需要针对 Tofino 系列架构编程，除开 P4 本身的语法，还需要学习 Tofino 系架构的特点，可以看 [TNA 文档](https://github.com/barefootnetworks/Open-Tofino/blob/master/PUBLIC_Tofino-Native-Arch.pdf)。虽然似乎也可以用 V1Model 架构跑在 TNA 的模型上，但是整个开发时还是使用 TNA 或者 T2NA 架构跑 Tofino 或 Tofino2 模型。

TNA 和 T2NA 相较于 V1Model 多了 Ingress 的 Deparser 和 Egress 的 Parser，也就是 Ingress 和 Egress 都是 Parser 到 Gress 逻辑到 Deparser。Register 只能在同一个 Gress 中使用，不可以在 Ingress 和 Egress 访问同一个 Register，因为 Ingress 和 Egress 的内存是不共享的（也不知道是不是学 Go 通过通信来共享内存）。所以造成了 Ingress 读不了 Egress 上的队列长度，所以才有个 Ghost Thread 打补丁，但是这个补丁完全是还技术债，Juniper 的 [Trio](https://people.csail.mit.edu/ghobadi/papers/trio_sigcomm_2022.pdf) 早就共享内存了（当然其目的并非只是为了获得 Egress 的队列长度）。

其余不同仅在于架构提供的接口不同，比如 V1Model 中的 `Clone` 在 TNA 中是 `Mirror` 等。看文档实现即可。

### 学习路线

个人建议是通过一个 [example](https://github.com/p4lang/open-p4studio/tree/main/pkgsrc/p4-examples/p4_16_programs) 入手，比如 `tna_counter` 或者 `tna_mirror`，把数据面（P4 代码）和控制面（`test.py`）对着看。因为数据面和控制面是分离的，但是又有关系，对起来看才能知道它到底在干什么。

## 举例

以 [Bolt](https://www.usenix.org/conference/nsdi23/presentation/arslan) 为例讲讲大概的实现。Bolt 中比较重要的是探测是否拥塞和回发拥塞通知，所以分两个部分来讲。

### 探测拥塞

探测拥塞部分是 Ghost Thread 来做的，但鉴于目前 Ghost Thread 好像不能用（[#5422](https://github.com/p4lang/p4c/issues/5422)），并且 Tofino 并不支持 Ghost Thread，因此一个兼容的方法是使用定时 App 生成 probe 包，通过 recirculation port 将 Egress 的数据带回 Ingress，但这将占用一部分带宽和包转发能力。

因为要探测每个端口的队列长度，所以需要定时给每个端口都发 probe 包，这个包只需要带个以太网头加个自定义的采集头就可以，不需要 payload，所以可以很小（大概 20B）。但问题在于端口很多，所以 probe 包也需要很多种标识，让它去探测不同的端口，而一个 App 只能配一种 probe 包，最多只能配置 8 个 App，没有办法给每个端口开一个 App 来生成对应的标识。可以利用 `pktgen_timer_header_t` 的 `batch_id` 和 `packet_id` 来批量生成对不同 Egress ID 的探测包。设置 1 个 Batch 生成端口个数个 packet，`packet_id` 当要探测的端口用即可，放在 `metadata_t` 或者其他地方都可以。这样，包就从 App port 的 Ingress 发送到了指定 probe 的 Egress 里了。

到达 Egress 后采集信息，之后我们要把这个包导回 Ingress，这时候就需要用到 recirculation。因为这个包已经在被 probe 的 Egress 里了，没办法再换端口了，所以此时使用 Egress-to-Egress Mirror 重新确定端口，首先把原有的包丢掉，然后 Mirror 一个新的，端口指定为 recirculation port 即可。在使用 Mirror 时 `session_id` 需要填不同的，类似于数据库主键。

当 recirculation port 收到了回环回来的 probe packet，更新拥塞信号后把包丢掉即可。

### 回发拥塞通知

回发拥塞通知相较于探测拥塞简单点，因为 P4 可以自定义包头，所以把拥塞信号塞包头里，没有拥塞的时候不激活，拥塞的时候激活即可。如果在 Ingress 发现拥塞，使用 Ingress-to-Egress Mirror 生成拥塞通知，目标端口为 Ingress 端口（原路发回去），I2E Mirror 之后包直接进入 Egress，在 Egress 处把 Mirror 包的源和目的 MAC 地址，源和目的 IP 地址交换一下，更新 checksum 就可以了。在 Mirror 时控制面可以配置 Mirror 的最大报文长度，设置成所有包头长度和就行，这样就可以把 payload 删掉了。 

## 一些坑和技巧

### P4

- 不知道为什么 Deparser 里不能像其他的代码一样直接写一个 `pkt.emit(hdr)` 就完事，一定要把每个 `hdr` 都 `emit` 一下才对
- 一个 table 的 action 里似乎只能用一个 Counter（条件有些记不清了），会编译错误，按提示改
- 报文定义必须字节对齐

### 调试

- 调试中可能可以用 GDB，在 `run_switchd` 加参数 `-g` 就可以，但是没用过，一直用 Counter 来调试
- 运行中在 `run_model` 的终端中可以看到包处理的详细过程，但是在 Ingress 和 Egress 的 Parser 前交换机会把一些自己的 metadata 放在包的开头，所以一开始应该先解析 metadata 而不是以太网头，输出的传给 Parser 的包也不是以以太网头开始的
- 可以使用 `bfrt_python` 看交换机的控制面，在 `run_switchd` 的终端中启动 `bfrt_python`，然后通过 `bfrt.<p4_name>.pipe.<...>` 进到 control block 中，比如一个叫 `test` 的 P4 程序，数据面上可能有 `Ingress`，就可以用 `bfrt.test.pipe.Ingress` 进到 control block 里看一些状态，但前提是 `Ingress` 中是存在 `table` 或者有状态 Register 的，否则控制面看不到
- `bfrt_python` 中代码补全非常完善，按 tab 还有提示，直接按 tab 就好了
- 读取 Counter 或者 Register 中数据的时候要先和数据面同步一下，在 `bfrt_python` 里用 `operation_counter_sync` 和 `operation_register_sync`，脚本里 `entry_get` 的时候用 `{from_hw: true}`

## 个人感受

不写这一节我可能会被活活憋死，不看也罢。学习一门语言从没有这么痛苦过，基本是经历了从疑惑到懵圈到质疑自己智力低下到懊恼到愤怒到麻木的过程。

首先 Tofino 和 Tofino2 是个半残架构。Ingress 和 Egress 做内存隔离是可以理解的，但是数据面上甚至不能读交换机的全局状态，这就过于挠头了。数据面也不会对全局状态写，只有一个写方剩下都是读方，也不会有什么并发问题。更甚者 Counter 居然不是数据面可读的，完全不理解为什么会这么设计。

其次 P4 是个半残语言。P4 的数据面和控制面分了但又没完全分，你必须要理解它「数据面上有逻辑，控制面下发数据让那套逻辑转起来」的抽象逻辑才能搞清楚它在干什么。而且 P4 完全可以用现在的 C 做前端，table 什么的提供 API 就是了，llvm 做个后端处理 IR 我感觉也完全能对上需求。结果搞出个勾八 P4 这么个怪胎出来，非得自创一堆乱七八糟语法，连带着半残架构写出一堆摸门不着的玩意。自家编译器特么编译 Ingress 和 Egress 访问同一 extern Register 直接报编译器有 bug，这种质量真有生产敢用吗？仓库里一翻一堆编译器 bug 没修，半死不活。现在是有了 [p4mlir-incubator](https://github.com/p4lang/p4mlir-incubator)，但我看没有社区活跃度换啥也没用，趁早埋了吧。

最后现在是个文章就有实际实验，何必呢，写出来只起到一个安慰审稿人的作用，然后增加了无效内卷。Tofino 和 CX5 又不是人人都有，但是现在好像没有这些就做不了研究一样。希望审稿人不要反思算法是否能实现，而是反思现在的硬件设计是不是太特么落后了，都特么的本质图灵机怎么就不能实现了，现在硬件实现不了难道不应该去改进硬件吗，特么的 ASIC 和 FPGA 都发展到现在这样了再问是不是不礼貌了，而且改进硬件的事又凭啥让算法设计来干，说到底怕不是来偷方案的，或者瞎找理由毙人的。
