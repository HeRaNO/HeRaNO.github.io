---
title: 2021, 2022 年电子科技大学 ICPC 暑假前集训数学与几何专题部分题解
date: 2022-07-01 21:15:12
categories: 'OI/ICPC'
tags:
	- 博弈论
	- 数论
	- 组合数学
	- 极角序
	- 凸包
	- 旋转卡壳
description: ' '
---

只有部分题解，完整题解实在太多了，写不过来了（主要是不会）……

排序主要按难度和相关性，并没有按时间顺序和题号顺序排。

以下多组数据题目的时间复杂度均标注的是单组数据的时间复杂度。

## 黑灰游戏

### 题目描述

$n$ 个石子从左到右排成一排，两人每人每次从左右两端中的某一端恰好拿走 $k$ 个石子，拿不走 $k$ 个石子的话游戏停止。拿走石子数多者胜。问最后谁会胜利。

数据范围：$1\le k\le n\le 10^9$。

### 题解

如果 $\lfloor \frac{n}{k}\rfloor$ 是奇数，则 Kate 获胜，否则二人平局。因为这个游戏与从左取，从右取还是任意取黑灰是无关的，无论先后手状态都是确定的。如果 $\lfloor \frac{n}{k}\rfloor$ 是奇数，则先手会比后手多拿一轮 $k$ 个石子，偶数的话两人拿的轮数是一样多的。

来自 [Codeforces Round #425 (Div. 2) A. Sasha and Sticks](https://codeforces.com/problemset/problem/832/A)。

时间复杂度：$\mathcal{O}(1)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2653.cpp)

## 黑灰游戏 · 改

### 题目描述

$n$ 堆石子从左到右排成一排，第 $i$ 堆石子有 $a_i$ 个。令轮数从 $1$ 开始增加，游戏按如下步骤进行：

- 奇数轮由 Kate 指定一堆非空的石子堆开始。然后 Emilico 从这堆石子中拿走至少一个石子。
- 偶数轮由 Emilico 指定一堆非空的黑灰堆开始。然后 Kate 从这堆石子中拿走至少一个石子。

取走最后一个石子的一方获胜。两人均使用最优策略，请判断 Emilico 在这场游戏中是否一定可以获胜？

数据范围：$1\le n\le 10^5$。

### 题解

设：

- $A$ 为只有一个石子的石子堆数；
- $B$ 为石子个数严格大于 $1$ 的石子堆数；
- $H$ 为现在指定的石子堆的石子个数。

如果 $B=0$，那么当且仅当 $A$ 为奇数时我们才会获胜，否则必败。这是显然的，因为我们只有当 $A$ 为奇数时才会取走最后一个石子。

下面讨论 $B>0$ 的情况，我们可以将指定堆后的状态分为如下三种情况：

- （W 态）$A$ 是偶数；
- （L 态）$A$ 是奇数且 $H=1$；
- （O 态）$A$ 是奇数且 $H>1$。

下面我们将说明，只有 L 态是必败态，而 W 态和 O 态都是必胜态。

当 $A$ 是偶数时，如果 $H=1$，我们总可以移除这个石子，然后指定另一个仅有一个石子的石子堆让对方取，因为 $A$ 是偶数，并且 $H=1$，所以 $A>0$，我们总可以找到这样的石子堆，这样就引导对方进入了 L 态。

如果 $H>1$ 且 $B=1$，那么我们可以直接取走指定的石子堆中所有石子，这样对方就进入了 $B=0$ 且 $A$ 为偶数的必败态。

如果 $H>1$ 且 $B>1$，我们可以取走指定堆中 $H-1$ 个石子，也就是剩一个石子，并且指定这个石子让对方取走，这样对方也进入了 L 态。

当 $A$ 是奇数且 $H=1$ 时，对方总会进入 W 态。

当 $A$ 是奇数且 $H>1$ 时，如果 $B=1$，我们可以取走指定堆中 $H-1$ 个石子，这样 $A$ 就变为了偶数，$B=0$，对方就进入了必败态。如果 $B>1$，我们可以取走指定堆的所有石子，并指定一堆只有一个石子的石子堆，因为 $A$ 是奇数，所以这样的石子堆一定存在。因此对方一定会进入一个 L 态或必败态。

每个 W 态或 O 态都会引导对方进入 L 态或者 $B=0$ 的一些必败态，而每个 L 态总会让对方进入 W 态。在每次转移时，石子总数均减小。因此，所有的 W 态或 O 态都是必胜态，所有 L 态都是必败态。

因此，先根据 $B$ 是否等于 $0$ 讨论后，再根据 $A$ 的奇偶性讨论均可。由于 Kate 总采用最优策略，因此最开始时，如果可以让 Emilico 为 L 态，Kate 总会让她为 L 态。

改编自 [CEOI 2021 Day2 T1](https://www.luogu.com.cn/problem/P8174)。原题是一个交互，要求模拟这个过程。

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2800.cpp)

## Kanade Hates Modular Multiplicative Inverse

### 题目描述

给定 $x,p$，求

$$
a\equiv bx \pmod p
$$

的一组最小解 $(a,b)$，满足 $b$ 最小的同时 $0< a < b< p$。

数据范围：$3\le p\le 10^{15},1< x < p$，保证 $p$ 是一个质数。

### 题解

不妨考虑

$$
bx-py=a
$$

这一方程，其中 $y$ 为一个自然数。

由于 $0< a< b$，则有 $0< bx-py< b$。化简一下

$$
\frac{p}{x}<\frac{b}{y}<\frac{p}{x-1}
$$

注意到 $a=bx-py$，我们要让 $a$ 和 $b$ 都最小，那么 $y$ 也要最小。下面这个问题就变成了求一个分子和分母最小的分数，使得这个分数在两有理数之间，这个问题就可以用 Stern-Brocot 树解决了。

还有一个[更简单一些的做法](https://codeforces.com/blog/entry/67091?#comment-512389)，复杂度和 Stern-Brocot 树相同。

来自：[HDU 6624. fraction](https://acm.hdu.edu.cn/showproblem.php?pid=6624)

时间复杂度：$\mathcal{O}(\log p)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2664.cpp)

## Rust

### 题目描述

有一 $m$ 阶多项式

$$
f(x)=\sum_{i=0}^m b_ix^i
$$

对于给定正整数 $a,n$，求

$$
S(n)=\sum_{k=0}^n a^kf(k)
$$

的值，对 $10^9+7$ 取模。

数据范围：$1\le n,a\le 10^9,1\le m\le 5\times 10^3$。

### 题解

先对式子变一下形

$$
\begin{aligned}
S(n)&=\sum_{k=0}^n a^kf(k)\\
&=\sum_{k=0}^n a^k\sum_{i=0}^mb_ik^i\\
&=\sum_{i=0}^mb_i\sum_{k=0}^na^k k^i
\end{aligned}
$$

后半部分 $\sum_{k=0}^n a^k k^i$ 的求值实际上是一个[板子](https://judge.yosupo.jp/problem/sum_of_exponential_times_polynomial)，可以在 $\mathcal{O}(d)$ 的时间复杂度下求值，其中 $d$ 为 $k$ 的指数部分。

这个板子实际上不想讲，主要是从多项式角度推的，但是我不会。详细推导过程可以看[这篇日语博客](https://nyaannyaan.github.io/library/fps/sum-of-exponential-times-poly.hpp)。Min25 的博客关了，悲。

我们下面介绍一种会二项式定理就能推的方法。令 $f(x)=\sum_{k=0}^n a^k k^x$。

现在我们不想要这个 $n$（因为太大了），希望把计算的复杂度下降到 $\mathcal{O}(\operatorname{poly} x)$ 的（因为不大），那么我们可以考虑去隐藏 $n$ 这个细节。所以我们把这个求和看成一个整体（也就是一个多项式），通过某种方法把它解出来。那么我们需要在右边把 $f(x)$ 构造出来，我们考虑稍微变个形。

$$
\begin{aligned}
f(x)=&\sum_{k=0}^n a^k k^x\\
&=a^0 0^x+\sum_{k=0}^n a^{k+1}(k+1)^x-a^{n+1}(n+1)^x\\
&=0^x+\sum_{k=0}^n a^{k+1}(k+1)^x-a^{n+1}(n+1)^x
\end{aligned}
$$

上一步化简中，由于 $0^0=1$，所以 $a^0=1$，但 $0^k=0\ (k\ge 1)$，所以需要保留上述形式。

我们扰动了一下 ，是因为我们希望用二项式定理提出来 $x$，并且隐藏 $n$，构造 $f(x)$。

接着化简

$$
\begin{aligned}
f(x)&=0^x+\sum_{k=0}^n a^{k+1}(k+1)^x-a^{n+1}(n+1)^x\\
&=0^x+\sum_{k=0}^n a^{k+1}\sum_{i=0}^x\binom{x}{i}k^i-a^{n+1}(n+1)^x\\
&=0^x+\sum_{i=0}^x\binom{x}{i}\sum_{k=0}^na^{k+1}k^i-a^{n+1}(n+1)^x
\end{aligned}
$$

我们看到后面的部分大概就像一个 $f(x)$ 了，但是多了一个 $a$，那么我们提出来

$$
\begin{aligned}
f(x)&=0^x+\sum_{i=0}^x\binom{x}{i}\sum_{k=0}^na^{k+1}k^i-a^{n+1}(n+1)^x\\
&=0^x+a\sum_{i=0}^x\binom{x}{i}\sum_{k=0}^na^{k}k^i-a^{n+1}(n+1)^x\\
&=0^x+a\left[\sum_{i=0}^x\binom{x}{i}f(i)-a^{n}(n+1)^x\right]
\end{aligned}
$$

容易发现，对于 $i$ 的求和最后一项就是 $f(x)$ 了，但是这里我们形式化一点，因为将 $f(x)$ 从和式中抽出来，会使求和范围变为 $0$ 到 $x-1$，而当 $x=0$ 时，这个范围是不规范的，所以我们单独讨论一下 $x=0$。

实际上，当 $x=0$ 时，$f(0)=\sum_{k=0}^n a^kk^0=\sum_{k=0}^n a^k$，实际上是一个等比数列求和的形式。只需特殊考虑一下公比 $a=1$ 的情况，其余情况均可使用公式求出答案。

下面我们只考虑 $x>0$ 时，那么 $f(x)$ 可以进一步化简

$$
\begin{aligned}
f(x)&=0^x+a\left[\sum_{i=0}^x\binom{x}{i}f(i)-a^{n}(n+1)^x\right]\\
&=af(x)+a\left[\sum_{i=0}^{x-1}\binom{x}{i}f(i)-a^{n}(n+1)^x\right]\\
(1-a)f(x)&=a\left[\sum_{i=0}^{x-1}\binom{x}{i}f(i)-a^{n}(n+1)^x\right]
\end{aligned}
$$

这里我们需要把 $1-a$ 除到等号右边，就能获得 $f(x)$ 的一个递推式了，但是 $a=1$ 时等号左边就变成 $0$ 了，我们仍需讨论 $x>0,a=1$ 的情况。

实际上，$x>0,a=1$ 时，$f(x)=\sum_{k=0}^n 1^k k^x=\sum_{k=0}^n k^x$，这是一个求自然数幂和问题，可以参考[这篇知乎回答](https://www.zhihu.com/question/369313777/answer/999977759)。

那么我们考虑 $a>1$ 时，直接除到右边

$$
\begin{aligned}
f(x)&=\frac{a}{1-a}\left[\sum_{i=0}^{x-1}\binom{x}{i}f(i)-a^{n}(n+1)^x\right]\\
&=\frac{a}{a-1}\left[a^{n}(n+1)^x-\sum_{i=0}^{x-1}\binom{x}{i}f(i)\right]
\end{aligned}
$$

这样，我们就可以在 $\mathcal{O}(m^2)$ 的时间复杂度下递推出 $0$ 到 $m$ 范围内所有的 $f(x)$ 了，之后乘系数求和即可。请注意整个过程中只需要使用一次快速幂求出 $a^n$ 的值，并不需要用快速幂求 $(n+1)^x$ 的值，因为递推的时候就可以 $\mathcal{O}(1)$ 计算了。

求组合数也不需要杨辉三角，只需要 $\mathcal{O}(m)$ 求一下阶乘和逆元的阶乘即可。如果模合数的话就需要使用杨辉三角了。

事实上，这个时间复杂度与使用上文提到的多项式板子的时间复杂度相同。

加强自 [2017 年清华大学研究生招生计算机类上机考试 - 多项式求和](https://www.acwing.com/problem/content/1337/)。

时间复杂度：$\mathcal{O}(m^2)$，空间复杂度：$\mathcal{O}(m)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2746.cpp)

## 天结

### 题目描述

对于两个计数器，每次先从 $[0,Q]$ 等概率随机选择一个整数，如果这个数不是 $0$，则先给第一个计数器加一，记目前第一个计数器的值为 $r$，然后给第二个计数器加 $r^k$。问这个过程进行 $n$ 次后，第二个计数器的值的期望，乘 $(Q+1)^n$ 后模 $998244353$ 输出。

数据范围：$1\le n,Q\le 9\times 10^8,0\le k\le 10^5$。

### 题解

令 $p=\frac{Q}{1+Q}$，则

$$
E(Y)=\sum_{r=1}^n\binom{n}{r}p^r(1-p)^{n-r}\sum_{i=1}^r i^k
$$

对概率化简

$$
\begin{aligned}
E(Y)&=\sum_{r=1}^n\binom{n}{r}p^r(1-p)^{n-r}\sum_{i=1}^r i^k\\
&=(1-p)^n\sum_{r=1}^n\binom{n}{r}\left (\frac{p}{1-p}\right )^r\sum_{i=1}^r i^k\\
\end{aligned}
$$

而 $\frac{p}{1-p}=Q$，则

$$
\begin{aligned}
E(Y)&=(1-p)^n\sum_{r=1}^n\binom{n}{r}Q^r\sum_{i=1}^r i^k\\
&=\left(1-\frac{Q}{1+Q}\right)^n\sum_{r=0}^n\binom{n}{r}Q^r\sum_{i=1}^r i^k\\
&=\frac{1}{(1+Q)^n}\sum_{r=1}^n\binom{n}{r}Q^r\sum_{i=1}^r i^k
\end{aligned}
$$

实际上，所求答案为

$$
\sum_{r=1}^n\binom{n}{r}Q^r\sum_{i=1}^r i^k
$$

注意到有

$$
i^k=\sum_{j=0}^k {k\brace j}\binom{i}{j}j!
$$

代入得

$$
\begin{aligned}
\sum_{r=1}^n\binom{n}{r}Q^r\sum_{i=1}^r i^k&=\sum_{r=1}^n\binom{n}{r}Q^r\sum_{i=1}^r \sum_{j=0}^k {k\brace j}\binom{i}{j}j!\\
&=\sum_{j=0}^k{k\brace j}j!\sum_{r=1}^n\binom{n}{r}Q^r\sum_{i=1}^r  \binom{i}{j}\\
&=\sum_{j=0}^k{k\brace j}j!\sum_{r=1}^n\binom{n}{r}Q^r\left(\binom{r+1}{j+1}-\binom{0}{j}\right)\\
&=\sum_{j=0}^k{k\brace j}j!\sum_{r=1}^n\binom{n}{r}Q^r\binom{r+1}{j+1}-\sum_{j=0}^k{k\brace j}j!\sum_{r=1}^n\binom{n}{r}Q^r\binom{0}{j}\\
&=\sum_{j=0}^k{k\brace j}j!\sum_{r=1}^n\binom{n}{r}Q^r\binom{r+1}{j+1}-[k=0]\left[(Q+1)^n-1\right]\\
\end{aligned}
$$

只考虑前半部分式子，注意到 $\binom{r+1}{j+1}=\frac{r+1}{j+1}\binom{r}{j}$，代入之

$$
\begin{aligned}
\sum_{j=0}^k{k\brace j}j!\sum_{r=1}^n\binom{n}{r}Q^r\binom{r+1}{j+1}&=\sum_{j=0}^k{k\brace j}j!\sum_{r=1}^n\binom{n}{r}Q^r\frac{r+1}{j+1}\binom{r}{j}\\
&=\sum_{j=0}^k{k\brace j}j!\sum_{r=1}^n\binom{n}{j}\binom{n-j}{n-r}Q^r\frac{r+1}{j+1}\\
&=\sum_{j=0}^k{k\brace j}\binom{n}{j}\frac{j!}{j+1}\sum_{r=1}^n\binom{n-j}{r-j}Q^r(r+1)\\
&=\sum_{j=0}^k{k\brace j}\binom{n}{j}\frac{j!}{j+1}\sum_{r=\max(1-j,0)}^{n-j}\binom{n-j}{r}Q^{r+j}(r+j+1)\\
&=\sum_{j=0}^k{k\brace j}\binom{n}{j}j!\frac{Q^j}{j+1}\sum_{r=\max(1-j,0)}^{n-j}\binom{n-j}{r}Q^{r}(r+j+1)\\
\end{aligned}
$$

分两部分考虑后面的和式，当 $j=0$ 时要减去 $r=0$ 时的值，也就是 $j+1=1$。

当 $j\neq 0$ 时

$$
\begin{aligned}
\sum_{r=0}^{n-j} \binom{n-j}{r}(r+j+1)Q^r&=(j+1)\sum_{r=0}^{n-j} \binom{n-j}{r}Q^r+\sum_{r=0}^{n-j} \binom{n-j}{r}rQ^r
\end{aligned}
$$

前面部分可以直接二项式定理逆用，后边考虑 $r\cdot \binom{n}{r}=n\cdot \binom{n-1}{r-1}$ 后再用二项式定理

$$
\begin{aligned}
\sum_{r=0}^{n-j} \binom{n-j}{r}(r+j+1)Q^r&=(j+1)\sum_{r=0}^{n-j} \binom{n-j}{r}Q^r+\sum_{r=1}^{n-j} \binom{n-j-1}{r-1}(n-j)Q^r\\
&=(j+1)(Q+1)^{n-j}+(n-j)\sum_{r=0}^{n-j} \binom{n-j-1}{r}Q^{r+1}\\
&=(j+1)(Q+1)^{n-j}+Q(n-j)\sum_{r=0}^{n-j} \binom{n-j-1}{r}Q^r\\
&=(j+1)(Q+1)^{n-j}+Q(n-j)(Q+1)^{n-j-1}
\end{aligned}
$$

那么合起来

$$
\begin{aligned}
\sum_{r=0}^n\binom{n}{r}Q^r\sum_{i=0}^r i^k&=\sum_{j=0}^k{k\brace j}\binom{n}{j}j!\frac{Q^j}{j+1}\sum_{r=0}^{n-j} \binom{n-j}{r}(r+j+1)Q^r\\
&=\sum_{j=0}^k{k\brace j}\binom{n}{j}j!Q^j\left(\frac{(j+1)(Q+1)^{n-j}+Q(n-j)(Q+1)^{n-j-1}}{j+1}-[j=0]\right)\\
&-[k=0]\left[(Q+1)^n-1\right]
\end{aligned}
$$

注意到我们需要求的第二类 Stirling 数 $k\le 10^5$，于是需要使用 NTT 加速计算。

时间复杂度：$\mathcal{O}(k\log k)$，空间复杂度：$\mathcal{O}(k)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2888.cpp)

## 新月之舞

### 题目描述

求

$$
\sum_{i=1}^n \sum_{j=1}^n i\bmod j
$$

数据范围：$1\le n\le 10^{12}$。

### 题解

我们分两段考虑

$$
\begin{align}
a_n&=\sum_{i=1}^n\sum_{j=1}^i i\bmod j+\sum_{i=1}^n\sum_{j=i+1}^n i\bmod j\\
&=\sum_{i=1}^n\sum_{j=1}^i i-\lfloor \frac{i}{j}\rfloor j +\sum_{i=1}^n i(n-i)\\
&=\sum_{i=1}^ni^2 -\sum_{i=1}^n\sum_{j=1}^i\lfloor \frac{i}{j}\rfloor j+\frac{(n+1)n(n-1)}{6}\\
&=\frac{n^2(n+1)}{2}-\sum_{i=1}^n\sum_{j=1}^i\lfloor \frac{i}{j}\rfloor j\\
&=\frac{n^2(n+1)}{2}-\sum_{i=1}^n\sum_{j=1}^i \sigma(j)\\
&=\frac{n^2(n+1)}{2}-\sum_{i=1}^n\sigma(i)(n-i+1)\\
\end{align}
$$

单独考虑后面的和式

$$
\begin{align}
b_n&=\sum_{i=1}^n\sigma(i)(n-i+1)\\
&=(n+1)\sum_{i=1}^n\sigma(i)-\sum_{i=1}^n i\sigma(i)
\end{align}
$$

对于 $\sum_{i=1}^n \sigma(i)$ 的部分，接着展开。

$$
\sum_{i=1}^n \sigma(i)=\sum_{i=1}^n\sum_{d\mid i} d
$$

反向考虑 $d$ 对答案的贡献，对于 $1\sim n$ 中的正整数，有 $\lfloor \frac{n}{d}\rfloor$ 个整数有 $d$ 这个因子，每个整数对于答案的贡献为 $d$，所以有

$$
\sum_{i=1}^n\sum_{d\mid i} d=\sum_{d=1}^n d\lfloor \frac{n}{d}\rfloor
$$

接着考虑 $\sum_{i=1}^n i\sigma(i)$ 的部分。

$$
\begin{align}
\sum_{i=1}^n i\sigma(i)&=\sum_{i=1}^n i\sum_{d\mid i} d\\
&=\sum_{i=1}^n \sum_{d\mid i} id\\
\end{align}
$$

接着考虑枚举 $d$，类似地，对于 $1\sim n$ 中的正整数，有 $\lfloor \frac{n}{d}\rfloor$ 个整数有 $d$ 这个因子，每个整数对于答案的贡献为 $id$，我们考虑使用 $pd$ 这样的方式来枚举这 $\lfloor \frac{n}{d}\rfloor$ 个整数，其中 $p\in [1,\lfloor \frac{n}{d}\rfloor]$，则有

$$
\begin{align}
\sum_{i=1}^n\sum_{d\mid i}id&=\sum_{d=1}^n \sum_{p=1}^{\lfloor\frac{n}{d}\rfloor} pd\times d\\
&=\sum_{d=1}^n d^2\sum_{p=1}^{\lfloor\frac{n}{d}\rfloor} p\\
&=\sum_{d=1}^n d^2 \frac{S(S+1)}{2}
\end{align}
$$

其中 $S=\lfloor\frac{n}{d}\rfloor$，注意上式中的变形。

让我们把 $b_n$ 捏起来。

$$
b_n=\sum_{d=1}^n (n+1)dS-\frac{1}{2}d^2S-\frac{1}{2}d^2S^2
$$

使用数论分块求解即可。

求解时需注意除法能否整除，可以把后两项合并起来求和，就可以避免除 2 时的整数与否的判断了。因为答案级别是 $\mathcal{O}(n^3)$ 的，因此需要使用 `__int128`。

来自 UOJ 群友某天问的一道题。

时间复杂度：$\mathcal{O}(\sqrt n)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2739.cpp)

## Жизнь по кругу

### 题目描述

现有一个半径为 $r$ 的圆，问圆上整点数个数。

数据范围：$1\le r\le 10^{18}$。

### 题解

结论是，设 $n=\prod_{i=1}^m p_i^{k_i}$，其中 $n$ 为圆的半径，$p_i$ 为 $n$ 的唯一分解。则答案为

$$
\prod_{i=1}^m [p_i\bmod 4=1](2k_i+1)
$$

证明见 3B1B 的[视频](https://www.bilibili.com/video/BV1kx411q7kK)。

于是用 Pollard rho 分解质因数即可。

加强自 HAOI 2008. 圆上的整点

时间复杂度：$\mathcal{O}(r^{1/4})$，空间复杂度：$\mathcal{O}(\omega(r))$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2668.cpp)

## 利兹与青鸟

### 题目描述

求

$$
G(n)=\sum_{k=1}^n \gcd(k,n)\cos\frac{2k\pi}{n}
$$

和

$$
L(n)=\sum_{k=1}^n \text{lcm}(k,n)\cos\frac{2k\pi}{n}
$$

的值。

数据范围：$1\le n\le 10^9$。

### 题解

这道题涉及一些复分析方面的知识，但不超过学了 FFT 的复分析知识范围。

对于求 $G(n)$ 的过程，已有[研究 [1]](https://math.stackexchange.com/questions/1644366/find-sum-i-12000-gcdi-2000-cos-left-frac2-pi-i2000-right)，可知 $G(n)=\varphi(n)$。

下面考虑求 $L(n)$ 的过程。

$$
\begin{align}
\sum_{k=1}^n \text{lcm}(n,k)\cos\frac{2k\pi}{n}&=\sum_{k=1}^n \frac{nk}{\gcd(n,k)}\cos\frac{2k\pi}{n}\\
&=n\sum_{k=1}^n \frac{k}{\gcd(n,k)}\cos\frac{2k\pi}{n}
\end{align}
$$

则

$$
\begin{align}
\sum_{k=1}^n \frac{k}{\gcd(n,k)}\cos\frac{2k\pi}{n}&=\sum_{d=1}^n\sum_{k=1}^n[\gcd(n,k)=d]\frac{k}{d}\cos\frac{2k\pi}{n}\\
&=\sum_{d\mid n}\sum_{k=1}^n[\gcd(n,k)=d]\frac{k}{d}\cos\frac{2k\pi}{n}
\end{align}
$$

令 $k=k'\cdot d,k'\in [1,\frac{n}{d}]$，则

$$
\sum_{d\mid n}\sum_{k=1}^n[\gcd(n,k)=d]\frac{k}{d}\cos\frac{2k\pi}{n}=\sum_{d\mid n}\sum_{k'=1}^{n/d}[\gcd(\frac{n}{d},k')=1]k'\cos\frac{2k'\pi}{\frac{n}{d}}
$$

我们来考虑

$$
\sum_{k'=1}^{n/d}[\gcd(\frac{n}{d},k')=1]k'\cos\frac{2k'\pi}{\frac{n}{d}}
$$

不妨设现在要求的是

$$
S(n)=\sum_{k=1}^{n}[\gcd(n,k)=1]\cos\frac{2k\pi}{n}\cdot k
$$

最终答案就是

$$
L(n)=n\sum_{d\mid n}S(\frac{n}{d})
$$

了。

我们猜测

$$
S(n)=
\begin{cases}
1 & n=1\\
\frac{n\mu(n)}{2} & n\ge 2
\end{cases}
$$

也可以简写为

$$
S(n)=\frac{n\mu(n)+[n=1]}{2}
$$

在 $n\le 10000$ 的情况都符合（实在不行打个表系列）。下面考虑证明之。

考虑 Möbius 反演的一个补充结论

$$
\gcd(i,j)=\sum_{d\mid \gcd(i,j)} \mu(d)
$$

这里考虑 $\mu * 1=\varepsilon$ 可知，详细过程可参考 [莫比乌斯反演 - OI Wiki](https://oi-wiki.org/math/number-theory/mobius/#_6)。代入之

$$
\begin{align}
S(n)&=\sum_{k=1}^n [\gcd(n,k)=1]\cos\frac{2k\pi}{n}\cdot k\\
&=\sum_{k=1}^n \cos\frac{2k\pi}{n}\cdot k\sum_{d\mid \gcd(n,k)} \mu(d)\\
&=\sum_{k=1}^n\sum_{d\mid n\land d\mid k}\mu(d)\cos\frac{2k\pi}{n}\cdot k\\
\end{align}
$$

考虑如何变换交换和号，由于 $d\mid k$，考虑 $k=k'\cdot d$，则可以把上式改写为

$$
\sum_{d\mid n}\sum_{k'=1}^{n/d} \mu(d)\cos\frac{2k'd\pi}{n}\cdot k'd
$$

则可提取与化简为

$$
\sum_{d\mid n}d\mu(d)\sum_{k'=1}^{n/d}\cos \frac{2k'\pi}{\frac{n}{d}}\cdot k'
$$

不妨设

$$
C(n)=\sum_{k=1}^n k\cdot \cos\frac{2k\pi}{n}
$$

则所求为

$$
S(n)=\sum_{d\mid n}d\mu(d)C(\frac{n}{d})
$$

实际上，对于 $C(n)$ 的求和是一个经典复分析问题，结论为

$$
C(n)=
\begin{cases}
1 & n=1\\
\frac{n}{2} & n>1
\end{cases}
$$

我们对其的证明放在最后。则根据其结论可化简 $S(n)$。

$$
\begin{align}
S(n)&=\sum_{d\mid n}d\mu(d)C(\frac{n}{d})\\
&=n\mu(n)+\sum_{d\mid n\land d< n}d\mu(d)\frac{n}{2d}\\
&=n\mu(n)+\frac{n}{2}\sum_{d\mid n\land d< n}\mu(d)\\
&=n\mu(n)+\frac{n}{2}\left(-\mu(n)+\sum_{d\mid n}\mu(d)\right)\\
&=\frac{n\mu(n)+[n=1]}{2}
\end{align}
$$

我们就这样证明了上述猜测的正确性。

那么答案就是

$$
L(n)=n\frac{1+\sum_{d\mid n}\frac{n}{d}\mu(\frac{n}{d})}{2}
$$

其中 $\sum_{d\mid n}\frac{n}{d}\mu(\frac{n}{d})$ 是 [A023900](https://oeis.org/A023900)，等于

$$
\prod_{p} (1-p)
$$

其中 $p$ 表示 $n$ 的质因数分解中的质数。

$S(n)$ 和 $C(n)$ 的求和是 rockdu 证的。

可以在 $\mathcal{O}(\pi(\sqrt n))$ 的复杂度进行质因子分解，众所周知 $\pi(n)\sim \frac{n}{\ln n}$，之后对于这些质因子分别计算，代价为 $\mathcal{O}(\omega(n))$。但当 $n=10^9$ 时，$\max\{\omega(n)\}=9$，因此统计答案时间代价可忽略不计。使用 $\mathcal{O}(\sqrt n)$ 的分解方法是无法通过的，第 4, 5 两组数据都是大质数。

因为整个不除以 $2$ 的部分可能会占用 $64$ 位（因为带符号），因此直接用 `double` 的话将引入精度误差（即使除以 $2$ 是无误差的，但将 `long long` 转成 `double` 的时候精度误差就存在了），可以使用 `long double` 解决，或者写无精度误差的方法（但是要注意 C++ 对于负数除法并非向 $0$ 舍入，需要转为绝对值）。

The Art of Shaving Logs.

来自 OI Wiki 群友问的问题（大概 4 月份？），当时没人回答。

时间复杂度：$\mathcal{O}(\frac{\sqrt n}{\ln n})$，空间复杂度：$\mathcal{O}(\sqrt{n})$。

实际上本题仍可以将 $n$ 的范围出到 $10^{18}$，并使用 Pollard rho 算法解决。但考虑到编码的困难和可以预见的数据类型使用问题，故作罢（$L(n)$ 大概是 $\mathcal{O}(n^2)$ 级的）。

附录一：证明 $C(n)$，也就是证明

$$
\sum_{k=1}^nk\cdot \cos\frac{2k\pi}{n}=\frac{n+[n=1]}{2}
$$

下面记 $\omega_n$ 为 $n$ 次单位根，我们实际上求的就是

$$
\sum_{k=1}^nk\cdot \omega_{n}^k
$$

的实部。

我们考虑上式为等差乘等比，高考题之

$$
\begin{align}
C(n)&=1\cdot\omega_{n}^1+2\cdot\omega_{n}^2+\ldots+n\cdot \omega_{n}^n\\
\omega_{n}C(n)&=1\cdot \omega_{n}^2+2\cdot \omega_n^3+\ldots+(n-1)\cdot \omega_{n}^n+n\cdot \omega_{n}^{n+1}
\end{align}
$$

上下两式作差

$$
(1-\omega_n)C(n)=\omega_n^1+\omega_n^2+\ldots+\omega_n^n-n\cdot \omega_n^{n+1}
$$

实际上前面的求和部分是一个 $1$ 到 $n$ 次单位根求和，当 $n>1$ 时，有

$$
\sum_{k=1}^n \omega_n^k=0
$$

上式从复分析角度可知。

那么

$$
(1-\omega_n)C(n)=-n\cdot \omega_n^{n+1}=-n\cdot \omega_n
$$

请注意，$\omega_1=1$，当 $n=1$ 时 $C(n)$ 前的系数为 $0$，不能简单相除。可以直接计算 $n=1$ 时的结果为 $C(1)=1$。

当 $n>1$ 时，$C(n)=-n\frac{\omega_n}{1-\omega_n}$。再次根据复分析知识，$\text{Re}(\frac{\omega_n}{1-\omega_n})=-\frac{1}{2}$，则 $C(n)=\frac{n}{2}$。请注意 $C(n)$ 是一个实数，因此对于单位根的计算我们取实部。

由此，$C(n)$ 已被证明。

下面简单介绍需要的复分析知识（十分不严谨，仅供参考），如果已经学习过 FFT 或相关知识可以跳过。

根据 Taylor 展开可知

$$
e^{ix}=\cos x+i\sin x
$$

这就是 Euler 公式。根据此公式有

$$
e^{2k\pi i}=1
$$

我们实际上就解出了所谓的「$n$ 次单位根」

$$
\omega_n=e^{\frac{2\pi i}{n}}
$$

我们首先来证明 $n>1$ 时

$$
\text{Re}(\frac{\omega_n}{1-\omega_n})=-\frac{1}{2}
$$

我们考虑 Euler 公式

$$
\begin{align}
\frac{\omega_n}{1-\omega_n}&=\frac{\cos \theta+i\sin\theta}{1-\cos \theta-i\sin\theta}\\
&=\frac{(\cos\theta+i\sin\theta)(1-\cos\theta+i\sin\theta)}{(1-\cos\theta-i\sin\theta)(1-\cos\theta+i\sin\theta)}\\
&=\frac{\cos\theta-1+i\sin\theta}{(1-\cos\theta)^2+\sin^2\theta}\\
&=\frac{\cos\theta-1+i\sin\theta}{2-2\cos\theta}
\end{align}
$$

那么实部就是 $\frac{\cos\theta-1}{2-2\cos\theta}=-\frac{1}{2}$。

关于 $\omega_{n}^k$ 的前 $n$ 项和，直接使用等比数列求和公式即可。注意 $\omega_n^n=1$。

附录二：如何猜答案

先写个暴力打 $n\le 10$ 的表，关于 $G(n)$，建议把 $\varphi(n)$ 表背下来。

对于 $L(n)$，首先我们考虑观察这些值有正负，还有 $.0$ 和 $.5$ 的值，首先考虑将它们都乘 $2$。乘完 $2$ 之后发现好像这些值越来越大，可以考虑猜一个 $\bmod n=0$，然后对这些值都除 $n$。首先观察所有质数位置，都等于 $2-p$，然后考虑观察 $n=pq$，其中 $p,q$ 都是质数的位置，我们考虑分解一下，发现没有什么卵用，但是考虑 $-1$ 之后都能拆成 $(1-p)(1-q)$，我们就猜它分解之后 $(1-p)$ 乘起来，加 $1$ 乘 $n$ 除 $2$。然后就猜出来了。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2799.cpp)

## Violet mag Veilchen

### 题目描述

有一个半径为 $r$ 的圆，求圆内（包括圆上）整点数。

数据范围：$1\le r\le 10^9$。

### 题解

[In memory of Min25.](https://web.archive.org/web/20211009144532/https://min-25.hatenablog.com/entry/2018/05/03/145505)

[这里](https://mathoverflow.net/questions/85368/convex-hull-of-lattice-points-in-a-circle)也有一些粗略的介绍。

一个很有意思的结论是，圆内（包括圆上）所有整数点形成的凸包规模并不大。究其原因，是因为函数较为平滑，考虑圆在第一象限内的图象，其二阶导的绝对值小于 $1$ 的范围很大（具体有多大呢，~~留作课后作业~~后面再说）。实际上，圆内（包括圆上）所有整数点形成的凸包规模为 $\mathcal{O}(n^{2/3})$。其中 $n$ 为圆的半径。

首先圆有很好的对称性，在这里我们仅考虑圆域位于第一象限的部分。最终答案就是第一象限的部分的答案乘 $4$。再根据第一象限的圆域是以 $y=x$ 为轴对称的，所以考虑答案实际上是一个八分之一圆内的答案乘 $8$（当然还要处理一些边界问题）。

我们设 $(x_0,y_0)$ 是凸包的一个顶点，并且只考虑圆域位于第一象限且在 $y=x$ 与 $x$ 轴之间的部分，下面我们要按顺时针方向找凸包的下一个顶点 $(x_1,y_1)$。

根据单调性，有 $x_0< x_1$ 且 $y_0>y_1$。并且 $x_1$ 应该是大于 $x_0$ 且满足条件的最小值，对于 $y_0$ 也是同理的。这样才能满足是下一个顶点。这样考虑很难下手，因为并没有用到凸包性质，所以我们转而考虑斜率。

因为都是凸包上都是整数点，所以对于凸包的任何边，其斜率都是有理数。不妨设

$$
\frac{a}{b}<\frac{y_0-y_1}{x_0-x_1}\le \frac{c}{d}
$$

我们的目的是求这样的一组 $(x_1,y_1)$ 满足上述条件。可以使用 Stern-Brocot 树，并可以发现这个过程和在树上二分挺像的。因为我们每次计算的中间值是 $\frac{a+c}{b+d}$，由于 Stern-Brocot 树的性质，会保证这个分数是最简的，所以有 $bc-ad=1$ 这样的性质。

为了统计方便，因为我们要算的圆域是闭区域，我们需要把圆周也包括进来，所以考虑的凸包是能够完整覆盖这个圆的最小整点凸包。不妨设 $p=a+c,q=b+d$，引入圆和凸包性质分情况讨论。

如果 $(x_0+q)^2+(y_0-p)^2\le n^2$，则令 $c\leftarrow p,d\leftarrow q$ 继续讨论。这里是递归到的值不在圆外的情况，表示斜率的相反数仍可以减小（理解上来说，就是现在凸包上的边太竖直了，减小后让他不那么竖直，保证下个点仍在圆外）。

如果 $(x_0+q)^2+(y_0-p)^2> n^2$，则需要讨论：

- $-f' (x_0+q)\ge \frac{c}{d}$，则下一个点就可以确定为 $(x_1,y_1)\leftarrow (x_0+q,y_0-p)$，因为这样的话下一个点处圆的切线斜率更大，可以保证下一个点的下个点一定可以在圆外，从树的性质来说又可以保证满足要求，因此可以确定；
- 否则，就表示凸包边的斜率大于切线处斜率。根据圆的一阶导和凸包性质，下一个点就会在圆内，所以扩大斜率的相反数，即 $a\leftarrow p,b\leftarrow q$。

由于凸包的边的斜率是单调的，所以考虑用栈将每次递归到的分数存起来，就可以达到理想的复杂度了。

最前面有个问题，圆在第一象限内的部分二阶导绝对值小于 $1$ 的范围有多大。实际上是一道~~微积分~~ MATLAB 题，下面放一段过程，如果有问题就凑合看吧……

可知

$$
y''=-\frac{r^2}{(r^2-x^2)^{3/2}}
$$

易得 $y''<0$ 在研究定义域 $[0,r)$ 恒成立。

我们实际要解

$$
\frac{r^2}{(r^2-x^2)^{3/2}}\le 1
$$

分母恒正，移项，两边平方，去括号，移项，化为多项式，然后丢 MATLAB 解出来（考虑实根即可）

$$
x^6-3r^2x^4+3r^4x^2+r^4-r^6\le 0
$$

（实际上令 $t=x^2$ 就是三次方程了）

考虑不等号左边的单调性，可得范围为

$$
\left[0,r^{\frac{2}{3}}\sqrt{r^{\frac{2}{3}}-1}\right]
$$

易知 $r-1\le r^{\frac{2}{3}}\sqrt{r^{\frac{2}{3}}-1}$，证明留作课后作业。

时间复杂度：$\mathcal{O}(n^{2/3})$，空间复杂度：$\mathcal{O}(n^{2/3})$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2667.cpp)

## 模拟人生 2077

### 题目描述

平面上 $n$ 个点，求由这 $n$ 个点组成面积最大和面积最小的三角形面积。

数据范围：$3\le n\le 2000$。

### 题解

对于求最大三角形，直接旋转卡壳即可。

对于求最小三角形，极角序是错的。这个方法以每个点为原点做点的极角序，统计极角序相邻的两个与原点形成三角形的面积取最小值。但需要注意最小三角形并不一定出现在极角序相邻的两个点，还需要考虑极径长度。

对于求最小三角形，考虑枚举每一条线段，用离这个线段最近的两个点更新答案。那么现在只有点到线段的距离有用了，考虑对线段按斜率排序，这里也可以利用极角序。对于排完序后的线段，因为现在只有点到线段的距离是我们需要考虑的，只要线段之间的相对位置不变即可。

考虑一种排序方式为按从 $y$ 轴负半轴到正半轴逆时针排序，将点按横坐标从小到大排序。我们枚举一条线段 $(x,y)$，表示由点 $x$ 和点 $y$ 组成的线段。根据上面的排序方式，我们只需要考虑点 $x$ 的前一个点和点 $y$ 的后一个点即可。其中一个事实是 $x$ 和 $y$ 一定是顺序上相邻的。切换到下一条线段时，按照排序后的序列，只有 $x$ 和 $y$ 对于下一条线段的相对位置变化了。交换两者的顺序即可。维护两个序列，一个表示某个点的位置，一个表示某个位置的点即可维护这样相邻交换的问题。

时间复杂度：$\mathcal{O}(n^2)$，空间复杂度：$\mathcal{O}(n^2)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2551.cpp)
