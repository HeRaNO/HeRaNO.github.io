---
title: Educational Codeforces Round 71 (Rated for Div. 2) 题解
categories: 'OI/ICPC'
toc: true
date: 2019-08-23 14:06:31
tags:
	- 模拟
	- 数学
	- 数论
	- DP
	- AC 自动机
	- 树状数组
---

(5-1)/7 Penalty: 224 Rank: 1243/1082

<!-- more -->

小号被 hack 人生大落大起大落还能上蓝我是真的佛了。

## A. There Are Two Types Of Burgers

### 题目描述

做一个汉堡需要两片面包和一片牛肉，做一个鸡肉汉堡需要两片面包和一块鸡肉。现在有 $b$ 片面包，$p$ 片牛肉和 $f$ 片鸡肉。做一个汉堡挣 $h$ 元，做一个鸡肉汉堡挣 $c$ 元。求最大总利润。

数据范围：$1\le b,p,f,h,c\le 100$。

### 题解

数据范围很小，直接暴力都行……

时间复杂度 $\mathcal{O}(Tpf)$，空间复杂度 $\mathcal{O}(1)$

如果 $p,f$ 范围大的话，根据每个汉堡的价格贪心即可。

但是一分钟手速掉这道题发现网太慢了交不上去……

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1207A.cpp)

## B. Square Filling

~~（反正这里是熊猫头，下面是脏话）~~

### 题目描述

一个 $n\times m$ 的矩阵，初始全是 $0$，每次可以选择一个 $2\times 2$ 的范围将它们全部填充为 $1$，求是否能填充成给定的 $n\times m$ 矩阵，若能则输出填充最少步数的方案，否则输出 $-1$。

### 题解

直接枚举每个 $2\times 2$ 区域填充即可，最后判断一下是不是和给定矩阵一样。

注意只能填充 $2\times 2$ 范围内全部是 $1$ 的范围。

然后手贱把 $m$ 打成 $n$ 了之后被 hack 了。

~~（反正这里是熊猫头，下面是脏话）~~

时间复杂度 $\mathcal{O}(nm)$，空间复杂度 $\mathcal{O}(nm)$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1207B.cpp)

## C. Gas Pipeline

### 题目描述

给了一条街道和一个 $01$ 串，如果串的某一位为 $1$ 则表示这一段是路口，管道高度必须为 $2$，否则高度为 $1$ 或 $2$ 均可。一个单位长度管道价格为 $a$，一单位支管道的柱子价格为 $b$，求建管道的最小花费。

数据范围：$1\le \sum n\le 2\times 10^5,1\le a,b\le 10^8$。

### 题解

队友贪心过去了，实际上和 DP 打的是一样的……

记 $f_{i,0/1}$ 表示第 $i-1\sim i$ 段路高度为 $1/2$ 的花费。

对于高度没有要求的路段，转移如下：

$$
\begin{align}
f_{i,0}&=\min\{f_{i-1,0}+a,f_{i-1,1}+2a\}+b\\
f_{i,1}&=\min\{f_{i-1,1}+a,f_{i-1,0}+2a\}+2b\\
\end{align}
$$

对于高度有要求的路段，转移如下：

$$
\begin{align}
f_{i,0}&=+\infty\\
f_{i,1}&=f_{i-1,1}+a+2b\\
\end{align}
$$

初值为：

$$
\begin{align}
f_{0,0}&=b\\
f_{0,1}&=+\infty\\
\end{align}
$$

答案就是 $f_{n,0}$ 了。

时间复杂度 $\mathcal{O}(T\sum n)$，空间复杂度 $\mathcal{O}(n)$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1207C.cpp)

## D. Number Of Permutations

### 题目描述

有 $n$ 个二元组 $(a_i,b_i)$，一个坏排列定义为这 $a_i$ 单调不减或 $b_i$ 单调不减，求好排列数量对 $998244353$ 取模。

数据范围：$1\le n\le 3\times 10^5,1\le a_i,b_i\le n$。

### 题解

好排列数量等于 $n!$ 减去坏排列数量。统计一下坏排列数量个数就行了。

坏排列数量直接容斥一下，就是 $a_i$ 单调不减的排列数量加 $b_i$ 单调不减的排列数量减去 $a_i,b_i$ 均单调不减的排列数量。

$a_i$ 单调不减的排列数量只需要统计 $a_i$ 中元素相同的个数，然后乘一下全排列就行了，$b_i$ 和 $(a_i,b_i)$同理，但是注意 $(a_i,b_i)$ 统计后需要再检查一下这个序列是不是真的单调不减（排完序之后会出现第二维减的情况），如果不是则这部分贡献为 $0$。

然后输出答案就行了……

时间复杂度 $\mathcal{O}(n\log n)$，空间复杂度 $\mathcal{O}(n)$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1207D.cpp)

## E. XOR Guessing

### 题目大意

可以询问 $100$ 或 $200$ 个数，要求猜一个给定的数。Grader 会返回询问的数中的某一个与要求猜的数的异或值。

数据范围：$0\le x\le 2^{14}-1$

### 题解

分二进制前 $7$ 位为 $0$ 和后 $7$ 位为 $0$，分别查询 $100$ 个数即可。

数据范围很良心，最大查询是 $12800$ 正好在 $2^{14}$ 范围内……

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1207E.cpp)

## F. Remainder Problem

### 题目描述

给定一个长度为 $5\times 10^5$ 的序列，有 $q$ 个操作，分为两类：

- 给位置 $x$ 的元素加 $y$；
- 查询位置编号对 $x$ 取模为 $y$ 的所有元素值之和。

数据范围：$1\le q\le 5\times 10^5$。

### 题解

下面记 $n=5\times 10^5$。

CF 又双叒叕出原题辣，在[这里](https://www.luogu.com.cn/problem/P3396)。

2014 年集训队论文里也写辣，详见《根号算法——不只是分块》。

我们想对直接统计不是那么优秀的合并统计，直接统计很快的暴力过去，于是我们统计所有 $x\le \sqrt n$ 的答案，这样就合并了这些统计出来很慢的东西，然后对于大于 $\sqrt n$ 的部分，直接暴力，复杂度是 $\sqrt n$ 的。

对于修改，暴力过去就完事了……

时间复杂度 $\mathcal{O}(q\sqrt n)$，空间复杂度 $\mathcal{O}(n)$。

和 [Codeforces Beta Round #80 (Div. 1 Only) D. Time to Raid Cowavans](https://codeforces.com/problemset/problem/103/D) 感觉上是一道题……

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1207F.cpp)

## G. Indie Album

### 题目描述

给一棵 Trie 树，询问一个字符串在某条链上的出现次数。

数据范围：$1\le n,m\le 4\times 10^5$。

### 题解

又被熊大爷教育了……

「$S_1$ 在 $S_2$ 中的出现次数转化为有多少个 $S_2$ 的前缀以 $S_1$ 作为后缀，$S_2$ 的前缀以 $S_1$ 作为后缀当且仅当代表前者的点在代表 $S_1$ 的点的子树中。」

这里子树指 Fail 树。

然后就是基本操作了……对询问离线，插到 AC 自动机里，跑一遍 dfs 序，再 dfs 一下 Fail 树处理询问即可……

时间复杂度 $\mathcal{O}(n+m\log n)$，空间复杂度 $\mathcal{O}(n+m)$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Codeforces/1207G.cpp)
