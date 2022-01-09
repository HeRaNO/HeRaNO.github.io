---
title: 'Codeforces Round #538 Div. 2 题解'
categories: 'OI/ICPC'
toc: true
date: 2019-02-11 16:06:30
tags:
	- 模拟
	- 贪心
	- 线段树
	- 数论
---

Penalty: 2568 (3/6) Rank: 959
<!-- more -->

明明是一场能 $4$ 题的上分场，可是，为什么呢……

比赛链接在[这里](https://codeforces.com/contest/1114)

## A. Got Any Grapres?

题目大意：A，D，M 三人分别至少要吃 $x,y,z$ 个葡萄，A 只吃绿葡萄并且至少要吃 $a$ 个绿葡萄，D 不吃黑葡萄，M 啥都行。给了绿葡萄，紫葡萄和黑葡萄分别 $a,b,c$ 个，问能不能满足他们的要求。

判断题，如果 $x\gt a$ 果断不行，如果 $y\gt a-x+b$ 果断不行，如果 $a+b+c\lt x+y+z$ 果断不行，剩下都行。

时间复杂度：$\mathcal{O}(1)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1114A.cpp)

## B. Yet Another Array Partitioning Task

题目大意：对数组 $a$ 进行一个 $k$ 划分（即划分为 $k$ 个连续数组），使得划分出的所有连续子数组前 $m$ 大值的和最大。输出这个和与划分方案。

考虑贪心，给这一列数排序，我们选的数一定是前 $mk$ 个。因为数组至少要 $m$ 个数，并且要一个 $k$ 划分。如果有负数，那么一定是最优的，如果均为正数，那么加入再多的数对于答案也没有贡献，因此划分一定是这样的。

所以排个序，对于前 $mk$ 个数标记一下在原数组的位置，然后输出的时候每 $m$ 个标记点分一段，输出 $k-1$ 次即可。

时间复杂度：$\mathcal{O}(n\log n)$，空间复杂度：$\mathcal{O}(n)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1114B.cpp)

## C. Trailing Loves (or L'oeufs?)
题目大意：问 $n!$ 在 $b$ 进制下末尾有多少 $0$。

CF 居然出原题辣！原题见[这里](https://www.luogu.com.cn/problem/P3927)。

考虑 $b=p_1^{k_1}p_2^{k_2}\ldots p_n^{k_n}$，考虑对 $n!$ 进行相同的拆分，即 $n!=p_1^{x_1}p_2^{x_2}\ldots p_n^{x_n}P$，其中 $P$ 与任意 $p_i$ 互质。

如果能整除 $y$ 个 $p_i$，那么结果的末尾就有 $y$ 个 $0$。

于是我们要求的是选择哪个 $p_i$ 能除最少的次数，因为如果比它多将有因子除不尽。

时间复杂度：$\mathcal{O}(\sqrt b+\log b\log n)$，空间复杂度：$\mathcal{O}(\sqrt b)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1114C.cpp)

## D. Flood Fill
题目大意：定义连通分量为有相同颜色的连续子数组。每一次你可以选择一个位置，并且把这个位置所在的一个连通分量改为一种颜色，代价为 $1$。求把整个数组改为同一种颜色的最小代价。

打比赛的时候去看 F 了，没看 D……

看起来像一个区间 DP，但是并不需要 $\mathcal{O}(n^3)$ 的复杂度。

考虑一个区间拓展一位，那么合并后区间的颜色为 $c_L$ 或 $c_R$，因此设 $f_{i,j,0/1}$ 为让线段 $[i,j]$ 都改成和 $c_i$ 或 $c_j$ 一样的最小代价，一共四种转移 yy 一下就好了。

但是注意转移顺序。

时间复杂度：$\mathcal{O}(n^2)$，空间复杂度：$\mathcal{O}(n^2)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1114D.cpp)

## E. Arithmetic Progression
题目大意：给定一个打乱顺序的等差数列的前 $n$ 项，猜等差数列的通项公式。你可以问一个位置的值，还可以问有没有元素大于你给定的值。但是询问最多 $60$ 次。

不想做交互，略过。

题解是二分和随机化。

## F. Please, another Queries on Array?
题目大意：给定一个数列 $\{a_i\}$，有以下两个操作：

1. 给 $[L,R]$ 区间的数乘一个值 $c$；
2. 求 $\varphi (\prod_{i=L} ^R a_i) \bmod (10^9+7) $。

考试时想出来怎么做了，结果没调完……

把思路告诉了一个人，结果他 A 了……

考虑一种计算欧拉函数的方法：

$$
\varphi(x) = x\prod_{i=1}^ n(1-\frac{1}{p_i})
$$

因为 $\{ a_i \}$ 和 $c$ 都在 $[1,300]$ 之内，因此所有可能出现数的质因数都在 $300$ 以内，只需要处理 $300$ 以内的质数就好了，共 $62$ 个。

分部分处理，前面有一个乘以 $x$，于是需要处理区间积，后面 $1-\frac{1}{p_i}$ 需要处理逆元，然后处理这个区间中 $p_i$ 是否作为质因子出现过，然后统一处理逆元。

处理区间积的时候需要一个 lazy 标记，然后处理质因子出现情况的时候也需要一个标记！不能把大区间的情况直接覆盖下去！

然后就是粘代码注意是不是少粘了什么……

一个 Trick 是预先把 $[1,300]$ 内的数先做一下质因数分解，能消掉一个 $\log $。但是最后加了读入优化时间没有什么特别大的改观（2511ms 变成了 1777ms……）

逆元直接打表就行了……没多大……

建线段树时间复杂度窝怎么感觉是 $\mathcal{O}(n)$ 的……

时间复杂度：$\mathcal{O}(T(\log n+\pi (300)))$，空间复杂度：$\mathcal{O}(n)$。

据说卡空间，没感觉……

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1114F.cpp)