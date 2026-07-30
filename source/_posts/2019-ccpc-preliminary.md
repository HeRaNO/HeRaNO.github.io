---
title: 2019 中国大学生程序设计竞赛（CCPC）网络选拔赛验题报告与部分题解
categories: 'OI/ICPC'
toc: true
date: 2019-08-23 14:06:31
tags:
	- 模拟
	- 数学
	- 数论
	- 贪心
	- 堆
---

2019.8.17 6/11

<!-- more -->

验题真爽，虽然直接认真打了 A 题数据水了没验出来……

题解可以在[这里](/downloads/tutorial.pdf)下载。

## A. ^ & ^
<!-- 命题人：QYitong -->

### 题目描述
找到一个正整数 $C$，使得 $(A\oplus C) \ \texttt{and} \ (B\oplus C)$ 最小。

如果 $C=0$ 时表达式的值为 $0$，则输出 $1$。

数据范围：$T\le 100,1\le A,B\lt 2^{32}$

### 验题过程

Idea：ZXyang & Vingying，Code：HeRaNO

这道题是在过完 G 之后做的……

ZXyang 先提出了算法，然后我顺手敲了，WA 了一发之后和 Vingying 讨论，加了一个补丁之后过了。

然后据说一些提交在这样的数据上有锅：

```plain
1
2 4
```

答案应该是 $2$，但是有不少队伍 A 掉了。

### 题解

首先这种题可以按位考虑，既然要最小，那么如果在这一位上 $A,B$ 都是 $1$，$C$ 就需要是 $1$，否则任何情况都应该为 $0$。

然后这样就会 WA 掉，比如 $A=2,B=5$。

于是做完上述过程后，还需判断 $C$ 是否为 $0$。如果是，答案就是 $A,B$ 二进制中较低的一位 $1$ 所在位置。

后来那组 hack 没挂还挺高兴……不知道是不是正解……

时间复杂度 $\mathcal{O}(T\log \max \{a,b\})$，空间复杂度 $\mathcal{O}(1)$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU6702.cpp)

## B. array
<!-- 命题人：zzb111 -->

### 题目描述
有一个长度为 $n$ 的数组 $a$，初始时 $a_i$ 互不相同，有 $m$ 个操作分为两类：

- 把某个位置的值加 $10^7$；
- 询问这个数组的某个前缀中不小于 $k$ 且与前缀中任何值都不相等的最小值。

本题强制在线。

数据范围：$T\le 10,\sum n,\sum m\le 5.1\times 10^5$

### 验题过程

Idea：ZXyang，Code：HeRaNO

中期打的，当时我和 Vingying 口胡是 mex 结果没继续往下推，Vingying 去看 I 了，我还在写 E 的代码。E 写完之后 ZXyang 说推出来了但是不会写线段树，我就把这个敲掉了……

1A 还是很高兴的 23333

### 题解

首先我们可以把第一个操作想象成把这个数给删掉了，注意到 $1\le k\le n$，那么最差情况答案应该是 $n+1$。这个其实和 mex 是类似的……

再次观察初始 $a_i$ 应该是一个 $1\sim n$ 的排列。（废话

然后考虑权值线段树维护出现位置即可。每次查询实际上是在 $[k,n]$ 中查询第一个大于 $r$ 的位置，如果找不到这个位置的话答案就是 $n+1$。

然后修改操作就很轻松了，单点修改 $a_i$ 的出现位置，改成无穷大就行了。

于是并不需要主席树什么的，简单线段树就完事了。

时间复杂度 $\mathcal{O}(\sum m\log \sum n)$，空间复杂度 $\mathcal{O}(n)$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU6703.cpp)

## C. K-th occurrence
<!-- 命题人：forgottencsc -->
### 题目描述
给定一个字符串 $S$，$m$ 次询问 $S$ 中的一个子串 $S[l\ldots r]$ 在 $S$ 中第 $k$ 次出现的起始位置。

数据范围：$T\le 10,1\le n,m\le 10^5$，只有五组数据 $n\ge 10^3$。

### 验题过程

Idea：HeRaNO，Code：HeRaNO

开场就看见一个字符串，再看好像是 SAM 乱搞，然后想到了暑假前集训的 L 题……

感觉有点类似，但是不会维护第 $k$ 次出现，于是放弃了……

赛后说和 L 一样但是比 L 简单，查一个区间第 $k$ 大就完事了……

### 题解

SAM 的 parent 树拖下来，然后静态区间第 $k$ 大……

我不会字符串 QAQ

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU6704.cpp)

时间复杂度 $\mathcal{O}(n+m\log n)$，空间复杂度 $\mathcal{O}(n\log n)$。

## D. path
<!-- 命题人：QWsin -->
### 题目描述
一个 $n$ 个点，$m$ 条边的有向带权图，路径的权值为经过所有边权之和。$q$ 次询问这张图的第 $k$ 小路径权值。

数据范围：$T\le 100,\sum n,\sum m,\sum q\le 2.5\times 10^5$，保证 $k$ 合法。

### 验题过程

Idea：UESTC_Guest_WiFi，Code：HeRaNO

这是写的最后一道题，放飞自我了……优先队列直接能往里加边就加边，然后统计答案。

其实相当于是直接上去就写了个时间空间都 $\mathcal{O}(n^2)$ 的，没过还 MLE 了……

然后发现正解和我们写的差不多……

然后 cjj 说这道题是签到题%%%

![](/images/path666.png)

### 题解

我只记得 $\mathcal{O}(n^2)$ 咋写的，没那么多 `break` 就是 $\mathcal{O}(n^2)$ 的……

代码见上图……只是核心代码……（比如要对边权排序什么的……）

时间复杂度 $\mathcal{O}(k\log (m+k))$，空间复杂度 $\mathcal{O}(m+k)$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU6705.cpp)

## E. huntian oy
<!-- 命题人：moeis0 -->

### 题目描述
$T$ 组数据，求

$$
\sum_{i=1}^n\sum_{j=1}^i \gcd(i^a-j^a,i^b-j^b)[\gcd(i,j)=1]
$$

的值，对 $10^9+7$ 取模。

数据范围：$T=10^4,1\le n\le 10^9$，但只有 $10$ 组数据 $n\ge 10^6$。保证 $a,b$ 互质。

### 验题过程

Idea：UESTC_Guest_WiFi，Code：HeRaNO

验题之前就说有一道数论题用了代数数论的结论……于是就是这道题了……

ZXyang 上来先让我打了个表，然后打了之后发现很神奇。

然后我就在那里推，他去看 H 了……

然后他 A 了 H，又去看 B，我才推完，写了一发还挂样例了……

然后和 Vingying 讨论了一下发现可以化简……

然后就是一个大力原题杜教筛……

倒是拿了个 1A 的一血挺开心的。

### 题解

当然是数论上来先打表，因为 $a,b$ 互质上去先搞个 $2,3$ 上去看，发现 $j+\gcd(i^a-j^a,i^b-j^b)=i$，换了挺多组互质的 $a,b$ 发现结论都成立，然后就可以大力拆项化简了：

$$
\begin{align}
&\sum_{i=1}^n\sum_{j=1}^i \gcd(i^a-j^a,i^b-j^b)[\gcd(i,j)=1]\\
&=\sum_{i=1}^n\sum_{j=1}^i (i-j)[\gcd(i,j)=1]\\
&=\sum_{i=1}^n\sum_{j=1}^i i[\gcd(i,j)=1] - \sum_{i=1}^n\sum_{j=1}^i j[\gcd(i,j)=1]
\end{align}
$$

对于前一个求和，直接把 $i$ 提到第二个和号外即可，后面就是一个欧拉函数。

对于后一个求和，首先的想法是把它转化成 $1\sim i$ 的和减去约数和，然后式子长度暴增。最后打挂了。

然后 Vingying 大爷就打了几项去 OEIS 了一下没想到还真出来了……

实际上对于每一个 $j$，枚举这个 $j$ 被统计进答案的次数即可。

接着化简：

$$
\begin{align}
&\sum_{i=1}^n\sum_{j=1}^i \gcd(i^a-j^a,i^b-j^b)[\gcd(i,j)=1]\\
&=\sum_{i=1}^n\sum_{j=1}^i i[\gcd(i,j)=1] - \sum_{i=1}^n\sum_{j=1}^i j[\gcd(i,j)=1]\\
&=\sum_{i=1}^n i\varphi(i)-\sum_{i=1}^n (\frac{i\varphi(i)}{2}+[i=1])\\
&=\sum_{i=1}^n \frac{i\varphi(i)-1}{2}
\end{align}
$$

然后是一个经典套路 [BZOJ 4916](https://www.cnblogs.com/alphainf/p/8481032.html)，杜教筛即可。

最后写得很奇怪倒是……但是 A 了……

时间复杂度 $\mathcal{O}(n^{2/3})$，空间复杂度 $\mathcal{O}(n^{2/3})$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU6706.cpp)

## F. Shuffle Card
<!-- 命题人：QYitong -->
### 题目描述
一共有 $n$ 张牌，现在有 $m$ 次操作，每次把编号为 $s_i$ 的牌抽出来放到最上面，问牌的最后顺序。

数据范围：$1\le n,m\le 10^5$

### 验题过程

Idea：ZXyang，Code：ZXyang

全是他写的，当时我好像在看 E

### 题解

把抽牌序列倒过来输出，如果输出过了则不输出，然后把剩下没输出的按原序列顺序依次输出。

因为总是把抽出来的牌放在第一位的说……

HDU 判格式真的太屑了，每个数字后面都要有一个空格，并且不能有回车。

~~对就是和答案一样你才能过~~

Lutece 貌似没 PE 所以没验出来……

时间复杂度 $\mathcal{O}(n)$，空间复杂度 $\mathcal{O}(n)$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU6707.cpp)

## G. Windows Of CCPC
<!-- 命题人：QYitong -->

### 题目描述
一阶 CCPC 旗是题里第一个图那样，$k$ 阶 CCPC 旗是三个 $k-1$ 阶 CCPC 旗放在左上，右上和右下角，一个$k-1$ 阶 CCPC 旗 P, C 互换放在左下角。问 $k$ 阶 CCPC 旗长什么样。

数据范围：$1\le k\le 10$。

### 验题过程

Idea：ZXyang，Code：HeRaNO

开场这道题，手速慢了没抢上一血。

~~因为把 CCPC 打成 CCCP 了直接同志伏特加了……~~

### 题解

这个是唯一没啥好说的……$k$ 阶 CCPC 旗的大小是 $2^k\times 2^k$ 的，直接模拟即可。

时间复杂度 $\mathcal{O}(T4^k)$，空间复杂度 $\mathcal{O}(k4^k)$。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU6708.cpp)

## H. Fishing Master
<!-- 命题人：eom -->

### 题目描述
鱼塘里有 $n$ 条鱼，炖一条鱼至少要 $t_i$ 分钟，抓一条鱼要 $k$ 分钟，每次只能抓一条鱼，锅也只有一个。你可以存鱼，但是抓鱼的时候不能对锅进行操作（也就是不能把锅里炖好的鱼拿出来后把抓上来并存起来的鱼放锅里）。问至少需要多长时间才能把这 $n$ 条鱼全炖了。

数据范围：$1\le T\le 20,1\le n\le 10^5$。

### 验题过程

Idea：ZXyang & Vingying，Code：ZXyang

我在打 E 的时候他们在想 H，然后写了一发挂了，之后怎么样我也不知道，然后就过了……

据说是一个堆的可撤销贪心……

### 题解

不是我打的……直接上代码吧……

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU6709.cpp)

## I. Kaguya
<!-- 命题人：UESTC_Sphinx -->

### 题目描述
一张二分图，左边 $n$ 个点右边 $m$ 个点，左右任意两点之间有一条边的概率为 $\frac{1}{2}$，求左右任意两点最短距离之和的期望。若不连通则认为距离为 $0$。答案需要对 $P$ 取模。

数据范围：$1\le n,m\le 30,772001\le P\le 10^9+7$，保证 $P$ 是质数。

### 验题过程

Idea：Vingying & HeRaNO，Code：NaN

实际上 Vingying 大爷一直在那里想 I 但是没想出来……

我最后也跟着一起想，没想出来……

### 题解

直接看题解吧……后三道全是神题……

这道题是 $\mathcal{O}(n^5)$ 的 DP。

## J. Touma Kazusa's function
<!-- 命题人：Forsaken -->

### 题目描述

定义：

$$
S(l,r)=\sum_{i=l}^r\sum_{j=l}^r \varphi(\gcd(a_i,a_j))\text{lcm}(a_i,a_j)
$$

现有一长度为 $n$ 的序列 $a_i$，$q$ 次询问 $S(l,r)\bmod 2^{32}$ 的值。

数据范围：$1\le \sum n,\sum q\le 3\times 10^5$，保证 $a_i$ 是随机的。

### 验题过程

Idea：ZXyang & HeRaNO，Code：NaN

又是我俩……上来我先用唯一分解定理推出来一些东西但是不能维护，ZXyang 说接着打表，但是没啥规律，作罢。

唯一和题解一样的是肯定用莫队……

（毕竟出题人莫队与数论大神）

### 题解

真就莫队……还有随机化……

UOJ 群友 5 分钟正解 NB！

rqy 分析期望 NB！

是一个用期望分析复杂度的莫队，时间复杂度据说是 $\mathcal{O}(11n\sqrt n)$ 的。

## K. sakura
<!-- 命题人：Mstdream -->

### 题目描述

一个三维网格空间，起始时刻 $a_{i,j,k}=1$，首先在起始时刻修改 $m$ 个格子 $a_{n,x_i,y_i}$ 的值为 $v_i$，然后每个时刻，每个格子变成题干里给出的那个形式，问 $n$ 秒后 $a_{0,0,0}$ 的值对 $998244353$ 取模后的值。

数据范围：$1\le n\le 10^9,1\le \sum m\le 2\times 10^5$。

### 验题过程

Idea：HeRaNO，Code：NaN

因为和某次队内赛的题有点像，已经知道肯定是算贡献了，但是不会往下搞了……

### 题解

个人觉得出得质量比较高的一道题……（指数据）

海星……我看不懂……

算每一个修改的贡献时先用费马小定理降幂，然后一部分快速幂解决，另一部分还需要继续拆分模数变成两个卢卡斯定理和一个扩展卢卡斯定理，然后 CRT 合并，最后把每一个修改的贡献乘起来。

使用扩展卢卡斯定理时因为模数为 $2^{23}$ 可以用与运算继续优化时间。

然后就成为了卡常~~毒瘤~~题……时间复杂度 $\mathcal{O}(m\log n)$，卡 $\log^2 n$ 的。

代码在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/HDU/HDU6712.cpp)

## L. zh and zz6d（并不存在）
<!-- 命题人：zz6d -->

### 题目描述

题面过于毒瘤（但是据说做法很简单就是一个数学分析），被撤了。

具体题目我也忘了……

### 验题过程

看都没看，并且谁也没看

嘿嘿嘿

### 题解

被撤了，也没题解。
