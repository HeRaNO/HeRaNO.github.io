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

> 实际上，这里的速率或字节计数器是被包长度归一化的，因为要利用 ECN 概率进行计算，而 ECN 是逐包的，所以在计算上需要进行归一化。

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

> 强行解释就是我们希望计算在 $\tau'$ 时间内 $\alpha$ 的平均变化率，而 $\tau'$ 是一个极小量，然后变化量期望代表着平均变化量，一除就是平均变化率了，反正不是的话也推不出后面的东西。这⾥⽤的是流体均值近似 ODE，作为建模来说相当粗糙。

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

> 但为什么是这样是很无厘头的。首先不是所有的发送数据都形成队列，假设这些数据都形成队列了，虽然 ECN 是根据队列长度变化的，但是按理说这个概率应该和包个数有关，而不是字节数，这里的指数位置考虑已经进行了归一化。然后是打 ECN 标记的时间并非控制回路延迟 $\tau^\ast$，综合起来这公式混合了不同时刻发生的事情，虽然差距很小但还是有差距，但是数据中心内这点差距也不算啥，数学是天体物理老师教的，就只能感性理解，不要用它来计算。长距链路上带入这个 $\tau^\ast$ 显然就是错的了。

### 公式 9

$$
\begin{aligned}
\frac{\mathrm{d}R_C}{\mathrm{d}t}=&-\frac{R_C(t)\alpha(t)}{2\tau}\left(1-(1-p(t-\tau^\ast))^{\tau R_C(t-\tau^\ast)}\right)\\
&+\frac{R_T(t)-R_C(t)}{2}\frac{R_C(t-\tau^\ast)p(t-\tau^\ast)}{(1-p(t-\tau^\ast))^{-B}-1}\\
&+\frac{R_T(t)-R_C(t)}{2}\frac{R_C(t-\tau^\ast)p(t-\tau^\ast)}{(1-p(t-\tau^\ast))^{-TR_C(t-\tau^\ast)}-1}
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

> 有一个没什么意义的推导。如果发送端每隔 $\tau$ 时间就收到一个 CNP，$\alpha$ 也是收敛的。设 $n=\lfloor \tau/\tau'\rfloor$，在等待下一个 CNP 的时间 $\tau$ 内，$\alpha$ 衰减 $n$ 次。因此在**下一个 CNP 刚好到达前**，$\alpha$ 衰减到：
> $$
> \alpha_{\text{nxt}} = [(1-g)\alpha_\text{now}+g](1-g)^n
> $$
> 因为当前 $\alpha$ 处于稳态，所以有 $\alpha_{\text{nxt}}=\alpha_{\text{now}}$，令稳态 $\alpha^\ast=\alpha_{\text{nxt}}=\alpha_{\text{now}}$，则有
> $$
> \alpha^\ast = [(1-g)\alpha^\ast+g](1-g)^n
> $$
> 解得
> $$
> \alpha^\ast=\frac{g(1-g)^n}{1-(1-g)^{n+1}}
> $$
> 考虑 $\alpha$ 更新的递推式
> $$
> \alpha_{k} = \left[ (1-g)\alpha_{k-1} + g \right] (1-g)^n
> $$
> 为了方便计算，我们令一个衰减常数 $A = (1-g)^{n+1}$。将上式展开并整理，得到一个标准的一阶线性递推数列：
>
> $$
> \alpha_{k} = A \cdot \alpha_{k-1} + g(1-g)^n
> $$
> 已知该数列的最终收敛稳态为 $\alpha^\ast$，我们可以利用稳态值将递推式改写为：
>
> $$
> \alpha_{k} - \alpha^\ast = A \cdot (\alpha_{k-1} - \alpha^\ast)
> $$
> 不断向下递推，可以得到第 $k$ 次迭代后的 $\alpha_k$ 与初始值 $\alpha_0$ 的关系：
>
> $$
> \alpha_{k} - \alpha^\ast = A^k \cdot (\alpha_0 - \alpha^\ast)
> $$
> 如果进入稳态，则对任意给定 $\varepsilon>0$，存在正整数 $N$ 使得对于 $k>N$ 都有
> $$
> | \alpha_k - \alpha^\ast | < \varepsilon
> $$
> 成立。
>
> 将上面的通项公式代入：
>
> $$
> A^k \cdot | \alpha_0 - \alpha^\ast | \le \varepsilon
> $$
> 两边同时取自然对数（注意 $A < 1$，所以 $\ln(A)$ 是负数，除以负数时不等号要变号）：
>
> $$
> k \ge \frac{\ln\left( \frac{\varepsilon}{|\alpha_0 - \alpha^\ast|} \right)}{\ln(A)}
> $$
> 化简得
> $$
> k\ge \frac{ \ln\varepsilon + \ln\left( \frac{1 - (1-g)^{n+1}}{1 - (1-g)^n} \right) }{ (n+1)\ln(1-g) }
> $$
> 但由于不可能一直收到 CNP，这意味着拥塞一直不缓解，并且有时进入收敛态的 $n$ 可能很大，所以计算这个确实没什么意义。

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
g'(p)=\frac{-C(1-p)^{C-1}p-(1-p)^C+1}{p^2}
$$
其中，$-C(1-p)^{C+1}p\ge 0$ 在 $[0,1]$ 上恒成立，考虑 $h(p)=1-(1-p)^C$ 在 $[0,1]$ 上的取值，继续求导。
$$
h'(p)=-C(1-p)^{C-1}
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
\frac{1}{c}&=\frac{(1-p^\ast)^{-B}-1}{(1-p^\ast)^{FB}p^\ast}=(1-p^\ast)^{-FB}\cdot \frac{1}{b}\\
\frac{1}{e}&=\frac{(1-p^\ast)^{-TR_C^{(i)\ast}}-1}{(1-p^\ast)^{FTR_C^{(i)\ast}}p^\ast}=(1-p^\ast)^{-FTR_{C}^{(i)\ast}}\cdot \frac{1}{d}
\end{aligned}
$$
由于 $(1-p^\ast)^{-FB}$ 和 $(1-p^\ast)^{-FTR_C^{(i)\ast}}$ 也在 $[0,1)$ 上单调增，所以 $\frac{1}{c}$ 和 $\frac{1}{e}$ 也是单调增的。

类似第二项分析并联函数，第三项同样单调递增。至此，可以证明原式单调递增。

回顾原式
$$
\frac{\alpha^{(i)\ast}a^2}{(b+d)(c+e)}=\tau^2R_{\text{AI}}R_C^{(i)\ast}
$$
当 $p^\ast$ 取 $0$ 时，左式小于右式，$p^\ast$ 取 $1$ 时，左式大于右式，因此 $p^\ast$ 在 $(0,1)$ 上有唯一不动点，这使得队列长度也存在唯一不动点 $q^\ast$。

在收敛性一节中证明了每条流的不动点是 $R_C^{(i)\ast}=C/N$，接下来估计 $p^\ast$ 的大小。由于 $p^\ast$ 通常非常接近 0，因此考虑在 $p=0$ 处 Taylor 展开。考虑 $(1-p)^x=1-xp+o(p)\approx 1-xp$，因此

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

忽略左侧产生的 $o(p^3)$，代入原式
$$
\frac{(\tau R_C^{(i)\ast}p^\ast)^2\tau'R_C^{(i)\ast}p^\ast}{(1/B+1/TR_C^{(i)\ast})^2}=\tau^2R_{\text{AI}}R_C^{(i)\ast}
$$
化简
$$
(p^\ast)^3=\frac{R_\text{AI}}{\tau'(R_C^{(i)\ast})^2}\left(\frac{1}{B}+\frac{1}{TR_C^{(i)\ast}}\right)^2
$$
代入 $R_C^{(i)\ast}=C/N$，化简得
$$
p^\ast\approx \sqrt[3]{\frac{R_\text{AI} N^2}{\tau' C^2} \left( \frac{1}{B} + \frac{N}{TC} \right)^2}
$$

### 稳定性

DCQCN 论文中并没有直接证明稳定性，而是依赖 QCN 的稳定性证明。QCN 也没有直接证明稳定性，而是通过如下路径来证明：

1. QCN 利用了 Average Principle（似乎没有对应的汉语翻译）；
2. AP 在线性控制系统中和 PD（比例微分）控制器代数意义上等价，即给 AP 和 PD 等价的输入，它们的输出是等价的；
3. PD 具有稳定性，那么 AP 也具有稳定性；
4. 所以 QCN 是稳定的。

在 CoNEXT 论文中通过数值方法证明了一部分稳定性问题。首先通过系统线性化和 Laplace 变换得到 DCQCN 的特征方程。

仍然考虑公式 9，我们直接将公式写成如下形式。
$$
\begin{aligned}
\frac{\mathrm{d}R_C}{\mathrm{d}t} &=- \frac{R_C \alpha}{2\tau} \left( 1 - (1-p)^{\tau R_C} \right)\\ 
&+ \frac{R_T - R_C}{2} \left[ \frac{R_C p}{(1-p)^{-B} - 1} + \frac{R_C p}{(1-p)^{-TR_C} - 1} \right]
\end{aligned}
$$

> 这里从第一步开始就忽略了反馈延迟。

在平衡点附近，标记概率 $p$ 非常接近 0，所以仍然考虑 Taylor 展开。

对于 $(1-p)^{\tau R_C}$，当 $p \to 0$ 时，$(1-p)^x \approx 1 - xp$，因此
$$
1 - (1-p)^{\tau R_C} \approx 1 - (1 - \tau R_C p) = \tau R_C p
$$
对于后面的分式，其核心为 $g(x) = \frac{p}{(1-p)^{-x} - 1}$（其中 $x$ 为 $B$ 或 $TR_C$）。考虑利用 Taylor 展开化简 $(1-p)^{-x}$ 的部分，为了保证线性化的精度，分母必须保留到二阶：
$$
(1-p)^{-x} = 1 + xp + \frac{x(x+1)}{2}p^2 + o(p^2)
$$
将结果代回分式
$$
g(x) \approx \frac{p}{xp(1 + \frac{x+1}{2}p)} = \frac{1}{x} \left(1 + \frac{x+1}{2}p\right)^{-1}
$$
由于 $p\to 0$，那么 $\frac{x+1}{2}p\to 0$，由于 $(1+x)^{-1}=\sum_{n=0}^\infty (-1)^nx^n\approx 1-x$，仍可进一步写成
$$
g(x)\approx \frac{1}{x}\left(1-\frac{x+1}{2}p\right)=\frac{1}{x}-\left(\frac{1}{2}+\frac{1}{2x}\right)p
$$
把 $x=B$ 和 $x=TR_C$ 代进去
$$
\begin{aligned}
&\left[ \frac{1}{B} - \left(\frac{1}{2} + \frac{1}{2B}\right)p \right] + \left[ \frac{1}{TR_C} - \left(\frac{1}{2} + \frac{1}{2TR_C}\right)p \right]\\
&= \left(\frac{1}{B} + \frac{1}{TR_C}\right) - p - \frac{1}{2}\left(\frac{1}{B} + \frac{1}{TR_C}\right)p
\end{aligned}
$$
令 $A=\frac{1}{B}+\frac{1}{TR_C}$，则上式可以写成
$$
A - p - \frac{A}{2}p = A - \left(1 + \frac{A}{2}\right)p
$$
把所有部分都合起来
$$
\begin{aligned}
\frac{\mathrm{d}R_C}{\mathrm{d}t} &=- \frac{R_C \alpha}{2\tau} \left( 1 - (1-p)^{\tau R_C} \right)\\ 
&+ \frac{R_T - R_C}{2} \left[ \frac{R_C p}{(1-p)^{-B} - 1} + \frac{R_C p}{(1-p)^{-TR_C} - 1} \right]\\
&=- \frac{R_C \alpha}{2\tau} \tau R_Cp+ \frac{R_C(R_T - R_C)}{2} \left[ A - \left(1 + \frac{A}{2}\right)p \right]
\end{aligned}
$$
拆开，约分，化简
$$
\begin{aligned}
\frac{\mathrm{d}R_C}{\mathrm{d}t} =- \frac{1}{2}R_C^2 \alpha p+\frac{A}{2}(R_TR_C - R_C^2) - \left(\frac{1}{2}+ \frac{A}{4}\right)(R_TR_Cp - R_C^2p)
\end{aligned}
$$

> 如果不忽略反馈延迟，应该是
> $$
> \begin{aligned}
> \frac{\mathrm{d}R_C}{\mathrm{d}t} &=- \frac{1}{2}R_C(t)R_C(t-\tau^\ast) \alpha p\\
> &+\frac{A}{2}(R_TR_C(t-\tau^\ast) - R_C(t)R_C(t-\tau^\ast))\\
> &- \left(\frac{1}{2}+ \frac{A}{4}\right)(R_TR_C(t-\tau^\ast)p - R_C(t)R_C(t-\tau^\ast)p)
> \end{aligned}
> $$
> 这里的 $\alpha$ 都是 $\alpha(t)$，$R_T$ 都是 $R_T(t)$，$p$ 都是 $p(t-\tau^\ast)$，$A$ 中的 $R_C$ 都是 $R_C(t-\tau^\ast)$。

现在我们得到的系统方程仍然不是线性的，只有线性化后才能进行 Laplace 变换，所以我们在不动点处对 $\frac{\mathrm{d}R_C}{\mathrm{d}t}$ 进行一阶 Taylor 展开。

令
$$
F(R_C, R_T, p, \alpha)=- \frac{1}{2}R_C^2 \alpha p+\frac{A}{2}(R_TR_C - R_C^2) - \left(\frac{1}{2}+ \frac{A}{4}\right)(R_TR_Cp - R_C^2p)
$$
对其在不动点处进行 Taylor 展开。这个过程十分掉 san。

首先需要复习一下多元函数的一阶 Taylor 展开

> 设多元函数 $f(x_1,x_2,\ldots,x_n)$ 在 $(a_1,a_2,\ldots,a_n)$ 处一阶可偏导，则有：
> $$
> \begin{aligned}
> f(x_1,x_2,\ldots,x_n)&=f(a_1,a_2,\ldots,a_n)+\sum_{i=1}^n(x_i-a_i)\frac{\partial f}{\partial x_i}(a_1,a_2,\ldots,a_n)\\
> &+R(x_1,x_2,\ldots,x_n)
> \end{aligned}
> $$
> 其中 $R(x_1,x_2,\ldots,x_n)$ 为余项。但是接下来写的所有式子全部忽略余项。

对于第一个式子
$$
\begin{aligned}
F_1(R_C,\alpha,p)&=F_1(R_C^\ast,\alpha^\ast,p^\ast)\\
&-R_C^\ast\alpha^\ast p^\ast(R_C-R_C^\ast)-\frac{1}{2}(R_C^\ast)^2p^\ast(\alpha-\alpha^\ast)-\frac{1}{2}(R_C^\ast)^2\alpha^\ast(p-p^\ast)\\
\end{aligned}
$$
对于第二个式子，令 $A^\ast=\frac{1}{B}+\frac{1}{TR_C^\ast}$。
$$
\begin{aligned}
F_2(R_C,R_T)=&\frac{1}{2}\left(\frac{1}{B}+\frac{1}{TR_C}\right)(R_TR_C - R_C^2)\\
=&\frac{1}{2}\left(\frac{1}{B}(R_TR_C - R_C^2)+\frac{1}{T}(R_T - R_C)\right)\\
=&F_2(R_C^\ast,R_T^\ast)\\
&+\frac{1}{2}\left(\frac{R_C^\ast}{B}+\frac{1}{T}\right)(R_T-R_T^\ast)\\
&+\frac{1}{2}\left(\frac{R_T^\ast}{B}-\frac{2R_C^\ast}{B}-\frac{1}{T}\right)(R_C-R_C^\ast)\\
=&F_2(R_C^\ast,R_T^\ast)\\
&+\frac{1}{2}\left(\frac{1}{B}+\frac{1}{TR_C^\ast}\right)R_C^\ast(R_T-R_T^\ast)\\
&+\frac{1}{2}\left(\frac{R_T^\ast - 2R_C^\ast}{B} + \frac{R_T^\ast - 2R_C^\ast}{TR_C^\ast} - \frac{R_T^\ast - 2R_C^\ast}{TR_C^\ast} - \frac{1}{T}\right)(R_C-R_C^\ast)\\
=&F_2(R_C^\ast,R_T^\ast)\\
&+\frac{1}{2}\left(\frac{1}{B}+\frac{1}{TR_C^\ast}\right)R_C^\ast(R_T-R_T^\ast)\\
&+\frac{1}{2}\left(\frac{1}{B}+\frac{1}{TR_C^\ast}\right)\left(R_T^\ast - 2R_C^\ast \right)(R_C-R_C^\ast)\\
&-\frac{R_T^\ast - R_C^\ast}{2TR_C^\ast}(R_C-R_C^\ast)\\
=&F_2(R_C^\ast,R_T^\ast)+\frac{A^\ast}{2}R_C^\ast(R_T-R_T^\ast)+\frac{A^\ast}{2}\left(R_T^\ast - 2R_C^\ast \right)(R_C-R_C^\ast)\\
&-\frac{R_T^\ast - R_C^\ast}{2TR_C^\ast}(R_C-R_C^\ast)\\
\end{aligned}
$$
> 参考推不动点的过程，其实稳态时 $R_T^\ast\neq R_C^\ast$，所以最后还不能省掉。

再对第三项同样展开，会很长很长。
$$
\begin{aligned}
F_3(R_C,R_T,p)=&-\frac{1}{2}\left(1+\frac{1}{2B}+\frac{1}{2TR_C}\right)(R_TR_Cp - R_C^2p)\\
=&-\frac{1}{2}\left((R_TR_Cp - R_C^2p)+\frac{1}{2B}(R_TR_Cp - R_C^2p)+\frac{1}{2T}(R_Tp - R_Cp)\right)\\
=&F_3(R_C^\ast,R_T^\ast,p^\ast)\\
&-\frac{1}{2}\left(R_C^\ast p^\ast+\frac{R_C^\ast p^\ast}{2B}+\frac{p^\ast}{2T}\right)(R_T-R_T^\ast)\\
&-\frac{1}{2}\left(R_T^\ast p^\ast-2R_C^\ast p^\ast+\frac{R_T^\ast p^\ast-2R_C^\ast p^\ast}{2B}-\frac{p^\ast}{2T}\right)(R_C-R_C^\ast)\\
&-\frac{1}{2}\left(R_T^\ast R_C^\ast-(R_C^\ast)^2+\frac{R_T^\ast R_C^\ast-(R_C^\ast)^2}{2B}+\frac{R_T^\ast-R_C^\ast}{2T}\right)(p-p^\ast)\\
=&F_3(R_C^\ast,R_T^\ast,p^\ast)\\
&-\frac{1}{2}\left(1+\frac{1}{2B}+\frac{1}{2TR_C^\ast}\right)R_C^\ast p^\ast(R_T-R_T^\ast)\\
&-\frac{1}{2}\left(R_T^\ast p^\ast-2R_C^\ast p^\ast+\frac{R_T^\ast p^\ast-2R_C^\ast p^\ast}{2B}+\frac{R_T^\ast p^\ast-2R_C^\ast p^\ast}{2TR_C^\ast}-\frac{R_T^\ast p^\ast-2R_C^\ast p^\ast}{2TR_C^\ast}-\frac{p^\ast}{2T}\right)(R_C-R_C^\ast)\\
&-\frac{1}{2}\left(1+\frac{1}{2B}+\frac{1}{2TR_C^\ast}\right)(R_T^\ast R_C^\ast-(R_C^\ast)^2)(p-p^\ast)\\
=&F_3(R_C^\ast,R_T^\ast,p^\ast)\\
&-\frac{1}{2}\left(1+\frac{1}{2B}+\frac{1}{2TR_C^\ast}\right)R_C^\ast p^\ast(R_T-R_T^\ast)\\
&-\frac{1}{2}\left(1+\frac{1}{2B}+\frac{1}{2TR_C^\ast}\right)\left(R_T^\ast p^\ast-2R_C^\ast p^\ast\right)(R_C-R_C^\ast)\\
&+\frac{R_T^\ast p^\ast-R_C^\ast p^\ast}{4TR_C^\ast}(R_C-R_C^\ast)\\
&-\frac{1}{2}\left(1+\frac{1}{2B}+\frac{1}{2TR_C^\ast}\right)(R_T^\ast R_C^\ast-(R_C^\ast)^2)(p-p^\ast)\\
=&F_3(R_C^\ast,R_T^\ast,p^\ast)\\
&-\left(\frac{1}{2}+\frac{A^\ast}{4}\right)R_C^\ast p^\ast(R_T-R_T^\ast)\\
&-\left(\frac{1}{2}+\frac{A^\ast}{4}\right)\left(R_T^\ast p^\ast-2R_C^\ast p^\ast\right)(R_C-R_C^\ast)\\
&-\left(\frac{1}{2}+\frac{A^\ast}{4}\right)(R_T^\ast R_C^\ast-(R_C^\ast)^2)(p-p^\ast)\\
&+\frac{R_T^\ast - R_C^\ast}{4TR_C^\ast}p^\ast(R_C-R_C^\ast)
\end{aligned}
$$
令 $\delta R_C(t)=R_C(t)-R_C^\ast$，$\delta R_T(t)=R_T(t)-R_T^\ast$，$\delta p(t)=p(t)-p^\ast$，$\delta \alpha(t)=\alpha(t)-\alpha^\ast$。把上面的式子合起来。

> 实际上所有的 $p$ 都是 $p(t-\tau^\ast)$。

$$
\begin{aligned}
\frac{\mathrm{d}R_C}{\mathrm{d}t}=& F_1 + F_2 + F_3 \\
=& -\frac{1}{2}(R_C^\ast)^2\alpha^\ast\delta p - \frac{1}{2}p^\ast R_C^\ast\alpha^\ast\delta R_C \notag \\
&-\frac{1}{2}p^\ast R_C^\ast\alpha^\ast\delta R_C - \frac{1}{2}p^\ast(R_C^\ast)^2\delta\alpha \notag \\
&+\frac{A^\ast}{2}\left(R_C^\ast\delta R_T - R_C^\ast\delta R_C + R_T^\ast\delta R_C - R_C^\ast\delta R_C\right) \notag \\
&-\left(\frac{1}{2}+\frac{A^\ast}{4}\right)\left(p^\ast R_C^\ast\delta R_T - p^\ast R_C^\ast\delta R_C + p^\ast R_T^\ast\delta R_C\right) \notag \\
&-\left(\frac{1}{2}+\frac{A^\ast}{4}\right)\left(p^\ast R_C^\ast\delta R_C + R_C^\ast R_T^\ast\delta p - (R_C^\ast)^2\delta p\right) \notag \\
&-\frac{R_T^\ast - R_C^\ast}{2TR_C^\ast}\delta R_C + \frac{R_T^\ast - R_C^\ast}{4TR_C^\ast}p^\ast\delta R_C
\end{aligned}
$$
由于稳定点处导数等于 0，所以可以将数值部分直接求出。

> 但这个方程并不严谨，我们仍需考虑 $\tau^\ast$ 项的影响。考虑之后应该长这样
> $$
> \begin{aligned}
> \frac{\mathrm{d} R_C(t)}{\mathrm{d}t} =& \left[ -\frac{1}{2}(R_C^\ast)^2 \alpha^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)(R_T^\ast R_C^\ast - (R_C^\ast)^2) \right] \delta p(t-\tau^\ast)\\ 
> &+ \left[ -\frac{1}{2}(R_C^\ast)^2 p^\ast \right] \delta \alpha(t)\\ 
> &+ \left[ \frac{A^\ast}{2}R_C^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)R_C^\ast p^\ast \right] \delta R_T(t)\\ 
> &+ \left[ -\frac{1}{2}R_C^\ast \alpha^\ast p^\ast - \frac{A^\ast}{2}R_C^\ast + \left(\frac{1}{2}+\frac{A^\ast}{4}\right)R_C^\ast p^\ast \right] \delta R_C(t)\\ 
> &+ \left[ -\frac{1}{2}R_C^\ast \alpha^\ast p^\ast + \frac{A^\ast}{2}(R_T^\ast - R_C^\ast) - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)(R_T^\ast - R_C^\ast )p^\ast \right] \delta R_C(t-\tau^\ast)
> \end{aligned}
> $$
> 至于论文里为什么是 $\frac{\mathrm{d}\delta R_C}{\mathrm{d}t}$，因为 $\delta R_C=R_C(t)-R_C^\ast$，对常数求导是 0 所以也可以写成这个东西。感觉并不重要。

接下来，对上面线性化的方程进行 Laplace 变换，以进行稳定性分析。

首先我们需要知道 Laplace 变换是什么。

> 对于所有实数 $t\ge 0$，函数 $f(t)$ 的 Laplace 变换是函数 $F(s)$，定义为：
> $$
> F(s)=\int_0^{\infty} \exp(-st)f(t) \mathrm{d}t
> $$
> 其中频率参数 $s$ 是一个复数：$s=\sigma+\mathrm{i}\omega$，其中 $\sigma$ 和 $\omega$ 是实数。
>
> 对 $f$ 的 Laplace 变换可记作 $\mathcal{L}\{f\}$。

那么我们现在是对一个微分进行 Laplace 变换，即求 $\mathcal{L}(f')$。那么利用分部积分
$$
\begin{aligned}
\int_{0}^{\infty} \exp(-st)f'(t) \mathrm{d}t &=\int_{0}^{\infty} \exp(-st) \mathrm{d}f(t)\\
&= \left[ \exp(-st)f(t) \right]_{0}^{\infty} - \int_{0}^{\infty} f(t) \cdot (-s \exp(-st)) \mathrm{d}t\\
&=s\int_{0}^{\infty} \exp(-st) f(t)  \mathrm{d}t-f(0)\\
&=sF(s)-f(0)
\end{aligned}
$$
定义 $R_C(s)$ 为 $\mathcal{L}\{R_C(t)\}$。类似定义，$R_T(s)=\mathcal{L}\{R_T(t)\}$，$p(s)=\mathcal{L}\{p(t-\tau^\ast)\}$，$\alpha(s)=\mathcal{L}\{\alpha(t)\}$。

> 这里符号确实乱了，将就用吧。

可以发现等式右侧是各种 $\delta$ 的线性组合，积分一个就能以类似形式得到其他的。比如
$$
\begin{aligned}
\mathcal{L}\{\delta R_C(t)\}&=\int_0^{\infty}\exp(-st)\delta R_C(t)\mathrm{d}t\\
&=\int_0^{\infty}\exp(-st) R_C(t)\mathrm{d}t-\int_0^{\infty}\exp(-st) R_C^\ast\mathrm{d}t\\
&=\int_0^{\infty}\exp(-st) R_C(t)\mathrm{d}t-\frac{R_C^\ast}{s}\int_{-\infty}^{0}\exp(-st) \mathrm{d}(-st)\\
&=R_C(s)-\frac{R_C^\ast}{s}
\end{aligned}
$$
又如
$$
\begin{aligned}
\mathcal{L}\{\delta p(t-\tau^\ast)\}&=\int_0^{\infty}\exp(-st)\delta p(t-\tau^\ast)\mathrm{d}t\\
&=\int_0^{\infty}\exp(-st) p(t-\tau^\ast)\mathrm{d}t-\int_0^{\infty}\exp(-st) p^\ast\mathrm{d}t\\
&=\int_0^{\infty}\exp(-s\tau^\ast)\exp(-s(t-\tau^\ast)) p(t-\tau^\ast)\mathrm{d}t-\int_0^{\infty}\exp(-st) p^\ast\mathrm{d}t\\
&=\exp(-s\tau^\ast)\int_{-\tau^\ast}^{\infty}\exp(-su) p(u)\mathrm{d}u-\int_0^{\infty}\exp(-st) p^\ast\mathrm{d}t\\
&=\exp(-s\tau^\ast)\int_{-\tau^\ast}^{\infty}\exp(-su) p(u)\mathrm{d}u-\frac{p*}{s}\\
&=\exp(-s\tau^\ast)\int_{0}^{\infty}\exp(-su) p(u)\mathrm{d}u-\frac{p*}{s}\\
&=\exp(-s\tau^\ast)p(s)-\frac{p*}{s}
\end{aligned}
$$

> 这里的扰动视为因果信号，因此有 $\mathcal{L}\{f(t-\tau)\}=\exp(-s\tau)F(s)$。

然后我们把这堆东西合进去。
$$
\begin{aligned}
s R_C(s) - R_C(0) =&\ \left[-\frac{1}{2}(R_C^\ast)^2\alpha^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)R_C^\ast R_T^\ast + \left(\frac{1}{2}+\frac{A^\ast}{4}\right)(R_C^\ast)^2\right] e^{-s\tau^\ast} p(s) \\
&\ + \left[-\frac{1}{2}p^\ast R_C^\ast \alpha^\ast - \frac{A^\ast}{2}R_C^\ast + \left(\frac{1}{2}+\frac{A^\ast}{4}\right)p^\ast R_C^\ast\right] e^{-s\tau^\ast} R_C(s) \\
&\ + \left[-\frac{1}{2}p^\ast R_C^\ast \alpha^\ast - \frac{A^\ast}{2}R_C^\ast + \frac{A^\ast}{2}R_T^\ast + \left(\frac{1}{2}+\frac{A^\ast}{4}\right)p^\ast R_C^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)p^\ast R_T^\ast\right] R_C(s) \\
&\ - \frac{1}{2}p^\ast (R_C^\ast)^2 \alpha(s) + \left[\frac{A^\ast}{2}R_C^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)p^\ast R_C^\ast\right] R_T(s) \\
&\ - \frac{1}{s} \left[-\frac{1}{2}(R_C^\ast)^2\alpha^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)R_C^\ast R_T^\ast + \left(\frac{1}{2}+\frac{A^\ast}{4}\right)(R_C^\ast)^2\right] p^\ast e^{-s\tau^\ast} \\
&\ - \frac{1}{s} \left[-\frac{1}{2}p^\ast R_C^\ast \alpha^\ast - \frac{A^\ast}{2}R_C^\ast + \left(\frac{1}{2}+\frac{A^\ast}{4}\right)p^\ast R_C^\ast\right] R_C^\ast e^{-s\tau^\ast} \\
&\ - \frac{1}{s} \left[-\frac{1}{2}p^\ast R_C^\ast \alpha^\ast - \frac{A^\ast}{2}R_C^\ast + \frac{A^\ast}{2}R_T^\ast + \left(\frac{1}{2}+\frac{A^\ast}{4}\right)p^\ast R_C^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)p^\ast R_T^\ast\right] R_C^\ast \\
&\ + \frac{1}{s} \left[\frac{1}{2}(p^\ast)^2 (R_C^\ast)^2\right]\alpha^\ast - \frac{1}{s} \left[\frac{A^\ast}{2}R_C^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)p^\ast R_C^\ast\right] R_T^\ast
\end{aligned}
$$

> 同样的，这个公式也不严谨，考虑 $\tau^\ast$ 后为
> $$
> \begin{aligned}
> s R_C(s) - \delta R_C(0) =& \left[ -\frac{1}{2}(R_C^\ast)^2 \alpha^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)(R_T^\ast R_C^\ast - (R_C^\ast)^2) \right] e^{-s\tau^\ast} p(s)\\ 
> &+ \left[ -\frac{1}{2}(R_C^\ast)^2 p^\ast \right] \alpha(s)\\ 
> &+ \left[ \frac{A^\ast}{2}R_C^\ast - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)R_C^\ast p^\ast \right] R_T(s)\\ 
> &+ \left[ -\frac{1}{2}R_C^\ast \alpha^\ast p^\ast - \frac{A^\ast}{2}R_C^\ast + \left(\frac{1}{2}+\frac{A^\ast}{4}\right)R_C^\ast p^\ast \right] R_C(s)\\ 
> &+ \left[ -\frac{1}{2}R_C^\ast \alpha^\ast p^\ast + \frac{A^\ast}{2}(R_T^\ast - R_C^\ast) - \left(\frac{1}{2}+\frac{A^\ast}{4}\right)(R_T^\ast  - R_C^\ast)p^\ast \right] e^{-s\tau^\ast} R_C(s)
> \end{aligned}
> $$
> 这个公式考虑吸收了 $\frac{1}{s}$ 的部分，实际上论文的公式也吸收了 $\frac{1}{s}$ 的部分。

### 收敛性

收敛性的详细证明在 CoNEXT 论文的 3.3 节，是分析多次一次降速后的多次升速过程，通过证明任意两条流之间的速率差随时间指数级别减小来说明多条流可以收敛，最后推导出收敛至公平。但是需要特别注意的是，分析过程中仅存在计时器引导的升速，而不存在字节计数器引导的升速。

需要注意的是，字节计数器引导的升速是乘性增。根据前面的公式是可以算出 $R_C$ 关于 $t$ 的关系的，但由于我已修为尽失所以只能问 Gemini。

考虑不触发 CNP 且仅有字节计数器影响时，每发送 $B$ 字节触发一次升速，两次触发升速之间的时间为 $\Delta t=B/R_C$，对 $R_C$ 和 $R_T$ 类似求导
$$
\frac{\mathrm{d}R_C}{\mathrm{d}t}\approx \frac{\Delta R_C}{\Delta t}=\frac{R_T-R_C+R_\text{AI}}{2B}R_C\\
\frac{\mathrm{d}R_T}{\mathrm{d}t}\approx \frac{\Delta R_T}{\Delta t}=\frac{R_\text{AI}}{B}R_C\\
$$
上下两式相除，得
$$
\frac{\mathrm{d}R_C}{\mathrm{d}R_T}+\frac{1}{2R_\text{AI}}R_C=\frac{R_T+R_\text{AI}}{2R_\text{AI}}
$$
这是一个一阶线性常微分方程，求解得到
$$
R_C(R_T)=R_T-R_\text{AI}+C_1\exp\left(-\frac{R_T}{2R_\text{AI}}\right)
$$
由于 $R_T\gg R_\text{AI}$，因此后一项趋近于 0，因此 $R_T-R_C\approx R_\text{AI}$。那么
$$
\frac{\mathrm{d}R_C}{\mathrm{d}t}\approx \frac{\Delta R_C}{\Delta t}=\frac{R_T-R_C}{2B}R_C\approx \frac{2R_\text{AI}}{2B}R_C=\frac{R_\text{AI}}{B}R_C
$$
考虑速率从 $R_\min$ 开始上升这一过程，实际上在最初主要起作用的是计时器，因为速率很低，升速计时器触发时发送字节数还没有达到 $B$，因此在最初还是进行加性增，直到一计时器时间间隔内发送数据量大于等于 $B$，则此时开始的发送速率为 $B/T$，之后的增速由字节计数器驱动，因此后面的增速过程可以表达为
$$
R_C(t)=\frac{B}{T}\exp\left(\frac{R_\text{AI}}{B}t\right)
$$

也就是说这个乘性增只在增速到一定程度后才起作用，此时可能网络空闲带宽很多，需要快速探测带宽，这样的设计倒也是有其合理性。

## 问题

我一直觉得流体模型不适用网络分析，分析一般流体（比如水流）的时候是可以微分的，但是分析网络数据的时候，分析对象没办法对时间微分，因为数据都是以包为单位传输的，这样在求极限的时候显然就爆炸了，比如在 $t\to 0$ 时间内即使就发了一个包，求出的所谓「速率」也是一个极吊诡的值，是没有物理意义的。说到底，所有的发送速率都是一种平均速率，在微观层面分析的时候只有两个包之间的发送间隔是有意义的，但是对于整体分析并无帮助。在选择符号方面，应该避免使用微分，而应该选择平均速率，变化率等表达，并且不能求极限。

那网络分析应该怎么分析，包传输过程是量子化的，但量子理论里似乎也没啥可借鉴的东西。所以与其微积分不如简单一点，用平均变化率表达，这样符号也对物理意义也有，但是能分析出啥就不知道了。
