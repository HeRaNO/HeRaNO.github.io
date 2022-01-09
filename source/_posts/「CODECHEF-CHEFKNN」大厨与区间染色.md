---
title: 「CODECHEF CHEFKNN」大厨与区间染色
categories: 'OI/ICPC'
toc: true
date: 2019-03-26 18:33:50
tags:
	- NTT
	- 数学
	- 多项式相关
---

这篇文章为题解翻译，原题解详见[这里](https://discuss.codechef.com/questions/123905/chefknn-editorial)。

<!-- more -->

这道题被 Lutece 搬运了，但是 Lutece 无法对外访问……

原题面在[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/CodeChef/Problems/CHEFKNN.pdf)。

## 难度

困难

## 前置技能

数论变换（NTT），组合数学

## 问题

给出一个长为 $N$ 的数组，最初，每一个数组中的元素都用颜色 $0$ 染色。给你 $K$ 次机会。在第 $i$ 次机会你可以把任何一个非空子数组全部染成颜色 $i$，这个操作可以覆盖任何已经存在的颜色。你需要回答最后的数组有多少种不同的颜色，答案对 $163577857$ 取模。

## 题解

让我们尝试找到 $F(K,N)$ 的表达式。此处 $F(K,N)$ 代表对于我们的问题的答案。

为了转化这个问题，我们遍历最后可见的颜色数。我们设最后数组有 $r$ 种可见的颜色。那么，从 $K$ 种给定颜色中选取 $r$ 种的方案数就是 $\binom {K-1}{r-1}$，这是由于你只需要从 $K-1$ 种颜色中选择 $r-1$ 种，因为第 $K$ 次涂的颜色永远是可见的。

这样，我们能写出如下公式：

$$
F(K,N) = \sum_{r=1}^{K} \binom {K-1}{r-1} e(r,N)
$$

此处，$e(r,N)$ 表示在最后长度为 $N$ 的数组中有 $r$ 种颜色可见的方案数。

### 找到 $e(r,N)$ 的表达式

让我们逆向考虑。设第 $r$ 种颜色占据了长为 $x_1$ 的区间。做到同样效果的方案数为 $N-x_1+1$。现在设第 $r-1$ 种颜色在剩下的 $N-x_1$ 长度中占据了长为 $x_2$ 的区间。需要注意 $x_1$ 不会影响 $x_2$。一般地，有如下公式：

$$
e(r,N) = \sum_{x_1=1}^{N-r+1} (N-x_1+1) \times e(r-1,N-x_1)
$$

尝试展开这个公式，可以写成如下形式：

$$
e(r,N) = \sum_{x_1=1} \sum_{x_2=1}\cdots \sum_{x_r=1} (N-x_1+1)(N-x_1-x_2+1)\cdots (N-x_1-x_2-\cdots -x_r+1)
$$

如果你仔细观察，你会发现这是从集合 $\{1,2,3,\cdots, n\}$ 中取出 $r$ 个数的乘积的和。因此，我们可以判定 $e(r,N)$ 是多项式 $P_N(x)$ 中 $x^{n-r}$ 项的系数。并且 $P_N(x)$ 如下：

$$
P_N(x) = (x+1)(x+2)(x+3)\cdots (x+N)
$$

我们现在剩下的就只是计算多项式 $P_N(x)$ 然后我们就做完了。

（OI 选手：woc 这不就是个 sb Stirling 数吗水了水了）

### 计算 $P_N(x)$

#### 子任务 1 和子任务 2

我们可以暴力计算每一项，可以在 $\mathcal{O}(n^2)$ 的复杂度下通过这两个子任务。

#### 子任务 3

这个子任务能用 NTT 和分治法在 $\mathcal{O}(n\log ^2 n)$ 的时间复杂度下通过。

目前的任务是计算 $n$ 次形如 $(x+i)$ 的多项式乘积。设 $A(x)$ 为前一半 $(x+i)$ 的乘积，$B(x)$ 为剩下部分的乘积。我们递归计算 $A(x)$ 和 $B(x)$，然后用 NTT 计算 $A(x)\cdot B(x)$。这样时间复杂度为：

$$
\mathcal{T}(n)=2\mathcal{T}(\frac{n}{2})+\mathcal{O}(n\log n)
$$

因此 $\mathcal{T}(n)=\mathcal{O}(n\log ^2 n)$，这个想法由下面的伪代码表述：

```plain
MUL(L,R) // computes (x+L)(x+L+1)...(x+R)
  if (L==R) then return (x+L)
  mid = (L+R)/2
  A = MUL(L,mid)
  B = MUL(mid+1,R)
  return A*B //A*B is computed by FFT
```

### 子任务 4

我们将会提出一个 $\mathcal{O}(n\log n)$ 的解法，因为 $\mathcal{O}(n\log^2 n)$ 的解法会因为严格的时间限制而无法通过。

#### 非正式的解法

让我们仔细观察这个分治算法的时间复杂度：

$$
\mathcal{T}(n)=\mathcal{T}(\text{left}_{\text{half}})+\mathcal{T}(\text{right}_{\text{half}})+\mathcal{O}(\text{merge})
$$

因为这里合并过程的复杂度为 $n\log n$，$\mathcal{T}(n)$ 最后的结果就是 $\mathcal{O}(n\log ^2 n)$。但是如果我们能够用一些方法，使得在 $\mathcal{O}(\text{merge})$ 的时间内能用左半部分计算出右半部分的答案就好了。这样递归关系将为：

$$
\begin{align}
\mathcal{T}(n)&=\mathcal{T}(\text{left}_{\text{half}})+\mathcal{O}(\text{merge})+\mathcal{O}(\text{merge})\\
&\Rightarrow \mathcal{T}(n)=\mathcal{T}(\frac{n}{2})+\mathcal{O}(n\log n)
\end{align}
$$

所以最后的结果为 $\mathcal{T}(n)=\mathcal{O}(n\log n)$，但是有些常数因子。所以我们只剩下如何用左半部分计算出右半部分的答案了。

#### 正式解法

我们可以用这个特征计算：

$$
P_{2N}(x)=P_N(x)\times P_N(x+N)
$$

主要问题是，当知道了 $P_N(x)$ 已知时，需要找到 $P_N(x+N)$ 的表达式。定义：

$$
P_N(x) = \prod_{i=1}^N (x+i) = \sum_{i=0}^N c_ix^i
$$

这里的 $c_i$ 均已知。然后，

$$
\begin{align}
&P_N(x+N) = \prod_{i=1}^N (x+N+i) = \sum_{i=0}^N c_i(x+N)^i\\
\Rightarrow &P_N(x+N) = \sum_{i=0}^N h_ix^i
\end{align}
$$

这里，

$$
\begin{align}
h_i &= \sum_{j=i}^N c_j  \binom{j}{i}  N^{j-i}\\
\Rightarrow h_i &= \frac{1}{i!} \sum_{j=i}^N (c_j j!)\left (\frac{n^{j-i}}{(j-i)!} \right )\\
\Rightarrow h_i &= \frac{1}{i!} (\text{coefficient of}\  x^{N-i} \ \text{in}\  A(x)B(x))
\end{align}
$$

这里，

$$
\begin{align}
A(x) &= \sum_{i=0}^{N} (c_{N-i}(N-i)!)  x^i\\
B(x) &= \sum_{i=0}^{N} \left( \frac {N^i}{i!} \right)  x^i
\end{align}
$$

现在我们就有了一个当 $P_N(x)$ 已知时，利用 $A(x)$ 与 $B(x)$ 的卷积在 $\mathcal{O}(n\log n)$ 的时间内找到 $P_N(x+N)$ 的表达式的算法。现在我们看一下解决这个子任务的伪代码：

```plain
MUL(N) // computes (x+1)(x+2)...(x+N) in O(NlogN)
  if N==1: 
    return (x+1)
  C = MUL(N/2)
  H = convolute(A,B) // use C to obtain A
  ANS = convolute(C,H)
  if N is odd:
    ANS *= (x+N) // naive multiplication will do - O(N)
  return ANS
```

最后一件事是如何计算两个多项式的乘积对 $163577857$ 取模。注意到 $163577857 = 39\times 2^{22} + 1$，这个数是 NTT 友好的。这就减轻了我们的负担。对于不了解 NTT 的人来说可以参考[这个链接](http://www.apfloat.org/ntt.html)。

### 优化

当你暴力模拟这个算法的时候，你会发现处理 $N=10^6$ 的数据花费时间过多。不幸的是，大 $\mathcal{O}$ 记号里隐藏的常数因子太高了！这个时间复杂度是正确的，所以我们需要附加的技巧与消除这部分常数并且能够把这个解法卡到时限里去。这里有一些优化方法：

- 尝试减少取模操作；
- 用数组代替 `vector` 来做 NTT，简单来说，找个跑得快的 NTT 板子；
- 内存使用：不要用多余的内存，因为这会减慢运行速度。用 130M 内存和用 70M 内存时间相差 0.5 秒以上（当然也取决于其他参数）；
- 除此之外，如果还是卡不进去，你可能还需要变换数据类型等其他优化方法。

## 其他解法

这个解法出乎意料，然而，有 $40\%$ 的选手用了这个技巧。这个想法是子任务 3 的解法通过子任务 4。一个暴力的子任务 3 的实现时间复杂度是 $\mathcal{O}(TN\log^2 N)$。如果我们能够消除复杂度里的 $T$，然后加一些优化，我们同样能够通过子任务 4。

如何消除这个 $T$ 呢？我们设不同的测试点的 $N$ 值为 $N_1,N_2,\cdots, N_T$。我们一次性输入这些数，然后按 $N$ 的升序排序。设 $n_i$ 是排序后的序列 $N$。我们可以调用 $\texttt{MUL}(1,n_1)$，$\texttt{MUL}(n_1,n_2)$，……，$\texttt{MUL}(n_{T-1}+1,n_T)$。现在，我们可以用前缀积计算这些项，并且求出所有测试点的答案。这个的时间复杂度可以近似用下面的求和计算：

$$
\mathcal{O}(N_1\log^2N) + \mathcal{O}((N_2-N_1)\log^2N) + \cdots + \mathcal{O}((N_T-N_{T-1})\log^2N) = \mathcal{O}(N\log^2N)
$$

时间复杂度：$\mathcal{O}(TN\log N)$ 或 $\mathcal{O}(N\log^2 N)$。

[出题人解法](http://www.codechef.com/download/Solutions/MARCH18/Setter/CHEFKNN.cpp)，打不开见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/CodeChef/March%20Challenge%202018/CHEFKNNA.cpp)。

[验题人解法](http://www.codechef.com/download/Solutions/MARCH18/Tester/CHEFKNN.cpp)，打不开见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/CodeChef/March%20Challenge%202018/CHEFKNNB.cpp)。

[原题解作者解法](http://www.codechef.com/download/Solutions/MARCH18/Editorialist/CHEFKNN.cpp)，打不开见[这里](https://github.com/HeRaNO/OI-ICPC-Codes/blob/master/CodeChef/March%20Challenge%202018/CHEFKNNC.cpp)。

由于只是题解搬运，就不给出我写的了（意思是我不写了）……
