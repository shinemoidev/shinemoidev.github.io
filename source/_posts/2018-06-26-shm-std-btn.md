---
title: UI组件-Button-WDStandardButton
date: 2018-06-26 09:20:57
toc: YES
tags:
- 组件
- UI组件
---

标准按钮控件，全局适用

<!--more-->

## 尺寸 (WDStdBtnSizeType)

### 大 (WDStdBtnSizeTypeBig)

高度: 50

宽度: 初始为距离屏幕两边*15/20* (视手机屏幕尺寸而定)，可以使用 _resizeToFitText_ 调整为适应文字宽度

最小宽度: 不限

自适应文字宽度规则: 文字宽度两边各*+30*

圆角: 5

文字大小: 18

![shm-std-btn-size-big](http://img.jokinryou.online/shm-std-btn-size-big.png)

### 中 (WDStdBtnSizeTypeMiddle)

高度: 40

宽度: 初始为距离屏幕两边*15/20* (视手机屏幕尺寸而定)，可以使用_resizeToFitText_调整为适应文字宽度

最小宽度: 140

自适应文字宽度规则: 文字宽度两边各*+30*

圆角: 4

文字大小: 16

![shm-std-btn-size-mid](http://img.jokinryou.online/shm-std-btn-size-mid.png)

### 小 (WDStdBtnSizeTypeSmall)

高度: 26

宽度: 初始为文字宽度左右各*+10* (自动宽度)

最小宽度: 45

自适应文字宽度规则: 文字宽度两边各*+10*

圆角: 2.6

文字大小: 14

![shm-std-btn-size-small](http://img.jokinryou.online/shm-std-btn-size-small.png)


## 基础色 (WDStdBtnBaseColorType)

基础色是按钮在normal状态下的颜色，同时用于按照不同的颜色策略，合成pressed和disabled状态的颜色

默认: 品牌色

## 颜色策略 (WDStdBtnColorSynScheme)

### 主 (WDStdBtnColorSynSchemeMajor)

![shm-std-btn-color-major](http://img.jokinryou.online/shm-std-btn-color-major.png)

### 次 (WDStdBtnColorSynSchemeMinor)

![shm-std-btn-color-minor](http://img.jokinryou.online/shm-std-btn-color-minor.png)

### 灰 (WDStdBtnColorSynSchemeGray)

![shm-std-btn-color-gray](http://img.jokinryou.online/shm-std-btn-color-gray.png)

## 反色按钮

多用在深色背景下，尺寸根据 _WDStdBtnSizeType_ 确定，有自己的颜色合成策略

![shm-std-btn-color-inverse](http://img.jokinryou.online/shm-std-btn-color-inverse.png)

## 代码

### Protocol

WDStdBtnColorConfig

颜色配置协议，用于指定按钮颜色、文字颜色等

WDStdBtnSizeConfig

尺寸配置协议，用于指定按钮高度、宽度、圆角、文字尺寸、按钮是否自适应文字大小等

### Enumeration

WDStdBtnColorSynScheme

指定按钮各种状态颜色策略

```
typedef NS_ENUM(NSUInteger, WDStdBtnColorSynScheme) {
    WDStdBtnColorSynSchemeMajor = 111,
    WDStdBtnColorSynSchemeMinor,
    WDStdBtnColorSynSchemeGray,
};
```

---
WDStdBtnSizeType

指定按钮尺寸类型

```
typedef NS_ENUM(NSUInteger, WDStdBtnSizeType) {
    WDStdBtnSizeTypeBig = 999,
    WDStdBtnSizeTypeMiddle,
    WDStdBtnSizeTypeSmall,
};
```

---
WDStdBtnBaseColorType

指定按钮基础色

```
typedef NS_ENUM(NSUInteger, WDStdBtnBaseColorType) {
    WDStdBtnBaseColorBrand = 777,
    WDStdBtnBaseColorSubLink,
    WDStdBtnBaseColorSuccess,
    WDStdBtnBaseColorVIP,
    WDStdBtnBaseColorCaution,
    WDStdBtnBaseColorYellow,
    WDStdBtnBaseColorRedPacket,
};
```

### Class

#### WDStandardButton

##### 类方法

指定颜色策略、尺寸类型
```
+ (instancetype)buttonWithColorSynScheme:(WDStdBtnColorSynScheme)colorSynScheme sizeType:(WDStdBtnSizeType)sizeType;
```
指定基础色、颜色策略、尺寸类型
```
+ (instancetype)buttonWithBaseColorType:(WDStdBtnBaseColorType)baseColorType colorSynScheme:(WDStdBtnColorSynScheme)colorSynScheme sizeType:(WDStdBtnSizeType)sizeType;
```
反色按钮初始化方法
```
+ (instancetype)inverseButtonWithBaseColorName:(SMTColorName)baseColorName colorType:(WDStdBtnColorSynScheme)colorType sizeType:(WDStdBtnSizeType)sizeType;
```
以品牌色为基础色，使用指定的颜色策略生成的颜色配置
```
+ (id <WDStdBtnColorConfig>)colorConfigWithColorSynScheme:(WDStdBtnColorSynScheme)colorSynScheme;
```

##### 实例方法

使用自定义的颜色配置和尺寸配置初始化按钮
```
- (instancetype)initWithColorConfig:(id <WDStdBtnColorConfig>)colorConfig sizeConfig:(id <WDStdBtnSizeConfig>)sizeConfig;
```
自动调整按钮尺寸为适应文字大小，内边距、最小、最大宽度遵循初始化时指定的尺寸类型 (尺寸配置)
```
- (void)resizeToFitText;
```
还原为按钮默认尺寸
```
- (void)restoreToDefaultSize;
```
指定新的颜色配置，刷新按钮
```
- (void)refreshWithColorConfig:(id<WDStdBtnColorConfig>)colorConfig;
```
当前按钮颜色配置
```
- (id <WDStdBtnColorConfig>)buttonColorConfig;
```

## 自定义

可自定义子类，遵循 _WDStdBtnColorConfig_ 或 _WDStdBtnSizeConfig_ ，实现协议里定义的方法，用于构造自定义按钮

参考工程中 *WDSelctOKBtnSizeConfig*