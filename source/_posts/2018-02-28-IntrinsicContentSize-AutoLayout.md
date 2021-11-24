---
title: IntrinsicContentSize 与 AutoLayout 的使用剖析
date: 2018-02-28 16:09:24
tags:
- iOS
---

在使用AutoLayout时，经常会产生约束冲突，我们可以通过改变约束优先级，及模糊约束来解决冲突。

<!--more-->

> Content Hugging (不想变大约束)：如果组件的此属性优先级比另一个组件此属性优先级高的话，那么这个组件就保持不变，另一个可以在需要拉伸的时候拉伸。

> Content Compression (不想变小约束)：如果组件的此属性优先级比另一个组件此属性优先级高的话，那么这个组件就保持不变，另一个可以在需要压缩的时候压缩。

### Content Hugging／Content Compression

`Content Hugging`，`Content Compression` 是两个UIView自带的模糊约束，而这两个约束存在的条件则是UIView必须指定了 Intrinsic Content Size。

### Intrinsic Contenet Size

Intrinsic Content Size：固有大小。顾名思义，在AutoLayout中，它作为UIView的属性，意思就是说我知道自己的大小，如果你没有为我指定大小，我就按照这个大小来。 比如：大家都知道在使用AutoLayout的时候，`UILabel`是不用指定尺寸大小的，只需指定位置即可，就是因为，只要确定了文字内容，字体等信息，它自己就能计算出大小来。

`UILabel`，`UIImageView`，`UIButton`等这些组件及某些包含它们的系统组件都有 Intrinsic Content Size 属性。 也就是说，遇到这些组件，你只需要为其指定位置即可。大小就使用Intrinsic Content Size就行了。

在代码中，上述系统控件都重写了`UIView` 中的`-(CGSize)intrinsicContentSize:` 方法。 并且在需要改变这个值的时候调用：`invalidateIntrinsicContentSize` 方法，通知系统这个值改变了。

所以当我们在编写继承自UIView的自定义组件时，也想要有Intrinsic Content Size的时候，就可以通过这种方法来轻松实现。

### Intrinsic冲突

一个`UIView`有了 Intrinsic Content Size 之后，就可以只指定位置而不指定大小，但有的时候会有问题。举个栗子：水平两个`UILlabel`，label1距左边50，label2距右边也50，中间间隔10。如果按照约束来布局，则没办法满足两个`UIlabel`都使用 Intrinsic Content Size，至少某个`UILabel`的宽度大于Intrinsic Content Size。这种情况，我们称之为2个组件之间的“Intrinsic冲突”。

解决“Intrinsic冲突”的方案有2种：

> 1、两个UIlabel都不使用Intrinsic Content Size。为两个UIlabel增加新的约束，来显式指定它们的大小。

> 2、可以让其中一个`UIlabel`使用Intrinsic Content Size，另一个label则自动占用剩余的空间。这时候就需要用到 Content Hugging 和 Content Compression了。
