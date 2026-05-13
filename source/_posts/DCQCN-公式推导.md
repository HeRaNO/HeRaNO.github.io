---
title: DCQCN 公式推导
date: 2026-04-08 10:19:28
categories: 'Networking'
tags:
	- '笔记'
description: ' '
---

我一直觉得 DCQCN 里的流体模型是鸡肋（包括 PowerTCP 里的 Lyapunov 稳定性），就纯为了凑论文字数糊上去的，说是做了分析但也就是让人看着觉得确实分析了罢了（虽然我自己的文章也是这样），除了给 Reviewer 心理安慰让他给过之外也没啥作用。最近老师让看 DCQCN 的公式推导，所以系统记录一下这个过程。

DCQCN 的流体模型来源于 SIGMETRICS '11 的 [Stability Analysis of QCN: The Averaging Principle](https://dl.acm.org/doi/10.1145/1993744.1993751) 这篇文章，在这届会议中同样还有分析 DCTCP 的稳定性的文章（[Analysis of DCTCP: stability, convergence, and fairness](https://dl.acm.org/doi/10.1145/1993744.1993753)，出自同一人之手）。总之可以互相参考着看。

DCQCN 文章中有一些更详细的公式推导在 CoNEXT '16 的 [ECN or Delay: Lessons Learnt from Analysis of DCQCN and TIMELY](https://dl.acm.org/doi/10.1145/2999572.2999593)，其中还有关于 TIMELY 的稳定性分析，但本文并不关心 TIMELY。

## 符号表

|          符号          |                        意义                        |
| :--------------------: | :------------------------------------------------: |
|         $R_C$          |                    当前发送速率                    |
|         $R_T$          | 目标速率，即恰好在最后一个反馈消息到来前的发送速率 |
|        $\alpha$        |                减速因子，初值为 $1$                |
|          $q$           |                      队列长度                      |
|          $t$           |                        时间                        |
| $K_\min,K_\max,p_\max$ |                  ECN 标记所用参数                  |
|          $g$           |       论文里没有描述，但其是一个移动平均因子       |
|          $N$           |                   瓶颈处的流条数                   |
|          $C$           |                   瓶颈链路的带宽                   |
|          $F$           |                  快速恢复的阶段数                  |
|          $B$           |             速率增长中所用的字节计数器             |
|          $T$           |               速率增长中所用的计时器               |
|     $R_\text{AI}$      |          速率增长的幅度（固定为 40Mbps）           |
|      $\tau^\ast$       |                   控制回路的延迟                   |
|        $\tau'$         |              $\alpha$ 更新的时间周期               |
|         $\tau$         |                   CNP 的生成周期                   |

## 前提假设

分析时，假设有 $N$ 条贪心流（指如果有空闲带宽就会抢占）通过单个容量为 $C$ 的瓶颈链路。

但最重要的假设是 DCQCN 一定在 PFC 前起作用。

> 长距场景下上面的假设非常鸡肋，不满足 PFC 后于 DCQCN 触发的情况很多。

## 流体模型

### 公式 5

$$
p(t)=
\begin{cases}
0 & q(t)\le K_\min\\
\frac{q(t)-K_\min}{K_\max-K_\min}p_\max & K_\min<q(t)\le K_\max\\
1 & q(t)>K_\max
\end{cases}
$$

这个公式描述的是拥塞时给包打 ECN 标记的概率随出队列长度变化的变化情况。图象是非常经典的三段，如果你做网络的话应该是刻 DNA 里了所以就不贴图了。

这个公式本身也没什么好说的，意思就是我们认为 $K_\min$ 以下的队列是合理的，因为[统计复用系统避免不了排队](https://zhuanlan.zhihu.com/p/446635134)。但是大于 $K_\min$ 的时候可能是拥塞的预兆，这时做 RED，但是不丢包，而是随机打标记。如果大于 $K_\max$ 则认为是真拥塞了，每个包都打拥塞标记。

### 公式 6

$$
\frac{\mathrm{d}q}{\mathrm{d}t}=NR_C(t)-C
$$

假设所有流的速率都相等，这个公式描述了队列长度 $q$ 随时间 $t$ 变化的情况。等式左边是队列变化率，右边是队列入速率减出速率，就是一个水槽放水的问题，也没什么大不了的。但是需要注意这个公式十分理想，现实情况下至少 $NR_C(t)$ 应写作 $\sum_{i=1}^N R_{C_i}(t)$，并且这也没考虑传输时延的影响。

后面说他可以放宽所有流的速率都相等的假设，但是说实话我没看懂他是怎么放宽的。

### 公式 7

$$
\frac{\mathrm{d}\alpha}{\mathrm{d}t}=\frac{g}{\tau'}\left(\left(1-\left(1-p(t-\tau^\ast)\right)^{\tau'R_C(t-\tau^\ast)}\right)-\alpha(t)\right)
$$

这是一个十分困难的公式，但是这个类型和 8 与 9 类似。

这个公式考虑 $\alpha$ 随时间 $t$ 的变化的情况。根据 DCQCN 算法，当收到一个 CNP 时，$\alpha$ 会进行一次「向 $1$ 更新」，也就是 $\alpha \gets (1-g)\alpha+g$，然后重置 $\alpha$ 更新计时器。当触发 $\alpha$ 更新计时器时，$\alpha$ 会进行一次「向 $0$ 更新」，也就是 $\alpha \gets (1-g)\alpha$。因此，对于「向 $1$ 更新」，$\alpha$ 变化量为 $\Delta \alpha_1=g(1-\alpha)$，对于「向 $0$ 更新」，$\alpha$ 变化量为 $\Delta \alpha_0=-g\alpha$。

> 这里的「向 $1$ 更新」和「向 $0$ 更新」是我自己造的，区分的是更新后 $\alpha$ 会往哪儿移动。「向 $1$ 更新」是一种 EWMA，标准的 EWMA 是 $A(t)=w X(t)+(1-w) A(t-1)$，这里 $A(t)$ 就是 $\alpha$，$w$ 就是 $g$，$X(t)$ 是 $1$，因此在移动混合过程中会逐渐消解 $A(t)$ 而过渡到 $X(t)$。「向 $0$ 更新」很 trivial，假设更新 $k$ 次，值变成 $(1-g)^k\alpha$，而 $g<1$，因此 $(1-g)^k\to 0$，基础微积分。

但是，是否收到 CNP 是取决于是否有 ECN 标记的包，而 ECN 标记是依概率的，因此 $\alpha$ 是一个随机变量，$\alpha(t)$ 是一个随机过程。

> 从道理来说，$\alpha(t)$ 是一个一维随机游走，不应该这么定义变化率。

考虑到设计时，CNP 的生成周期 $\tau$ 是小于 $\alpha$ 的更新周期 $\tau'$ 的。因此，在一个 $\tau'$ 的时间窗口内，要么收到了一个 CNP 后进行「向 $1$ 更新」，之后 $\alpha$ 更新计时器被重置，但假设不重置的情况下，在这个时间窗口内仍只会有这一个 CNP；要么一直没有收到 CNP，到窗口末期进行「向 $0$ 更新」。因此，一个时间窗口内，最多进行一次「向 $1$ 更新」，如果没进行「向 $1$ 更新」，则进行一次「向 $0$ 更新」。

> 这里文章的符号是乱的，我自己采用了新的符号。$\tau$（后文还会出现）在文章里是 $N$，$\tau'$ 上下文符号不一致，第一次出现的符号是 $K$。

为了计算 $\alpha$ 的变化率，我们首先需要承认如下近似。

$$
\frac{\mathrm{d}\alpha}{\mathrm{d}t}\approx \frac{\mathbb{E}(\Delta \alpha)}{\tau'}
$$

> 强行解释就是我们希望计算在 $\tau'$ 时间内 $\alpha$ 的平均变化率，而 $\tau'$ 是一个极小量，然后变化量期望代表着平均变化量，一除就是平均变化率了，反正不是的话也推不出后面的东西。严格来说这么写都是错的，$\alpha(t)$ 不可导，应该写成类似 Ito 积分的形式，虽然选了应用随机过程但已经过了四年（但好像讲的时候 Ito 积分也就提了一嘴，没细讲），现在啥也不会了。

设在 $\tau'$ 时间窗口内没有 CNP 的概率为 $p_0$，那么

$$
\begin{aligned}
\mathbb{E}(\Delta \alpha)&=p_0\Delta \alpha_0+(1-p_0)\Delta \alpha_1\\
&=-p_0g\alpha+(1-p_0)g(1-\alpha)\\
&=g(1-p_0-\alpha)
\end{aligned}
$$

则

$$
\frac{\mathbb{E}(\Delta \alpha)}{\tau'}=\frac{g}{\tau'}(1-p_0-\alpha)
$$

比照给出的公式，发现 $p_0=(1-p(t-\tau^\ast))^{\tau'R_C(t-\tau^\ast)}$。我们仅考虑一个 $\tau'$ 时间窗口内的变化。由于网络存在控制回路延迟 $\tau^\ast$，当前时刻 $t$ 收到的反馈，对应的是 $t-\tau^\ast$ 时刻的发送状态。因此，如果这个窗口内无 CNP，发送的总数据量为

$$
\tau'R_C(t-\tau^\ast)
$$

为了使这个窗口无 CNP，就不能被 ECN 打标记，因此在这一窗口内所有数据都不被 ECN 标记的概率是

$$
p_0=(1-p(t-\tau^\ast))^{\tau'R_C(t-\tau^\ast)}
$$

由此得证。

> 但为什么是这样是很无厘头的。首先不是所有的发送数据都形成队列，假设这些数据都形成队列了，虽然 ECN 是根据队列长度变化的，但是按理说这个概率应该和包个数有关，而不是字节数。然后是打 ECN 标记的时间并非控制回路延迟 $\tau^\ast$，综合起来这公式混合了不同时刻发生的事情，虽然差距很小但还是有差距，但是数据中心内这点差距也不算啥，数学是天体物理老师教的，就只能感性理解，不要用它来计算。长距链路上带入这个 $\tau^\ast$ 显然就是错的了。

### 公式 9

$$
\begin{aligned}
\frac{\mathrm{d}R_C}{\mathrm{d}t}=&-\frac{R_C(t)\alpha(t)}{2\tau}\left(1-(1-p(t-\tau^\ast))^{\tau R_C(t-\tau^\ast)}\right)\\
&+\frac{R_T(t)-R_C(T)}{2}\frac{R_C(t-\tau^\ast)p(t-\tau^\ast)}{(1-p(t-\tau^\ast))^{-B}-1}\\
&+\frac{R_T(t)-R_C(T)}{2}\frac{R_C(t-\tau^\ast)p(t-\tau^\ast)}{(1-p(t-\tau^\ast))^{-TR_C(t-\tau^\ast)}-1}
\end{aligned}
$$

由于 $R_C$ 的更新相对简单，我们先看公式 9。

由于降速还是升速取决于是否有 CNP，还是一个随机过程，因此仍然仿照公式 7 推导，写出

$$
\frac{\mathrm{d}R_C}{\mathrm{d}t}\approx \frac{\mathbb{E}(\Delta R_C)}{t}
$$

但是这里降速周期和升速周期并不一样，降速周期是 $\tau$ 但是升速周期受字节计数器和升速计时器限制，所以我们拆开分析

$$
\frac{\mathrm{d}R_C}{\mathrm{d}t}\approx \frac{\mathbb{E}_\text{dec}(\Delta R_C)}{\tau}+\frac{\mathbb{E}_\text{inc}(\Delta R_C)}{t}
$$

> 这里用到 $E(X+Y)=E(X)+E(Y)$，我们把除法也放进去就好了。

#### 降速过程

当收到 CNP 时会降速，而 CNP 每 $\tau$ 时间生成一个，因此在这里我们以 $\tau$ 为时间窗口分析。降速过程中 $R_C$ 的变化量 $\Delta R_C=(1-\frac{\alpha}{2})R_C-R_C=-\frac{\alpha}{2}R_C$，同样根据公式 7 中推导已知，$\tau$ 时间内最多产生一个 CNP，并且在 $\tau$ 时间内产生至少一个 CNP 的概率是 $1-(1-p(t-\tau^\ast))^{\tau R_C(t-\tau^\ast)}$，所以

$$
\frac{\mathbb{E}_\text{dec}(\Delta R_C)}{\tau}=-\frac{R_C(t)\alpha(t)}{2}\cdot\frac{1}{\tau}\left(1-(1-p(t-\tau^\ast))^{\tau R_C(t-\tau^\ast)}\right)
$$

#### 升速过程

升速过程受字节计数器和计时器限制。先看一次升速产生的变化量，虽然 DCQCN 中有 FastRecovery，AdditiveIncrease 和 HyperIncrease 三种状态，但是更新 $R_C$ 的方法都是同一种，即 $R_C\gets (R_C+R_T)/2$，因此 $\Delta R_C=\frac{R_T(t)-R_C(t)}{2}$。并且无论 $F$ 取值多少，都会这样更新 $R_C$，所以推导中不会出现 $F$。

再看多长时间才能进行一次升速，首先看字节计数器影响的升速。网卡要发送 $B$ 个字节且期间不收到任何 CNP，才能触发一次提速。我们要求的就是期望发送多少数据，才能出现连续发送 $B$ 个字节均不被标记。这是一个 Markov 链经典问题（QCN 稳定性论文中也有提及，但并没有做推导），答案是

$$
\mathbb{E}(S)=\frac{\left(1-p(t-\tau^\ast)\right)^{-B}-1}{p(t-\tau^\ast)}
$$

那么进行一次字节计数器导致的升速所需的期望时间为

$$
\begin{aligned}
\Delta T_\text{byte} &= \frac{\mathbb{E}(S)}{R_C(t-\tau^\ast)}\\
&=\frac{\left(1-p(t-\tau^\ast)\right)^{-B}-1}{R_C(t-\tau^\ast)p(t-\tau^\ast)}
\end{aligned}
$$

同理，对于计时器，网卡要发送 $T\cdot R_C$ 个字节且期间不收到任何 CNP，才能触发一次提速，重新带入即可。

$$
\Delta T_\text{timer} =\frac{\left(1-p(t-\tau^\ast)\right)^{-TR_C(t-\tau^\ast)}-1}{R_C(t-\tau^\ast)p(t-\tau^\ast)}
$$

然后

$$
\frac{\mathbb{E}_\text{inc}(\Delta R_C)}{\Delta T}=\frac{\Delta R_C}{\Delta T_\text{byte}}+\frac{\Delta R_C}{\Delta T_\text{timer}}
$$

就是原式了。

综合如上两部分原式得证。

### 公式 8

$$
\begin{aligned}
\frac{\mathrm{d}R_T}{\mathrm{d}t}=&-\frac{R_T(t)-R_C(t)}{\tau}\left(1-(1-p(t-\tau^\ast))^{\tau R_C(t-\tau^\ast)}\right)\\
&+R_{\text{AI}}R_C(t-\tau^\ast)\frac{(1-p(t-\tau^\ast))^{FB}p(t-\tau^\ast)}{(1-p(t-\tau^\ast))^{-B}-1}\\
&+R_{\text{AI}}R_C(t-\tau^\ast)\frac{(1-p(t-\tau^\ast))^{FTR_C(t-\tau^\ast)}p(t-\tau^\ast)}{(1-p(t-\tau^\ast))^{-TR_C(t-\tau^\ast)}-1}
\end{aligned}
$$

公式 8 的大致思路和公式 9 是一样的，只是换了一下参数而已。

在降速阶段，由于进行了 $R_T\gets R_C$，因此 $\Delta R_T=R_C-R_T$，剩下的讨论同公式 9。

在升速阶段，虽然 DCQCN 中 AdditiveIncrease 和 HyperIncrease 两种升速所用的 $R_\text{AI}$ 不同，但是为了方便起见公式中用的是一样的。由此 $\Delta R_T=R_\text{AI}$。

与 $R_C$ 更新不同的是，至少更新 $F$ 次 $R_C$ 后才会更新 $R_T$，即 FastRecovery 不会更新 $R_T$，而是在 Increase 阶段才更新。因此，公式 9 字节计数器升速时讨论的「网卡要发送 $B$ 个字节且期间不收到任何 CNP，才能触发一次提速」就变成了「网卡要发送 $FB$ 个字节且期间不收到任何 CNP，才能触发一次提速」，对于计时器升速同理，指数部分多乘个 $F$ 就是了。

综合起来原式得证。

## 稳定性

DCQCN 论文中并没有直接证明稳定性，而是依赖 QCN 的稳定性证明。QCN 也没有直接证明稳定性，而是通过如下路径来证明：

1. QCN 利用了 Average Principle（似乎没有对应的汉语翻译）；
2. AP 在线性控制系统中和 PD（比例微分）控制器代数意义上等价，即给 AP 和 PD 等价的输入，它们的输出是等价的；
3. PD 具有稳定性，那么 AP 也具有稳定性；
4. 所以 QCN 是稳定的。

至于证明已经完全不想看了。

## 用处

那么到最后 DCQCN 用这个流体模型干什么了呢？

### 不动点

这部分内容在 CoNEXT '16 文章中的 3.2 节。

首先置公式 6 左式为 $0$，即
$$
0=\frac{\mathrm{d}q}{\mathrm{d}t}=\sum_{i=1}^N R_C^{(i)}(t)-C
$$
也就是说，如果 DCQCN 存在一个不动点，则必须满足
$$
\sum_{i=1}^N R_C^{(i)\ast}(t)=C
$$

> 因为不动点处一定有队列长度不变，说明速率收敛了

在任一不动点处，设 $p$ 的值为 $p^\ast$，所有流都共享此值。此时，队列长度的不动点与每条流的 $\alpha^{(i)}$ 的不动点可以由公式 5 和 7 确定。

假设流收敛在队列长度为 $K_\min$ 到 $K_\max$ 之间（其原因并没有说明）。则直接将 $p^\ast$ 带入公式 5，得到队列长度的不动点
$$
q^\ast=\frac{p^\ast}{p_\max}(K_\max-K_\min)+K_\min
$$
对于 $\alpha^{(i)}$ 的不动点，仍然置公式 7 左式为 0，即
$$
0=\frac{\mathrm{d}\alpha}{\mathrm{d}t}=\frac{g}{\tau'}\left(\left(1-\left(1-p(t-\tau^\ast)\right)^{\tau'R_C(t-\tau^\ast)}\right)-\alpha^{(i)\ast}(t)\right)
$$
由于 $\frac{g}{\tau'}\neq 0$，因此只能是后面括号内部为 0，则可解得
$$
\alpha^{(i)\ast}=1-(1-p^\ast)^{\tau'R_C^{(i)\ast}}
$$
接下来我们需要证明 $p^\ast$ 存在，并且被 $R_C^{(i)\ast}$ 唯一确定。

为了简单需要确定五个代换变量
$$
\begin{aligned}
a&=1-(1-p^\ast)^{\tau R_C^{(i)\ast}}\\
b&=\frac{p^\ast}{(1-p^\ast)^{-B}-1}\\
c&=\frac{(1-p^\ast)^{FB}p^\ast}{(1-p^\ast)^{-B}-1}\\
d&=\frac{p^\ast}{(1-p^\ast)^{-TR_C^{(i)\ast}}-1}\\
e&=\frac{(1-p^\ast)^{FTR_C^{(i)\ast}}p^\ast}{(1-p^\ast)^{-TR_C^{(i)\ast}}-1}
\end{aligned}
$$
置公式 8 左式为 0，得到
$$
\begin{aligned}
0=\frac{\mathrm{d}R_T}{\mathrm{d}t}=&-\frac{R_T^{(i)\ast}-R_C^{(i)\ast}}{\tau}a\\
&+R_{\text{AI}}R_C^{(i)\ast}c\\
&+R_{\text{AI}}R_C^{(i)\ast}e
\end{aligned}
$$
则
$$
\frac{R_T^{(i)\ast}-R_C^{(i)\ast}}{\tau}a=R_{\text{AI}}R_C^{(i)\ast}(c+e)
$$
置公式 9 左式为 0，得到
$$
\begin{aligned}
0=\frac{\mathrm{d}R_C}{\mathrm{d}t}=&-\frac{R_C^{(i)\ast}\alpha^{(i)\ast}}{2\tau}a\\
&+\frac{R_T^{(i)\ast}-R_C^{(i)\ast}}{2}R_C^{(i)\ast}b\\
&+\frac{R_T^{(i)\ast}-R_C^{(i)\ast}}{2}R_C^{(i)\ast}d
\end{aligned}
$$
则
$$
\frac{R_C^{(i)\ast}\alpha^{(i)\ast}}{2\tau}a=
\frac{R_T^{(i)\ast}-R_C^{(i)\ast}}{2}R_C^{(i)\ast}(b+d)
$$
解得
$$
R_T^{(i)\ast}-R_C^{(i)\ast}=\frac{\alpha^{(i)\ast}a}{\tau(b+d)}
$$
带入公式 8 得到的结果，有
$$
\frac{\alpha^{(i)\ast}a^2}{\tau^2(b+d)}=R_{\text{AI}}R_C^{(i)\ast}(c+e)
$$
移项得
$$
\frac{\alpha^{(i)\ast}a^2}{(b+d)(c+e)}=\tau^2R_{\text{AI}}R_C^{(i)\ast}
$$
为 CoNEXT 论文中公式 11。

> 此处，由于我们已经是考虑稳态中情况了，$R_C^{(i)\ast}$ 实际上为一常数，可以根据上式求解 $p^\ast$。

下面我们要考虑不动点是否只有一个，也就是求解上式是否可以唯一确定 $p^\ast$。可以发现，当 $p\in [0,1]$ 时，左式是关于 $p^\ast$ 的单调递增函数。这部分证明论文没写，虽然我猜他们是猜的，但是确实是单调递增的。

考虑左式拆为
$$
f(p)=a^2\alpha^\ast\cdot \frac{1}{b+d}\cdot \frac{1}{c+e}
$$
如果这三部分均关于 $p$ 在 $[0,1]$ 上单调递增，那么 $f(p)$ 在 $[0,1]$ 上单调递增。

首先，$a=1-(1-p^\ast)^{\tau R_C^{(i)\ast}},\alpha^{(i)\ast}=1-(1-p^\ast)^{\tau'R_C^{(i)\ast}}$，两部分都是关于 $p^\ast$ 的多项式函数，因为指数部分都是常数。考虑 $(1-p)^C$（$C$ 为一常数）在 $[0,1]$ 上单调递减，则 $1-(1-p)^C$ 在 $[0,1]$ 上单调递增成立，因此第一部分在 $[0,1]$ 上单调递增成立。

第二部分和第三部分形式类似。对于第二部分，考虑 $b$ 的倒数和 $d$ 的倒数
$$
\begin{aligned}
\frac{1}{b}&=\frac{(1-p^\ast)^{-B}-1}{p^\ast}\\
\frac{1}{d}&=\frac{(1-p^\ast)^{-TR_C^{(i)\ast}}-1}{p^\ast}
\end{aligned}
$$
考虑函数
$$
g(p)=\frac{(1-p)^C-1}{p}
$$
其中 $C<0$，我们想知道它在 $[0,1]$ 上的单调性。

对 $g(p)$ 求导
$$
g'(p)=\frac{-C(1-p)^{C+1}p-(1-p)^C+1}{p^2}
$$
其中，$-C(1-p)^{C+1}p\ge 0$ 在 $[0,1]$ 上恒成立，考虑 $h(p)=1-(1-p)^C$ 在 $[0,1]$ 上的取值，继续求导。
$$
h'(p)=C(1-p)^{C+1}
$$
$h'(p)\ge 0$ 在 $[0,1]$ 上恒成立，因此 $h(p)$ 在 $[0,1]$ 上单调增，也就是 $h(p)\ge h(0)=0$ 成立，因此 $g'(p)\ge 0$ 在 $[0,1]$ 上恒成立，$g(p)$ 在 $[0,1]$ 上单调增。因此 $1/b$ 和 $1/d$ 在 $[0,1]$ 上均单调增。

那么
$$
\frac{1}{b+d}=\frac{1}{\frac{1}{1/b}+\frac{1}{1/d}}=\frac{(1/b)(1/d)}{(1/b)+(1/d)}
$$
由于 $1/b$ 和 $1/d$ 均单调增，那么它们的并联函数也单调增，因此第二部分也单调增。

>设 $f(x)$ 和 $g(x)$ 均为单调增函数，且 $f(x)$ 和 $g(x)$ 均不为 $0$，现在考虑它们的并联函数
>$$
>h=\frac{f\cdot g}{f+g}
>$$
>求导
>$$
>\begin{aligned}
>h'&=\frac{(f\cdot g)'(f+g)-f\cdot g(f+g)'}{(f+g)^2}\\
>&=\frac{(f'\cdot g+f\cdot g')(f+g)-f\cdot g(f'+g')}{(f+g)^2}\\
>&=\frac{(f\cdot f'\cdot g+f^2\cdot g'+f'\cdot g^2+g\cdot f\cdot g')-(f'\cdot f\cdot g+g'\cdot f\cdot g)}{(f+g)^2}\\
>&=\frac{f^2\cdot g'+f'\cdot g^2}{(f+g)^2}
>\end{aligned}
>$$
>由于 $f$ 和 $g$ 均单调增，那么 $f'$ 和 $g'$ 均大于 $0$，因此 $h'>0$ 成立，即 $h$ 单调增。

对于第三部分，类似考虑 $c$ 的倒数和 $e$ 的倒数
$$
\begin{aligned}
\frac{1}{c}&=\frac{(1-p^\ast)^{-B}-1}{(1-p^\ast)^{FB}p^\ast}\\
\frac{1}{e}&=\frac{(1-p^\ast)^{-TR_C^{(i)\ast}}-1}{(1-p^\ast)^{FTR_C^{(i)\ast}}p^\ast}
\end{aligned}
$$
都把分母中 $(1-p)^C$ 的部分除上去，得到类似下面的函数
$$
h(p)=\frac{(1-p)^{C_1}-(1-p)^{C_2}}{p}
$$
其中 $C_1<C_2<0$。由于求导十分麻烦，因此在 $p=0$ 处对分子 Taylor 展开
$$
\begin{aligned}
k(p)&=(1-p)^C\\
&=1-Cp+\frac{C(C-1)}{2!}p^2-\frac{C(C-1)(C-2)}{3!}p^3+\dots
\end{aligned}
$$
则
$$
\begin{aligned}
h(p)&=(C_2-C_1)\\
&+\frac{C_1(C_1-1)-C_2(C_2-1)}{2!}p\\
&+\frac{C_2(C_2-1)(C_2-2)-C_1(C_1-1)(C_1-2)}{3!}p^2\\
&+\dots
\end{aligned}
$$
观察 $p$ 前系数，分奇偶讨论，可以得出系数均为正。因此对展开式求导，可知 $h'(p)$ 大于 0 恒成立。

> 考虑函数
> $$
> f(x)=\prod_{i=0}^n(x-i)
> $$
> 求导得到
> $$
> f'(x)=\sum_{j=0}^n\left(\prod_{i=0,i\neq j}^n (x-i)\right)
> $$
> 分析当 $x < 0$ 时的各项符号：
>
> 1. 由于 $i$ 的取值范围是 $0, 1, 2, \dots, n$，当 $x < 0$ 时，所有的 $(x-i)$ 都是负数。
> 2. 在 $f'(x)$ 的求和公式中，每一项 $\prod_{i \neq j} (x-i)$ 都是从 $n+1$ 个负数因子中抽掉了一个，因此每一项恰好是 $n$ 个负数的乘积。
>
> 因此，当 $n$ 为偶数时，$f'(x) > 0$ 恒成立，函数在负半轴上单调递增。当 $n$ 为奇数时，$f'(x) < 0$ 恒成立，函数在负半轴上单调递减。

类似第二项分析并联函数，第三项同样单调递增。至此，可以证明原式单调递增。

回顾原式
$$
\frac{\alpha^{(i)\ast}a^2}{(b+d)(c+e)}=\tau^2R_{\text{AI}}R_C^{(i)\ast}
$$
当 $p^\ast$ 取 $0$ 时，左式小于右式，$p^\ast$ 取 $1$ 时，左式大于右式，因此 $p^\ast$ 在 $(0,1)$ 上有唯一不动点，这使得队列长度也存在唯一不动点 $q^\ast$。

在收敛性一节中证明了每条流的不动点是 $R_C^{(i)\ast}=C/N$，接下来估计 $p^\ast$ 的大小。由于 $p^\ast$ 通常非常接近 0，因此考虑在 $p=0$ 处 Taylor 展开，忽略 $O(p^4)$。

> 实际上只展开了到 $p$ 忽略的是 $o(p^2)$。原论文大小 $o$ 还打错了。

考虑 $(1-p)^x=1-xp+o(p^2)\approx 1-xp$，因此
$$
\begin{aligned}
\alpha^{(i)\ast}&\approx \tau'R_C^{(i)\ast}p^\ast\\
a&\approx \tau R_C^{(i)\ast}p^\ast\\
b&\approx \frac{p^\ast}{(1+Bp^\ast)-1}=\frac{1}{B}\\
c&\approx \frac{(1-FBp^\ast)p^\ast}{(1+Bp^\ast)-1}\approx \frac{1}{B}\\
d&\approx \frac{p^\ast}{(1+TR_C^{(i)\ast}p^\ast)-1}=\frac{1}{TR_C^{(i)\ast}}\\
e&\approx \frac{(1-FTR_C^{(i)\ast}p^\ast)p^\ast}{(1+TR_C^{(i)\ast}p^\ast)-1}\approx \frac{1}{TR_C^{(i)\ast}}\\
\end{aligned}
$$

> 由于 $p^\ast$ 非常接近 0，因此同样忽略了 $c$ 和 $e$ 中的 $Cp^\ast$ 部分（$C$ 指某个常数）。

代入原式
$$
\frac{(\tau R_C^{(i)\ast}p^\ast)^2\tau'R_C^{(i)\ast}p^\ast}{(1/B+1/TR_C^{(i)\ast})^2}=\tau^2R_{\text{AI}}R_C^{(i)\ast}
$$
化简
$$
(p^\ast)^3=\frac{\tau^2R_\text{AI}}{\tau'(R_C^{(i)\ast})^2}\left(\frac{1}{B}+\frac{1}{TR_C^{(i)\ast}}\right)^2
$$
代入 $R_C^{(i)\ast}=C/N$，化简得
$$
p^\ast\approx \sqrt[3]{\frac{R_\text{AI} N^2}{\tau' C^2} \left( \frac{1}{B} + \frac{N}{TC} \right)^2}
$$


### 收敛性

收敛性的详细证明在 CoNEXT 论文的 3.3 节，是分析多次一次降速后的多次升速过程，通过证明任意两条流之间的速率差随时间指数级别减小来说明多条流可以收敛，最后推导出收敛至公平。但是需要特别注意的是，分析过程中仅存在计时器引导的升速，而不存在字节计数器引导的升速。

需要注意的是，字节计数器引导的升速是乘性增。根据前面的公式是可以算出 $R_C$ 关于 $t$ 的关系的，但由于我已修为尽失所以只能问 Gemini。

考虑不触发 CNP 且仅有字节计数器影响时，每发送 $B$ 字节触发一次升速，两次触发升速之间的时间为 $\Delta t=B/R_C$，对 $R_C$ 和 $R_T$ 类似求导
$$
\frac{\mathrm{d}R_C}{\mathrm{d}t}\approx \frac{\Delta R_C}{\Delta t}=\frac{R_T-R_C}{2B}R_C\\
\frac{\mathrm{d}R_T}{\mathrm{d}t}\approx \frac{\Delta R_T}{\Delta t}=\frac{R_\text{AI}}{B}R_C\\
$$
上下两式相除，得
$$
\frac{\mathrm{d}R_C}{\mathrm{d}R_T}+\frac{1}{2R_\text{AI}}R_C=\frac{R_T}{2R_\text{AI}}
$$
这是一个一阶线性常微分方程，求解得到
$$
R_C(R_T)=R_T-2R_\text{AI}+C_1\exp\left(-\frac{R_T}{2R_\text{AI}}\right)
$$
由于 $R_T\gg R_\text{AI}$，因此后一项趋近于 0，因此 $R_T-R_C\approx 2R_\text{AI}$。那么
$$
\frac{\mathrm{d}R_C}{\mathrm{d}t}\approx \frac{\Delta R_C}{\Delta t}=\frac{R_T-R_C}{2B}R_C\approx \frac{2R_\text{AI}}{2B}R_C=\frac{R_\text{AI}}{B}R_C
$$
我们考虑速率从 $R_\min$ 开始上升，则
$$
R_C(t)=R_\min\exp\left(\frac{R_\text{AI}}{B}t\right)
$$

## 问题

我一直觉得流体模型不适用网络分析，分析一般流体（比如水流）的时候是可以微分的，但是分析网络数据的时候，分析对象没办法对时间微分，因为数据都是以包为单位传输的，这样在求极限的时候显然就爆炸了，比如在 $t\to 0$ 时间内即使就发了一个包，求出的所谓「速率」也是一个极吊诡的值，是没有物理意义的。说到底，所有的发送速率都是一种平均速率，在微观层面分析的时候只有两个包之间的发送间隔是有意义的，但是对于整体分析并无帮助。在选择符号方面，应该避免使用微分，而应该选择平均速率，变化率等表达，并且不能求极限。

那网络分析应该怎么分析，包传输过程是量子化的，但量子理论里似乎也没啥可借鉴的东西。所以与其微积分不如简单一点，用平均变化率表达，这样符号也对物理意义也有，但是能分析出啥就不知道了。
