---
title: UESTC 第十届 ICPC 趣味程序设计竞赛第五场（正式赛）题解
categories: 'OI/ICPC'
date: 2018-12-08 17:30:43
tags:
	- 模拟
	- 数学
	- 数论
	- 贪心
toc: true
---

3/5 Penalty: 04:00:50 Rank 14
<!-- more -->

woc 就差一个比较就能 $4$ 题了啊啊啊啊啊我石乐志啊啊啊啊啊

因为是英文场，作为 LibreOJ 翻译组的一个 sb，懒得翻译了……

## A. Cooking - 做饭

题目大意：给了 $n$ 个桶，每次选 $k$ 个扔一个球，求扔多少次之后每个桶都至少有 $m$ 个球，输出这个次数的 $t$ 倍。

啊啊啊啊啊啊啊我就差一个判断就 A 了啊啊啊啊啊啊啊

时间复杂度：$\mathcal{O}(1)$，空间复杂度：$\mathcal{O}(1)$

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2353.cpp)。

## B. You Know Nothing, Jon Snow - 你什么都不知道，Jon Snow

（本题梗来自：冰与火之歌）

题目大意：给了一个数 $N$，每次减去它的最小质因子，求一共会减多少次，这个数变成 $0$。

分成两种情况，如果这个数是个偶数，那么每次都是减 $2$，输出 $\frac{n}{2}$ 即可，如果是奇数，首先它的质因子一定没有 $2$，所以第一次减掉的是一个奇数因子，然后它就变成了偶数，然后回到前一种情况。

时间复杂度：$\mathcal{O}(\sqrt n)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2354.cpp)。

## C. Pour Tea for Master oy - 给 oy 大爷倒茶

题目大意：每种茶有个编号，给出它们的二进制表示，定义 DIO 值是两种茶叶编号的异或值的二进制表示中 $1$ 的个数，现在要买一种茶，使得这种茶与给的 $n$ 种茶的 DIO 值的最大值最小，求这种茶的编号的二进制表示。

暴力之。

时间复杂度：$\mathcal{O}(n2^k)$，空间复杂度：$\mathcal{O}(n)$

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2355.cpp)。

## D. Slay The Spire - 杀戮尖塔

（本题梗来自：杀戮尖塔）

题目大意：一个勇士有一个初始 HP 为 $m$，现在他要上 $n$ 层塔，每上一层塔会掉 $a_i$ 点血，但是上到下一层就会回 $6$ 点血。他有一个烟雾弹，用了之后他就会无伤上到下一层，回血照样回，求他能不能通关。

一个烟雾弹相当于一个药瓶，药瓶能回这层掉的全部血量，然后在快没血的时候判断一下使不使用即可。

这个药瓶的血量是动态变化的，如果要在第 $i$ 层用烟雾弹，那么这个药瓶的血量是前 $i-1$ 层血量的最大值。

然后就可以玩了……

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(1)$

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2356.cpp)。

## E. The Party - 聚会

题目大意：有 $n$ 个数，定义 $f(i)$ 表示在这 $n$ 个数中选 $i$ 个数，它们的最大公约数的最大值。求 $f(1)\sim f(n)$。

感觉这种类型见了很多次了……

最大公约数一定不会超过这 $n$ 个数的最大值，所以我们枚举最大公约数。

然后，所有是这个数的整数倍的数的最大公约数就是这个数。统计出来有几个，更新一下答案。

最后，答案是单调不升的，但是可能有些位置并没有答案。我们可以发现，如果 $k$ 个数的最大公约数为 $r$，那么 $k-1$ 个数的最大公约数至少为 $r$，而大于 $r$ 的话答案一定可以被统计到，所以最后修改一下就行了。

时间复杂度：$\mathcal{O}(m\sqrt m)$，空间复杂度：$\mathcal{O}(m)$，其中 $m=\max_{i=1}^n \{a_i\}$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2357.cpp)。
