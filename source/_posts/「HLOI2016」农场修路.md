---
title: 「HLOI2016」农场修路
categories: 'OI/ICPC'
date: 2017-02-18 16:42:00
tags:
	- HLOI
	- 最小生成树
toc: true
description: HLOI2016 Day2 T1
---

## 题目描述
小明是一个快乐的农场主，他有 $N$ 个农场，编号从 $1$ 到 $N$，因为每个农场距离很远，现在他想修一些路使得任意两个农场都有道路能够到达。设计师给出了 $N$ 个农场由 $M$ 条公路相连的设计图，小明希望修的道路尽可能的少并且在修完的所有道路中最长的路程和最短的路程之间差值最小。

## 输入
第一行给出 $N,M$；

接下来 $M$ 行，每一行包含三个整数：$u,v,c$，表示农场 $u$ 和农场 $v$ 之间有一条路可以互达，路程 $c$；

保证无重边，无自环。

## 输出
输出一行包括一个整数，表示在修完的所有道路中最长的路程和最短的路程之间的最小差值。如果不能实现任意两个农场都有道路能够互达，输出 $-1$。

## 样例输入
```
3 3
1 2 1
1 3 2
2 3 3
```
## 样例输出
```
1
```

## 数据范围及约定
对于 $30\%$ 的数据有 $2\le N\le 20$；

对于 $60\%$ 的数据有 $2\le N\le 50$；

对于 $100\%$ 的数据有 $2\le N\le 100,0\le M\le \frac{N\times (N-1)}{2},0\lt c\le 10^7$。

## 时间限制与空间限制
时间限制：$5000\text{ms}$，空间限制：$128\text{MB}$。

## 题解
黑历史题，详见 [UVa1395 Slim Span](https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=4141)。

因为当最小边确定时，最小生成树唯一确定，所以枚举所有最小边，生成最小生成树，然后比较答案，选择最优解。

时间复杂度： $\mathcal{O}(m\log m+mn)$，空间复杂度：$\mathcal{O}(m)$。

我也不知道为什么要开五秒。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HSAHRBNUOJ/P38xx/P3874.cpp)。
