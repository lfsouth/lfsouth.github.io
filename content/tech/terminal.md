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