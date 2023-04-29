---
title: Security and Performance in the Delegated User-level Virtualization 论文浅述
date: 2023-04-08 16:01:37
categories: 'DevOps'
tags:
	- '云计算'
---

[IPADS-DuVisor / DuVisor](https://github.com/IPADS-DuVisor/DuVisor)

<!-- more -->

云计算安全课要看一篇论文并且汇报，正好前几天看知乎的 IPADS 组有这么一篇论文，还挺合适的，所以就拿来汇报了。但是汇报的时候 OSDI '23 还没开，参考的论文是 arXiv 上的[预印本](https://arxiv.org/abs/2201.09652)。等到正式论文出来之后再更新与预印本不同的细节。

看完之后大受震撼，但是由于我不是做这个方向的，描述上可能有谬误，敬请指正。

文章中图片，表格和数据全部来源于论文。

> 个人评论仍用引用表达，并且可能有失偏颇，请勿引用。

## Abstract

今天的主流虚拟化系统由两个互相配合的组件组成：一个是访问虚拟化硬件的内核驻留驱动程序，一个是提供虚拟机管理和 I/O 虚拟化的用户态辅助进程。然而，这种虚拟化架构在安全性（具有大攻击面）和性能方面都有固有问题。虽然有很多工作试图通过将功能卸载到用户态中来减少在内核驻留的驱动程序，但它们面临着安全和性能之间的基本权衡：卸载更多函数到用户态可能会缩小内核的攻击面，但会增加辅助进程和驱动程序之间的运行时状态切换，从而增加性能开销。

本文将探讨一种名为托管式虚拟化的新设计，它将控制平面（内核驱动）与数据平面（辅助进程）完全分开，从而使内核驱动免于运行时的干预。由此产生的用户级管理程序，称为 DuVisor，可以处理所有的虚拟机操作，而不需要在内核驱动完成初始化后切换到内核态。DuVisor 用新的托管虚拟化扩展来改造现有的硬件虚拟化支持，以直接处理 VM exit，配置虚拟化寄存器，管理二阶段页表和用户态下的虚拟设备。我们已经在开源的 RISC-V CPU 上实现了硬件扩展，并在硬件之上建立了一个基于 Rust 的管理程序。在 FireSim 上的评估显示，DuVisor 在各种实际应用中比 KVM 的性能高出 47.96%，并大大缩小了攻击面。

> 这里的控制面指内核的控制面，内核驱动只控制虚拟机的启停和一些内存溢出等严重错误，剩下 VM exit 事件都是由数据面来负责操作的。

## 研究的问题

目前的虚拟化方案正处于第三阶段，即使用虚拟化硬件扩展，将一部分虚拟化函数放在硬件扩展中，以提高性能。虚拟化硬件扩展如 Intel VMX，AMD SVM 和 ARM VE/VHE 等，这些扩展扩展允许硬件执行大多数 VM 代码，启用二阶段地址转换等。但这种设计同时面临安全问题和性能问题。如下表展示了对 KVM，Xen 和 VMware ESXi 的 CVE 分析，Host 代表可能导致对主机内核攻击的漏洞。PE、DoS 和 DL 分别代表特权升级、拒绝服务和数据泄漏。LoC 表示他们在 x86-64 架构上的代码行数（VMware 的代码大小并不清楚，因为它是专有的），Starting Years 代表漏洞暴露的最早年份。

![](/images/duvisor/1.png)

KVM 为了实现虚拟化引入了巨大的可信计算基（Trusted Computing Base, a.k.a. TCB），由此引入了大攻击面。其余虚拟化 Hypervisor 均有这些问题。

除了安全问题之外，虚拟机运行导致的用户态与内核态之间的状态切换也引入了性能问题。内核驱动需要与内核中大量函数交互，但是内核中都是通用函数，不会为虚拟化而优化，一些 Hypervisor 在面对 VM exit 事件时可能会把事件转发给用户态的管理 VM 处理，这样又增加了状态切换次数。为了处理上面的问题，一些 I/O 虚拟化函数又放回了内核（比如 vhost-net），但是因为实现问题导致了虚拟机逃逸（V-gHost）。

这一切的问题根源在于硬件虚拟化扩展和内核之间不必要的紧耦合，由于要缩小内核部分，就要将不必要在内核态的虚拟化函数挪到用户态，但是移动函数到用户态势必带来状态切换，这样又导致了性能损失。在这种情况下，安全和性能两边都无法实现令人满意的效果。

而如今，硬件又出现了新的改进，如 Intel 和 RISC-V 允许用户态进程处理物理中断，称为用户级中断。并且 RISC-V 的 Physical Memory Protection（a.k.a. PMP）技术支持在硬件层面限制一个程序能访问的内存范围。所以如何结合新的硬件改进，从根本问题入手，将硬件虚拟化扩展和内核进行解耦，开发新的虚拟化的方法以取得虚拟化安全和性能的平衡即为本研究的主要研究内容。

> 这篇文章出来我才知道有这么个指令集，十分震撼。

## 研究的目标

1. 提出托管式虚拟化，以打破传统管理程序所面临的安全/性能权衡。
2. 设计一个用户级管理程序，在运行时不涉及内核驱动的情况下为虚拟机提供服务。
3. 基于 RISC-V 实现硬件扩展，同时开发一个基于 Rust 的管理程序，并将代码开源。
4. 使用周期精确的 Firesim 和一套真实世界的应用，对 AWS F1 FPGA 上的虚拟机管理程序进行评估。

## 研究的方法

结合硬件新特性，设计托管虚拟化（delegated virtualization）。用户态 Hypervisor（DuVisor）可以安全高效地在用户态控制所有虚拟化函数，DuVisor 直接利用虚拟化扩展的寄存器和指令，为敏感指令、二阶段页表缺页和 I/O 操作引起的 VM exit 事件提供服务，而不需要进入内核态。内核态驱动只负责初始化 DuVisor 和处理严重错误，一个 VM 由一个 DuVisor 管理，使用 PMP 技术来限制 DuVisor 所能访问的物理内存。如果 DuVisor 非法访问其他 VM 的内存，就会产生一个严重错误，交给内核处理，确保安全性。整体架构图如下图所示。

![](/images/duvisor/2.png)

为了实现 DuVisor，研究中设计了针对 DuVisor 的硬件扩展 DV-Ext，DV-Ext 基于 RISC-V Rocket CPU 实现，复用已有的硬件特性，如虚拟化扩展（H-Ext）和用户级中断扩展（N-Ext），并添加或修改了一些寄存器指令。DV-Ext 在进行 VM exit 的操作流程图如下图所示。

![](/images/duvisor/3.png)

首先由内核驱动进行初始化，让控制面开启 DV-Ext（将寄存器 h_enable 设为 1），后将 VM exit 托管到 HU 模式，DuVisor 将 VM exit 的 handler 写到 hu_ehb 中，保存遇到 VM exit 时要执行的函数指针。启动 VM 之前，由 DuVisor 搭建 VM 运行环境，包括准备二阶段页表，准备虚拟 I/O 设备，初始化通用寄存器和系统寄存器，准备结束后执行 HURET 指令，进入 V 模式运行用户代码。当出现 VM exit 事件时，DuVisor 直接调用 hu_ehb 中的 handler，然后 handler 可以直接读 hu_er 和 hu_einfo 等等寄存器中的信息，获得 VM exit 的原因以处理 exit 事件，处理完后再执行 HURET 指令，回到 V 模式。

DV-Ext 在进行内存访问操作时执行过程如下图所示。

![](/images/duvisor/4.png)

首先由虚拟机内系统内核进行用户虚拟地址到用户物理地址的转换，然后交由 DuVisor 进行用户物理地址到宿主机物理地址的转换。在转换后，还需要经过 RISC-V 的物理地址检查机制，如果要访问的物理地址处于 DuVisor 的管辖范围内，则执行这次访问，否则由内核生成一个错误，交给内核中的驱动程序处理。

DV-Ext 在进行 I/O 和中断虚拟化的流程图如下图所示。

![](/images/duvisor/5.png)

如果一台虚拟机内的一个 vCPU 要发送 I/O 中断，它将调用 HUSUIPI 指令发起一次用户态中断，DuVisor 在检查它请求中断的 vCPU 和 VMID 合法后，将向目标 vCPU 发送一个 UIPI，导致对应 vCPU 进入 VM exit 处理中断。

文章进行了一些实验。首先是精细的基准测试。在基准测试中将测试一系列操作，测试将在DuVisor和KVM之间进行，测量十万次操作中两个虚拟机执行指令次数的平均值。测试结果如下图所示。

![](/images/duvisor/6.png)

可以发现，DuVisor 在各类操作的执行指令数上均优于 KVM，在 MMIO 和虚拟 IPI 上远优于 KVM。这证明 DuVisor 将 VM exit 等事件托管到用户态处理确实可以取得一定的优化效果。

研究中选取了一些真实环境软件进行了测试。测试中以 KVM 中运行的时间为基准，测量 DuVisor 中相同软件运行时间快了多少，除了 Memcached 外，其余软件的改善幅度均不超过 10%，而 Memcached 改善了超过 30%。Memcached 是一个网络密集型应用，根据基准测试，这个性能改善的是由于网络 I/O 的 notification 执行的指令次数少。研究中进一步收集了测试中 VM exit 执行的时间开销，发现由于 MMIO 导致的 VM exit 占比最大，而 DuVisor 使用 UIPI 优化了这一过程，所以导致了大幅度的性能提升。但研究者认为在提供硬件中断虚拟化（如 ARM vGIC）以减少过多因 MMIO 导致的 VM exit 的平台上，DuVisor 架构不会带来如此巨大的性能提升。其他 I/O 密集型应用的改进不到 10%：使用 FileIO 和 iperf3 测试的目的是使 I/O 带宽饱和，而 virtio 通过减少 I/O notification 的频率来优化这种情况。因此，较少的 I/O notification 是其性能改善小于 Memcached 的原因。Netperf 在客户端和服务器之间以 ping-pong 的方式评估网络延迟。尽管 DuVisor 的 I/O notification 要快得多，但客户端和服务器中的网络堆栈的时间成本在整个过程中占主导地位（网络堆栈的成本在总的 60 微秒中占 50 微秒），降低了性能的改善。

> 汇报的时候老师问的这个 DuVisor 内部跑的计算任务是在 Intel 平台上还是 RISC-V 平台上，然后我就开始口胡了。因为我没用过 FPGA 集群，文章中提到它的环境是 two FPGA boards. Each FPGA board has eight RISC-V cores (3.2GHz, rv64imafdc), 16GB RAM and 115GB storage. 然后 Both FPGA boards are controlled by an EC2 instance that runs CentOS 7.6.1810 on 16-core Intel E5-2686 v4 CPU (2.3GHz) and 240GB RAM. 我理解的 FPGA 集群是 FPGA 承担一些加速计算任务，整体控制计算流和另一部分计算任务是放 Intel 上的，但是后面提到 The baseline is KVM (with H-Ext support) and kvmtool v3.18.0 running on Linux 5.11.0-rc3 (hereinafter called KVM). 还说 Note that kvmtool is the only available hypervisor for KVM on RISC-V platforms, so that we have to choose it as the performance baseline rather than other hypervisors (e.g., QEMU). 然后我就懵了。我认为的理想中的 DuVisor 是只有 DuVisor 跑在 FPGA 的，剩下的所有计算都发到 Intel CPU 上完成，其实发到什么 CPU 上完成都没有关系，有 GPU 也无所谓，FPGA 只是用来加速 DuVisor 的那个 Hypervisor 的，剩下的计算随便放哪儿都好，但是后面又写 baseline 是 KVM with H-Ext Support，这暗示了 KVM 是在 RISC-V 平台上跑的，那计算好像就得放在 RISC-V 上了。可以保证的是，KVM 和 DuVisor 在测试时所处平台是相同的，但是它们的计算都落在 RISC-V 上还是 Intel 上，我汇报的时候说的是在 Intel 上，但是冷静了一下之后感觉，如果按上面的推理的话，比如用的指令是 HUSUIPI，是在 RISC-V 上的，所以整个计算是应该在 RISC-V 上的。这样可能实用性就不是很高，因为很多场景下都希望在 x86 下跑程序，在 RISC-V 上跑可能是一个新的尝试，但是业务肯定不敢大规模往上迁。

同样，研究者还测量了 DV-Ext 对 KVM 执行的影响，测试以官方原始内核的 KVM 中上下文切换次数为基准，测量 DuVisor 对内核的修改和 DV-Ext 对 KVM 上下文切换次数的影响，结果显示影响均在 1% 以下，可以证明 DV-Ext 对 KVM 几乎无影响，DuVisor 产生的性能提升不是由于 KVM 性能下降导致的。

> 这组对照实验做得很有道理。

## 文章结论

本文通过使用新的托管式虚拟化扩展来改造现有的硬件，提出了托管式虚拟化。在硬件扩展之上建立了一个用户级的管理程序，称为 DuVisor，它直接处理所有的虚拟机操作，而不需要在运行时切换进内核态。实验结果表明，DuVisor 的性能优于传统的管理程序。

## 个人评价

本文提出了一种全新的虚拟化方法，并且通过测试说明了在性能基本与 KVM 持平时具有更好的安全性。在 *Network Algorithmics* 一书中，作者提出了十五个优化网络系统的准则，其中一点是添加硬件改善性能，现在 FPGA 做计算加速的例子也很多，文章利用 FPGA 硬件加速虚拟化是一个十分新的尝试。DuVisor 的代码量在 30,000 行左右，并且开源了内核插件和其他测试脚本，可以说明工作是真实的。从行文来说，文章的实现是扎实的，测试也比较充分，成果可信度较高。其中测试 DV-Ext 对 KVM 的影响实验提高了前面测试的可靠性，这也是值得学习的实验设计技巧。

但是从实用性的角度，一个 DuVisor 只对应一个虚拟机，并且引入这种硬件必然增加成本。虽然在性能比 KVM 略有提升，安全性完全强于KVM，但是其应用场景不足。虽然目标是赋能云厂商向租户提供当前虚拟化方案所难以保证的高可用性，但现实中已有的权衡尚且令人满意，可能在现实中并没有大规模使用的场景，设计目标应该针对的是要求高安全性的场景，如 VMM 等。并且我自己还没搞清楚整个计算是落在 RISC-V 上还是 Intel 上，如果是落在 RISC-V 上实用性就更差了，作为业务人员，感觉迁到这儿来没有什么动力，或者说我就是卖 x86 虚拟机的，我肯定也不会采用这个方案。当然，我认为这篇文章的尝试是值得肯定的，作为一项研究来说是比较完善的，在汇报的时候老师也在质疑实用性，我也持同样观点。不过 Intel 也在搞类似 RISC-V 的用户级虚拟化，这种模式今后或许也可以全部搬到 x86 平台下。

All it needs is time.
