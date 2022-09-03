---
title: CNSS 2020 Recruit Writeups
categories: 'CTF'
toc: true
date: 2020-10-14 11:18:35
tags:
	- Golang
	- 数学
	- 数论
---

Score: 12xxx, Rank: 9

<!-- more -->

老年人闲得没事去凝聚招新赛玩，结果被捶个半死。

以下代码并不全是能获得 flag 的，只说方法。

## Web

### (baby)更简单的计算题

直接 F12，发现输入框限制了 `maxlength="5"`，提交按钮被 disable 了，直接改源码把这些限制去掉，然后就能提交获得 flag 了。

### (baby)第一次冒险

打开网页，发现要 `GET` 一个 `answer`。

于是用 Python 的 `requests` 搞，`GET` 一个 `{'answer': 'yes'}`，发现还要换个网页 `POST` 一个 `answer`，于是按提示再 `POST` 一个 `{'answer': 'yes'}` 发现可以拿 flag 了。

flag 藏在响应头的 Cookie 里，Cookie 里面是一段 Base64 编码，后面等号用 `%3D` 转义了，于是 decode 之即为 flag。

然后 Web 剩下的都不会了……

## Reverse

### Baby C Code

flag 是 `code` 数组中所有元素的取反，稍微分析一下源码即可。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Reverse/babyc.cpp)

### baby_pyc

先用 uncompyle6 反编译一下，发现 `s` 是明文的 Base64 码，先解码 Base64 得到明文为 `cnss{flag}`，注意真正的 flag 是这段明文的 MD5。

发现这个文件里其实运行会挂，还要自己引入一下库再跑。加个 `hashlib` 就行了。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Reverse/baby_pyc.py)

### babyre

小学二年级的时候，我们就学习过用记事本打开可执行文件然后用搜索字符串的方式获取 flag。

把后缀名改成 `.txt`，按照 flag 格式搜索 `cnss` 即可。

### cppRe

小学二年级的时候，我们就学习过用记事本打开可执行文件然后用搜索字符串的方式获取 flag。结果就发现只有一个 `the flag is : cnss{your input}`。

怒而 IDA 反编译，这份可执行文件是 64 位的，结果发现全是 vector 状物。

继续分析有无常量，还真有那么个常量 `sub_15464`，一共 32 位，然后分析 `sub_55477`，它会把 `((a3 + 10) ^ 0x6F) - a4` 的值扔进一个 vector，然后如果这个 vector 内部所有元素均为 0，就说明输入是 flag。

a3 传的是 v4，a4 传的是 v3，v3 是那个常量数组，v4 是输入。

其实就是输入进来的东西的 ASCII 码加 10 再与 0x6F 异或要与那个常量数组相同。

把那堆常量复制下来即可，对常量异或 0x6F 后再减 10 即可还原。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Reverse/cppRe.cpp)

## Crypto

### (baby) 龙王的代码I

输出 $0\sim 999999$ 之间的质数平方和。

线性筛一遍即可，答案不超过 `long long` 能表示的范围。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Crypto/dragon1.cpp)

### (baby) 龙王的代码II

求解一次同余方程组。

直接 exCRT 即可，一定要用 Python 写。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Crypto/dragon2.py)

### (easy) 龙王的代码III

求解一个大数在大质数下的二次剩余。

先看模数，因为数太大了以为不是质数，结果看一堆人 A 了就很慌，最后发现这个数是质数……

然后直接把熊板拿 Python 写一下就好了。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Crypto/dragon3.py)

### (baby) babyFib

求 Fibonacci 数列第 $m$ 项，对 $1926081719260817$ 取模。因为模数太大了，建议用 Python 写（`__int128` 其实也行）。

因为后台 flag 设错了挂了一次……

~~做法？不会真有人不会写矩阵快速幂吧？~~

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Crypto/babyFib.py)

### (easy) notFib

设 $\{f_n\}$ 为一数列，有如下递推关系：

$$
f_n=f_{n-1}+f_{n-2}+r(n-1)
$$

其中，$r(x)$ 表示 $x$ 可以被 $a$ 数组中的多少个整除。求 $f_m$ 的值。

这题刚上线一天就被四位大爷秒了，我推了半个月才推完……

下面记 $\{F_n\}$ 为 Fibonacci 数列，$\{L_n\}$ 为 Lucas 数列。

展开 $f_n$ 得

$$
\begin{aligned}
f_3&=f_2+f_1+r(2)\\
f_4&=2f_2+f_1+r(2)+r(3)\\
f_5&=3f_2+2f_1+2r(2)+r(3)+r(4)\\
f_6&=5f_2+3f_1+3r(2)+2r(3)+r(4)+r(5)\\
\ldots
\end{aligned}
$$

可利用数学归纳法知

$$
f_n=F_n+\sum_{i=2}^{n-2} F_{n-i}r(i)
$$

而

$$
r(x)=\sum_{i=1}^n [x\bmod a_i=0]
$$

所以

$$
f_m=F_m+\sum_{i=2}^{m-2} F_{m-i}\sum_{j=1}^n [i\bmod a_j=0]
$$

交换求和顺序并化简

$$
f_m=F_m+\sum_{i=1}^n\sum_{j=2}^{m-2}F_{m-j}[j\bmod a_i=0]
$$

可以看出后一个和式的 $j$ 只在 $j=ka_i$ 时有一个贡献 $F_{m-j}$，因此又可写成

$$
f_m=F_m+\sum_{i=1}^n \sum_{2\le m-ka_i\le m-2} F_{m-ka_i}
$$

后一项求和的所有 $F_{m-ka_i}$ 的下标构成等差数列。由于 $n$ 阶递推隔 $k$ 项取一项后构成的新数列仍为 $n$ 阶线性递推数列，因此可以找个规律，发现有

$$
F_{a+bn}=L_bF_{a+b(n-1)}-(-1)^bF_{a+b(n-2)} \quad (n\ge 2)
$$

成立。

在 [Lucas Number - Wikipedia](https://en.wikipedia.org/wiki/Lucas_number) 中，有一个公式

$$
F_{n+k}+(-1)^kF_{n-k}=L_kF_n
$$

通过这个公式可以直接推导上述公式，但是这个公式我不会证……或许可以直接通过通项证。

既然仍是二阶线性递推，求和可构造矩阵

$$
\mathbf A=
\begin{bmatrix}
L_b & (-1)^{b+1} & 0\\
1 & 0 & 0\\
1 & 0 & 1
\end{bmatrix}
$$

构造列向量

$$
r=
\begin{bmatrix}
F_{a+b}\\
F_a\\
F_a
\end{bmatrix}
$$

则 $\mathbf A^nr$ 的第三个分量即为答案，也就是

$$
\sum_{i=0}^n F_{a+bi}
$$

需要手动求出 $a,b$ 的值和项数。

~~新人这能过四个真的 tql~~

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Crypto/notFib.py)

### (easy) Feistel

观察代码，考虑把最后得到密文的分两段考虑。

模拟一下加密过程，发现加密 16 次后，左边是明文右半部分异或上一些密钥，右边是明文左半部分异或明文右半部分再异或上一些密钥。考虑把两边多异或的密钥计算出来，只需要知道这些密钥的异或就好，然后再利用整体的异或还原明文。

注意加密代码最后一步对密文还要交换一下，因此解密时也要先交换再解密。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Crypto/feistel.py)

## Pwn

### nc

直接无脑 `nc` 进去就完事了，注意是 Linux 命令。

### (easy) 不安全的计网实验

其实改了 [TCP 单进程单循环服务器](https://github.com/HeRaNO/ChickenRibs/blob/master/ComputerNetworking/sigsrvsigcli/new/tcp_echo_srv.c)的代码，不会真有人不会 `diff` 吧。

~~当时写这份计网实验代码都给我写吐了……~~

当然……我没有部署这个东西……大家也不用找我挂黑页……

## Dev

### 0x00 Hello World

~~啊？要写六种才能拿 2.0 倍率？~~

### 0x01 GitHub

~~啊？不会真有人没有 Github 账号吧？~~

### 0x02 程序打桩机

根据 CSAPP 里 7.13.3 节「运行时打桩」的方式写一个动态链接库运行即可，还是挺轻松的。

**注意给下发的可执行文件加运行权限！**代码就不给了……照着书上的例程改改就好。

~~我真是把我傻哭了~~

### 0x02 仓鼠大师第一步

~~啊？不会真有人不会用 Go 写多线程吧？~~

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Dev/multithread.go)

### 0x04 仓鼠大师第二步

~~啊？都会写多线程了不会写计网实验？~~

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Dev/proxy.go)

### KV Engine [ Step 0 ]

~~啊？不会真有人不会用 unordered_map 吧？~~

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Dev/kv0.cpp)

[Test code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Dev/kv0_test.cpp)

## SA

### [🐧-0] Baby Linux

~~啊？不会真有人不会用 Linux 吧？~~

### [🐳-0] Hello Docker

~~啊？不会真有人不会用 Docker 吧？~~

### [🐳-1] Docker Build

~~啊？不会真……~~

确实，我不会用 Docker 了……

Dockerfile 写得很丑了……Alpine 本身 5M 左右，最后构建出的东西有 1.3G……

RUN 命令需要写在一起，构建证书那段需要分阶段构建。

~~但是懒得改了……~~

[nginx-quic](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/SA/nginx-quic)

## Design

一个题都没写……

## Misc

### Hello World - 1

~~啊？不会真有人不会输出 Hello, World! 吧？~~

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Misc/hello1.c)

### Hello World - 2

要求不能出现各种引号，可以考虑用 `putchar()`，只需要打 ASCII 码就行了。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Misc/hello2.c)

### Hello World - 3

要求不能出现分号……

~~你是魔鬼吗.jpg~~

有什么语句能不带分号呢？

`if` 语句套进去可以不带分号……

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Misc/hello3.c)

### Hello World - 4

要求用 C++ 实现一个不引入库就能输出的程序。

老压行选手肯定会记得 C 语言中有这么一种写法。

```c
main(){ /* code */ }
```

但是 C++ 不能这么写。就考虑用 `extern "C"` 引入 `printf` 搞定之。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Misc/hello4.cpp)

### Hello World - 5

要求不能出现 `%:?#({<[]>})`……

提示是用 Shellcode 写，但是我不会。

想用汇编，但是怎么想都得用几个括号……

### MOD 3

写一个求 $x\bmod 3$ 的函数，要求在 `int` 范围内正确。负数取余方式与 C 语言处理方式相同。只能用位运算和加法。

首先考虑任何指令集都没有直接的模运算指令，能用的运算和 RISC 指令集是类似的，所以就去用 [Compile Explorer](https://gcc.godbolt.org/) 搞出它在 MIPS 上的汇编。大概就 15 条指令完成，但是有些看不懂。

切回 x86-64 架构，终于有注释了，大概意思是先乘一个 Magic Number 之后再截断一下，加加减减之后就完事了。乘法那部分就用 MIPS 补足即可。

我们小学二年级的时候就学过 `-b = (~b) + 1`。利用这个实现减法即可。

总符号使用数为 20。

[Code](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2020/Misc/mod3.cpp)

### 电报

~~啊？不会真有人不会用 Telegram 吧？~~
