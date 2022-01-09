---
title: 2019 年电子科技大学 ACM-ICPC 暑假前集训数学与几何专题解题报告
categories: 'OI/ICPC'
toc: true
date: 2019-07-03 00:00:00
tags:
	- BSGS
	- 博弈论
	- 组合数学
	- 数论
	- 凸包
	- 旋转卡壳
	- 最小圆覆盖
	- 线段树
	- 扫描线
---

5/10/12
<!-- more -->

牺牲博弈论的微积分与大学物理保护法。

## A. 韬哥哥抽卡

### 题目描述

求：

$$
\Large m^{\sum_{d\mid n}{n\choose d}}\bmod p
$$

数据范围：$1\le n,m\le 10^9,p<10^6$ 且保证 $\mu(p-1)\neq 0$，$p$ 为质数。

### 题解

因为指数可能很大，所以先考虑扩展欧拉定理，当 $p$ 是质数时 $a^b\equiv a^{b\bmod \varphi(p)}\pmod p$，然后代入 $\varphi(p)=p-1$ 即可。

然后就单独考虑指数，就是怎么求 $\sum_{d\mid n}\binom{n}{d}\bmod (p-1)$，因为 $p$ 是质数，那么 $p-1$ 一定是一个偶数，那么肯定就不是质数了。观察到 $p-1$ 比较小，所以考虑扩展 Lucas 定理直接求解，其实就是分解模数然后 CRT 合并上去。根据 $\mu(p-1)\neq 0$ 可知每个因子只有一个，还好算了挺多。

最后一遍快速幂即可。

时间复杂度 $\mathcal{O}(p+\sqrt{np}\log_p n)$，空间复杂度 $\mathcal{O}(p)$。

反正跑得贼快。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2277.cpp)

## B. 别问，问就是 AK

### 题目描述

多次询问，设 $f(i)=\sum_{x=1}^{p-1}\sum_{y=1}^m[(x+y)^i\equiv x^i\pmod p]$，求 $\sum_{i=1}^{p-1}if(i)$。

数据范围：$T\le 100,1\le m\le p-1,2\le p\le 10^9+7$，且 $p$ 为质数。

### 题解

对于这样带质数还恒等的一般考虑模数 $p$ 的原根。记 $p$ 的原根为 $g$。

则存在唯一 $a,b$ 使得 $x+y\equiv g^a\bmod p,x\equiv g^b\bmod p$。

根据计数规则，有：$g^{ai}\equiv g^{bi}\pmod{\varphi(p)}$，即 $i(b-a)\equiv 0 \pmod{\varphi(p)}$。

显然 $b-a=\frac{k\varphi(p)}{\gcd(\varphi(p),i)}$，其中 $k\in[1,\gcd(\varphi(p),i))$。

那么现在问题是有多少符合条件的 $b-a$ 使得 $i(b-a)\bmod p=0$。

设 $z=b-a$，则：

$$
\begin{aligned}
x+y&\equiv g^{a+z} \mod p\\
x+y&\equiv xg^z\mod p\\
y&\equiv x(g^z-1)\mod p\\
x&\equiv y(g^z-1)^{-1}\bmod p
\end{aligned}
$$

因为 $p$ 是质数，那么小于 $p$ 的所有数逆元一定存在，所以对于任意一个 $y$，都存在 $\gcd(\varphi(p),i)-1$ 个对应的 $x$。

于是答案就变成了 $f(i)=m(\gcd(\varphi(p),i)-1)$，代入和式并化简得到最终答案为：

$$
m(\sum_{i=1}^{p-1}i\gcd(p-1,i)-\sum_{i=1}^{p-1}i)
$$

对于后一个求和可以直接用等差数列求和公式求。

前一个求和枚举 $\gcd$，可得：

$$
\sum_{i=1}^{p-1}i\gcd(p-1,i)=\sum_{d\mid p-1}\frac{d^2\varphi(\frac{p-1}{d})\frac{p-1}{d}+[d=p-1]}{2}
$$

预先求出 $2$ 在 $10^9+7$ 下的逆元统计答案即可。

时间复杂度 $\mathcal{O}(Tp^{0.75})$，空间复杂度 $\mathcal{O}(1)$。

实际上时间复杂度应该是一个 $\sum_{d\mid n}\sqrt{\frac{n}{d}}$ 的东西，这个和杜教筛的时间复杂度类似但是比杜教筛快……但是我也不知道这个跑得为啥这么快……

原题来自：[HDU 6051. If the starlight never fade](https://vjudge.net/problem/HDU-6051)

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2279.cpp)

## C. oy function

### 题目描述

多组询问，求：

$$
\sum_{i=1}^n\sum_{j=1}^n \varphi(\gcd(i,j))
$$

数据范围：$1\le T\le 5\times 10^3,1\le n\le 10^7$。

### 题解

看起来像 Möbius 反演，可以先看一下 H 题的题解。但是实际上不用……

还是考虑枚举 $\gcd$：

$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^n\varphi(\gcd(i,j))&=\sum_{d=1}^n\varphi(d)\sum_{i=1}^n\sum_{j=1}^n[\gcd(i,j)=d]\\
&=\sum_{d=1}^n\varphi(d)\sum_{i=1}^{\lfloor \frac{n}{d}\rfloor}\sum_{j=1}^{\lfloor \frac{n}{d}\rfloor}[\gcd(i,j)=1]
\end{aligned}
$$

记 $S(n)=\sum_{i=1}^n \varphi(i)$，那么：

$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^n\varphi(\gcd(i,j))&=\sum_{d=1}^n\varphi(d)(2S(\lfloor \frac{n}{d}\rfloor) -1)\\
&=2\sum_{d=1}^n\varphi(d)S(\lfloor \frac{n}{d}\rfloor)-S(n)
\end{aligned}
$$

对于前一个求和，利用数论分块即可解决。

时间复杂度 $\mathcal{O}(n+T\sqrt n)$，空间复杂度 $\mathcal{O}(n)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2280.cpp)

## D. 如果早知道 wf 题也会被 ak - 年度版

### 题目描述

已知三阶递推式 $f_i=(f_{i-1}+f_{i-2}+f_{i-3})\bmod p\ (i\ge 4)$，$f_1=1,f_2=1,f_3=2$。给出一段连续的 $f_{i-1},f_i,f_{i+1}$，问最小满足条件的 $i$。保证有解。

数据范围：$p=10^9+7$。

### 题解

因为这样的三阶递推式可以构造矩阵：

$$
A=
\begin{bmatrix}
1&1&0\\
1&0&1\\
1&0&0
\end{bmatrix}
$$

所以我们求的就是最小的 $x$ 使得 $A^x=B$，其中：

$$
B=
\begin{bmatrix}
f_{i+1}&f_{i}&f_{i-1}\\
f_{i}+f_{i+1}&f_{i+1}-f_i&f_{i}-f_{i-1}\\
f_i&f_{i-1}&f_{i-2}
\end{bmatrix}
$$

然后利用 BSGS 求解即可。

令 $x=im-j$，则 $A^{im}=BA^j\pmod p$，这样就不用求逆矩阵了。最后返回的值改一下即可。

时间复杂度 $\mathcal{O}(\sqrt p\log p)$，空间复杂度 $\mathcal{O}(\sqrt p)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2242.cpp)

## E. xzlnb 教你避铳

### 题目描述

~~期末了，牺牲博弈论的微积分与大学物理保护法~~

### 题解

## F. xzlnb 教你胡牌

### 题目描述

~~期末了，牺牲博弈论的微积分与大学物理保护法~~

### 题解

## G. 你怎么还没睡啊

### 题目描述

一共 $nm$ 个盒子 $k$ 个球，盒子可以互相区分，球可以互相区分，其中一个球装在一个确定的盒子里，剩下的球可以任意放在任意的盒子里。求放后每个盒子都有球的概率。

数据范围：$1\le nm,k\le 10^6$。

### 题解

因为有一个球已经确定了位置，高中学过这样的情况要分类讨论。

- 如果这个球放的盒子里只有它一个球，剩下的情况就是 $nm-1$ 个可区分的盒子里放 $k-1$ 个球，要求盒不为空；
- 如果这个球放的盒子里还有其他球，剩下的情况就是 $nm$ 个可区分的盒子里放 $k-1$ 个球，要求盒子不为空。

总情况就是往 $nm$ 个可以互相区分的盒子里放 $k-1$ 个球，两者相除即为所求答案。

对于 $n$ 个互不相同的盒子里放 $m$ 个互不相同的球，盒子不允许为空的情况，方案数为 $n!S(m,n)$。$S(n,m)$ 为第二类 Stirling 数。

对于 $n$ 个互不相同的盒子里放 $m$ 个互不相同的球，盒子允许为空的情况，方案数为 $n^m$。

然后求第二类 Stirling 数有一个卷积式子：

$$
S(n,m)=\sum_{k=0}^m\frac{(-1)^k}{k!}\cdot \frac{(m-k)^n}{(m-k)!}
$$

这样就可以快速求出需要的第二类 Stirling 数了。

答案式子为：

$$
\frac{(nm)!(S(k-1,nm-1)-S(k-1,nm))}{nm^{k-1}}
$$

时间复杂度 $\mathcal{O}(k)$，空间复杂度 $\mathcal{O}(k)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2276.cpp)

## H. world domination

### 题目描述

在三维空间坐标系中，求在 $(0,0,0)$ 处能看到多少整点，并且整点点坐标的绝对值都小于等于 $n$。

这里定义看到为这个点到原点的连线上没有其他点。

数据范围：$0\le n\le 10^7$。

### 题解

这个和 SDOI 2008 的[「仪仗队」](https://www.luogu.com.cn/problem/P2158)一题很相似，但是这里是对这道题的三维拓展。

还和 [SPOJ VLATTICE](https://www.spoj.com/problems/VLATTICE/) 差不多，但是 SPOJ 那道题只需要算第一卦限和三个坐标面三个坐标轴上的值就行了。

我们考虑点坐标绝对值均大于等于 $1$ 的情况，其中一个或两个为 $0$ 的情况和均为 $0$ 的情况，并考虑对称性，在各个卦限和四分之一坐标面上答案均相等，可以得到答案为：

$$
8\sum_{i=1}^n\sum_{j=1}^n\sum_{k=1}^n[\gcd(i,j,k)=1]+12\sum_{i=1}^n\sum_{j=1}^n[\gcd(i,j)=1]+7
$$

最后的 $7$ 为点坐标有两个为 $0$ 和三个都等于 $0$ 的总情况数，为 $6+1$。

考虑 Möbius 反演，先对简单的二维情况推导。

构造一个函数 $f(n)=\sum_{d\mid n} \mu(d)$，可以发现这个函数很有意思：

$$
f(n)=
\begin{cases}
1 & n=1\\
0 & n>1 
\end{cases}
$$

其实是我们对着 Möbius 反演干了一件很无聊的事情：这里 $F(n)=1$，$f(n)=[n=1]$，观察到 $F(n)=\sum_{d\mid n}f(d)$，因为任意正整数都有一个因子是 $1$，所以有以上结论。

那么我们要求的就变成了：

$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^n [\gcd(i,j)=1]
&=\sum_{i=1}^n\sum_{j=1}^n f(\gcd(i,j))\\
&=\sum_{i=1}^n\sum_{j=1}^n \sum_{d\mid \gcd(i,j)} \mu(d)\\
&=\sum_{i=1}^n\sum_{j=1}^n \sum_{d\mid i\wedge d\mid j} \mu(d)
\end{aligned}
$$

我们交换后一个的求和顺序，变成枚举因子的形式：

$$
\begin{aligned}
\sum_{i=1}^n\sum_{j=1}^n[\gcd(i,j)=1]&=\sum_{i=1}^n\sum_{j=1}^n \sum_{d\mid i\wedge d\mid j} \mu(d)\\
&=\sum_{d=1}^n \mu(d)\sum_{i=1}^{\lfloor \frac{n}{d}\rfloor}\sum_{j=1}^{\lfloor \frac{n}{d}\rfloor} 1\\
&=\sum_{d=1}^n\mu (d)\left(\lfloor \frac{n}{d} \rfloor\right)^2
\end{aligned}
$$

然后这个式子就可以在 $\mathcal{O}(n)$ 的复杂度内求出了。

对于三维情况，有类似推导，可以得到：

$$
\sum_{i=1}^n\sum_{j=1}^n\sum_{k=1}^n[\gcd(i,j,k)=1]=\sum_{d=1}^n\mu (d)\left(\lfloor \frac{n}{d} \rfloor\right)^3
$$

同样做一下即可。

时间复杂度 $\mathcal{O}(n)$，空间复杂度 $\mathcal{O}(n)$。

实际上对于 $\lfloor \frac{n}{d}\rfloor$ 可以利用数论分块，枚举变化点来求值。但是线性筛就是 $\mathcal{O}(n)$ 的也不是多组查询就不优化了。

据说还可以杜教筛做到 $n^{\frac{2}{3}}$，但是太鬼畜了（

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2278.cpp)

## I. 快去睡觉

### 题目描述

$nm$ 个盒子 $k$ 个小球，多次询问四种把小球放进这些盒子使得盒子不空的方案数：

- 小球可互相区分，盒子可互相区分；
- 小球可互相区分，盒子不可互相区分；
- 小球不可互相区分，盒子可互相区分；
- 小球不可互相区分，盒子不可互相区分。

数据范围：$1\le n,m,k\le 10^3$。

### 题解

很显然的是，如果 $nm>k$，那么一定会出现空盒子，所以先把这部分判掉。

对于小球可互相区分，盒子可互相区分的情况，这个和 G 是一样的，是第二类 Stirling 数乘一个阶乘。

对于小球可互相区分，盒子不可互相区分的情况，是第一种情况除以一个阶乘。

对于小球不可互相区分，盒子可互相区分的情况，显然插板法这个是高中题。

对于小球不可互相区分，盒子不可互相区分的情况，这个是一个类似 DP 的递推。

具体可以参考 [HaHaTa 的博客 (Web Archive)](https://web.archive.org/web/20210227115934/http://hahata.org/647/)。

于是我们先预处理一下第二类 Stirling 数，组合数，那个 DP 和阶乘，然后计算即可。

时间复杂度 $\mathcal{O}(k^2)$，空间复杂度 $\mathcal{O}(k^2)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2275.cpp)

## J. 白给的几何题 1

### 题目描述

求矩形面积并。

数据范围：$1\le n\le 10^5,0\le x,y\le 10^5,T\le 300$。

### 题解

矩形面积并有经典的离散化后扫描线+线段树算法。用扫描线维护 $y$ 轴区域，然后查询 $x$ 轴上有多长有效长度，乘起来就是面积了。

原题来自：[POJ 1151. Atlantis](http://poj.org/problem?id=1151)

时间复杂度 $\mathcal{O}(n\log n)$，空间复杂度 $\mathcal{O}(n)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2281.cpp)

## K. 白给的几何题 2

### 题目描述

求平面最远点对。

数据范围：$1\le n\le 5\times 10^4$。

### 题解

平面最远点对一定在凸包上，所以求凸包上的最远点对就完事了。

如果数据是随机的，XJB 暴力的复杂度是对的，因为随机 $n$ 个点的凸包大小是 $\log n$ 级别的，暴力就是 $(\log n)^2$。

但是不是随机的话，这样显然被卡，直接生成一个 $n$ 个点的凸多边形随便卡，于是利用旋转卡壳就可以了。

对于平面最远点对在凸包上的证明，可以利用调整法证明，如果一个点不在凸包上，那么调整到凸包上一定会增大答案。实际上咋证我也不会。

时间复杂度 $\mathcal{O}(n)$，空间复杂度 $\mathcal{O}(n)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2282.cpp)

## L. 白给的几何题 3

### 题目描述

给定 $n$ 个点，求能够覆盖这 $n$ 个点的最小圆。

数据范围：$1\le n\le 10^5$。

### 题解

最小圆覆盖是一个神奇的东西，利用随机增量法，先打乱所有输入，枚举一个点 $i$，如果 $i$ 不在当前圆内，就把它设成圆心，半径为 $0$，然后枚举第二个点 $j$，如果这个点不在圆内，就把答案圆设成以 $i,j$ 为直径的圆，再枚举第三个点 $k$，如果这个点不在圆内，就把答案设成三角形 $ijk$ 的外接圆。这样看起来是 $\mathcal{O}(n^3)$ 的，但是实际上是期望 $\mathcal{O}(n)$ 的，证明是通过进入循环的概率证明的。实际上怎么证我也不会。

时间复杂度 $\mathcal{O}(n)$，空间复杂度 $\mathcal{O}(n)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2283.cpp)
