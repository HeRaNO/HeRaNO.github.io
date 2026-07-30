---
title: 交互题和通信题是怎么跑起来的
date: 2026-04-28 14:42:35
categories: 'OI/ICPC'
tags:
	- 开发
description: ' '
---

新人教育的时候发现新人不懂交互题是怎么跑起来的，因此写了这些东西。

## 前置知识

通常意义下理解的交互题模型类似一个 C/S 架构应用，你的交互器作为服务端（Server），选手程序作为客户端（Client），选手所发送的询问和回答都可以看做一次对服务端的请求（Request）。

- 交互器和选手程序实现了题面所规定的通信协议（Protocol），包括语法，语义和同步。题面中应定义这三部分

通常意义下理解的通信题模型实际与通信过程无关，更一般的理解是多次运行（multi-pass）类问题，在这类问题中，选手程序会被多次运行，而两次运行之间会有裁判程序介入以处理选手的上一次输入和准备下一次输出。选手程序每次运行时的模型（是以传统方式运行还是交互方式运行）是可以任意的（但取决于比赛系统实现，一般是固定一种的）。

## 交互题后台运行流程

运行流程根据比赛系统实现有些许差异，但整体流程大致统一。

1. 首先准备两条管道，一条管道连接交互器的 stdout 到选手的 stdin，一条管道连接选手的 stdout 到交互器的 stdin；

   - 这里省略了启动交互器进程和选手进程细节，在不是写一个比赛系统的情况下此细节可忽略
   - 这里建立的管道一般为单向管道

2. 之后，交互器可以从 `inf` 读入数据并预处理；

   - `inf` 是 `registerInteraction` 通过命令行参数中传参配置的，此时输入是作为文件来读入的（因为 stdin 已经被管道连上了）

   - 应当很轻松地发现并理解，选手的输入输出都是被管道连接的，选手能读到的东西是可以被交互器控制的
   - 选手应当无法读取输入文件，这是比赛系统保证的
   - 因此，输入文件中可以写任何有助于交互过程的数据，只要交互器自己保留（作为答案或过程变量），不传递给选手就没有问题

3. 最后，交互器和选手通过管道交互，交互结束后各自退出。

   - 交互器从 `ouf` 读取选手输出，并且把数据写到 stdout 后 flush，和选手交互过程对称
   - 此时，如果选手输出格式有误，交互器会自动退出，导致管道断裂，而选手程序并不会注册 SIGPIPE 信号，这样如果选手再向标准输出中写入，就可能触发运行时错误，导致选手其实是 WA 但是可能系统里看是 RE，选手 panic
   - 理想状态是 Server 需要 Graceful Shutdown（在 Server 退出时 Client 可以知道自己要结束数据处理并进入退出过程，TCP 挥手是 Graceful 的），也就是说交互器在退出时要给选手发个信号（理想情况下就是给选手程序发个 $-1$ 之类好读入好处理的东西），但是由于 testlib 实现问题这个信号很难发。比如，testlib 在读入整数发现整数超过范围时，直接会触发 wrong output format 的 quit 然后停止程序，这个过程中没有任何 hook 可以让我们去发这样的信号
   - 这种问题有一个兜底逻辑，比赛系统可以先尊重交互器的判题结果，再去看选手程序的退出状态，就可以确保是一致的 WA

**一些 DOMjudge 和 Polygon 上的运行差异细节**

- DOMjudge 不处理 `tout`，而 Polygon 处理 `tout`，两者运行流有一些区别
  - DOMjudge 实现的是传统意义上的交互过程，只有交互器和选手程序进行交互，最终看交互器的结束状态和判题结果来返回判题结果
  - Polygon 在此之上多了一步，交互器可以向 `tout` 输出数据，结束之后 Polygon 会将 `tout` 的输出内容与答案（答案指 Main Correct Solution 与交互器交互后生成的 `tout` 的输出）进行比对。最后综合交互器的判题结果和 `tout` 比对结果来返回判题结果
  - 所以，在 Polygon 造题时，交互题 `checker` 直接选 lcmp 并且不管 `tout`，在交互器返回所有可能结果，导入 DOMjudge 就不会出问题

## 通信题后台运行流程

通信题的实现五花八门，目前并不趋同，因此分别介绍 DOMjudge 上和 Polygon 上的运行方式。

注：下面所称「交互器」并不一定起交互功能，只是从交互题中借用术语。

### Polygon

Polygon 上只支持选手程序运行最多两次。如果要在 Polygon 上造通信，必须勾选 `Is problem interactive`，在 `Advanced` 页面勾选 `Run solutions twice`。`Interactive second invocation` 是指第二次运行是不是按交互题运行，因为第一次运行一定是按交互题运行的。

如果不勾选 `Interactive second invocation`，运行流程大致如下：

1. 第一次运行：交互器与选手交互，交互器需要将下一次运行的输入写到 `tout` 中，如果这次交互发现选手输出有问题可以直接返回 WA，系统不会进行第二次运行
2. 第二次运行：系统自动把上一次写到 `tout` 的内容当做选手程序的输入，选手程序按传统方式运行
3. 最后，交互器作为 checker 比较选手第二次运行的输出与答案

如果勾选 `Interactive second invocation`，运行流程大致如下：

1. 第一次运行：交互器与选手交互，交互器需要将下一次运行的输入写到 `tout` 中，如果这次交互发现选手输出有问题可以直接返回 WA，系统不会进行第二次运行
2. 第二次运行：交互器还是与选手交互，还是如同交互题的交互过程

### DOMjudge

DOMjudge 支持选手程序运行**最多** `validation_passes` 次，以不产生 `nextpass.in` 文件为终止条件。如果经过了 `validation_passes` 次运行之后发现还有 `nextpass.in` 系统会直接 WA。如果系统发现第一次运行后没产生 `nextpass.in` 并且交互器 OK，系统就不进行后续的运行，直接返回 AC。

DOMjudge 并没有解耦选手程序每次运行时的模型。如果 Types 选了 `multi-pass`，就是每次运行都是传统模式，如果**又**选了 `interactive`，就是每次运行都是交互模式（前提是 `pass-fail` 肯定要选）。

- 由于 DOMjudge 觉得 `multi-pass` 又 `interactive` 这种题自己还没测好，所以导入包时不支持同时识别 `multi-pass` 和 `interactive`，此时**必须**以 interactive 题目类型导入（确保交互器 Executable 类型是 run，如果正常 `multi-pass` 类型是 compare，这样跑不了），然后手动去 web 端加上 `multi-pass` 类型后才能正常判，注意还要同时设置 `validation_passes`。
  - 或者去 backport Dup4 的一个 [commit](https://github.com/Dup4/domjudge/commit/81f5fe82cbb641e775d7ad1066463be842ea0436) 就好了，但导入时 `validation` 的顺序必须是 `interactive multi-pass`

如果只选了 `multi-pass`，运行流程大致如下：

1. 第一次运行：选手正常进行一次传统形式的运行，交互器需根据选手输出与第一次输入准备 `nextpass.in`；
2. 第二次运行：选手正常进行一次传统形式的运行，交互器需根据选手输出与第二次输入准备 `nextpass.in`；
3. 重复
4. 第 $n$ 次运行（最后一次）：选手正常进行一次传统形式的运行，交互器需根据选手输出与第 $n$ 次输入判断选手程序是否正确，**不能创建 `nextpass.in`**。

如果又选了 `interactive`，就是把每次传统方式都变成交互方式运行，区别不大。

注意：

- 最后一次运行前的所有运行一定要创建 `nextpass.in`。
- 最后一次运行一定不能创建 `nextpass.in`。

### 让你的交互器具备通用性

你的交互器**必须**保证既能在 Polygon 上跑，又能在 DOMjudge 上跑，但是两个系统逻辑不同，就需要借助宏来判断当前在哪个环境。p2d 打包时会添加 `DOMJUDGE` 宏，来给交互器传个信息，Polygon 上当然没有这个宏，所以可以借助这个宏来让交互器具有通用性。

例如最常用的创建 `nextpass.in` 的逻辑，这个只在 DOMjudge 上有，Polygon 上没有，所以你需要在第一次运行中进行：

```cpp
#ifdef DOMJUDGE
	tout.open(make_new_file_in_a_dir(argv[3], "nextpass.in"), std::ios_base::out);
	ensure(tout.is_open());
#endif
```

来创建这个文件。之后对 `tout` 的写入在两个平台上是一样的。

又或者仅 `multipass` 类型问题，由于 Polygon 上第一次运行永远是交互，而 DOMjudge 上第一次运行可以是传统方式，所以在 Polygon 上你需要把第一次运行的输入通过 stdout 输出出来给选手程序，否则选手程序会卡死在读入上，这就需要类似：

```cpp
#ifdef DOMJUDGE
	tout.open(make_new_file_in_a_dir(argv[3], "nextpass.in"), std::ios_base::out);
	ensure(tout.is_open());
#else
	std::cout << ops << "\n";
	std::cout << ori << std::endl;
#endif
```

这样的逻辑。

至于这种情况下第一次运行选手是否需要 flush，DOMjudge 上是不需要的，因为每次运行都是传统方式，Polygon 上我尝试的情况下也不需要 flush，其实比较奇怪，但是 flush 肯定是没错的。
