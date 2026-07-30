---
title: UESTC 第十届 ICPC 趣味程序设计竞赛第四场（正式赛）题解
categories: 'OI/ICPC'
date: 2018-12-02 17:46:27
tags:
	- 模拟
	- 数学
	- 概率与期望
toc: true
---

2/5 Penalty: 02:15:58 Rank 94
<!-- more -->

~~我不露一手你们真的以为我很强？~~

被第二题打自闭（第二题是大一大爷出的不敢说话），然后就出现了 $26$ 个 AK 的……

今天智商不在线实锤，滚去写微积分作业。

我有毒，真的，我觉得别人觉得简单的场很难，我觉得别人觉得比较难的场简单……

包括数学考试，英语考试，全部应验……

比赛链接早已失效，不放了。

## A. 就是要众生平等

题目大意：一个数列的所有元素和要变成 $nk$，最少要加多少？

mdzz

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Contest/The%2010th%20UESTC%20Fun%20Programming%20Contest%20-%202nd%20Phase/A.cpp)。

## B. Sindar家的气球

题目大意：有一些气球排成一列做匀速直线运动，气球的质量都相等，并且有一个速度。求经过碰撞后最快与最慢的气球到达终点的用时。

mdzz 我 TM 直接去想动量定理了。

不就是交换速度吗，可以看成两个气球位置交换……

所以就当没有碰撞就好了……

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Contest/The%2010th%20UESTC%20Fun%20Programming%20Contest%20-%202nd%20Phase/B.cpp)。

## C. 蚂蚁的旅途

题目大意：一个点从数轴上的原点出发，每一秒等概率地向左或向右移动一个单位长度，求 $t$ 秒后在 $m$ 位置的概率。

数据不满，跑暴力。

时间复杂度：$\mathcal{O}(T2^t)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Contest/The%2010th%20UESTC%20Fun%20Programming%20Contest%20-%202nd%20Phase/C.cpp)。

## D. XOR 最长路径

这道题排版满分，点名表扬。

题目大意：求 $0$ 到 $n$ 的一个字典序最小的排列，使得所有相邻两个数的异或的 NIM 和最大。

暴力打错，手动再见。

写出这个式子，因为异或具有结合律，并且 $a\oplus a=0$，因此就令第一个数与最后一个数的异或值最大即可。

一列数中选两个数异或值最大有经典的 Trie 树做法，直接上即可，但要记录这两个数的值。最后把这两个数排序放在序列的头尾，剩下的数排个序即可。

但是观察一下，选的这两个数的异或应该是 $2^p-1$ 这样，因为要使第一个数尽可能小，所以要使最后一个数尽可能大，一直大到 $n$ 就行。

然后就不用 Trie 树了，直接做就好了……

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(n)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Contest/The%2010th%20UESTC%20Fun%20Programming%20Contest%20-%202nd%20Phase/D.cpp)。

## E. 奥本海姆的衣柜

题目大意：还是题解写得明白……

不想说话，直接粘[题解](https://blog.csdn.net/qq_39471664/article/details/84719578)……

时间复杂度：$\mathcal{O}(nk)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/Contest/The%2010th%20UESTC%20Fun%20Programming%20Contest%20-%202nd%20Phase/E.cpp)。
