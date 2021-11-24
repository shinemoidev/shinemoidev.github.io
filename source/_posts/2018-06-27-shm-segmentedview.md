---
title: UI组件-WDSegmentedView
date: 2018-06-27 10:32:00
toc: YES
tags:
- 组件
- UI组件
---

分段控制器

继承自[HMSegmentedControl](https://github.com/HeshamMegid/HMSegmentedControl)

此文章只讨论我们自己添加的相关属性和方法，更多设置请参考[HMSegmentedControl](https://github.com/HeshamMegid/HMSegmentedControl)

如无特殊情况，请尽量只在使用我们自己提供的API的前提下完成需求。(使用 _HMSegmentedControl_ 原生参数可能会导致UI不统一)

Dynamic

![](http://img.jokinryou.online/shm-segmented-view-dynamic.png)

Fixed

![](http://img.jokinryou.online/shm-segmented-view-fixed.png)

<!--more-->

## 尺寸

宽度: 屏幕宽度，不可修改

高度: 40，固定，不可修改

## 样式

### Dynamic

每个选项区域宽度为文字长度左右各 *+edgeMargin*

edgeMargin默认为 *15*

![](http://img.jokinryou.online/shm-segmented-view-dynamic-marked.png)

### Fixed

每个选项区域宽度为 MAX(屏幕宽度/count, 文字宽度左右各 *+edgeMargin*)

edgeMargin默认为 *0*

如无特殊情况，此样式下 edgeMargin 使用 *0* 即可

![](http://img.jokinryou.online/shm-segmented-view-fixed-marked.png)

## 代码

### Enumeration

HMSegmentedControlSegmentWidthStyle

```
typedef NS_ENUM(NSInteger, HMSegmentedControlSegmentWidthStyle) {
    HMSegmentedControlSegmentWidthStyleFixed, // Segment width is fixed
    HMSegmentedControlSegmentWidthStyleDynamic, // Segment width will only be as big as the text width (including inset)
};
```

### Class

#### WDSegmentedView

##### Property

```
@property (nonatomic, weak) id<WDSegmentedViewDelegate> delegate;
```

选中的项目文字和指示器的颜色，默认 _SMTColorNameLinkColor_，可设置
```
@property (nonatomic, strong) UIColor *tintColor;
```
```
@property (nonatomic, strong, readonly) UIFont *normalTextFont;
```
```
@property (nonatomic, strong, readonly) UIFont *boldTextFont;
```

##### 实例方法

指定titles，style默认为 _HMSegmentedControlSegmentWidthStyleDynamic_，edgeMargin默认为为 _15_
```
- (instancetype)initWithTitles:(NSArray *)titles;
```

style需要设置为 _HMSegmentedControlSegmentWidthStyleFixed_ 时，edgeMargin 一般设置为 *0*
```
- (instancetype)initWithTitles:(NSArray *)titles widthStyle:(HMSegmentedControlSegmentWidthStyle)style edgeMargin:(CGFloat)edgeMargin;
```

当选择项不只包含文字时使用此方法，需要自行给定合适的 normalImages 和 selectedImages
```
- (instancetype)initWithNormalImages:(NSArray *)normalImages
                      selectedImages:(NSArray *)selectedImages
                          widthStyle:(HMSegmentedControlSegmentWidthStyle)widthStyle
                          edgeMargin:(CGFloat)edgeMargin;
```
```
- (void)configSegmentedControlWithWidthType:(HMSegmentedControlSegmentWidthStyle)widthStyle edgeMargin:(CGFloat)edgeMargin;
```

选择项有变动的情况下使用
```
- (void)resetTitles:(NSArray *)titles selectedIndex:(NSInteger)selectedIndex;
```

### Delegate

```
- (void)segmentedView:(WDSegmentedView *)segmentedView didSelecteIndex:(NSUInteger)index;
```