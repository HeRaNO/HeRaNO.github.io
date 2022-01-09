---
title: 电子科技大学第十一届 ACM 趣味程序设计竞赛第三场（正式赛）题解
categories: 'OI/ICPC'
date: 2019-11-23 17:00:00
tags:
	- 数学
	- 模拟
	- 贪心
toc: true
description: ' '
---

## A. 双十一
### 题目描述
有 $n$ 个物品在 $m$ 个商店出售，给出在每个商店的售价，问如何买才能使花费最小。输出最小花费。

数据范围：$1\le n,m\le 100,-1\le a_{i,j}\le 100$。

### 题解
对于每种商品，在售价最便宜的商店中买即可。

时间复杂度：$\mathcal{O}(nm)$，空间复杂度：$\mathcal{O}(n)$。

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2331.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2331.java)。

Problem Setter: Macaron_lin

## B. 我们身边的狼
### 题目描述
有 $n$ 个人分为好人和坏人，其中好人数量不少于坏人。好人只会说真话，坏人只会说假话，他们都知道每个人的身份。现在可以问他们任意一人（包括自己）的身份，他们一定会回答并只回答是或否。问能否在所有情况下通过一系列问题问出每个人的身份（好人或坏人）。

数据范围：$1\le n\le 10^9$。

### 题解
根据样例可以推测，如果 $n$ 为奇数即可推出，否则不能。

我们假设好人有 $a$ 个，坏人有 $b$ 个。由题可知 $a\ge b$，且 $a+b=n$。

现在分成三部分考虑，当 $a>b+1$ 时，无论怎样都可以判断出哪些是好人，哪些是坏人。我们可以随机挑选出一个人，问剩下的人这个人是好人吗？回答相同的人为一类，并且人数多的一方为好人。然后再找好人问选出的那一个人是好是坏即可。

当 $a=b$ 时，无论如何询问，都会出现因为不清楚被选出的那个人是好人还是坏人而无法判断人数多的那一方到底是好人还是坏人。所以 $a=b$ 时永远不可能分清哪些是好人，哪些是坏人。

当 $a=b+1$ 时，我们可以同 $a>b+1$ 一样区分开，如果遇到选出的那个人为好人的情况，我们可以再挑选一个人，可以保证总有一次可以选到一个坏人，并区分开好人和坏人。

综上所述，当存在一种情况需要区分 $a=b$ 时，就永远无法分出来了。因此，当 $n$ 为偶数时存在这种情况，而当 $n$ 为奇数时不存在。所以有一开始的结论。

题目样例说明可以提示证明方法。

时间复杂度：$\mathcal{O}(1)$，空间复杂度：$\mathcal{O}(1)$。

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2346.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2346.java)。

类似思路来自于 ICPC 2019 Nanjing Regional Onsite H. Prince and Princess

Problem Setter: Vingying


## C. %%BlackRed·Z
### 题目描述
给定 $a_1,a_2,\ldots ,a_n$，求 $f(x)=\sum_{i=1}^n (x\bmod a_i)$ 的最大值。

数据范围：$2\le n\le 3\times 10^3,2\le a_i\le 10^5$。

### 题解
既然要 $f(x)$ 最大，那么可能的最大值应该为 $\sum_{i=1}^n (a_i-1)$。也就是说下列同余方程组的解即为最大值点：

$$
\begin{align}
x &\equiv a_1-1 \pmod{a_1}\\
x &\equiv a_2-1 \pmod{a_2}\\
&\ldots \\
x &\equiv a_n-1 \pmod{a_n}\\
\end{align}
$$

可知，一个解为 $\text{lcm}(a_1,a_2,\ldots ,a_n)-1$，这个解是存在的。

由 $\text{lcm}(a_1,a_2,\ldots ,a_n) \bmod a_i =0$，则 $(\text{lcm}(a_1,a_2,\ldots ,a_n)-1) \bmod a_i =-1$，在模意义下 $-1\equiv a_i-1\pmod {a_i}$，因此有上述结论。

因此，答案即为 $\sum_{i=1}^n (a_i-1)$。

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(1)$。

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2337.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2337.java)。

Problem Setter: michaelshuoliu

## D. 炉石传说
### 题目描述
一共 $n$ 个数 $a_1,a_2,\ldots ,a_n$，每次可以让其中一个数减 $Y$，或者让所有数都减 $X$，如果每一次减后某个数小于等于 $0$ 则从这个序列中删除。问至少多少次能够使得所有数都被删除。

数据范围：$1\le n\le 5\times 10^3,1\le X,Y,a_i\le 5\times 10^3$。

### 题解
对于本题，直接入手比较困难，但观察到每个数全部减 $X$ 的话减的次数可能很少，多余的部分就可以用 $Y$ 减掉。因此考虑枚举打全体伤害的次数，然后计算打单体伤害的次数。对于打了 $i$ 次全体伤害，第 $j$ 个数即为 $a_j-Xi$，然后计算 $\lceil \frac{a_j-Xi}{Y} \rceil$ 即可。如果 $a_j-Xi$ 是负数，则说明全体伤害已经伤害溢出，不需要再打单体了。答案即为对于每种打全体伤害的次数，两者和的最小值。

时间复杂度：$\mathcal{O}(nX)$，空间复杂度：$\mathcal{O}(n)$。

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2329.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2329.java)。

Problem Setter: hbji

## E. 大地的裂变
### 题目描述
给出二维平面上的 $n$ 条直线，问对于每个 $t\ (2\le t\le n)$，坐标值为非负整数的交点被 $t$ 条直线经过的交点数量。

数据范围：$2\le n\le 10^4,0\le k,b\le 100$。

### 题解
考虑交点坐标的范围，设两条直线 $y=k_1x+b_1$ 与 $y=k_2x+b_2$，联立后可解得：

$$
x=\frac{b_1-b_2}{k_2-k_1}\\
y=\frac{k_2b_1-k_1b_2}{k_2-k_1}
$$

由于要求的是非负整数坐标，那么根据数据范围，可知 $0\le x\le 100,0\le y\le 10^4$。因此统计这部分区域每个非负整数点有多少条直线经过即可。最后利用数组统计输出。

时间复杂度：$\mathcal{O}(kb^2)$，空间复杂度：$\mathcal{O}(kb)$。

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2326.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2326.java)。

Problem Setter: ZXyaang
