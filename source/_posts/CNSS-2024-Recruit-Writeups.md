---
title: CNSS 2024 Recruit Writeups
date: 2024-10-20 09:13:15
categories: 'CTF'
toc: true
tags:
	- 数学
	- 数论
description: ' '
---

## Crypto

### 😢 雨霖铃

就把明文按 RSA 加密的形式加密了发过去就行，不用想太多。

[yulinling.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2024/Crypto/yulinling.py)

### 😊 快乐王子

RSA 解密，算一下 $d$ 就好了。因为 $n$ 是一个质数，所以 $\varphi(n)=n-1$，去年夏令营的题。

[prince.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2024/Crypto/prince.py)

### 🦆 滕王阁序

给出了前三个生成的数，可知

$$
\begin{cases}
ax_0+b=x_1 \pmod{n}\\
ax_1+b=x_2 \pmod{n}\\
\end{cases}
$$

上下相减可得 $a(x_0-x_1)=x_1-x_2\pmod {n}$，这是一个线性同余方程，可以直接求 $a$。

之后就好算了，根据一个方程求出来 $b$，然后往后推就行了。

[tengwangge.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2024/Crypto/tengwangge.py)

### 😴 苦昼短

考虑共模攻击。但是这里 $\gcd(e_1,e_2)=2$，所以找到的 $se_1+te_2$ 是 $2$。也就是 $c_1^sc_2^t=m^2\pmod{n}$。

现在想求 $m$，虽然是一个二次剩余，但是 $n$ 不是质数。感觉应该有一些别的方法，但是都不好用，选择了一种鬼畜的方法。猜它 flag 一定是 $35$ 位的，因为明文的前 $5$ 个字符一定是 `cnss{`，并且给了后 $7$ 个字符，也就是知道了 $m$ 的高 $40$ 位和低 $56$ 位。构造多项式 $(h+x+l)^2$，$h$ 是高 $40$ 位，$l$ 是低 $56$ 位，然后求 $f(x)=(h+x+l)^2-c$ 的零点，用 Coppersmith。跑得比较慢。 

[kuzhouduan.sage](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2024/Crypto/kuzhouduan.sage)

### 🐬 詩超絆

根据 OFB 算法，解密第一个块需要加密用的初始向量，但是解密第二个块只需要明文第一块和密文第一块异或一下就好。PNG 格式的前 $16$ 字节是固定的，也就是第一个块已经知道了，所以可以不用初始向量，直接异或一下当初始向量，从第二个块开始解密即可。然后记得把第一块拼回去。

[utakotoba.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2024/Crypto/utakotoba.py)

## Misc

### 我很会C++

Python 处理一下代码就行，缺了两个数。

### ❓ 璇椾汉鎻℃寔

很喜欢的一个网站 [乱码恢复](https://www.ff98sha.me/tools/messycode/)，然后恢复出来是一堆大写中文数字，人脑挨个敲回去就是了。

### 👑 我意同君共天涯

suid 提权，看一下什么可以提权，结果发现了 curl，所以用 curl 访问一下 flag 就出来了。

### 🚗 求种子……

Torrent 格式里可以直接带文件，但是用 bencode 那个 Python 库说文件格式损坏，很怪。

上网找了一个 [Bencode Online](https://chocobo1.github.io/bencode_online/) 把种子转化一下，后面就有 `pieces` 信息，里面是文件的 SHA-1 编码，然后上网找个破解工具把 27 个字符全搞出来就行了。

对不起，今年 Misc 挺有意思的，但是一直搞论文没时间做了。
