---
title: "我对于terminal的认知"
date: 2022-07-26T21:27:10+02:00
draft: false
tags:
- terminal
- shell
---

## physical terminal和terminal emulator

很久以前，terminal是一个机器，最有名就是这个VT-100了。

![VT-100](/images/DEC_VT100_terminal.jpeg)

屏幕是输出，键盘是输入。

键盘上的击打会被转化为电信号通过RS-232电缆发送到Computer对应的接口上，这个电信号在被Computer接受到之后，通过硬件和驱动，最终被OS接收到；OS识别输入的具体意义，可能进行一些操作之后回复，也可能立刻回复，回复的内容，通过驱动和硬件转化为电信号并发送回terminal，而terminal接受到电信号的操作是一致的，将其按照既定规则转化为屏幕上的字符。

所以，虽然感觉上是你敲击键盘直接在屏幕上打出文本，但其实键盘和屏幕之间是没有任何联系的；键盘的电信号只会发送给computer，而屏幕需要的电信号只会来自于computer。

现在，没人用VT-100了。

现在，都是用的是terminal emulator。

我用的最多的是iterm2，一个MAC OS上的terminal emulator。

打开iterm2，你得到的是一个graphical application，除了边框，就是一块textual screen。

什么是textual screen？就是一块被划分了行和列的screen。换句话说，这是一块铺满格子的screen，每一个格子都可以被一个行标和一个列标唯一定位。

你在MAC OS上打开Finder等其他应用，你得到的是也是一个graphical application，一块screen，但是这个screen是以pixel为单元的，每一个pixel都可以被唯一定位。

两者只是不同的computer display mode。

前者很古老，效率高，内存需求低，没多少花样可玩，最多也就是sl的效果。

后者同样很古老，内存效率自然不如前者，但这种消耗对当下的电脑来说也不值一提，并且花样多，是毫无疑问的主流。

## terminal emulator和shell

terminal emulator是一个executable，shell也是一个executable。

因为绝大多数terminal emulator打开之后在textual screen上呈现出来的是一个prompt，所以总给人一种terminal emulator和shell是不分你我的观感。

实际上，两者是完全独立的两个process。

将两者联系起来的是pseduoterminal，pseduoterminal是属于kernel space的，换句话说，任何对pseduoterminal的操作都是通过system calls来完成的。

你可以将pseduoterminal看成是一个沟通不同process的桥梁，也就是一种IPC。

terminal emulator处于IPC的一端，shell处于IPC的另一端。

terminal emulator可以通过pseduoterminal将数据发送到shell，shell也可以通过pseduoterminal将数据发送到terminal emulator。

流程上，创建一个terminal emulator的process，创建一个executable的process，创建一个pseduoterminal，绑定terminal emulator process到pseudoterminal的一端，绑定shell process到pseduoterminal的另一端。

下图是TLPI介绍terminal related system calls时候使用的图片，对应的，可以认为terminal emulator就是一个driver program，shell就是一个terminal-oriented program；其中从driver program到terminal-oriented program的fork&exec过程不是必须的，但是是常见的操作。

![Terminal-001](/images/terminal-001.png)

terminal emulator是一个textual graphical application，打开这样的一个application，然后敲击键盘，屏幕上显示你敲击的符号，感觉上是你敲击键盘导致屏幕符号的产生，但这是错觉。

你敲击键盘，对windowing system来说，这是key event；这些key events被terminal emulator这个window捕获，terminal emulator将之转化为数据通过pseduoterminal发送给shell，shell对发送过来的数据进行解释，比如执行ls命令之类的，然后将执行的结果转化为数据通过pseduoterminal发送会terminal emulator，而terminal emulator将根据数据调整textual screen上的视图，用户就在屏幕上看到结果。

## terminal emulator和shell和ssh

在我的MAC OS上，我的iterm2是这样的：

![Terminal-002](/images/terminal-002.png)

在我的MAC OS上，我的iterm2是这样的：

![Terminal-003](/images/terminal-003.png)

所以，实际发生了什么呢？

![Terminal-004](/images/terminal-004.png)

在Mac上打开iterm2，这里iterm2是一个terminal emulator，其和一个zsh通过pty相连，当我在输入ssh 的时候，iterm2捕获了这个这一行命令，通过pty发送给了zsh，zsh fork&exec一个ssh client，ssh client通过TCP/IP和远端的ssh server相连。
作为响应，远端的ssh server fork&exec一个login shell，并通过pty将ssh server和login shell连接起来。

这样一个长长的通路就建立了，login shell发送的ANSI escape codes sequence通过pty，ssh server，tcp/ip，ssh client，zsh，pty最终到达iterm2，然后iterm2这个terminal emulator根据这个sequence修改其textual screen的试图，这就是屏幕上所显示的东西。

Mac上的zsh我做了多样的配置，明显提供了更复杂的sequence，呈现出来的效果也很好，而vm里面的login shell只是默认的配置，其sequence没那么多花样，出现的效果就很朴素。

## terminal emulator和shell和vim

![Terminal-005](/images/terminal-005.png)

这是在Mac的iterm2直接使用vim打开一个文件的界面。

其背后的实现时这样的：

不同的是，当textual screen里面展示的是prompt的时候，terminal处于canonical状态，这个时候数据在pty上是以行为单位来传输的。

而当textual screen里面展示的是如上vim的操作界面的时候，terminal实际上时处于noncanionical状态的，这个时候数据是以一个字符未单位来传输的。

## device & driver


