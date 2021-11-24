---
title: Xcode中使用LLDB调试
date: 2019-01-11 09:55:42
tags:
---

### 在Xcode中调试程序

对于我们日常的开发工作来说，更多的时候是在Xcode中进行调试工作。因此上面所描述的流程，其实Xcode已经帮我们完成了大部分的工作，而且很多东西也可以在Xcode里面看到。因此，我们可以把精力都集中在代码层面上。

在[苹果的官方文档](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-command-examples.html#//apple_ref/doc/uid/TP40012917-CH3-SW5)中列出了我们在调试中能用到的一些命令，我们在这重点讲一些常用的命令。

<!--more-->

### 打印

打印变量的值可以使用`print`命令，该命令如果打印的是简单类型，则会列出简单类型的类型和值。如果是对象，还会打印出对象指针地址，如下所示：

```
(lldb) print a
(NSInteger) $0 = 0
(lldb) print b
(NSInteger) $1 = 0
(lldb) print str
(NSString *) $2 = 0x0000000100001048 @"abc"
(lldb) print url
(NSURL *) $3 = 0x0000000100206cc0 @"abc"
```

在输出结果中我们还能看到类似于`$0`,`$1`这样的符号，我们可以将其看作是指向对象的一个引用，我们在控制面板中可以直接使用这个符号来操作对应的对象，这些东西存在于LLDB的全名空间中，目的是为了辅助调试。如下所示：

```
(lldb) exp $0 = 100
(NSInteger) $9 = 100
(lldb) p a
(NSInteger) $10 = 100
```

另外`$`后面的数值是递增的，每打印一个与对象相关的命令，这个值都会加1。
上面的`print`命令会打印出对象的很多信息，如果我们只想查看对象的值的信息，则可以使用`po`(`print object`的缩写)命令，如下所示：

```
(lldb) po str
abc
```

当然，`po`命令是`"exp -O --"`命令的别名，使用`"exp -O --"`能达到同样的效果。
对于简单类型，我们还可以为其指定不同的打印格式，其命令格式是`print/`，如下所示：

```
(lldb) p/x a
(NSInteger) $13 = 0x0000000000000064
```

格式的完整清单可以参考[Output Formats](https://sourceware.org/gdb/onlinedocs/gdb/Output-Formats.html)。

### expression

在开发中，我们经常会遇到这样一种情况：我们设置一个视图的背景颜色，运行后发现颜色不好看。嗯，好吧，在代码里面修改一下，再编译运行一下，嗯，还是不好看，然后再修改吧～～这样无形中浪费了我们大把的时间。在这种情况下，`expression`命令强大的功能就能体现出来了，它不仅会改变调试器中的值，还改变了程序中的实际值。我们先来看看实际效果，如下所示：

```
(lldb) exp a = 10
(NSInteger) $0 = 10
(lldb) exp b = 100
(NSInteger) $1 = 100
2015-01-25 14:00:41.313 test[18064:71466] a + b = 110, abc
```

`expression`命令的功能不仅于此，正如上面的po命令，其实际也是`"expression -O --"`命令的别名。更详细使用可以参考[Evaluating Expressions](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-command-examples.html#//apple_ref/doc/uid/TP40012917-CH3-SW5)。


### image

`image`命令的用法也挺多，首先可以用它来查看工程中使用的库，如下所示：

```
(lldb) image list
[  0] 432A6EBF-B9D2-3850-BCB2-821B9E62B1E0 0x0000000100000000 /Users/**/Library/Developer/Xcode/DerivedData/test-byjqwkhxixddxudlnvqhrfughkra/Build/Products/Debug/test 
[  1] 65DCCB06-339C-3E25-9702-600A28291D0E 0x00007fff5fc00000 /usr/lib/dyld 
[  2] E3746EDD-DFB1-3ECB-88ED-A91AC0EF3AAA 0x00007fff8d324000 /System/Library/Frameworks/Foundation.framework/Versions/C/Foundation 
[  3] 759E155D-BC42-3D4E-869B-6F57D477177C 0x00007fff8869f000 /usr/lib/libobjc.A.dylib 
[  4] 5C161F1A-93BA-3221-A31D-F86222005B1B 0x00007fff8c75c000 /usr/lib/libSystem.B.dylib 
[  5] CBD1591C-405E-376E-87E9-B264610EBF49 0x00007fff8df0d000 /System/Library/Frameworks/CoreFoundation.framework/Versions/A/CoreFoundation 
[  6] A260789B-D4D8-316A-9490-254767B8A5F1 0x00007fff8de36000 /usr/lib/libauto.dylib 
```

我们还可以用它来查找可执行文件或共享库的原始地址，这一点还是很有用的，当我们的程序崩溃时，我们可以使用这条命令来查找崩溃所在的具体位置，如下所示：

```
NSArray *array = @[@1, @2];
NSLog(@"item 3: %@", array[2]);
```

这段代码在运行后会抛出如下异常：

```
2015-01-25 14:12:01.007 test[18122:76474] *** Terminating app due to uncaught exception 'NSRangeException', reason: '*** -[__NSArrayI objectAtIndex:]: index 2 beyond bounds [0 .. 1]'
*** First throw call stack:
(
	0   CoreFoundation                      0x00007fff8e06f66c __exceptionPreprocess + 172
	1   libobjc.A.dylib                     0x00007fff886ad76e objc_exception_throw + 43
	2   CoreFoundation                      0x00007fff8df487de -[__NSArrayI objectAtIndex:] + 190
	3   test                                0x0000000100000de0 main + 384
	4   libdyld.dylib                       0x00007fff8f1b65c9 start + 1
)
libc++abi.dylib: terminating with uncaught exception of type NSException
```

根据以上信息，我们可以判断崩溃位置是在`main.m`文件中，要想知道具体在哪一行，可以使用以下命令：

```
(lldb) image lookup --address 0x0000000100000de0
      Address: test[0x0000000100000de0] (test.__TEXT.__text + 384)
      Summary: test`main + 384 at main.m:23
```

可以看到，最后定位到了`main.m`文件的第`23`行，正是我们代码所在的位置。
我们还可以使用`image lookup`命令来查看具体的类型，如下所示：

```
(lldb) image lookup --type NSURL
Best match found in /Users/**/Library/Developer/Xcode/DerivedData/test-byjqwkhxixddxudlnvqhrfughkra/Build/Products/Debug/test:
id = {0x100000157}, name = "NSURL", byte-size = 40, decl = NSURL.h:17, clang_type = "@interface NSURL : NSObject{
    NSString * _urlString;
    NSURL * _baseURL;
    void * _clients;
    void * _reserved;
}
@property ( readonly,getter = absoluteString,setter = <null selector>,nonatomic ) NSString * absoluteString;
@property ( readonly,getter = relativeString,setter = <null selector>,nonatomic ) NSString * relativeString;
@property ( readonly,getter = baseURL,setter = <null selector>,nonatomic ) NSURL * baseURL;
@property ( readonly,getter = absoluteURL,setter = <null selector>,nonatomic ) NSURL * absoluteURL;
@property ( readonly,getter = scheme,setter = <null selector>,nonatomic ) NSString * scheme;
@property ( readonly,getter = resourceSpecifier,setter = <null selector>,nonatomic ) NSString * resourceSpecifier;
@property ( readonly,getter = host,setter = <null selector>,nonatomic ) NSString * host;
@property ( readonly,getter = port,setter = <null selector>,nonatomic ) NSNumber * port;
@property ( readonly,getter = user,setter = <null selector>,nonatomic ) NSString * user;
@property ( readonly,getter = password,setter = <null selector>,nonatomic ) NSString * password;
@property ( readonly,getter = path,setter = <null selector>,nonatomic ) NSString * path;
@property ( readonly,getter = fragment,setter = <null selector>,nonatomic ) NSString * fragment;
@property ( readonly,getter = parameterString,setter = <null selector>,nonatomic ) NSString * parameterString;
@property ( readonly,getter = query,setter = <null selector>,nonatomic ) NSString * query;
@property ( readonly,getter = relativePath,setter = <null selector>,nonatomic ) NSString * relativePath;
@property ( readonly,getter = fileSystemRepresentation,setter = <null selector> ) const char * fileSystemRepresentation;
@property ( readonly,getter = isFileURL,setter = <null selector>,readwrite ) BOOL fileURL;
@property ( readonly,getter = standardizedURL,setter = <null selector>,nonatomic ) NSURL * standardizedURL;
@property ( readonly,getter = filePathURL,setter = <null selector>,nonatomic ) NSURL * filePathURL;
@end"
```

可以看到，输出结果中列出了`NSURL`的一些成员变量及属性信息。
`image`命令还有许多其它功能，具体可以参考[Executable and Shared Library Query Commands](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-command-examples.html#//apple_ref/doc/uid/TP40012917-CH3-SW5)。


### 参考

>转载自: 南峰子的技术博客 ([LLDB调试器使用简介](http://southpeak.github.io/2015/01/25/tool-lldb/))

