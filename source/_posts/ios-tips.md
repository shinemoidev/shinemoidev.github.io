---
title: iOS Tips
date: 2021-12-03 09:51:01
tags: tips
---

#### 1. 检查二进制文件中是否有UIWebView

在工程根目录执行

```bash
find . -type f | grep -e ".a" -e ".framework" | xargs grep -s UIWebView
```

我们提交 App Store 都是打包成 .ipa 提交，所以我们先将 .ipa 重命名 .zip 并解压。 然后假设我们需要找的是 UIWebView, Terminal 搞开，到解压后的目录

```bash
find . -print0 | xargs -0 grep -s 'UIWebView'
```
