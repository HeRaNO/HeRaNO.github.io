---
title: k 次方和问题浅述
date: 2022-11-05 19:51:46
categories: 'OI/ICPC'
toc: true
tags:
	- 线段树
	- 数学
	- 数论
description: ' '
---

前置技能：线段树，组合数学，矩阵。

$k$ 次方和问题，实际上是维护 $\sum_{i=l}^r a_i^k$ 的一类问题。再进行深入研究，实际上是一系列推大式子的问题。

下面分组介绍。

## 幂和问题

幂和问题即为求 $\sum_{i=1}^n i^k$ 的一类问题，其中常量为 $n,k$。目前的做法有以下几种。

### $\mathcal{O}(n\log k)$ 

显然，直接利用枚举与快速幂可以做到这个时间复杂度。

### $\mathcal{O}(n)$

考虑到 $f(x)=x^k$ 为一个积性函数，所以我们可以用线性筛的方法解决。对于质数直接利用快速幂求解，对于合数则拆出任意一个因子进行计算。由于只对质数进行快速幂，根据素数定理 $\pi(x)\sim \frac{x}{\ln x}$，我们可以估计这个算法的复杂度为 $\Theta(n)$。然后做前缀和即可，复杂度仍为 $\Theta(n)$。

实现见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Template/Linear_i%5Ek.cpp)。

### $\mathcal{O}(k^2)$

公式是：

$$
S(k)=\frac{(n+1)^{k+1}-1-\sum_{i=0}^{k-1}\binom{k+1}{i}S(i)}{k+1}
$$

推导见[这里](https://www.zhihu.com/question/369313777/answer/999977759)。

### $\mathcal{O}(k)$

由于 Lagrange 基本多项式的分子分母特殊，因此利用 Lagrange 插值可以做到 $\mathcal{O}(k)$ 的复杂度。需注意到幂和为一个 $k+1$ 次多项式。

具体可参考 [riteme 的博客](https://riteme.site/blog/2017-3-18/lagrange-interpolation.html)。

### 不典型例题

求：

$$
\sum_{x=1}^N\binom{N}{x}x^k
$$

输出答案对 $10^9+7$ 取模后的值。$1\le N\le 10^9,1\le k\le 5\times 10^3$。

原题来自：[ICM Technex 2018 and Codeforces Round #463 (Div. 1 + Div. 2, combined) E. Team Work](https://codeforces.com/problemset/problem/932/E)。

$$
\begin{aligned}
\sum_{i=1}^n\binom{n}{i}i^k&=\sum_{i=1}^n\binom{n}{i}\sum_{j=1}^k{k\brace{j}}\binom{i}{j}j!\\
&=\sum_{i=1}^n \sum_{j=1}^k \frac{n!}{(n-i)!(i-j)!}{k\brace{j}}\\
&=\sum_{j=1}^k {k\brace{j}}\sum_{i=1}^n \frac{n!}{(n-i)!(i-j)!}\\
&=\sum_{j=1}^k {k\brace{j}}\sum_{i=1}^n \frac{(n-j)!}{(n-i)!(i-j)!}\times \frac{n!}{(n-j)!}\\
&=\sum_{j=1}^k {k\brace{j}}\frac{n!}{(n-j)!}\sum_{i=1}^n \frac{(n-j)!}{(n-i)!(i-j)!}\\
&=\sum_{j=1}^k {k\brace{j}}\frac{n!}{(n-j)!}\sum_{i=1}^n \binom{n-j}{n-i}\\
&=\sum_{j=1}^k {k\brace{j}}\frac{n!}{(n-j)!}2^{n-j}
\end{aligned}
$$

有一个比较常用的转化：

$$
x^k=\sum_{i=0}^k{k\brace i}\binom{x}{i}i!
$$

注意第四步的配凑和倒数第二步中组合数的化简，由于它能覆盖到 $0\sim n-j$ 的所有数，因此可化简为 $2^{n-j}$。因此总时间复杂度 $\mathcal{O}(k^2)$，空间复杂度 $\mathcal{O}(k^2)$。

[Solution](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/932E.cpp)

## 区间 $k$ 次方和问题

区间 $k$ 次方和问题是维护数列，并查询 $\sum_{i=l}^r a_i^k$ 或有关信息的一类问题。

当 $k=0$ 时，甚至可以 $\mathcal{O}(1)$ 回答询问……

当 $k=1$ 时，只需要维护区间和即可，根据修改的要求维护线段树。

一般询问的是 $k=2$ 和 $k=3$ 的情况。

线段树问题我们一般维护询问，当询问区间和，区间修改为区间加法时，有如下推导：

$$
\begin{aligned}
\sum_{i=l}^r(a_i+x)^2&=\sum_{i=l}^r(a_i^2+2a_ix+x^2)\\
&=\sum_{i=l}^ra_i^2+2x\sum_{i=1}^{r}a_i+(r-l+1)x^2\\
\sum_{i=l}^r(a_i+x)^3&=\sum_{i=l}^r(a_i^3+3a_i^2x+3a_ix^2+x^3)\\
&=\sum_{i=l}^ra_i^3+3x\sum_{i=1}^{r}a_i^2+3x^2\sum_{i=l}^ra_i+(r-l+1)x^3\\
\end{aligned}
$$

可以观察到，要维护 $k$ 次方的话一定要维护所有 $0\sim k$ 次方和，在更新的时候也需要从高次到低次幂维护。

但是对于更高次数时，这样维护太奔溃了，对于高次情况我们考虑使用矩阵来维护。

对于线段树的每个区间，维护如下矩阵：

$$
\begin{bmatrix}
\sum a^0& \sum a^1 & \ldots &\sum a^k
\end{bmatrix}
$$

对于区间加法，即可对于每个区间右乘（$x$ 为区间增量）：

$$
\begin{bmatrix}
1 & x & x^2 & x^3 & \ldots & x^k\\
0 & 1 & 2x  & 3x^2& \ldots & kx^{k-1}\\
0 & 0 & 1   & 3x  & \ldots & \binom{k}{2}x^{k-2}\\
0 & 0 & 0   & 1  & \ldots & \binom{k}{3}x^{k-3}\\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots\\
0 & 0 & 0 & 0 & \ldots & 1
\end{bmatrix}
$$

考虑到加法可以合并，我们可以直接维护区间加标记，下传时再构建上述矩阵进行标记下传中的维护。对于乘法，和区间乘法一样类似操作即可。因此整个操作的复杂度为 $\mathcal{O}(k^2q\log n)$。

当 $k$ 较小时，直接维护效果更好，复杂度为 $\mathcal{O}(\mathcal{q\log n})$，并且常数很小。

相关题目：

- [Lutece 2143](https://acm.uestc.edu.cn/problem/fang-chai/description)，[Solution](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2143.cpp)；
- [HDU 4578](https://vjudge.net/problem/HDU-4578)，[Solution](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU4578.cpp)；
- [ICPC 2019 Shanghai Onsite F.](https://ac.nowcoder.com/acm/contest/4370/F)，[Solution](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Contest/The%202019%20ICPC%20Asia%20Shanghai%20Regional%20Contest/F.cpp)。
