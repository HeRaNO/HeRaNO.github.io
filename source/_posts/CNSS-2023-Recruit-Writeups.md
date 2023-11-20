---
title: CNSS 2023 Recruit Writeups
date: 2023-11-20 09:13:15
categories: 'CTF'
toc: true
tags:
	- 数学
	- 数论
---

## 夏令营

### [Re] 🐍 [Mid]Pyfuck

把 `x` 数组和那堆怪浪怪浪的异或值拖出来算就好了，那堆值是根据位运算算出来的，`([]<[])` 是 `False`，但是 `~False` 是 `-1`，因为 `False` 是 $0$，所以这样。我也是第一次知道 Python 还有这种操作。

总之拖出来算完了异或就好了。

[pyfxxk.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Bin/pyfxxk.py)

### [Pwn] 😡 [Baby]让我访问！！！

这道题难点在于要把 `cat` 命令直接发过去，而不是进 interaction。或者在 Linux 下跑。

Windows 下进 interaction 之后发命令崩了，不知道为啥，看 debug 记录它是一个字符一个字符发过去的。

[letmein.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Bin/letmein.py)

### [Pwn] 👀 [Easy]你的名字

数组比较和整数存储方式，还是 4B 分个段然后反过来。本地测的时候可能有内存屏障导致溢出到数组的部分没被处理，可能是编译的时候要加点花活，不过总之这样是对的。

### [Pwn] 💞 [Easy]古明地觉模拟器

IDA 之后发现先和一个 char 数组比较，如果是 `KomeijiSatori` 就再和一个随机数比较，如果相等就 getshell。一看数组开的位置和读入的位置就知道直接把数组溢出覆盖掉那个随机数就行了。覆盖的方式也十分随意，直接搞一个长度大于 $28$，前缀是 `KomeijiSatori` 但是别太离谱的输入就行，因为 `memcmp` 有长度限制所以也不用加 `\0` 啥的，长度可以设为 $32$，然后本地 get 一下回显的覆盖出来的数，然后拿这个数上去输入就可以了。

### [Pwn] 🥰 [Easy]看看你的backd00r

上上学期做过的实验复现，IDA 可以看到有个 `backdoor` 函数能 getshell，然后考虑溢出 `buf` 就行，IDA 显示的距离 rbp 的距离是对的。再看跳转到哪儿，如果直接跳到 backdoor 的头上是不行的，因为 64 位程序包含 `system` 要考虑堆栈平衡，可以看[这篇文章](https://blog.csdn.net/qq_41560595/article/details/112161243)，总之直接跳到给 `system` 赋参数那行就行了。

[backdoor.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Bin/backdoor.py)

### [Crypto] 🌀 cyclic group

就是 RSA，一样做就好了。

[cyclic.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/cyclic.py)

### [Misc] 🔎 侦探 CNSS 娘的秘密

flag 在图片信息内，用 [Stegsolve](http://www.caesum.com/handbook/Stegsolve.jar) 看一下信息就有了，注意把空格删了。

### [Misc] 🏆 重生之我在 CNSS 当 OIer

运行时没做隔离可以直接读答案，虽然答案没和程序放一起，设置一下读答案的路径就行了。

忘设置了 WA 了一发，哈哈。

[oier.c](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Misc/oier.c)

### [Misc] 🔮 東方希缇符

我好像记得这个曾经出过。总之打开游戏发现是永夜抄，上谷去搜有没有解包，有一个 [VS 没编译过的工程](https://github.com/xfgryujk/THUnpacker)，还有个[只有二进制的](https://github.com/shirokurakana/Quick-Touhou-Extractor)。总之用那个只有二进制的，解包之后会出来一个 `flag.txt` 就是了。

### [Misc] ❓ 泻药，人在CNSS，刚打完招新

在 Timlzh 的[知乎回答](https://www.zhihu.com/question/561337166/answer/2730585153)里，居然埋了一年。

## Web

### 🙋 PHPinfo

提示访问 `/phpinfo.php`，然后就能看到 PHP info 了，然后搜一下就能找到 flag 了。

### 🦴 babyHTTP

上来先 get，然后 post，最后设置一个 cookie，还是挺简单的，但是 set 的 cookie 是 `admin=true`。

### 🏓 Ping

发现 IP 地址直接拼接到命令里，所以直接 getshell 了，把 ping 输出的东西干掉，然后 cat 一下 flag 就好了。

[ping.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Web/ping.py)

### 🥇 我得再快点

页面一秒会刷新一次，所以先 requests 一下看看逻辑，总之先从网页里拉下来 Key，然后算一下 MD5 再 GET 另一个 URL，不能超时，所以直接写就好了。

[fast.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Web/fast.py)

### 🐶 CNSS娘の宠物商店

SQL 盲注，直接密码填 `' or 1=1#` 就行了。因为用户名是 `admin`。

sqlmap 有一点点用，但不多。

### 🥵 2048

上去改分就行了，直接控制台把 `score` 改成很大的数，然后调一下 `getflag()` 就好了。

### 👤 换个头像先

大概就是[这个意思](https://blog.csdn.net/HasntStartIsOver/article/details/130910722)，配合一下 PHP 一句话木马就行了。可以把 `POST` 改成 `GET` 更简单一些。

## Re

### 😍 那个女人

二年级

### 😢 我的flag碎了一地

第一部分在 Strings 里，第二部分在 Function 栏最上面，第三部分需要 xref 一下，但是 `putchar` 部分不全，需要看汇编部分才能看到完整的最后一部分。

### ♾️ 亦真亦或亦假

还是那种异或，把那堆数放到 `char` 数组里，然后一样做就好了，还是很简单的。

[real.c](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Bin/real.c)

### 茶杯头大冒险

搜一下发现是 xtea 加密，找个解密就好了。最好是在 IDA 里把密文的负数转成无符号的，然后直接用无符号整数做就好了。注意密文大小端序。

[xtea.cpp](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Bin/xtea.cpp)

### ✈️ 打飞机高手

`get_flag` 函数在 `Show` 函数内部，点开 `Show` 函数才能看到。总之还是在程序最开头 patch 一下，直接调用 `get_flag` 函数就好了。

## Pwn

### 🎬 应聘 CNSS 娘中之人

用正则乱搞一下，把里面的信息用捕获组捕获一下就好了，写起来稍微还有点麻烦。主要是它自己输出了 `> `，需要 `recvuntil` 把它吃了。

[cnss.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Bin/cnss.py)

### 💝 Overflow Me

刚开始看岔别了，还以为要绕 `read` 里的读入长度限制，其实读入长度是长于数组的，于是直接把溢出部分构造过去就好了。pwntools 挺好用的。

[overflow.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Bin/overflow.py)

### ❤️‍🔥 Stack Overflow

ret2text，没啥好说的。

[stack_overflow.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Bin/stack_overflow.py)

### ❤️‍🩹 Unlock My Heart

其实啥预先准备都有，后门函数上面就有 `pop rdi` 和 `pop rsi`，怎么样搞一搞参数，跳到那个位置就好了，但是没整明白那个溢出数组的首地址，感觉也挺麻烦的，并且局部变量应该也用不了传参。所以直接 ret2libc 了，还是上上学期的大作业。但是 64 位的跳转和传参和 32 位有很大区别，可以参考[这个博客](https://blog.csdn.net/Mintind/article/details/128165311)。还有一个问题是要找函数地址，在目标机器上试了 `puts` 和 `__libc_start_main` 这两个函数的 got 表，结果都只有后三位每次是相同的，到 [libc database](https://libc.rip/) 搜一下，有两个 libc，试了一下 `libc6_2.35-0ubuntu3_amd64` 的地址就对了。其实 3.x 版本的地址都是一样的，只需要试 1 和 3。

其实需要注意使用返回地址，先从 `vuln()` 函数跳到 `setName()` 函数里，往全局变量 `name` 里设置 `/bin/sh` 这个字符串，然后再跳回有溢出的函数，再 `pop rdi` 和 `pop rsi` 里设置函数参数，再跳到后门里，但是没成功，只能说 ret2libc 挺好玩的。

[ez64.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Bin/ez64.py)

## Crypto

### 🔮 cnss娘的谜语

四个部分用四种不同方法编码，就再解码一下。由于 flag 是中文的，所以要全拼出来才能获得最后的 flag。

[riddle.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/riddle.py)

### 🐬 水龙吟

给 $p,g$ 和 $c=m\cdot g\bmod p$，求 $m$。

看岔别了，还以为 $p-1$ 之后怎么 CRT 合并，实际上是一个线性同余方程。熊板抄一下就完事了。

[shuilongyin.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/shuilongyin.py)

### 🌔 卜算子

三个质数的 RSA，一样算回去就完事了。

[busuanzi.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/busuanzi.py)

### 🐠 Fun_factoring

对于第一个 flag，首先是一个 $p-1$ 是光滑数，用 [Pollard's p-1 算法](https://ctf-wiki.org/crypto/asymmetric/rsa/rsa_module_attack/#p-1) 算法搞一下就好了。然后第二个 flag 是已知了 $e\cdot d$ 求 $p,q$，但是给出的 `hint1` 里面又多乘了一个质数。根据[这个](https://bbs.kanxue.com/thread-263069.htm) WP 来做，先出来的是多乘的质数，然后是 $pq$ 乘积，再做一遍就好了。至于为什么先出来的是多乘的那个我也不知道。

[fac.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/fac.py)

### 🦅 ezLattice

感觉题里啥也没给，直接对 C 矩阵做个 SVP，第一行就是了。从直觉来说确实是这样的，LLL 算出来之后因为 $p$ 很大，第一行的值都很小，而后面行的值都很大，所以直接取第一行为明文就行了。

因为忘了考虑符号懵住了很久，终于在把符号取反了之后才想起来符号确实是反的。

[lattice.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/lattice.py)

### 🦢 BabyCurve

椭圆曲线给定了公私钥求基点，也就是 $Q=e\cdot P$，给定 $Q,e$ 求 $P$。那么直接考虑类似 RSA，求 $de\equiv 1\pmod {\text{ord}(E)}$，然后 $P=d\cdot Q$ 就好了。跑起来挺慢的。

[BabyCurve.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/BabyCurve.py)

### 🔑 Small private key

主要参考 [New Attacks on RSA with Modulus N = p^2q Using Continued Fractions](https://iopscience.iop.org/article/10.1088/1742-6596/622/1/012019)，使用第一个算法就行了，第二个算法不行。和出题人找到了一样的文章但是没有用一样的算法，在算法二死磕了一晚上，换了个算法就好了。

我感觉他是算法二写错了，因为 $e/N$ 的连分数表示里没有解，我不好说。

需要注意这里 $\gcd(\varphi(n), n)=p$。

[spk.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/spk.py)

### 🐿️ 物不知数

看上去和 21 年的 ECDLP 是一样的，还是省略掉最后一个大质数然后把小质数搞出来 DLP。首先上来分解 $n$，速度居然还行。然后考虑如果 $h\equiv g^x\pmod n$，那么也有 $h\equiv g^x\pmod{p_i}$，于是解 DLP 我们就有了 $x\equiv d_i\pmod{\varphi(p_i)}$，之后 CRT 合并一下就行了。这里只能硬上，十分奇怪的是用 `Mod(g, p)` 这种写法套到 `discrete_log` 里就跑得奇慢，换成 `GF(p)` 然后用 `R(h).log(R(g))` 就奇快，十分神奇，就这个问题调了一下午。

[crt.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/crt.py)

### 😋 叒是欧几里得

其实根据题目名提示很容易想到构造多项式做 gcd，用 [Franklin–Reiter related-message attack](https://en.wikipedia.org/wiki/Coppersmith%27s_attack#Franklin%E2%80%93Reiter_related-message_attack) 就可以了。令 $m+k_1\cdot \delta=x$，则第二个 padding 就是 $x+r$，其中 $r=(k_2-k_1)\delta$，因为 bound 很小枚举一下计算就好了，这样我们其实求出的是 $(m+k_1\cdot\delta) \bmod n$，之后还得把噪声减去，减的时候也不用担心模，正常取就好了，因为明文长度不超过模数，总之就再枚举一遍 $k_1$ 试着去 decode 就好了。

[gcd.sage](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/gcd.sage)

### ✡️ Oracle of Nyx

和 HITCON 2023 的题差不多，可以看出题人的 [WP](https://bronson113.github.io/2023/09/08/hitconctf-2023-careless-padding.html)。但是感觉挺不一样的，总之不会做。

### ⚽ Permutation

和 [CF 612E](https://codeforces.com/problemset/problem/612/E) 是一样的，但是这道题得求出所有可能。相同长度偶环只能两两合并，相同长度奇环可以单扔出来也可以两两合并，合并的时候还可以对齐不同位置，总之超级大搜索，太难蚌了，摆了。

### 🧐 Odd_Curve

可以看这篇 [WP](https://github.com/p4-team/ctf/tree/master/2018-12-08-hxp/crypto_curve12833227)，这个题里的曲线是 $y^2=x(x+2)^2$，一样做就好了。但是直接解出来的 $k$ 不对，毕竟解出来应该是一个解系，$k+i\cdot\text{ord}(E)$ 都是解，还要挨个试。然后曲线上点和分数是同构的，所以算一下那个分数的阶就是 $\text{ord}(E)$ 了，直接 `u.order()` 算的是 $u$ 在的群的阶，得用 `multiplicative_order`。仔细想想最后要求一个 $nP=\infty$ 的 $n$，这里映射后 $\infty$ 映射到 $1$ 了，确实是求乘法阶的问题。

[odd.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Crypto/odd.py)

## SA

### 🚩 Flag in Image?

直接 export 那个 docker image 会发现 flag 文件在第一个 layer 里，只不过在最后删除了，直接打开那个 layer 就可以。

## Misc

### 🧐 招新平台的彩蛋

先 F12 可以看到一个 genshin.jpg，里面 ISBN 号被抹了，以为和 ISBN 有关，所以硬干了很久原神的 ISBN 号并没有啥收获（甚至想到了书店员本田的 OP），然后去干了 JPG 隐写也没有啥东西。直到鼠标挪到左上角 cnss 的徽标上，发现可以转，但也不算稀奇，但是它一直在转就很 nb。然后转着转着原神就出来了，抹掉的 ISBN 号的位置就是 flag，十分难蚌。

P.S. 平台貌似大小写不敏感，所以字母全大写也过了。

### ✨ 星光下的梦想

MP3 频谱隐写，应该能听出来是开头的左声道部分，但是 flag 不是很清楚，得仔细看。

注意要从波形切到频谱，哈哈。

### 🧡 杂鱼~ 杂鱼~ 找不到 flag 的杂鱼~

LSB 隐写，用 Stegsolve 分析一下，选 Data Extract，然后勾出来 RGB 的最低位 Preview 一下，一开头就是 flag 了。

### 🎉 扫码领取 flag

去[修复](https://merri.cx/qrazybox/)一下二维码的 Format Info Pattern，在网站的 Tools 里，直接 Brute Force 一下然后 Decode 就好了。

今年的 Misc 只能说强度很低。

### Hello World - 5

属于是补作业了，首先还是 shellcode，上上学期大作业刚做过，别的不说先得写个 64 位汇编。然后就是不让用 `()` 和 `{}`，说明这里面一个函数都不能定，主函数都不行，然后就在想怎么搞一个程序入口。也知道 gcc 下程序入口是 `_start`，直接编译一个空文件，ld 会提醒 `_start` 里没有 `main`，但是之后就不懂了，后来直接看到了 [plusls 的博客](https://blog.plusls.com/technical/binary/pwn/write-helloworld-without-bracket/)，才明白了直接开一个 `main` 数组就能替代了，然后把数组变成连续排布的变量就可以绕 `[]` 了，淦。

这样其实还不够，因为有 `-O2` 会优化掉内存排布，如果直接顺序那么写的话不开 O2 是能过的，但是开了 O2 会将其栈式处理，这样整个 shellcode 就颠倒了，objdump 一下就能看出来了。所以最后要把变量逆序定义。

[shellcode.asm](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Misc/shellcode.asm)

[make_code.py](https://github.com/HeRaNO/ChickenRibs/tree/master/CNSSRecruit/2023/Misc/make_code.py)

编译命令：

```bash
nasm -f elf64 shellcode.asm -o shellcode.o
ld shellcode.o -o shellcode
```

拿 shellcode 的命令（从 [StackOverflow](https://stackoverflow.com/questions/15593214/linux-shellcode-hello-world) 抄的）

```bash
for i in `objdump -d shellcode | tr '\t' ' ' | tr ' ' '\n' | egrep '^[0-9a-f]{2}$' ` ; do echo -n "\\x$i" ; done
```

剩下的问题感觉就是 shellcode 了，也就是各个平台的汇编怎么写的问题，aarch64 就得写 ARM 汇编，不会写，放弃了。
