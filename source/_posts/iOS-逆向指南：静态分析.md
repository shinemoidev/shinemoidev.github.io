---
title: iOS 逆向指南：静态分析
date: 2021-12-22 14:19:50
tags:
- 逆向
- 反编译
---

静态分析是指对二进制包进行反编译，分析静态的代码逻辑。

### 1、获取ipa

从 App Store 下载的 app 是经过加密的，需要对其进行解密后，才能进行分析。如果你懒得砸壳，可以直接去各种苹果助手下载越狱版 app，那些是已经解密过的。但是如果要找的 app 在助手上没有，就只能自己砸壳了，至于如何砸壳，自行google相关教程，本文只介绍如何逆向分析已经脱壳ipa。

### 2、class-dump工具安装和使用

class-dump是一个可以导出 Objective-C 头文件的工具，官网：[http://stevenygard.com]()

通过分析头文件里的 API，可以简单地分析一个类的实现，或者查找一些私有 API。

class-dump 官网上的版本不能导出用 swift 编写的工程的头文件，当出现`Error: Cannot find offset for address 0x3a546a04 in dataOffsetForAddress:`这样的错误时，就说明这个 app 可能是用 swift 编写的。

建议去 github 上手动编译最新版的 class-dump，或者使用 class-dump-z 代替，下载地址：[https://code.google.com/archive/p/networkpx/downloads]()

把下载到的class-dump执行文件拷贝到`/usr/local/bin/`目录下，这样就可以在终端使用 class-dump 命令了：class-dump -H `/Users/xxx/Payload/xxx.app` -o /Users/xxx/exportDir`了。

![](https://raw.githubusercontent.com/qq510304723/SourceFile/master/WX20211222-105825%402x.png)

### Hopper Disassembler工具安装和使用

Hopper Disassembler 是一个专门反编译 OC 程序的工具。官网：[http://www.hopperapp.com]()

试用版有功能限制，30分钟退出一次，不能保存和导入反编译后的文件，不能动态调试等。

打开 Hopper Disassembler，直接将xxx.app文件或者xxx.app里面的xxx可执行文件拖入，选择对应的 CPU 架构类型即可。

![](https://raw.githubusercontent.com/qq510304723/SourceFile/master/WX20211222-103959%402x.png)

![](https://raw.githubusercontent.com/qq510304723/SourceFile/master/WX20211222-103908%402x.png)

右上角选择if(b)f(x):可以看到近似源码的伪代码，双击可以查看下一级代码

![](https://raw.githubusercontent.com/qq510304723/SourceFile/master/WX20211222-112647%402x.png)