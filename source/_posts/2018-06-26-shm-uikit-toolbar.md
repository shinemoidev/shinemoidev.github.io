---
title: UI组件-WDToolBar
date: 2018-06-26 14:05:30
toc: YES
tags:
- 组件
- UI组件
---

常用于页面底部有一个或多个横向并排操作 (按钮样式)，也可用于页面任何位置

注意，WDToolBar跟系统的UIToolbar无任何关系

高度固定 *50* (自适应iPhone X时除外，但有效区域高度仍然为50)

<!--more-->

## 样式

### 一般样式

![](http://img.jokinryou.online/shm-toolbar-style-common.png)

### 强引导样式

![](http://img.jokinryou.online/shm-toolbar-style-guide.png)

### 情感倾向样式

![](http://img.jokinryou.online/shm-toolbar-style-emotional.png)

## 代码

### Enumeration

WDToolBarPos

toolBar位置

```
typedef NS_ENUM(NSUInteger, WDToolBarPos) {
    WDToolBarPosScreenBottom = 345, // toolBar在屏幕底部，自动适应iPhone X
    WDToolBarPosOther,              // 默认值，toolBar在除屏幕底部外的任何位置
};
```

WDToolBarItemStatusIndicationType

按钮状态标识类型
```
typedef NS_ENUM(NSUInteger, WDToolBarItemStatusIndicationType) {
    WDToolBarItemStatusIndicationDefault = 999, // icon、文字标识highlight状态和disable状态
    WDToolBarItemStatusIndicationFull           // icon、文字、背景同时标识highlight状态和disable状态
};
```

_WDToolBarItemStatusIndicationFull_ 只用于 *强引导样式*

WDToolBarItemIconPos

如果按钮中同时存在icon和text，指定 *icon相对于text* 的位置
```
typedef NS_ENUM(NSUInteger, WDToolBarItemIconPos) {
    WDToolBarItemIconPosNone = 222,
    WDToolBarItemIconPosUp,
    WDToolBarItemIconPosLeft,
    WDToolBarItemIconPosDown,
    WDToolBarItemIconPosRight,
    WDToolBarItemIconPosBack,
};
```

### Class

#### WDToolBarItem

##### Property

tag，用于delegate中标识当前点击的是哪个item
```
@property (nonatomic, copy) NSString *tag;
```
iconName，如果有图片，指定图片的iconfont icon name。不支持使用UIImage，因为要在不同的按钮状态下改变icon颜色
```
@property (nonatomic, copy) NSString *iconName; // iconfont icon name
```
name，按钮文案
```
@property (nonatomic, copy) NSString *name;
```
enabled，按钮是否可点击，默认YES
```
@property (nonatomic, assign) BOOL enabled;
```
iconColor，normal状态下图标颜色，pressed和disabled状态颜色会自动生成

默认 _SMTColorNameNoColorGray5_
```
@property (nonatomic, assign) SMTColorName iconColor;
```
textColor，normal状态下文字颜色，pressed和disabled状态颜色会自动生成

默认 _SMTColorNameNoColorGray5_
```
@property (nonatomic, assign) SMTColorName textColor;
```
backgroundColor，normal状态下按钮背景色，pressed和disabled状态颜色会自动生成

默认白色
```
@property (nonatomic, assign) SMTColorName backgroundColor;
```
iconPos，icon相对与文字的位置，支持上、下、左、右、后
```
@property (nonatomic, assign) WDToolBarItemIconPos iconPos;
```
showLeftVerticalSeparator，是否显示按钮左边的灰色竖向分割线
```
@property (nonatomic, assign) BOOL showLeftVerticalSeparator;
```
statusIndicationType，如何标识按钮状态，具体参照枚举 _WDToolBarItemStatusIndicationType_

默认 _WDToolBarItemStatusIndicationDefault_
```
@property (nonatomic, assign) WDToolBarItemStatusIndicationType statusIndicationType;
```

#### WDToolBar

##### Property

```
@property (nonatomic, weak) id <WDToolBarDelegate> delegate;
```

是否显示顶部阴影，默认YES
```
@property (nonatomic, assign) BOOL showTopShadow;
```

##### 实例方法

指定toolbarItems，不进行iPhone X自适应
```
- (instancetype)initWithToolBarItems:(NSArray<WDToolBarItem *> *)toolbarItems;
```
指定toolbarItems和toolBarPos，toolBarPos定义参考枚举 _WDToolBarPos_
```
- (instancetype)initWithToolBarItems:(NSArray<WDToolBarItem *> *)toolbarItems toolBarPos:(WDToolBarPos)toolBarPos;
```
使用一堆新的toolBarItems刷新toolBar
```
- (void)refreshWithToolBarItems:(NSArray<WDToolBarItem *> *)toolBarItems;
```
设置enabled状态，若index < 0，则设置全部
```
- (void)setItemEnabled:(BOOL)enabled atIndex:(NSInteger)index;
```

### Delegate

#### WDToolBarDelegate

toolBar被点击时的回调
```
- (void)toolBar:(WDToolBar *)toolBar itemDidPressed:(WDToolBarItem *)toolBarItem;
```