---
title: Viking 2022 Recruit Writeups
date: 2022-09-01 12:29:36
categories: 'CTF'
toc: true
description: ' '
---

## Web

主要参考了[这篇](https://blog.csdn.net/shuaicenglou3032/article/details/116951359) Writeup。

### easy_backup

中午莫名就会扫网站了，dirsearch 一下扫到了两个东西 `flag.php` 和 `robots.txt`。`flag.php` 里显示的没什么有用的东西，于是翻 `robots.txt`，里面有一个 `Disallow: /wwwroot.zip.bak` 的记录，把这个 zip 下载下来，打开 `flag.php`，里面注释的就是 flag 了。

### unserialize

晚上莫名就会反序列化洞了。在 `/secret.php` 下就能看到，感觉打 CNSS 2020 的时候就见到过反序列化的题，但是没做。

构造 payload

```plain
?d4wn=O:8:"UserInfo":2:{s:18:"%00UserInfo%00username";s:4:"root";s:18:"%00UserInfo%00password";s:4:"EnF5";}
```

即可。

## Re

### easy_re

小学二年级的时候，我们就学习过用记事本打开可执行文件然后用搜索字符串的方式获取 flag。

### easy_pwn

逆一下给的二进制，发现直接 getshell 了，所以 pwntools 直接进去就可以了。

### right number

逆完之后发现前半部分判断肯定是 `false`，没办法进到 `f` 函数里，所以直接看 `f` 函数。里面写得乱七八糟，大概意思是有两个数组，先异或一下再异或一个常数就是最后的 flag。把常量复制出来放数组里搞一搞就好了。

## Misc

### helloworld

flag 是 `flag{helloworld}`，在通知里。
