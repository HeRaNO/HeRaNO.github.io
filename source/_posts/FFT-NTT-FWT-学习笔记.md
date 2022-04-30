---
title: 'FFT, NTT, FWT 学习笔记'
categories: 'OI/ICPC'
date: 2017-05-05 23:07:06
tags:
	- FFT
	- NTT
	- FWT
toc: true
description: 前方高能！慎入！
---

## 为什么学 FFT
退役（很早）之前听说 FFT 很神（e）奇（xin），Po 姐来讲的时候也是膜（sha）了（ye）一（bu）发（dong），于是就放那里了。退役之后有（xian）了（de）时（mei）间（shi），~~并且在篮球赛之前立了赢一场的 flag 否则学 FFT 结果又双叒叕全输了~~，于是下面就是成果了……网上 FFT 讲解多得是不看也罢……

如果想要深入了解，请参考王逸松在 2018 年全国青少年信息学奥林匹克竞赛冬令营的讲课[傅里叶变换及其在 OI 中的应用](/downloads/FT-Lecture.pdf)。

## 什么是 FFT？为什么 FFT？

### 什么是 FFT？

FFT *(Fast Fourier Transform)*，全名快速傅里叶变换，这与 [Fourier 变换](https://en.wikipedia.org/wiki/Fourier_transform)**只有半毛钱的关系**，Fourier 变换是分析波的成分的方法，通过推广 Fourier 变换得到了 FFT ，为了展示区别和联系，下面是 Fourier 变换和 Fourier 逆变换的公式。

Fourier 变换：

$$
F(\omega)=\mathcal{F}[f(t)]=\int_{-\infty}^\infty f(t)e^{-i\omega t} \text{d}t
$$

Fourier 逆变换：

$$
f(t)=\mathcal{F}^{-1}[F(\omega)]=\frac{1}{2\pi} \int_{-\infty}^\infty F(\omega)e^{i\omega t} \text{d}\omega
$$

其中 $f(t)$ 为以 $t$ 为周期的周期函数，且满足 [Dirichlet 条件](https://en.wikipedia.org/wiki/Dirichlet_conditions)，$F(\omega)$ 称为 $f(t)$ 的像函数，$f(t)$ 称为 $F(\omega)$ 的原像函数。具体请参考数字信号处理相关书籍或实分析相关书籍。

### 为什么 FFT？

从一个简单问题说起：大整数乘法。在做 [Vijos P2000](https://vijos.org/p/2000) 的时候，看数据范围 $\mathcal{O}(n^2)$ 有点卡常？在做 [HDOJ1402](http://acm.hdu.edu.cn/showproblem.php?pid=1402) 的时候 $\mathcal{O}(n^2)$ 接着卡常？大整数乘法的时间复杂度只能是 $\mathcal{O}(n^2)$ 了？

并不是。从算法优化上可以实现为 $\mathcal{O}(n^{\log_2 3})$ 的方法（$\log_2 3\approx 1.585$，此方法为 Karatsuba 乘法，详解看[这里](http://web.archive.org/web/20200924035210/http://blog.miskcoo.com/2014/10/karatsuba-multiplication)），还是不太友好。那么就会用到 $\mathcal{O}(n\log n)$ 的 FFT 了。

## （初中知识）多项式

### 定义

我们把形如 $a_0+a_1x+a_2x^2+\ldots +a_{n-1}x^{n-1}$ 的式子叫做多项式，其中 $a_0,a_1,\ldots,a_{n-1}$ 称为多项式的系数，多项式中单项式个数为多项式的项数。因为多项式由单项式组成，规定最高次项的次数叫做多项式的次数，这里 $A(x)$ 的次数为 $n-1$。$x$ 为未知数。

以上内容均为初二（我记得是）的内容，不明白的自觉面壁思过（初中数学老师：这个定义抄 $10$ 遍给我）……

下面就是没学过的了。

为方便表示，我们记多项式 $a_0+a_1x+a_2x^2+\ldots +a_{n-1}x^{n-1}$ 为 $A(x)$，即：

$$
A(x)=\sum_{i=0}^{n-1}a_ix^i
$$

我们规定：严格大于 $n$ 的任意整数为这个多项式的次数界，即次数的范围。多项式中任意单项式次数均严格小于次数界。

下面无特殊说明，多项式均用 $A(x)$ 等类似形式表示。

## 多项式的两种表示法

### 系数（插值）表示法

我们知道，向量是个好东西（蛤？），矩阵也是个好东西（蛤？），于是我们把三者结合起来看（蛤玩意？）。

我们将 $n-1$ 次多项式的系数看成 $n$ 维向量，即 $\vec a=(a_0,a_1,\ldots,a_{n-1})$，则称 $\vec a$ 为多项式 $A(x)$ 的系数表示。

在选修 4-2 中，我们知道向量与矩阵有着密切联系，这就是为什么我们把多项式，向量和矩阵联系起来。（然而我们并没选 4-2 蛤蛤蛤）

于是这个 $n$ 维向量可以表示为一个 $n$ 维列向量，为：

$$
\begin{bmatrix}
a_0\\
a_1\\
\vdots \\
a_{n-1}
\end{bmatrix}
$$

### 点值表示法

我们把多项式 $A(x)$ 看做一个函数，选取不同的 $n$ 个值 $x_0,x_1,\ldots,x_{n-1}$ 代入 $A(x)$ 中，可以得到 $A(x)$ 的图像上 $n$ 个不同的点，那么称点集 $\alpha=\{(x_i,A(x_i))\mid i\in[0,n-1],i\in \mathbb{Z}\}$ 为多项式 $A(x)$ 的点值表示。

我们知道一个 $n$ 次多项式至少需要 $n+1$ 个两两不同的点才能确定。因为 $n$ 次多项式共 $n+1$ 个系数，需要 $n+1$ 个 $n+1$ 元方程组才能求解。

到这里，是不是差不多忘了要干什么了……

## 多项式的运算

### 多项式求值

对于一个多项式 $A(x)$，给出 $x$ 的值求 $A(x)$，就是多项式求值。

学过必修三吗？

学过必修三还不会？

拿出数学必修三翻到 $37$ 页你看到了什么？

秦九韶算法可以大量减少乘法和加法次数，并且把运算量简化为 $\mathcal{O}(n)$ 的。

### 多项式加法

没啥难度，直接加即可：

$$
\begin{aligned}
C(x)&=A(x)+B(x)\\
&=\sum_{i=0}^{n-1}a_ix^i+\sum_{j=0}^{n-1}b_jx^j\\
&=\sum_{i=0}^{n-1}(a_i+b_i)x^i\\
&=\sum_{i=0}^{n-1}c_ix^i
\end{aligned}
$$

这个操作可以 $\mathcal{O}(n)$ 完成。

### 多项式乘法
这就是个开括号的问题……

两个次数界为 $n_a,n_b$ 的多项式 $A(x),B(x)$ 相乘，得到的多项式 $C(x)$ 的次数界为 $n_a+n_b-1$。

我们把不足的次数用系数 $0$ 补充，变成两个齐次多项式。

考虑怎样的两项乘完后，结果的幂指数为同一个。即 $a_ix^i\times b_jx^j=c_kx^k$，$i,j$ 怎样取值才能使 $k$ 为定值。显然 $k=i+j$。

那么乘完之后，$C(x)$ 就可以知道了。

$$
\begin{aligned}
C(x)&=\sum_{i=0}^{2n-2}(\sum_{j=0}^ia_jb_{i-j})x^i\\
&=\sum_{i=0}^{2n-2}c_ix^i
\end{aligned}
$$

我们把中间的那个加法，即 $c_i=\sum_{j=0}^ia_jb_{i-j}$ 叫做卷积。

这样，朴素算法时间复杂度为 $\mathcal{O}(n^2)$，显然不是太好。

但是点值表达式的表示就很好了，直接乘起来就好了嘛！

设两个多项式的次数界均为 $n$，则它们相乘得到的多项式的次数界即为 $2n-1$，因此需要在每个多项式上取 $2n-1$ 个点才能确定乘出来的多项式。

设 $A(x)$ 的点值表示为 $\alpha=\{(x_i,A(x_i))\mid i\in[0,2n-2],i\in \mathbb{Z}\}$，设 $B(x)$ 的点值表示为 $\beta=\{(x_i,B(x_i))\mid i\in[0,2n-2],i\in \mathbb{Z}\}$，然后就能得到 $C(x)$ 的点值表示为：$\gamma=\{(x_i,A(x_i)B(x_i))\mid i\in[0,2n-2],i\in \mathbb{Z}\}$。

### 插值 - 点值转换
就要接近重点了同志们加油！

先定义两个运算：

1. 定义一种运算，可以将系数表示转化为点值表示，记为 $\text{DFT}(A(x))=\alpha$；
2. 定义其逆运算，即从一个多项式的点值表示确定其系数表示为插值。记为 $\text{DFT}^{-1}(\alpha)=\text{IDFT}(\alpha)=A(x)$。

第一个运算在 $x_i$ 确定时唯一确定，那么第二个在 $x_i$ 确定时可不可以唯一确定 $A(x)$ 呢？

答案是肯定的。我们把这些点代入，可以得出一个 $n$ 元一次方程组，当 $x_i$ 均不相同时，可以唯一确定 $a_0,a_1,\ldots ,a_n$。（如果有一个 $x_i$ 相同立刻变成不定方程）

~~当然可以拿矩阵证明网上多得是。~~

那么就很尴尬，$\text{DFT}(A(x))$ 的复杂度是 $\mathcal{O}(n^2)$ 的，它的逆运算是 $\mathcal{O}(n^3)$ 的（高斯消元）……还不如不优化了。

所以我们还需要一些姿势。

## （高中知识）复数
一些奇奇怪怪的问题无法解决，可以把它从实数域扩展到复数域上解决，加入了复数这种类似向量的东西，问题就会变得简单。

噫，向量？点值表示不就是个向量吗！

### 定义

可以参考 [复数 - OI Wiki](https://oi-wiki.org/math/complex/)。

一堆定义就不说了，选修 2-2 内容，这里~~简单的~~强调一些运算：

设 $i^2=-1$。

设 $z_1,z_2\in \mathbb{C}$，$z_1=a+bi,z_2=c+di$，则：

$$
\begin{align}
z_1\pm z_2&=(a\pm c)+(b\pm d)i\\
z_1\times z_2&=(ac-bd)+(ad+bc)i\\
\frac{z_1}{z_2}&=\frac{ac+bd}{c^2+d^2}+\frac{bc-ad}{c^2+d^2}i
\end{align}
$$

设 $z\in \mathbb{C}$，$z=a+bi$，则 $z$ 的共轭复数为 $\overline z=a-bi$。

这是学过的，下面就说说没学过的。

### 单位根
根？方程的根？什么方程的根？

定义：$x^n=1$ 的所有根叫做 $n$ 次单位根。

当我们把问题引入复数域，$x$ 就不能简单计算为 $x=\sqrt[n] 1=1$ 了，而是在复平面内解决问题。

### 欧拉公式
一个很强的公式。内容是：

$$
e^{ix}=\cos x+i\sin x
$$

式子很简单，证明如下：

由 Taylor 公式：

$$
\begin{aligned}
e^x&=1+x+\frac{x^2}{2!}+\ldots +\frac{x^n}{n!}+\ldots \\
\cos x&=1-\frac{x^2}{2!}+\frac{x^4}{4!}-\frac{x^6}{6!}+\ldots \\
\sin x&=x-\frac{x^3}{3!}+\frac{x^5}{5!}-\frac{x^7}{7!}+\ldots \\
\end{aligned}
$$

将 Taylor 公式推广到复数域上，有复数运算 $\pm i=\pm i,(\pm i)^2=-1,(\pm i)^3=\mp i,(\pm i)^4=1$，可得：

$$
\begin{aligned}
e^{ix}&=1+ix-\frac{x^2}{2!}-i\frac{x^3}{3!}+\frac{x^4}{4!}+i\frac{x^5}{5!}+\ldots \\
&=(1-\frac{x^2}{2!}+\frac{x^4}{4!}-\ldots)+i(x-\frac{x^3}{3!}+\frac{x^5}{5!}-\ldots)\\
&=\cos x+i\sin x
\end{aligned}
$$

证毕。

将 $x$ 换成 $\pi$，就出现了 $e^{\pi i}=-1$，即 $e^{\pi i}+1=0$，被誉为上帝创造的公式。

将 $x$ 换成 $2k\pi\ \ (k\in \mathbb{Z})$，就出现了 $e^{2k\pi i}=1$。

我们不是要解 $x^n=1$ 吗！

那么 $x=e^{\frac{2k\pi i}{n}}\ (k \in \mathbb{Z})$，记这些单位根为 $\omega_n^k=e^{\frac{2k\pi i}{n}}$。

下面就是一些数论问题了……

记 $k=nr+p$，其中 $p\in[0,n-1]$，则 $\omega_n^k=e^{\frac{2p\pi i}{n}}$。

$$ 
\begin{aligned}
\omega_n^k&=e^{\frac{2k\pi i}{n}}\\
&=e^{\frac{2(nr+p)\pi i}{n}}\\
&=e^{\frac{2nr\pi i}{n}+\frac{2p\pi i}{n}}\\
&=e^{\frac{2nr\pi i}{n}}\times e^{\frac{2p\pi i}{n}}\\
&=e^{2\pi ir}\times e^{\frac{2p\pi i}{n}}\\
&=1^r\times e^{\frac{2p\pi i}{n}}=e^{\frac{2p\pi i}{n}}
\end{aligned}
$$

这不一样吗……

所以可以证明 $k\in [0,n-1]$。也就是 $n$ 次单位根有 $n$ 个，不难发现，这 $n$ 个单位根就是单位圆上的 $n$ 个 $n$ 等分点。

于是记这些根为 $\omega_n^0,\omega_n^1,\ldots ,\omega_n^{n-1}$，这些根各不相同，并且在乘法意义下构成一个群。即：$\omega_n^i\omega_n^j=\omega_n^{i+j}=\omega_n^{(i+j)\bmod n}$，且 $\omega_n^{-1}=\omega_n^{n-1}$，证明如下：

$$
\begin{aligned}
\omega_n^{-1}&=1\times \omega_n^{-1}\\
&=e^{2\pi i}\times e^{\frac{-2\pi i}{n}}\\
&=e^{2\pi i-\frac{2\pi i}{n}}\\
&=e^{\frac{2(n-1)\pi i}{n}}=\omega_n^{n-1}
\end{aligned}
$$

问题得证。

### 三个引理
#### 消去引理
内容：当 $d\not =0$ 时，$\omega_{dn}^{dk}=\omega_n^k$。

证明：

$$
\begin{aligned}
\omega_{dn}^{dk}&=e^{\frac{2dk\pi i}{dn}}\\
&=e^{\frac{2k\pi i}{n}}=\omega_n^k
\end{aligned}
$$

很简单……~~也很智障~~

#### 折半引理
内容：如果 $n>0$ 且 $n$ 为偶数，$(\omega_n^k)^2=\omega_{\frac{n}{2}}^k$。

证明：

$$
\begin{aligned}
(\omega_{n}^{k})^2&=(e^{\frac{2k\pi i}{n}})^2\\
&=e^{\frac{2k\pi i}{\frac{n}{2}}}=\omega_{\frac{n}{2}}^k
\end{aligned}
$$

还行吧……可以看出，$n$ 个 $n$ 次单位根平方的集合就是 $\frac{n}{2}$ 个 $\frac{n}{2}$ 次单位根的集合，并且每个元素出现两次。可以通过单位根的对称性来理解。

更一般的结论：

$$
\begin{align}
\omega_n^{k+\frac{n}{2}}&=\omega_n^{\frac{n}{2}}\omega_n^k=-\omega_n^k\\
\omega_n^{\frac{n}{2}}&=e^{\pi i}=-1
\end{align}
$$

#### 求和引理
内容：对于 $\forall k\in(0,n)$，有 $\sum_{j=0}^{n-1}(\omega_n^k)^j=0$。

证明：

$$
\begin{aligned}
\sum_{j=0}^{n-1}(\omega_n^k)^j
&=\frac{1-(\omega_{n}^{k})^n}{1-\omega_{n}^{k}}\\
&=\frac{1-(\omega_{n}^{n})^k}{1-\omega_{n}^{k}}=0
\end{aligned}
$$

注意：$\omega_n^n=1$，第一步用到了等比数列求和，化简省略了部分~~很多~~步骤。

现在全都就绪了，终于要到目标了……

## 离散傅里叶变换（DFT）
DFT，即 Discrete Fourier Transform，离散 Fourier 变换，就是我们之前定义的那个，现在有了复数，我们就可以在 $\mathcal{O}(n\log n)$ 的时间内做完 DFT 运算。运用到了 Cooley-Tukey 算法。

我们想在尽可能快的时间里获得 $A(x)$ 的点值表示。并且为了满足之后的操作，我们需要在二倍的次数界下完成这个操作，也就是我们需要代入 $2n-1$ 个点。每次多项式求值的复杂度是 $\mathcal{O}(n)$ 的，因此朴素算法的复杂度为 $\mathcal{O}(n^2)$。

我们**将多项式按照指数的奇偶分类**，记原多项式为 $A(x)$，那么构造两个多项式：

$$
\begin{aligned}
A_0(x)&=a_0+a_2x+a_4x^2+\ldots +a_{n-2}x^{\frac{n}{2}-1}\\
&=\sum_{i=0}^{\frac{n}{2}-1}a_{2i}x^i\\
A_1(x)&=a_1+a_3x+a_5x^2+\ldots +a_{n-1}x^{\frac{n}{2}-1}\\
&=\sum_{i=0}^{\frac{n}{2}-1}a_{2i+1}x^i\\
\end{aligned}
$$

通过观察可知，$A(x)=A_0(x^2)+xA_1(x^2)$，根据折半引理，我们考虑将 $n$ 次单位根作为 $x$ 代入计算。又因为折半引理，我们即可分治操作。

那么，对于 $k<\frac{n}{2}$，有：

$$
\begin{aligned}
A(\omega_n^k)&=A_0((\omega_n^k)^2)+\omega_n^kA_1((\omega_n^k)^2)\\
&=A_0(\omega_{\frac{n}{2}}^k)+\omega_n^kA_1(\omega_{\frac{n}{2}}^k)\\
&=\sum_{i=0}^{\frac{n}{2}-1}a_{2i}\omega_{\frac{n}{2}}^{ki}+\omega_n^k\sum_{i=0}^{\frac{n}{2}-1}a_{2i+1}\omega_{\frac{n}{2}}^{ki}\\
A(\omega_n^{k+\frac{n}{2}})&=A_0((\omega_n^k)^2)+\omega_n^{k+\frac{n}{2}}A_1((\omega_n^k)^2)\\
&=A_0(\omega_{\frac{n}{2}}^k)-\omega_n^kA_1(\omega_{\frac{n}{2}}^k)\\
&=\sum_{i=0}^{\frac{n}{2}-1}a_{2i}\omega_{\frac{n}{2}}^{ki}-\omega_n^k\sum_{i=0}^{\frac{n}{2}-1}a_{2i+1}\omega_{\frac{n}{2}}^{ki}
\end{aligned}
$$

这样，由奇偶性使得需要代入的值范围减半，递归做下去就行了。

时间复杂度为：

$$
\mathcal{T}(n)=2\mathcal{T}(\frac{n}{2})+\mathcal{O}(n)=\mathcal{O}(n\log n)
$$

因此，对于单位根的多项式求值，其实是均摊 $\mathcal{O}(\log n)$ 的。

因为永远是奇偶合并，如果想要更方便地合并必然 $n$ 取 $2$ 的整数次幂，否则合并到一定时候左右不一定都是相同次数（可以考虑一个完全二叉树）。所以做的时候取 $2$ 的整数次幂作为 $n$。

## 逆离散傅里叶变换（IDFT）
IDFT，即 Inverse Discrete Fourier Transform，逆离散傅里叶变换，还是我们之前定义的那个。我们在 $\mathcal{O}(n\log n)$ 的时间里搞定了 DFT，可是 IDFT 还是 $\mathcal{O}(n^3)$ 的，所以我们继续观（tui）察（da）性（shi）质（zi）。

我们说过，IDFT 相当于解个 $n$ 元一次方程组，这个方程组写成矩阵形式是这样的：

$$
\begin{bmatrix}
(\omega_n^0)^0& (\omega_n^0)^1&(\omega_n^0)^2 &\ldots &(\omega_n^0)^{n-1}\\
(\omega_n^1)^0& (\omega_n^1)^1&(\omega_n^1)^2&\ldots &(\omega_n^1)^{n-1}\\
\vdots&\vdots &\vdots &\ddots &\vdots \\
(\omega_n^{n-1})^0 & (\omega_n^{n-1})^1& (\omega_n^{n-1})^2&\ldots & (\omega_n^{n-1})^{n-1}
\end{bmatrix}
\times
\begin{bmatrix}
a_0\\a_1\\ \vdots \\a_{n-1}
\end{bmatrix}=
\begin{bmatrix}
A(\omega_n^0)\\
A(\omega_n^1)\\
A(\omega_n^2)\\
\vdots \\
A(\omega_n^{n-1})\\
\end{bmatrix}
$$

这同时也是做 DFT 时我们利用的矩阵。只不过做 DFT 时右边是未知的，这次 $a_i$ 是未知的。

记上面的系数矩阵为 $D$，再考虑以下矩阵：

$$
\begin{bmatrix}
(\omega_n^0)^0& (\omega_n^0)^1&(\omega_n^0)^2 &\ldots &(\omega_n^0)^{n-1}\\
(\omega_n^{-1})^0& (\omega_n^{-1})^1&(\omega_n^{-1})^2&\ldots &(\omega_n^{-1})^{n-1}\\
\vdots&\vdots &\vdots &\ddots &\vdots \\
(\omega_n^{-(n-1)})^0 & (\omega_n^{-(n-1)})^1& (\omega_n^{-(n-1)})^2&\ldots & (\omega_n^{-(n-1)})^{n-1}
\end{bmatrix}
$$

记此矩阵为 $W$，则记 $E=D\times W$。可知：

$$
\begin{aligned}
E_{ij}&=\sum_{k=0}^{n-1}D_{ik}W_{kj}\\
&=\sum_{k=0}^{n-1}\omega_n^{-ik}\omega_n^{kj}\\
&=\sum_{k=0}^{n-1}\omega_n^{k(j-i)}
\end{aligned}
$$

当 $i=j$ 时：

$$
\begin{aligned}
E_{ij}&=\sum_{k=0}^{n-1}\omega_n^{k(j-i)}\\
&=\sum_{k=0}^{n-1}\omega_n^0=n\\
\end{aligned}
$$

当 $i\not =j$ 时，由求和引理：

$$
\begin{aligned}
E_{ij}&=\sum_{k=0}^{n-1}\omega_n^{k(j-i)}\\
&=\sum_{k=0}^{n-1}(\omega_n^{j-i})^k=0\\
\end{aligned}
$$

记单位矩阵为 $I$，所以 $I=\frac{1}{n}E$，即 $\frac{1}{n}D=W^{-1}$。
将由方程组推得那个矩阵左乘 $\frac{1}{n}D$，可得：

$$
\begin{bmatrix}
a_0\\a_1\\ \vdots \\a_{n-1}
\end{bmatrix}=
\begin{bmatrix}
(\omega_n^0)^0& (\omega_n^0)^1&(\omega_n^0)^2 &\ldots &(\omega_n^0)^{n-1}\\
(\omega_n^{-1})^0& (\omega_n^{-1})^1&(\omega_n^{-1})^2&\ldots &(\omega_n^{-1})^{n-1}\\
\vdots&\vdots &\vdots &\ddots &\vdots \\
(\omega_n^{-(n-1)})^0 & (\omega_n^{-(n-1)})^1& (\omega_n^{-(n-1)})^2&\ldots & (\omega_n^{-(n-1)})^{n-1}
\end{bmatrix}
\times
\begin{bmatrix}
A(\omega_n^0)\\
A(\omega_n^1)\\
A(\omega_n^2)\\
\vdots \\
A(\omega_n^{n-1})\\
\end{bmatrix}
$$

这和 DFT 的操作很像，所以用 DFT 的过程实现 IDFT 即可。

负号？把 $\omega_n^1$ 改成 $\omega_n^{-1}$ 即可；
$\frac{1}{n}$？做完之后再除个 $n$ 即可。

现在 IDFT 也是 $\mathcal{O}(n\log n)$ 的了。

至此，所有的时间都是 $\mathcal{O}(n\log n)$ 的了！即 FFT 的复杂度为 $\mathcal{O}(n\log n)$！

## NTT
我们学过了 FFT，发现很滋磁。但是 FFT 涉及浮点数，有可能有精度问题，所以这就很闹心了……发现精度问题出现在 DFT 和 IDFT 中求 $\sin$ 和 $\cos$ 导致精度问题和 IDFT 中乘以 $\frac{1}{n}$ 的精度问题，所以我们考虑能否在整数范围内实现 FFT 的过程。

那么整数范围内的问题需要数论解决，这就导致了 NTT（也叫 FNT）的诞生。

可以说，精度问题主要由 $n$ 次单位根引起。回顾 FFT，我们用了 $n$ 次单位根，也就是 $\omega_n^k$ 的哪些性质？

好像也就是三个引理吧……
列出来所用的性质们：（一些条件有省略，具体请详见 FFT 学习笔记）

1. $\omega_n^n=e^{2k\pi i}=1$；
2. $\forall i,j\in [0,n-1]$ 且 $i,j\in \mathbb{Z},i\not =j$，$\omega_n^i\not =\omega_n^j$；
3. （折半引理）$(\omega_n^k)^2=\omega_{\frac{n}{2}}^k$，$\omega_n^{k+\frac{n}{2}}=-\omega_n^k$；
4. （求和引理）$\sum_{j=0}^{n-1}(\omega_n^k)^j=0 \ (k\not =0)$。

所以，我们需要在整数范围内找到一个符合这四条性质的一种数。于是我们找到了一种特殊的数——原根。

定义如果所有与 $n$ 互质的数 $a$ 等于 $g^x\bmod n$，其中 $x$ 为一正整数，则 $g$ 为在模 $n$ 意义下的原根。

为了方便计算，我们取模数为一个质数 $P$。

然后，我们用 $g^{r\frac{\varphi (P)}{n}}$ 代替 $\omega_n^r$ 进行计算，由 $\varphi (P)=P-1$，要求 $\frac{\varphi (P)}{n}$ 为整数，$n $ 还是 $2$ 的整数次幂，所以要求 $P=k\times 2^q+1$，其中 $2^q\ge n$。

然后我们验证一下性质是否符合。

对于第一条，由 Fermat's little theorem，$g^{\varphi (P)}\equiv 1\ (\bmod P\ )$，因此第一条满足。

对于第二条，根据第一条可以看出，原根同单位根一样，构成了一个乘法群。具体可以用第一条证明。因此第二条也是满足的。

对于第三条，化简一下指数，拆成两项相乘的形式，然后接着用第一条，还是可以得证。

对于第四条，运用等比数列求和，然后也可以得证。

如果模数任意呢？

如果要模数是 $P$，那么多项式的所有系数和不会超过 $n(P-1)^2$。所以找出一组 $p$，使得 $p_i=k_i\times 2^{q_i}+1$，且：

$$
\prod_{i=0}^{k-1}p_i>n(P-1)^2
$$

然后按每个 $p $ 做 NTT，然后 CRT 合并……

又双叒叕 CRT 合并……

本身 NTT 常数就比较大，和 FFT 差不多吧……（中间有快速幂）。

Po 姐：「此外，一个答案模一个数的计数问题必须用 NTT。」

[这里](http://web.archive.org/web/20201214071224/http://blog.miskcoo.com/2014/07/fft-prime-table)是一些常用质数的原根。

## FWT
FWT 是一种类似于 FFT 的变换。我们都知道对于数组 $a$ 与数组 $b $ 的卷积 $c$ 可以表示为：

$$
c_j=\sum_{i=1}^ja_ib_{j-i}
$$

那么 FWT 只是把计算下标的加减运算改为了逻辑运算（与、或、异或等）。

比如与运算下的 FWT 表示为：

$$
c_k=\sum_{i\ \texttt{AND}\ j=k}a_ib_j
$$

那么怎么做……

以下三位大爷写得已经足够好就不造轮子了……

[Fast Walsh-Hadamard Transform - Picks's Blog](http://picks.logdown.com/posts/179290-fast-walsh-hadamard-transform)  
[FWT 详解 知识点 - neither_nor](https://blog.csdn.net/neither_nor/article/details/60335099)  
[HDU 5909 Tree Cutting 树形 DP+快速沃尔什变换 - PoPoQQQ](https://blog.csdn.net/popoqqq/article/details/52816762)


膜拜以上三位大爷%%%


## 实现问题
说是一回事，写是另一回事。

奇偶分组这个玩意就很坑，容易发现，设 $r_i$ 为将 $i$ 的高位翻转到低位得到的数字（如 $0011$ 就变成 $1100$），那么初始位置为 $i$ 的数就会被移动到 $r_i$，所以我们可以预先完成移动，然后合并计算。时间就会加快很多了。

不过 FFT 本身常数巨大，主要因为 `double` 类型的计算时间，还有使用 $\sin,\cos$ 函数带来的计算时间问题，所以常数问题一定要注意。

我们看到，DFT 做完后，得到的是将 $n$ 次单位根代入后表达式的点值表示，IDFT 做完后，得到的是两多项式相乘的系数表示。

FFT 和 FWT 都在图像处理，音频处理方面有较大应用。

终于把我的 flag 填完了我好开心啊！！！
