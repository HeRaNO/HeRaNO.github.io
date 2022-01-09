---
title: UESTC 第十届 ICPC 趣味程序设计竞赛第三场（正式赛）题解
categories: 'OI/ICPC'
date: 2018-11-24 16:49:52
tags:
	- 模拟
	- 概率与期望
	- 数学
toc: true
---

4/5 Penalty: 09:23:54 Rank 20
<!-- more -->

罚时爆炸，代码调不对，概率论不会，整段垮掉。

比赛链接在[这里](https://acm.uestc.edu.cn/contest/57/summary)。

## A. 秦皇炒饭

题目大意：将 $1\sim n$ 分成最少的组，使得组内元素两两互质。求这个组数。

第一眼就和 [Codeforces Round #400 B. Sherlock and his girlfiend](http://codeforces.com/problemset/problem/776/B) 混了，然后 XJB 调都不对，然后自闭了。

最后打了个暴搜，找到了规律，交上去过了……

结论：当 $n>1$ 时，分成最小组数为 $\lfloor \frac{n}{2} \rfloor$，当 $n=1$ 的时候输出 $1$ 次。

证明：因为 $2k$ 与 $2k+1$ 互质，一定能放到一组，偶数不满足两两互质不能放在一组，编号为 $1$ 的可以放在任意组。

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(1)$

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2347.cpp)。

## B. 摩天乐

题目大意：一栋楼的左边和右边都有电梯，上下移动只能坐电梯，求一点到另一点的最小距离。

这题 WA 了 $5$ 遍真是没救了……

只需要做出从左边电梯下楼和从右边电梯下楼的答案，然后输出小的即可。

注意同层情况……

时间复杂度：$\mathcal{O}(T)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2349.cpp)。

## C. Akane

题目大意：给定长度为 $n$ 的序列 $A$，重新排列 $A$，使有尽可能多的相邻数对，满足 $(A_i+A_{i+1})\bmod m \equiv 0$。

把式子拆开：$(A_i\bmod m+A_{i+1}\bmod m)\bmod m \equiv 0$

于是，我们对于每一对 $(i,j),\ i+j=m$ 进行配对即可。注意个数相等和个数等于 $0$ 情况的判断。

时间复杂度：$\mathcal{O}(n+m)$，空间复杂度：$\mathcal{O}(m)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2350.cpp)。

## D. 强哥打电话

题目大意：两个人会在一些时间段内打电话，另一个人给这两个中的一个打电话，他从一个时刻开始打，每次打会保持一段时间，如果没打通他会在挂断之后的一段时间后再打，求什么时候能打通。

把时间转化成秒数做，然后暴力……

时间复杂度：$\mathcal{O}(Tn)$，其中 $T=86400$，空间复杂度：$\mathcal{O}(T)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2351.cpp)。

## E. 永远的X日之都—暴击猫

题目大意：打出暴击的概率是 $p$，无 buff 加成的普通暴击伤害倍率是 $k$，一层 buff 提供的暴击伤害倍率提高 $a$，buff 最多叠加层数 $n$。求攻击无限次，装备与不装备该装备的总期望伤害之比。

不会概率论直接自闭了。

其实可以取攻击次数为 $10^7$ 等等大数，然后计算。

分析如下：

设 $f_i$ 为无限次攻击后，某一次攻击时 buff 有 $i$ 层的概率。那么：

$$
\begin{align}
f_0&=p\\
f_1&=p(1-p)\\
f_2&=p(1-p)^2\\
&\ldots\\
f_{n-1}&=p(1-p)^{n-1}
\end{align}
$$

对于 $f_0$，因为无论有多少层 buff，一旦暴击 buff 都清零。对于 $f_1$，是由 buff 为 $0$ 层的状态再加上上一次不是得来。

$f_n$ 不能由这个式子计算。因为 buff 的上限是 $n$，此时再不暴击 buff 层数也不会增加。

可以列方程：

$$
(f_{n-1}+f_n)\times (1-p)=f_n
$$

解得 $f_n=(1-p)^n$。

所以一次攻击的期望计算方式是：

$$
\begin{align}
m&=\sum_{i=0}^n [f_i\times (k+ia)\times p+f_i\times 1\times (1-p)]\\
n&=pk+(1-p)
\end{align}
$$

可得答案为 $\frac{m}{n}$。

时间复杂度：$\mathcal{O}(n)$，空间复杂度：$\mathcal{O}(1)$。

代码见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/UESTC/2352.cpp)。
