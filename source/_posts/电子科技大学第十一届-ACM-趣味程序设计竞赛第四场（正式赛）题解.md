---
title: 电子科技大学第十一届 ACM 趣味程序设计竞赛第四场（正式赛）题解
categories: 'OI/ICPC'
date: 2019-11-30 17:00:00
tags:
	- 数学
	- 模拟
	- 贪心
toc: true
description: ' '
---

## A. 天气之子
### 题目描述
给定 $n$ 个数 $a_1,a_2,\ldots ,a_n$，从左到右模拟以下过程：如果这个数大于 $w$，计数器加 $a_i-w$，后面所有数加 $k$。询问最后计数器的值。

数据范围：$1\le n\le 2\times 10^5,1\le k,a_i\le 10^4$。

### 题解
显然有一个 $\mathcal{O}(n^2)$ 的做法，即直接模拟这个过程。

仍然考虑从左向右枚举。对于后面的数整体加 $k$ 的这个操作，考虑维护增量，即可将暴力地给后面所有数都加 $k$ 的这部分时间省下。

需要注意数据范围和数据类型的使用。

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(1)$。

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2345.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2345.java)。

Problem Setter: Vingying

P.S. P for Park ^ ^

## B. 夜路
### 题目描述
给定一个 $n\times 2$ 的网格，如果上下两行出现不同列有障碍或者一行两个格子有障碍则无法通行。求从左上角走到底部有多少种方案。

数据范围：$1\le n\le 50$。

### 题解
考虑如果可以走到一行，那么走到这行就有这行空位数那么多种方法。在判断掉非法状态后，利用乘法原理统计即可。如果能够走到底部，答案为 $2$ 的 `..` 行数次方（除去第一行）。

这道题有一个显而易见的 DP 做法。实际上，DP 与这样统计是等价的。还需要注意范围与数据类型的使用。

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(n)$ 或 $\mathcal{O}(1)$。

（因为每行判断是否合法只取决于这一行与上一行的状态，可以只保存这两行的状态以达到空间最优，但是对于本题来说没有太大必要）

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2342.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2342.java)。

Problem Setter: OrangeRain

## C. 蔚蓝
### 题目描述
给定两个数列，每一个都有 $n$ 个数，需要在两个数列中分别取出 $\frac{n}{2}$ 个数，取出的数下标互不相同。这些数将组成一个长度为 $n$ 的新数列，求这个新数列的所有元素之和的最小值。

数据范围：$1\le n\le 10^3,1\le a_i,b_i\le 10^4$。

### 题解
本题可以看成一个集合划分问题，但是由于这道题的特殊性，我们可以换用其他解法。

首先，我们把原数组中的一个当做最终的答案数组，这里不妨设将 $\{a_i\}$ 作为最终的答案数组。但是显然这个答案是错的，因为要恰好挑出 $\frac{n}{2}$ 个数组 $\{b_i\}$ 中的元素，我们还需要把其中 $\frac{n}{2}$ 个 $a_i$ 换成 $b_i$。把一个 $a_i$ 换成 $b_i$，所有元素和就会加上 $b_i-a_i$。为了使得这个和最小，我们肯定要让加的这个值尽可能小，那么根据贪心的思路，我们选择最小的 $\frac{n}{2}$ 个 $b_i-a_i$ 加到所有元素和里，所得的结果就是答案了。

具体证明可以利用调整法，对于最优状态下任意一个被换走的 $a_i$，如果我们选择另一对 $a_i,b_i$ 换，那么 $b_i-a_i$ 的值将不小于目前这对，因此换走将可能会使答案变劣，换用相等的 $b_i-a_i$ 的话对答案无影响，只是影响选择方案罢了。

~~（这道题甚至可以输出方案，扩大范围或者问有多少种方案）（废话~~

这里并没有卡排序，写复杂度合理的各种排序算法均可，根据题目数据范围写桶排序或者基数排序也可（但是需要注意负数的处理）。

时间复杂度：$\mathcal{O}(n^2)$ 或 $\mathcal{O}(n\log n)$ 或 $\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(n)$ 或 $\mathcal{O}(A)$。

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2330.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2330.java)。

Problem Setter: hbji

## D. 外星货币
### 题目描述
求集合 $S$ 的最小大小。这个集合内的数的子集进行加减运算，可以凑出 $1$ 到 $n$ 之间的所有整数，要求每个数不能和自己进行加减运算。

数据范围：$1\le n\le 10^{18}$。

### 题解
考虑这个集合 $S$，假设它满足要求，那么它可以表示的数的范围为 $[1,p]$，其中 $p$ 为集合 $S$ 内所有元素的和。如果再向其中加入一个数 $a$，那么与 $a$ 组合能表示的数的范围为 $[a-p,a+p]$。为了使加入 $a$ 后满足要求，并且使得新表示的元素不与之前可以表示的数重复（即，不浪费新生成的数，这样可以使得集合尽可能小），则需要使得 $a-p=p+1$，因此 $a=2p+1$。

这样考虑递推，根据性质，如果集合 $S$ 内元素和为 $p$，那么集合 $S$ 可以为 $n\in [1,p]$ 时的答案。新加入的数永远为 $2p+1$。记 $S_n$ 为当 $|S|=n$ 时 $S$ 内所有元素和，那么有如下递推关系：

$$
S_{n+1}=S_n+2S_n+1=3S_n+1
$$

当 $n=1$ 时，$S_1=1$。

因为要求最小数目，所以答案为第一个 $n\le S_p$ 的位置 $p$。

根据递推关系，可知通项为 $S_n=\frac{3^n-1}{2}$。这个数列在第 $40$ 项时就已经超过了最大可能的 $n$，所以只需要求出这个数列的前 $40$ 项进行比较即可。

还有一份来自 ZXyaang 的题解。对于一种货币的面值，如果计入答案的话可以有三种选择：加，减或者不选。只要规定一个值是加，比他小的所有面值都可以选择三态中的一个，所以每多选一个货币新增 $3^n$ 个状态。答案就是 $3^n$ 的前缀和。

时间复杂度：$\mathcal{O}(\log n)$，空间复杂度：$\mathcal{O}(\log n)$。

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2333.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2333.java)。

Problem Setter: qqq17770027225

## E. 马拉松
### 题目描述
给定 $n$ 条匀速直线的运动轨迹，这些轨迹都经过 $y=ax+b$ 上的互不相同的点。问所有人的总相遇次数。

数据范围：$1\le n\le 10^5,|a|,|v_x|,|v_y|\le 100$。

### 题解
~~是不是感觉和大地的裂变有点像？~~

考虑两人相遇满足什么条件。考虑两人起始点分别为 $(p,ap+b),(q,aq+b)$，位移向量为 $(v_{xp}t,v_{yp}t),(v_{xq}t,v_{yq}t)$。则：

$$
\begin{cases}
p+v_{xp}t=q+v_{xq}t\\
ap+b+v_{yp}t=aq+b+v_{yq}t
\end{cases}
$$

对于第一个式子，移项可得 $(v_{xp}-v_{xq})t=q-p$。由题，$p\neq q$ 且 $t\neq 0$，由此 $v_{xp}\neq v_{xq}$。

两式联立，可化简为 $a(v_{xp}-v_{xq})=v_{yp}-v_{yq}$，进一步地，$av_{xq}-v_{yq}=av_{xp}-v_{yp}$。

即，如果两人相遇，需要满足：$av_{xq}-v_{yq}=av_{xp}-v_{yp}$ 且 $v_{xp}\neq v_{xq}$。利用数组统计一下即可。注意负数的处理。

~~（原来想卡 log 的但是被 Java 无情地拒绝了）~~

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(av_x^2)$。

参考程序：[C++](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2327.cpp)，[Java](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2327.java)。

Problem Setter: santongding

来自 [Codeforces Round #478 (Div. 2) D. Ghosts](https://codeforces.com/contest/975/problem/D)
