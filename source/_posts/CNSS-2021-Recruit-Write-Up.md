---
title: CNSS 2021 Recruit Writeups
categories: 'CTF'
toc: true
date: 2021-09-28 22:53:56
tags:
	- 数学
	- 数论
---

这次真就纯玩的……

<!-- more -->

## Web

### [👶Baby] Signin

进去以后发现要换个 Method，于是 POST 过去，返回了一堆 PHP。

然后看不懂 PHP，猜它啥意思。大概意思是要 POST 里面带个 `{"CNSS": "join"}` 的 payload，然后要有个 GET 参数 `web=like`。还有一个 `setcookie` 的函数，盲猜它对于已有 cookie 不会重复写，因此再带一个 cookie `flag` 为 `1`。然后把这堆东西 POST 过去就能拿到 flag 了。

[Signin.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Web/Signin.py)

### [👶Baby]D3buger

进去发现 F12 用不了，直接进行一个 GET 把源码拿下来。在最后一行发现实际起作用的是 `/js/fuck.js` 这段代码，再去访问这个代码，发现 flag 就在最下面注释里……

其实就发一个 GET 请求就完事了……不放代码了……

### [😃Easy]Ezp#p

首先是绕双重 MD5 校验，PHP 的一个 feature 是 `==` 不会判断类型是否相等，如果一个字符串是 `0e` 开头且后面全是数字，那么这个字符串会被求值而当做 0。这道题我们要一个两次求 MD5 之后是 `0e` 开头且后面全是数字的一个字符串放到 Cookie 里。[这里](https://0clickjacking0.github.io/2020/08/24/CTF%E4%B8%AD%E5%85%B3%E4%BA%8Emd5%E7%9A%84%E4%B8%80%E4%BA%9B%E6%80%BB%E7%BB%93/)有一些爆好的。

后面的 `preg_replace` 还是比较简单的，因为它只会执行一次，所以用一个完整的串插入到这个串的某个位置就好了，比如 `fifilterlter` 这样。

这样只能获得 flag 的前一半。后面要绕到 `verify`，估计是利用对数组 MD5 未处理异常的问题把 `===` 绕过去，然后显示 `flag`。但是不会了，放弃。

UPD：上面的后一半想复杂了，是 `extract` 的问题，只需要搞一下覆盖的问题就行了。让 `verify` 的内容是 `flag2`，实际上传 MD5 就可以了……

## Reverse

### 0x01 Hello Re!

还是改后缀名搜 flag 格式 trick，但是 flag 太迷惑了，撑死胆大的。

### 0x03 有手就行

丢到 IDA 里，发现挺简单的，就是大写字母从 A 开始隔一个字母取一个，取到 Y 为止。太简单了不写代码了。

### 0x04 Py 大师第一步

Python 字节码还挺好懂的，结合最右边的符号和中间的指令，自己写出源代码，发现又是异或一下加一下就完事的那种。于是逆回去就完事了。

[pymaster.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Reverse/pymaster.py)

## Pwn

### 0x01 Net Cat

就无脑 nc 进去 cat 一下 flag 就行了……

### 0xFF roshambo

IDA 逆一下程序，大概意思是给对方剪刀石头布的出法，你要在 1 秒钟内赢它，连续赢 50 次就可以 getshell。

于是就变成了简单的 pwntools 应用题……

[roshambo.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Pwn/roshambo.py)

## Crypto

### [😋] 大大超人的代码 I

之前跑出来一堆乱七八糟的东西，跟 MMonster 说了之后是题挂了，修了。

总之这道题是求逆元，这里 $n$ 是一个质数。

[BigBigSupermanI.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Crypto/BigBigSupermanI.py)

### [😋] 大大超人的代码 II

求一个大数的欧拉函数值，考虑欧拉函数的一个求法：

$$
\varphi(x)=x\prod_{p} 1-\frac{1}{p}
$$

其中 $p$ 是 $x$ 的质因子。

然后就硬上 Pollard Rho，发现质因子不是那么大。然后发现有 20 个质因子，然后历尽千辛万苦终于求出了这个值。

然后发现 SageMath 有直接求欧拉函数的函数，调用了一下跑得飞快……但是不知道具体啥原理……

[BigBigSupermanII.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Crypto/BigBigSupermanII.py)

### [😋] EZRSA

给一个 RSA 模数 $n$，这个 $n$ 是两个 1024 位的素数的积，现在用 $e=3$ 去加密明文，现在把密文和 $n$ 给出，要求破解明文。

好像在夏令营就有这个题，但是不知道怎么做，还在想三次剩余怎么解，没想出来对这种 $n=pq$ 形式的啥好方法。然后招新的时候谷歌了一下，发现有师傅写过[教程](https://www.tr0y.wang/2017/11/06/CTFRSA/#%E4%BD%8E%E5%8A%A0%E5%AF%86%E6%8C%87%E6%95%B0%E6%94%BB%E5%87%BB)，提到了低加密指数的攻击方式是就当取模没用……哈哈。

于是就直接尝试开个三次方根，就做完了……

[EZRSA.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Crypto/EZRSA.py)

### [😋] True Random

众所周知，当确定随机种子后得到的序列是相同的。因此代码并不是真正意义上的随机的。它的 `res` 是一个类似差分异或的计算方法，因此再异或逆回去就行了。

[TrueRandom.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Crypto/TrueRandom.py)

### [😋] 基地遭到攻击

因为经典密码很不会打，所以跳了。国庆某天晚上仔细看了一下，其实是先把明文三个三个分组，每组求出一个长度为 $5$ 的三进制串，前面补个 $0$，长度变成了 $16$，然后四个四个取，每一个长度为 $4$ 的三进制串转化成一个十进制数，对应到字典表里输出。主要问题是后面还有各种补充的操作，当时直接认为这个东西对密文有影响，然后就放着了。然后看了一下除去最后四个字符发现长度是 $4$ 的倍数，于是一切都好解释了，后面补全根本没用，所以直接反着做回去就行了。

[Invade.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Crypto/Invade.py)

### [(😋+😗)/2] Smooth Criminal

要解一个 $23^x\bmod p=h$ 的方程，于是容易想到求离散对数。但是这个 $p$ 太大了，BSGS 做不了。[CTF Wiki](https://ctf-wiki.org/crypto/asymmetric/discrete-log/elgamal/#2015-mma-ctf-alicegame) 里面提到了这种加密算法要求 $p-1$ 有大质因子，然而 PR 分解一下 $p-1$，最大的质因子还没到 $100$，于是可以用 Pohlig-Hellman，这个玩意[熊板](https://github.com/forgottencsc/ICPC/blob/master/Templates/Math/%E6%95%B0%E8%AE%BA.md#pohlig-hellman)有，但是还是用 SageMath 就可以处理了，虽然有点慢。

[SmoothCriminal.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Crypto/SmoothCriminal.py)

### [😗] ECDLP

看上去是一个解椭圆曲线的问题，ECDLP 本身就是解题方法。有一道类似的题的 [WP](https://ctftime.org/writeup/29157)。

观察 $E$ 的阶，分解质因子，发现有两个巨大的质因子，剩下都比较小。因为 ECDLP 的解决方法还是利用离散对数做，直接做 PH 和 BSGS 都做不了。

然后就莫名其妙地搜到了一篇一毛一样的 [WP](https://github.com/hgarrereyn/Th3g3ntl3man-CTF-Writeups/blob/master/2017/picoCTF_2017/problems/cryptography/ECC2/ECC2.md)，看了一下意思大概是在 PH 最后的 BSGS 中，模数除以最后两个大质因子的积，得到的结果是大于明文的范围的，因此可以不考虑最后两个大质因子的影响，因为在 BSGS 的合并中没有贡献。但是题里面规定的是明文范围小于 $2^{100}$，而模数除以最后两个大质因子的积也是小于 $2^{100}$ 的……但是赌他打错了，然后就求出了正确的结果。

因为除去两个大质因子，用其余的质因子做 CRT 得到的结果是模所有质因子乘积的，再加上两个大质因子已经不会取模了，因此就这么做就可以了。因此应该保证明文数字是小于模数除以最后两个大质因子的积的。

[ECDLP.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Crypto/ECDLP.py)

### [😗] RSA I

问 MMonster 说解法是搜索剪枝。寄了，根本不会剪枝。

之前写了一版剪枝，挂服务器上跑了很长时间，跑不出来。最后就在想是不是代码写错了，然后测小数据发现果然写错了，改掉之后再跑，又跑不出来，麻了。

但是给 $p\oplus q$ 和 $p\times q$ 是可做的，Plaid CTF [出过](https://ctftime.org/task/15578)，来源应该是 Stack Overflow 的一个[提问](https://math.stackexchange.com/questions/2087588/integer-factorization-with-additional-knowledge-of-p-oplus-q)。二进制从低位到高位搜就可以了，$p\times q$ 可以只看它二进制后几位来进行剪枝，类似无符号整形的溢出。这么做状态不多，但如果把一个数的二进制反过来再异或就很不 make sense，感觉没啥性质……

P.S. 其他 XCPC 选手仍然表示不会做，与去年如出一辙，甚至还想把 PlaidCTF 那道题出成交互。

### [😗] 大大超人的代码Ⅲ

给了 $a,b,p$，问最小的 $x$，使得存在 $y\in [1,p)$，满足 $a^x\equiv b^y\pmod p$。

分解一下 $\varphi(p)$，发现最大质因子很小，PH 没跑了，所以考虑转化为一个判定问题，从 $1$ 开始枚举这个 $x$，判断是否存在 $y$ 满足上面的等式。但是依然很慢，得跑大概半个小时才能出结果。

据 NamelessOIer 说单次应该是 $\mathcal{O}(\log p)$ 的，所以上面的暴力应该不是标解。顺便据说是某道区域赛的前半部分，应该是我没做过的东西了。

UPD：用原根乱搞，我是啥b

[BigBigSupermanIII.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2021/Crypto/BigBigSupermanIII.py)

### [😯] RSA II

看上去 $e$ 很大，还是在 tr0y 师傅的[教程](https://www.tr0y.wang/2017/11/06/CTFRSA/#%E4%BD%8E%E8%A7%A3%E5%AF%86%E6%8C%87%E6%95%B0%E6%94%BB%E5%87%BB) 提到了 Wiener Attack，但是并不能用，怒而 Factor DB，发现有人把这玩意分解好了，于是一通基础操作就拿到了明文，发现并不是 flag，哈哈，又是代码。

转而分析新代码，发现用同一个 $n,e$ 加密了两份明文，而且 $e$ 很小，猜它 padding 很小，然后 Coppersmith，结果猜错了，用不了，这题就寄了。

UPD：就是 Coppersmith，我没细看，我是啥b×2。

## Misc

### 无线电报

加 TG 群点开 PIN 的信息就有了。
