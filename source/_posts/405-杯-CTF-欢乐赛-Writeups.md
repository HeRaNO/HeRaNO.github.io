---
title: 405 杯 CTF 欢乐赛 Writeups
date: 2024-10-18 10:00:00
categories: 'CTF'
tags:
	- '数学'
	- '数论'
	- '欧拉路径'
toc: true
description: ' '
---

## Web

### 赛博上香

看一下 JS 是要点十万次上香，看一下 `worship()` 函数有一个 `cnt` 控制，直接把这个变量改成 $10^5$ 即可。

### webshell detect

自己构造了很久都没能传上去，最后还是在 Github 上搜到一个免杀 Webshell 的[项目](https://github.com/cseroad/Webshell_Generate)，然后随便生成一个。传上去，然后就过了，感觉一晚上白干。

### tiny image

首先这个题得把文件直接传上去绕过后缀检查，可以用 Burp 也可以 requests，为了方便用了 Burp。接着是如何构造这个 BMP，试了很多方案（比如只有 `BM` 文件头，BMP 文件只到宽和高，改偏移量等等），经过了无数次 reset 之后确认只有 24 位 BMP 的关于像素的 12 字节能动，所以需要构造尽可能短的 PHP 代码，就别想一句话木马了，直接 `exec` 得了。首先至少得要个 `echo` 还有 `exec` 类的函数，经过搜索搜到了 `<?=` 可以代替 `echo`，反引号可以代替 `exec` 执行（其实是做 webshell detect 的时候搜的），然后就是执行的命令也要尽可能短，但是 `flag` 已经占了四个字符了，所以考虑用通配符，然后 `<?=```cat /*```?>` 一共 13 个字符（这里三个反引号是一个反引号），还需要找一个能输出东西还比 `cat` 长度短的函数，找到了 `nl`。换成 `nl` 就可以了。

[tiny_image.py](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Web/tiny_image.py)

把 `new.bmp` 传上去，在 Burp 里把文件名改成 `new.php` 即可。

![](/images/405/29.png)

### 【教学】HelloWorld

Do something same as [[Easy] ezhttp](https://herano.github.io/2022/09/25/CNSS-2022-Recruit-Writeups/#easy-ezhttp)

[hw.py](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Web/hw.py)

### 【教学】跑马场

PHP 一句话木马，Do something same as [[Baby] Backdoor](https://herano.github.io/2022/09/25/CNSS-2022-Recruit-Writeups/#baby-backdoor)

总之没用蚁剑，懒得装了

[ma.py](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Web/ma.py)

需要找找环境变量，叫 `FLAG1`

### 有穷战争

没什么提示，看起来要玩，但是可以直接看 `world.js` 里搜一下有没有 Flag，还真有。

![](/images/405/4.png)

之后拼起来就好了。

## Reverse

### Flag挖掘机

IDA 逆一下发现枚举了 $30$ 个 $0$ 到 $15$ 的变量，看一下进去的函数

![](/images/405/24.png)

应该很快就能联想到欧拉路径，输入是点的遍历顺序，每条边只删一次，到最后图是空的。那个 `v13` 没有用，就是下标乘 $4$ 了，看一下最后的返回函数

![](/images/405/25.png)

其中要求欧拉序中某一些位置是固定的。

先把图从 `dword_140022000` 中提取出来

[extract_graph.cpp](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Bin/extract_graph.cpp)

画出来之后是

![](/images/405/23.png)

可以发现每个点的度都是偶数，欧拉序应组成一条欧拉回路。

心里寻思反正就这么 $16$ 个点手搓一下就好了，结果嗯搓了一个小时没搓出来，午休结束了都还在搓（感觉丢掉了一个一血）。最后发现我以为欧拉回路的起终点是 $3$，实际上看错了，$28$ 不是最后一位，怒而写暴力

[flag_digger.cpp](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Bin/flag_digger.cpp)

秒出 Flag。

### 修修补补

![](/images/405/9.png)

IDA 看一下 Strings View，第一个 Flag 就有了。

xref 一下第二个 Flag 的提示，可以看到第二个 Flag 是在函数 `sub_404680` 里算出来的，直接在 `_main` 中 patch 掉这里，改为直接调用这个函数。

![](/images/405/10.png)

第三个 Flag 也是类似的，但是函数多了两个传参。看一下这两个参数的意义应该是在地图中的坐标，毕竟是 `20 * x + y` 这种。

![](/images/405/12.png)

所以 `4280CC` 应该是行，`4280C8` 是列，然后看一下什么时候才会到 Flag3

![](/images/405/13.png)

走到 `Q` 的时候才会进到算 Flag3，但是 `Q` 不可达。看一下 `Q` 位置，应该取 `4280CC` 为 $6$，`4280C8` 为 $15$，参照原来函数的传参方法，按 32 位传参传进去再 call 它即可，注意找一个位置够的地方进行 patch。

![](/images/405/11.png)

> 最早忘了这是一道 Rev 题，到 10.11 晚上看突然意识到可以 patch 才把所有 flag 做出来

### 老学校逆向

由于设置了固定的随机数种子，因此生成的随机数是确定的，直接按照原来代码里的方式做回去就好。需要注意变量类型。

[old_school.c](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Bin/old_school.c)

### LazyRe

首先看后缀 `.hs` 可以知道是 Haskell 的代码，倒也不用 GPT，直接看结构分析一下还是挺简单的，输入 $a_i$ 变成 $(((a_i\oplus(2i+1))+195)\bmod 128)\oplus(99-2i)\bmod 128$，但是这里对 $128$ 取模就神烦，需要仔细考虑异或掉第一层之后结果和 $195$ 的关系再取模，不考虑的话会导致输出所有字母都是对的，然后猜的 Flag 是错的。手慢。

代码：

[lazyre.cpp](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Bin/lazyre.cpp)

## Pwn

### 奶龙笑传之答答题

PwnTools 应用题，你需要猜 18 个题的答案。网上所有能找到的都没有答案，但是联想到群主发奶龙，说不定是群主看过了。于是手动枚举猜答案即可。需要注意的一点是不要回显它的问题，在最后一个问题上很有可能超时，把回显都干掉了之后才进得去 interactive mode。猜了半个小时，太难蚌了。

[nlanswer.py](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Bin/nlanswer.py)

### 减肥备忘录

一眼 ret2text，写过 [WP](https://herano.github.io/2023/11/20/CNSS-2023-Recruit-Writeups/#pwn-easy%E7%9C%8B%E7%9C%8B%E4%BD%A0%E7%9A%84backd00r)，首先还是能看到 `win()` 函数是能 getshell 的。看一下位置，考虑堆栈平衡直接跳到给 `system()` 函数赋值的地址。然后看一下主函数逻辑，是 `gets` 的栈溢出，但是 `s` 后面多开了两个 `int` 变量，这里需要考虑在栈上的变量定义顺序，但是 IDA 有 `s` 到 `rbp` 的距离，取 `70h` 就好。

![](/images/405/7.png)

然后就很简单的 ret2text

[note.py](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Bin/note.py)

> 10.10 本地通了远程一直打不通，问了一下群主远程是不是 64 位的，结果是 64 位的，然后陷入了极深的自我怀疑，11 号刚起床就看见改附件了，上班逆完之后改改地址就通了。

### 奶龙汇编

IDA 看一下发现是 shellcode 直接注，但是 shellcode 不允许出现 `\x0f\x05`，`\x0f\x34` 和 `\x0f\x35`，其中 `\x0f\x05` 是 64 位的 `syscall`。

上网搜了一下怎么样绕过，找到了一个 [WP](https://github.com/VulnHub/ctf-writeups/blob/master/2015/ringzer0/shellcoding.md#level-4)，但是写得巨复杂。大概意思对 shellcode 编码，里面配一个 decoder，预先先对整个 shellcode 编码，上去运行的时候先解码 shellcode 然后运行。因为 x64 的 shellcode 比较简单，先找一个[能跑的](https://www.exploit-db.com/exploits/46907)，然后改改它的 decoder：

[decode.asm](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Bin/decode.asm)

主要就是改它的 shellcode 长度，是 $23$。

然后 nasm 编译一下：

```shell
nasm -f bin decode.asm
```

再把两部分拼到一起：

[merge.py](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Bin/merge.py)

然后发出去就是了

[nlasm.py](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Bin/nlasm.py)

### ascii_cfg

一进 IDA 报错 The graph is too big，然后看这个程序很大，没有任何输入，只有一个输出，刚开始就不知道从哪儿下手了。

直接跑一下这个程序

![](/images/405/21.png)

可以看出来前半段 Flag。

绕了很久，尝试反编译，改了最大函数段长度到 1M，但是反编译出来的东西没有任何用。搜了一个十分特殊的指令 `vfmaddsub132ps`，然后直接搜到了一个 [DEFCON](https://media.defcon.org/DEF%20CON%2023/DEF%20CON%2023%20presentations/DEF%20CON%2023%20-%20Chris-Domas-REpsych.pdf) 的 Pres，里面大概意思是用 Graph 做类似像素画的效果，果然还得是调那个 graph too big 的问题，搜了一下[解决方法](https://blog.csdn.net/LostSpeed/article/details/124586377)，然后卡了一段时间出 Graph 了。调成暗色模式并且缩放到最小。

![](/images/405/22.png)

可以看到上半部分就是 Flag 的后半段。想了一下为啥长度是 $30$，一看那个 `w` 的字体很不对，两个部分中间是有间距的（可能是修排版修多了导致的工伤），然后就凑到 $30$ 了。

## Crypto

### Yamoto

Python 实现的随机是 Mersenne Twister，搜到了一个 [PRNG](https://book.jorianwoltjer.com/cryptography/pseudo-random-number-generators-prng) 的介绍，说获得连续 624 个 32 位随机数即可构造一个生成剩余随机数列的 MT PRNG，但是题里的随机数是两个 PRNG 产生数的异或。这里应该证明两个种子不同的 MT PRNG 生成数列对应异或之后得到的数列可以通过构造一个 MT PRNG 来生成得到，但是不会证，写了一个实验验证。

[yamato_test.py](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Crypto/yamato_test.py)

跑过了，没什么问题，所以就当成一个随机数生成器做就好了。

代码：

[yamato.py](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Crypto/yamato.py)

### Feistel Collision

很有意思的一道题。代码是要求输入一个串，构造哈希碰撞，然后 padding 是先接长度再接 `\x00`，所以想去干哈希长度扩展，但是没什么收获，浪费了一天。

然后分析一下代码是类似 DES 加密，Github 搜代码找到了[差不多一样的代码](https://github.com/RobinDavid/pydes)。发现这个十分怪，把输入当 `key` 去循环加密 `\x00`，[CTF-Wiki](https://ctf-wiki.org/crypto/blockcipher/des/) 上说 `key` 只使用了 $56$ 位，有 $8$ 位丢了，才想到用丢的位构造。但是代码里 `CP_1` 换了一个置换，丢掉的不是每个位置的最后一位，输出看一下：

```patch
        key = string_to_bit_array(self.msg)
+       kkey = [i for i in range(64)]
+       pp = self.permut(kkey, CP_1)
+       for i in range(64):
+           if i not in pp:
+               print(i)
        g, d = nsplit(key, 28)
```

发现丢掉的是第 $17, 21, 25, 35, 39, 44, 46, 50$ 位，找一个位变一下就好了。

构造的输入：`Cre2y_4_V_mE_S0`

### 榴莲披萨

看一下 `task.py`，大概是一个数字签名，用 $A^sY^{h(m,R)}$ 来验签。$h(m,R)$ 中 $m$ 是明文，$R$ 是一个输入的矩阵，$h(m,R)$ 返回将明文和输入矩阵拍平之后依次连接形成的字符串的 MD5 值。如果 $A^sY^{h(m,R)}=R$，则验证成功获取 Flag。所有的计算是在模一个大质数的情况下做的，并且这个质数光滑。

感觉写得特别迷，$s,m,R$ 都是可控的。考虑令 $R$ 不变，$m$ 换成要签名的字符串，然后求 $s$ 就变成了求离散对数。这里应该证明 $s$ 的存在性，考虑到模质数意义下一定存在原根，如果 $A$ 是一个原根那 $s$ 是一定存在的，对应到矩阵域下应该也存在一个类似的原根概念，但是 $A$ 不是一个原根也可能存在 $s$，这里就很迷，不会系统证明。

最主要的问题是 sage 不能直接对矩阵求离散对数，搜一下怎么求找到了一篇 [WP](https://ctftime.org/writeup/31952) 提到用 Jordan 标准型。感慨了一下只有 71 的矩阵分析后对 $A$ 进行 Jordan 分解，发现直接分解出一个对角阵，因此只需要求模意义下对角线的某个元素的离散对数即可。求出的数就是 $s$。

[durian_pizza.sage](https://github.com/HeRaNO/ChickenRibs/tree/master/405/Crypto/durian_pizza.sage)

真算出来 $s$ 了。然后直接上去输入就可以了。

> 代码中将 $R$ 换成单位阵也是能跑出来 $s$ 的

## AI

### Neuro 的邪恶计划

不会 AI，也没看出来网页上哪儿有漏洞，分析 JS 也不得行，看上去混淆过了。但是发现它的出招方案是固定的，所以可以直接通过一定次数的交互把它搞出来。

能赢的一个序列：`2332122333222313131132333223322333222112112232321X`，剪刀石头布分别为 1, 2, 3。

> 搞了 40min，感觉和上面的答题一样痛苦。

### 奶龙检测器

> 我感觉我是乱做的，我不会 AI（again）。

因为提到了 One Pixel Attack，同时看了一下[欺骗神经网络](https://www.ruanx.net/cheat-neural-network/)，看起来大概是因为奶龙特征太单一了，神经网络训练的时候过拟合了。

要求的是改动最多 10 个像素，让分类器失效，所以直接启动画图，针对奶龙特征比较多的部分（比如眼睛周围，面部，身体什么的）随机点 10 个黑点

![](/images/405/nl.png)

本来就寻思娱乐一下的，但是没想到直接通过了。

### 复活吧！客服小祥

以 `预设词 flag` 为关键词搜索，搜到了一个 [套 Prompt](https://blog.csdn.net/m0_59598029/article/details/136727401) 的方法，直接用忽略指令就好了。

![](/images/405/26.png)

第一句纯属歪打正着，复制错了，但是获得了很好的效果。果然 Flag 在 Prompt 里。

> 上述方法只能这样处理，应该是必须先输入一些文本 AI 处理不了的东西 fallback 到预设场景外，然后再进去套 Prompt。但是换了个链接它就 fallback 不了，只能这样复现。
>
> 没看过 MyGO，想用一些其他方法套，奶奶哄睡法无效，换预设法无效，最后想试试换个人反而把我整急了。就这样吧，害。
>
> ![](/images/405/27.png)
>
> ![](/images/405/28.png)
>
> ——世界名画：《你爹攻击和扩展你爹攻击》

## Misc

### 签到

Flag 1 是群号，海报有。

Flag 2 去找个 EXIF 信息的 [Extractor](https://exif.tools/) 就好了，在 Comment 里。

### 密码游戏

看到是 Python 的打包，先[解一下包](https://github.com/extremecoders-re/pyinstxtractor)，然后[反编译](https://tool.lu/pyc/)一下 `passwdgame.pyc`，可以看到有 21 条规则，依次匹配就好了。

注意：

- 有些规则是相容的，可以利用相容性让一个部分的构造满足两个规则，比如以撒的结合部分可以直接用 `???` 来满足特殊字符，并且不影响后面的构造
- 罗马数字乘积部分仅考虑单个罗马数字，构造完了之后除了 `I` 以外的罗马数字字符都得禁用
- 数独的那个规则条件已经写死在正则里了
- 可能有些化学元素有共同前缀，比如 `F` 和 `Fe`，会考虑两遍原子序数
- `V` 是钒，也是罗马数字 `5`，总之化学元素部分的构造比较头疼，启发式调整
- 最后才说要密码包含密码长度和做 PoW，但是都已经解包了，可以预先调整
- 最后的 PoW 没说清字符表范围，是大小写字母，但是我猜 ASCII 码可见字符里应该也有

最后构造的密码：`$Zq$Lilac774???Vg1deadbeefeeeFeaustraliaiiiiRh2024I am WarRuoooooouuuuuTod`

### CatMine

首先能看到是 Unity 项目，因为之前做过（倒也不是）一堆游戏 Misc（[東方希缇符](https://herano.github.io/2023/11/20/CNSS-2023-Recruit-Writeups/#misc-%E6%9D%B1%E6%96%B9%E5%B8%8C%E7%BC%87%E7%AC%A6)，[Shino 的心跳大冒险](https://herano.github.io/2022/09/25/CNSS-2022-Recruit-Writeups/#middle-shino-%E7%9A%84%E5%BF%83%E8%B7%B3%E5%A4%A7%E5%86%92%E9%99%A9)），所以路径依赖了（包括密码游戏也是），搜一下[解包](https://github.com/Perfare/AssetStudio)，然后用解包工具看一下。

![](/images/405/14.png)

可以看到最后一个 scene 里有 Flag，然后去找这个 Flag 到底是哪个 asset，导出一下 asset 表搜一下 `level3`，把相关的 MonoBehaviour 全选上然后导出

![](/images/405/15.png)

导出的 `Text.dat` 是二进制文件，用二进制模式打开即可，里面有第一个 Flag

![](/images/405/16.png)

之后费了很长时间想一个 Rev 题应该 Rev 什么，Rev 了一晚上启动器和资源文件，无果，第二天以 `unity3d ctf misc` 为关键词搜，搜到了一篇并不怎么相关的[博客](https://www.cnblogs.com/chrysanthemum/p/11664798.html)，里面提到逆 `Assembly-CSharp.dll`，因为所有代码都打包到这儿了，然后用 IDA 逆了一下，发现 Strings View 用不了，手动找，直接把两个 Flag 都找到了。

![](/images/405/19.png)

![](/images/405/20.png)

xref 了一下发现这个 Flag 应该是打在日志里的，但是游戏运行的时候没看到有日志输出，估计是只能逆了才能看的。

> 所以应该当纯 Rev 题做的。

## 题外话 - 关于没过的题

基本所有题都看了一遍，这个部分是题外话，是关于没过的题的思考过程。

贴个外校师傅的 [WP](https://www.yuque.com/keyboard-ovrmx/scxvuu/ya9yiy8zgt8tg9v4)

### poker

先 `checksec` 了一下发现有 NX 有 PIE，然后 IDA 看一下发现是跟一个固定操作模式的 AI 打德扑，赢掉 AI 所有钱就 getshell，不会打德扑，算了。

### RealWorld

几乎和 real world 题绝缘，不会打 Web。

### UAF

堆上 Use After Free，看了一下 [CTF-Wiki](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/use-after-free/) 感觉还得结合别的攻击来做。实在没时间了。

### 奶龙大冒险

可能是个 ret2libc，因为它会给一个 `printf` 函数的位置，但是有 NX 有 PIE，用不了这些，可能是比较高级的 ROP 了。

### 宇宙的回响

最值得吐槽的一道题，听一下大概是 1:23 左右的时候右声道有杂音，搜一下题目里 `这种声音常在宇宙中传输图像信息` 可以知道是 SSTV 信号，然后就是需要对信号降噪，用的 Audacity。分离了左右声道，但是右声道里有左声道声音，混响了，并且左声道音量提高了，右声道音量降低了，先 normalize 了一下，然后试着用左声道的声音做特征去修右声道的声音，几乎没有效果，用 Robot36 解出来大概能看出是一张二维码，但是图像效果极差，关键的定位点都没有解出来。然后尝试去根据频率去滤波，会增强一些画质，但是还是没有修到根本问题上。问题在于左声道的声音频率和 SSTV 频率几乎是一致的，并且响度还大，干扰十分强。这个过程产生了巨量噪音，虽然可以用虚拟声卡代理一下，但是并没有用。

之后交流的时候说是左声道反相，调整音量后叠加右声道即可。但我也试了这种方案，没啥效果。害，不会修音频。

### 神秘文件

可能是 PDP-11 的程序机器码，将给的数字转成 16 进制再以字节形式写入（比如如果数字是 $1$ 就写 `\x00\x00\x00\x01`），然后找个[反编译](https://github.com/caldwell/pdp11dasm)，发现没成功，然后试了将每个字节倒序或者 $2,1,4,3$ 这种顺序也没获得什么信息。

> 实际上是 Base64，龙了

### eth-intro

可能是个区块链简单题，但是没时间细看了。

### rust_shell

一看是 Rust 就跑了，不会 Rust。
