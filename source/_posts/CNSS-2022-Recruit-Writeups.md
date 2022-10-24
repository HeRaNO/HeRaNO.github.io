---
title: CNSS 2022 Recruit Writeups
date: 2022-09-25 16:25:34
categories: 'CTF'
toc: true
tags:
	- 数学
	- 数论
---

这几年打下来感觉技术确实没什么长进了……

<!-- more -->

## Web

### 📝 [Baby] SignIn

进去发现 F12 不能用，那就直接找浏览器菜单把开发者模式找出来，进去之后改一下 Flag 的 button 属性即可。

### 🚪 [Baby] Backdoor

这题可以说是我被纯纯的降智了，首先被蚁剑安装橄榄，费了很大力气把 Windows Defender 关了，然后解压安装代码无法解压，错误是 `[object Object]`，又花了很长时间百度，终于发现要开管理员身份运行，然后终于打开了蚁剑。代码是个 RCE，添加数据的时候连接密码填 `cnss` 即可。然后进去发现里面有个 9.25 改动的 flag，以为是被人改了。问了出题师傅说没改过，主要因为蚁剑左边栏不显示文件只显示文件夹，直接被自己的智商深深地感动了。flag 在根目录下，直接 cat 一下即可。

### 📦 [Baby] Webpack

之前没啥想法，大概搜了下 Webpack，可以根据 sourceMap 进行反编译（？），看起来没有 map 文件，但是访问一下那个 js 文件后面加 `.map` 是有的，下载下来用 shuji 逆一下，flag 在 `App.js` 的注释里。

### 💧 [Easy] Leak

扫一下站发现有一个 `.index.php.swp`，应该是 Vim 的备份文件，`vim -r` 恢复之后和上一道题差不多，只不过方法从 POST 变成了 GET，所以利用漏洞更简单了。直接 GET 一个 `system('ls /');` 看一下 flag 文件名，再把它 cat 出来即可。需要注意的是 PHP 里面也是要有分号的，没加分号挂了好几次。

### 🎯 [Easy] ezhttp

上去先是只接受 CNSS 请求，一眼就看出来要把请求方法改成 CNSS，但是很懒不想做。某一天打开 Postman 突然发现它可以改请求方法，所以做了。然后要用安卓微信内置浏览器访问，于是要改 `User-Agent`，上网[搜一下](https://www.jianshu.com/p/b05b50d8d30d)就知道了。然后要改 `Referer` 和 `X-Forwarded-For`，一个改成 `cnss.io`，另一个改成 `127.0.0.1`，然后说「只接受 `uestc.edu.cn` 域名的请求」，要改 Host。然后加一个 `Content-Type` 是 `application/json`，然后要发一个 JSON，有 `name` 和 `password`，之后在 Cookie 里面也设置它俩，值和 JSON 一样，最后用一个 Basic Auth 再验证一下。总之就是 Postman 和 HTTP Header 使用集合，跟着提示做就行了。

### 🐘 [Easy] ezunserialize

和 Viking 2022 招新的那个反序列化差不多。但是要绕过 `__wakeup`，Viking 不需要绕，绕的话只需要把表示成员个数的值改大一些就好了。还要注意 `i_want2_say` 里面有一些奇怪的 UTF-8 字符把 `fssmsl` 放到后面去了，试了好几次最后把所有字符都用 URL 编码一下就过了，不知道为啥。

### 🕵️ [Mid] ezpad

首先是破解一个 MD5，因为前后缀必须是给定的，所以只能上脚本爆破，第一个 payload 是 `2022iWebCNSS`，然后可以拿到 flag 的 MD5。第二个要让 `cnss` 变量的整数值等于它逆序的整数值，它逆序的值怎么搞都只能是 0，所以只能让 `cnss` 上来整数值就等于 0，所以第二个 payload 是 `2022e-4CNSS`，这样就可以让前面是 0 了。

这样拿到了 flag 的长度是 31。考虑怎么能让 flag 的 MD5 不变或者能算出来变之后是什么。考虑 [Length extension attack](https://en.wikipedia.org/wiki/Length_extension_attack)，MD5 的计算从一个整块扩到两个整块，第二个整块对第一个整块的影响可以计算出来，第一个整块的值就是原来 flag 的 MD5。[这里](https://github.com/iagox86/hash_extender)有写好的可以用。

## Crypto

### ⏲︎ CNSS娘的代码 Ⅰ

一个 CRT，找之前写过的换个数据跑一下就好了。

[CNSSI.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Crypto/CNSSI.py)

### ⏭ CNSS娘的代码 Ⅱ

注：DLP 就是离散对数。

求

$$
\begin{cases}
3^{P+x}\equiv m_1 \pmod{2^{512}}\\
2^{P+x}\equiv m_2 \pmod{3^{512}}
\end{cases}
$$

中 $x$ 的值。其中 $P,m_1,m_2$ 为确定的值。

实际上 $P+x$ 的值已经大于了 $3$ 对 $2^{512}$ 的阶，但是小于 $2$ 对 $3^{512}$ 的阶。因此可以直接用 $m_2$ 算出 flag 来。

[CNSSII.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Crypto/CNSSII.py)

### ⛈️ CNSS娘的代码 Ⅲ

发现 $a=f\cdot B$，那么 $f=a\cdot B^{-1}$，Sage 求个矩阵逆就行了，果然 $B$ 是可逆的。

[CNSSIII.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Crypto/CNSSIII.py)

### 🐢 RSA Ⅰ

我们知道了 $p$ 对一个 mask 的与和或，同时我们知道 mask，那么就可以根据这些值还原 $p$，然后就知道 $q$ 了。这样我们就对 $n$ 进行了质因子分解。求出 $d=e^{-1}\pmod{\varphi(n)}$ 之后明文 $m=c^d\bmod n$。

[RSAI.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Crypto/RSAI.py)

### 🐍 RSA Ⅱ

给了 $p$ 和一个掩码的与，还有 $q$ 和另一个掩码的与。考虑 $p\times q \equiv (p\bmod 2^n)\times (q\bmod 2^n) \pmod{2^n}$，根据结果和掩码搜一下就好了，状态不太多。

[RSAII.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Crypto/RSAII.py)

### 🌘 HomoBlock

做得十分降智，观察一下加密方法，是每 8 个分一块加密的，那么只需弄清楚每块怎么加密的就好了。实际上看一下 `NotHomoFunction` 就行了。每一轮都会把上一轮得到结果的高低 32 位交换一下，然后异或上 `MASK2` 再异或上 `key`。这样交换的结果是换 4 次就会换回到初值，至于为什么，可以按规则手写五分钟写出来。那么代码里换 5 次，得到的值就是初始值异或 `MASK2` 再异或上 `key`。因为代码开头给了第一块的明文，所以根据这个明文先把 `key` 搞出来，然后再按规则把后续块的初值算出来即可。 

[HomoBlock.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Crypto/HomoBlock.py)

## Re

### 😇 [Baby] Welcome to Reverse World

二年级

### 💔 [Baby] Find me

第一段在 Strings 窗口里，第二段在 Functions 窗口里，在最上面，但是不要忘了加前面的下划线。第三段 xref 一下就能看到是在 sub736 里。最后一段已经给出了，补上去即可。

### 💫 [Easy] 回レ! 雪月花

看着 IDA 代码大眼瞪小眼了很长时间，实际上代码大概类似这样。

```cpp
unsigned char* encode(unsigned char* a1, unsigned char* a2, unsigned char* a3, unsigned char* a4)
{
	long long* v4;
	unsigned char *result;
	long long v6;
	// v4 = &v6;
	// *(&v6 - 3) = (long long)a1;
	// *(&v6 - 4) = (long long)a2;
	// *(&v6 - 5) = (long long)a3;
	// *(&v6 - 6) = (long long)a4;
	unsigned char f[5];
	f[4] = (*a1 << 7) | (*a2 >> 1);
	f[3] = ((*a4 >> 2) | (*a3 << 6)) ^ f[4];
	f[2] = ((*a1 >> 1) | (*a2 << 7)) ^ f[3];
	f[1] = ((*a3 >> 2) | (*a4 << 6)) ^ f[2];
	*a1 = f[4];
	*a2 = f[3];
	*a3 = f[2];
	*a4 = f[1];
	return result;
}
```

倒着做回去即可，和 hard 谱一样麻烦，一定要小心操作。

完全不知道为啥那么多人过了，我看了一下午+半晚上。

[maware.cpp](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Bin/maware.cpp)

### 👁 [Easy] 邪王真眼

逆一下发现是 Base64，但是索引表不一样。实际上按照那个 Buf2 先还原出按程序内部的索引表的 Base64 值，然后再根据程序索引表和一般索引表做个映射就可以了。但是 x86 机器上存变量是小端序，低位在低地址，然后数组是从低到高的，正好顺序反了，所以要在转化过程中 reverse 一下。调一晚上这玩意……

[encode.cpp](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Bin/encode.cpp)

### 🔪 [Easy] 恭喜你获得了flag提现机会！

看一下程序，主要是 `outputflag` 这个函数太长了，找不到入口，其实直接在程序开头 call 一下那个函数就行。所以 Patch 一下程序逻辑，先在 IDA view 里光标选中想改的行，选 `Edit-Patch program-Assemble`，之后改掉想改的行，改成 `call outputflag` 即可，最后在 `Edit-Patch program-Apply patches to input file` 保存一下 exe，运行即可。

### 💻 [Easy+] diannaobaozhale

给了一段汇编，把它人脑反编译回去。首先 `_putchar` 调用会输出 edi 位置的字符，先输出的五个字符就是 `cnss{`，最后一个就是 `}`。然后 `loc_1194` 和 `loc_11B0` 是互相跳转的，构成一个循环。循环变量是 `[rbp+var_4]`，初值是 0，如果大于 9 了就会跳出去。然后是循环内部，首先 `[rbp+var_5]` 这个值被挪到了 eax 里，然后被挪到了 edi 里，然后调用了 `_putchar`。这个值初值是 `c`。之后对这个值先加 2 再异或 1，然后进行下一次循环。

反正就照这个逻辑写就好了，不算很难，但是感觉比下面那个费脑子……

[diannaobaozhale.cpp](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Bin/diannaobaozhale.cpp)

### 🗣 [Easy+] Tell me your secret

前 16 位按照给的常量还原即可，后 16 位按 8, 4, 2, 2 位分隔，每一段都是原串的逆序，所以需要把它们翻转一下。这玩意比雪月花简单得多，不知道为啥放后面了。

[TellMeYourSecert.cpp](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Bin/TellMeYourSecert.cpp)

### ❤ [Middle] Shino 的心跳大冒险

打开游戏发现是用 [Yuri AVG Engine](https://github.com/rinkako/YuriAVGEngine) 写的，去 Github 搜一下能搜到这个项目（但是两年没更新了，估计弃坑了）。主要的场景都放在了 Scenario 文件夹下的 `main.sil` 文件里，打开发现是一堆 Base64。下载那些 Release，里面有一个 YuririCLI 是编译项目的，其实看项目介绍图也会发现有个 key，上源码搜有没有加密，发现[还真有](https://github.com/rinkako/YuriAVGEngine/blob/master/Lyyneheym/YuriPlatform/Utils/EncryptUtils.cs)，是个 DES 加密。把代码拖到本地编译，然后解密一下那个 sil 文件，key 就用默认的 `yurayuri`（key 不知道的话就变成 Crypto 题了），发现解密成功了，里面是一堆编译过的东西，有一堆怪浪怪浪的数字。但是这些数字出现还是有些规律的，比如有 dialog 里的，vol 里的什么的，猜它是 byte 转成数字存的，验证一下这些数字串长度都是三的倍数并且每三个都不超过 255，就直接按 flag 格式搜索字符串 `099110115115`，结果真被找到有一个，然后一直复制直到 `}` 的 ASCII 码 `125` 即可。然后将得到的字符串三位三位分割，之后转化成字符串输出即可。

代码就不放了，把那段加密代码拖下来编译就好。

## Pwn

### 🏄 Netcat 是一种 net 吗？

nc 进去 cat flag

### 🤞 更坑的计算题

pwntools 进去，发现要在 1s 内算一个乘法，哈哈。

还只有一组，直接算完了就完事。

[Calc.py](https://github.com/HeRaNO/ChickenRibs/blob/master/CNSSRecruit/2022/Bin/Calc.py)

## Misc

### 🕵️‍♀ 让我看看你的浏览记录

文件是 Wireshark 的流量记录文件，挨个 HTTP 包看就会找到 flag 了。

### 🗝️ 密码学专家（伪）

首先发现 `fake_flag.txt` 和压缩包里面的完全一致，所以考虑明文攻击。用 ARCHPR 选明文攻击就行了，问题主要是需要那个文件用 zip 压缩一下，我用的是 PeaZip，需要用「最大」的压缩模式才能被 ARCHPR 识别成功。跑了四分钟把密码跑出来了。解压出来的包的提示是密码为六位数字，于是接着跑暴力，跑出来了之后第三层提示的是弱密码，于是用 ARCHPR 的字典，密码是 123456，最后就能得到 flag 了。
