---
title: 部分 OJ 的 VJudge 方法
categories: '开发'
toc: true
date: 2021-01-27 22:19:07
tags:
	- 开发
description: 主要讲的就是怎么搭（mo）VJudge（yu）
---

这个人终于被 VJudge 整崩溃了，寒假集训的时候所有学校都在寒假集训，CF 交题一直 Pending，毕竟 VJudge 只有五个账号，到高峰期一等就是 10 多分钟，十分影响比赛体验（骂骂咧咧）。并且 VJudge 自己 fork 题面的话就不能用数学公式，很气。

加上 Lutece 的重构需求，大概的想法是将测评端（Judge Server）抽象出来，作为独立的服务来看，测评端的存储题面和数据，接受后端发来的测评和题目请求，返回结果。这样抽象的话就可以兼顾 Lutece 作为题库的需求和 VJudge 需求了。本地 Judge Server 提供题目相关接口和测评接口，后端接口一对接就完事了。爬来的题面渲染问题在前端解决应该就可以了。

其实也不怎么算 VJudge 了，主要是需要被动增加题目，不是主动去对方 OJ 搜题。

然后就是走一遍 VJudge 老路，看怎么登录和提交代码了。

主要参考的是 [VJudge 源码](https://github.com/hnshhslsh/virtual-judge)和 [BNUOJ VJudge 源码](https://github.com/BNUACM/bnuoj-vjudge)。两个项目一个是 Java 一个是 C++，互相参考理解可能更快一点吧。

原型在[这里](https://github.com/HeRaNO/ChickenRibs/tree/master/Lutece-VJudge-prototype)，都是 Python 写的，以供参考。

## Codeforces / Gym / SGU

这个是 VJudge 都会搞的。对于题面来说不是大事，总的题目可以通过 API 调取，然后根据调取结果爬题面即可。

对于 CF 的登录，需要从页面中拿到 CSRF token，还需要从页面 Cookies 中提取 `39ce7` 的神仙字段，然后计算出 `_tta`，作为 POST 的参数。很神仙，根本不懂这个 `_tta` 算出来有啥用。

CF 的提交仍然需要 CSRF token，但是不需要 `_tta` 了，然后提交语言需要参照列表里面的对应值填写，反正也是从页面爬。

Gym 的提交和 CF 提交大体一样。

SGU 的题放在 CF 上了，所以这部分也和 CF 一样。

## AtCoder

AtCoder 的登录还是简单一些，只需要 CSRF token 就好了。

AtCoder 的提交需要一顿好找，需要找到那些标签的名称，语言的话还是从页面爬到，填对应的值即可。

## Aizu

VJudge 起来超级舒服的 OJ，但是自己是个 sb。

提供了齐全的 API，文档在[这里](http://developers.u-aizu.ac.jp/index)，面向 API 搞就好了。

最后没搞懂提交的异步返回是怎么搞的，返回一个 token 之后不知道怎么查结果，或许应该 await 一下就好了……

## CodeChef

~~是个人会写成 CodeChef 这样的提交吗。~~

写这个的时候也参考了 [CodeChef CLI](https://github.com/sk364/codechef-cli) 的实现。

CodeChef 把一堆验证都设成了 hidden，需要在一面上好找一顿。提交时需要注意 `form_token` 有两个，一定要填 `id` 对应的，否则会出错。

而且必须要传个空文件上去，否则也会被拒绝~~，实属脑瘫设计~~。

调了一下午结果是语言代码写错了……

是个人能写成这样？

## Baekjoon OJ

OJ 地址是[这个](https://acmicpc.net)。

这个 OJ 上有一堆别的地方找不到的题，各种远古题看起来很全。

但是登陆要绕 reCAPTCHA 认证，很烦，虽然是隐藏的认证不用点图片或者输验证码什么的，但是还是不会绕。

看上去提交的时候还是有 CSRF token。

OJ 开发者远古的时候写了个自动提交代码的 Python 脚本，但是现在不好使了……

这个 OJ 也打算提供 API，但是不知道要等到啥时候了……

## LibreOJ

问题同 Baekjoon OJ，更新到 SYZOJ NG 之后有了全套 API，但是不知道文档在哪儿，并且登录还是得绕隐藏的 reCAPTCHA 认证……

VJudge 支持了，但是不知道 VJudge 咋写的，太神仙了……

Menci 说要手动登录获取 session 的 token，我决定放弃。

## HDU

HDU 到现在都是 HTTP 方式访问……

浏览器抓个包其实都看出来了，登录会重定向，但是不影响访问。

网站编码是 GBK 系的，提交的时候注意编码，但是我完全搞不懂。所以直接交 Base64 让后端解决去了……

可以用中文测试，也可以试试 [HDU 2815](http://acm.hdu.edu.cn/showproblem.php?pid=2815) 这道题，输出的那个单引号不是英文单引号……

## ZOJ

ZOJ 登录时需要拼图认证，同 LOJ 和 Baekjoon OJ 不会搞。

## Kattis

Kattis 上比赛挺多的。

~~看标志有点像恋爱裁判 PV 女主竟然有点萌~~

Kattis 登录有 CSRF token，并且会重定向，但是 HDU 不能自动重定向但是 Kattis 可以不知道为什么。

提交依旧提供了提交文件的选项，同 CodeChef 处理就行了。

## POJ / OpenJudge

POJ 原本是不想支持的，但是就当个标本摆着了。

因为我们也没怎么在 POJ 拉题倒是……

这两个 OJ 的表单数据都不怎么复杂，继续浏览器抓包就啥都有了。

OpenJudge 提供了 API，查状态还是很方便的。

## SPOJ

看 SPOJ 的 footer 有 API 吓到我了，结果就是个广告。

仍然没什么需要特别注意的地方……提交文件还是像 CodeChef 处理。

## HackerRank

登录有 CSRF token，并且有反爬手段，必须交一个伪 header。

剩下的没什么了。

## URAL

这个 OJ 很有意思，注册了之后你会获得一个 `JUDGE_ID`，然后凭这个 ID 提交，根本不用登录。

但是记这个 `JUDGE_ID` 感觉好烦……

## 计蒜客

计蒜客的验证挺烦的，CSRF 的 token 放在 Cookie 里了，传密码要传 MD5……但是响应是 json 的，还是挺方便的。

## SCU

作为一个远古 OJ，主要是~~嫖上面的省赛题~~训练为主。

登录还是很正常，但是提交的时候又双叒叕是 GBK 编码，这回没办法用 Base64 糊弄过去了，BNUOJ 的 VJugde 是把代码当 URL 查询参数传过去的，但是感觉这样太丑了……

非常震撼的是提交要验证码，但是我不会写验证码识别，看两个 VJudge 都是手写的验证码识别，应该不难，但是懒得搞了……也防 SCU 突然加强验证码导致提交失败……

## hihoCoder

hihoCoder 上貌似有些区域赛的题可以做。

真的没啥说的，应该是最简单的了……

## 51nod

~~51nod 上面有 lxl 题还是可以搬的。~~

51nod 的密码传输采用 RSA 算法加密，公钥如下：

```plain
-----BEGIN PUBLIC KEY-----
MIGeMA0GCSqGSIb3DQEBAQUAA4GMADCBiAKBgGTNlnTkCV1UvqhwEwoTBSgve1Ic
eORoDpB8lV7g358xqoXTHFj7i+4SP3mD48VmGFUyKu+ZiH8hN0Od6s6+bttQ7YbE
vSRwEjwfVC9DCCPukAHtirBSCDUqbFJPg8BfsHIOtPDx0CJCNmzutNkSnhD/lNad
JvHU/Gl9NitAceZlAgMBAAE=
-----END PUBLIC KEY-----
```

需要用公钥加密密码之后传过去……绝了……

登录成功返回一个 `-1`，我还以为登录挂了。

提交成功返回提交编号。

## TopCoder

过于奇妙以至于根本没看懂它是怎么 work 的。

## E-Olymp

听说这个 OJ 有很多奇妙的远古比赛。

登录不难，提交有一个 token 需要从提交页面获取。交完直接跳转到结果页面。

## UVa / UVALive

UVa 已经开始用 C++ 重构了，但是表单还是很离谱，在页面里藏了一堆 tag。

UVa 100 这个题输入范围是错的，并且不保证左边界小于等于右边界，很离谱。

UVALive 和 UVa 用的 OJ 差不多，方法也都类似。

## LightOJ

登录有 reCAPTCHA，不会搞。

## FZU

FZU 的 OJ 看起来好老啊（（（

然后页面编码居然是 ISO-8859-1，人都傻了。

但是提交代码是 UTF-8 的，支持中文，海星。

## ECNU

~~哦草后端真的写验证写起来不嫌麻烦是吧~~

ECNU 搞了以上所有 OJ 的验证与加密方法，密码用 RSA 加密，图片验证码，CSRF token，能用上的都用上了，我吐了。

公钥在 HTML 里面，可以直接找到。

交题的时候有 CSRF token，返回一个 json，能获得当前提交状态。

## NowCoder

牛客的登录外包出去了……转到 [GrowingIO](https://growingio.com/) 做验证……

## HIT

说实话去爬 HIT 和 HRBUST 的题也没啥必要……也没拉过他们的题……VJudge 也不支持 HIT 了……

坑还是有一些的。登录之前先获得验证码，里面有登录用的 token，登录之后还会获得一个 token，用来交题中的身份认证。

这里认证用的是 JWT，虽然不怎么重要但是确实是这样。

然后他们公开代码写反了是我没想到的，提交代码 public 选 false 才能看自己的代码，而且自己不能看自己没 public 的代码。

关键这种逻辑写挂的不是随便测一下就测出来了吗……

然后交题之后什么都不返回是我没想到的……只有一个状态码 200……

## HRBUST

登录的时候是把用户名密码信息的 JSON 转化成字符串之后用 RSA 加密发出去的，绝了……

公钥如下：

```plain
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC88fBRU1FaC5S537krMDVDapOZ
k44Nw+Yud69IPZYwk9HT0n6D7Pvp3mXp+Par6gp5HUW3tFs7/l3cTIqryEXMfJXR
F7FlneM64kLs/KZ2i0larVrz7QgcTb5oAuHeIE24Uc0ja+S83Hlc2Dk6z1TWkAjG
2f/p15xRfv/IO5yyywIDAQAB
-----END PUBLIC KEY-----
```

而且 OJ 不能在比赛中注册账号，绝了……

## Universal OJ / Dark BZOJ

UOJ 的密码是加盐 MD5 加密的，找了一晚上才发现是 HMAC-MD5 加盐加密的，甚至想手动照着 JS 代码实现一遍，但是 JS 写法太奔放了根本承受不住地址溢出……

还要在 JS 代码里面找 CSRF token 我真的是吐了。

但是所幸一个 session 里的 token 不会变……
