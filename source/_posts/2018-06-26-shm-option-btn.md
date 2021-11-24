---
title: UI组件-Button-WDOptionButton
date: 2018-06-26 10:57:15
toc: YES
tags:
- 组件
- UI组件
---

用于多项选择的按钮，全局适用

<!--more-->

## 尺寸

高度: 26

宽度: 初始为文字宽度左右各*+10* (自动宽度)

最小宽度: 不限

自适应文字宽度规则: 文字宽度两边各*+10*

圆角: 2.6

文字大小: 14

## 状态

未选中、选中、未选中无效 (不可选)、选中无效

![shm-option-btn](http://img.jokinryou.online/shm-option-btn.png)


## 使用示范

![shm-option-btn-example](http://img.jokinryou.online/shm-option-btn-example.png)

## 代码

### Enumeration

WDOptionButtonStatus

按钮状态
```
typedef NS_ENUM(NSUInteger, WDOptionButtonStatus) {
    WDOptionButtonStatusUnselected = 666,
    WDOptionButtonStatusSelected,
    WDOptionButtonStatusInvalid,
    WDOptionButtonStatusSelectedInvalid,
};
```

### Class

#### WDOptionButton

##### Property

设置按钮状态，设置之后改变按钮文字、背景颜色
```
@property (nonatomic, assign) WDOptionButtonStatus buttonStatus;
```
设置按钮文字，按钮大小会自适应文字
```
@property (nonatomic, copy) NSString *buttonText;
```

##### 类方法

```
+ (instancetype)buttonWithStatus:(WDOptionButtonStatus)buttonStatus text:(NSString *)text;
```