---
title: UI组件-WDActionAlertView
date: 2018-10-15 16:51:06
tags:
toc: true
---

行为很接近系统的UIAlertController，使用UIView实现，做了一些样式的自定义。

宽度**固定**，高度自增长

<!--more-->

## 支持的页面元素

![action-alert-view-elements](http://img.jokinryou.online/action-alert-view-elements.png)

各元素可任意添加与组合

## 接口

### WDActionAlertView.h

```objc
// 初始化方法，title 和 message 都可为nil
+ (instancetype)alertViewWithTitle:(NSString *)title message:(NSString *)message;
```

```objc
// 添加一个action，样式固定，action按照添加顺序**从左到右**排列
- (void)addAction:(WDActionAlertViewAction *)action;
```

```objc
// 添加取消action，样式固定
// 不论什么时候添加，调用此方法添加的action一定在**最左边**
// 多次调用，只有 __最后一次有效__
// title为nil或者@""时，默认为 "取消"
- (void)addCancelActionWithTitle:(NSString *)title handler:(WDActionAlertViewActionHandler)handler;
```

```objc
// 添加确定action，样式固定，粗体，品牌色
// 不论什么时候添加，调用此方法添加的action一定在**最右边**
// 多次调用，只有 __最后一次有效__
// title为nil或者@""时，默认为 "确定"
- (void)addOKActionWithTitle:(NSString *)title handler:(WDActionAlertViewActionHandler)handler;
```

```objc
// 添加一个textField，可以在configurator中配置textField
// 多次调用，添加多个textField
- (void)addTextFieldWithConfigurator:(WDActionAlertViewInputViewConfig *(^)())configurator;
```

```objc
// 添加一个textView，可以在configurator中配置textView
// 多次调用，添加多个textView
- (void)addTextViewWithConfigurator:(WDActionAlertViewInputViewConfig *(^)())configurator;
```

```objc
// 获取输入控件中的文字。当只有一个输入控件时，使用此方法快速获取输入的文字
- (NSString *)inputtedText;
```

```objc
// 通过identifier获取输入的文字
// 当同时有多个textField或者textView时，使用此方法获取输入的文字
- (NSString *)textWithIdentifier:(NSString *)identifier;
```

```objc
// 显示右上角的CloseButton，默认不显示
- (void)showCloseButton;
```

```objc
// 所有的元素添加完成后，显示actionAlertView
- (void)show;
```

```objc
// 手动让actionAlertView消失
- (void)dismissAnimated:(BOOL)animated completion:(void(^)())completion;
```

```objc
// 获取当前是否已有在显示的actionAlertView
+ (BOOL)hasVisibleAlertView;
```

### WDActionAlertViewAction

#### Property

```objc
// 自定义文字颜色
@property (nonatomic, strong) UIColor *titleColor;

// 指定action是否可点击，默认YES
@property (nonatomic, assign) BOOL enabled;

// 是否使用粗体
@property (nonatomic, assign) BOOL useBoldFont;

@property (nonatomic, copy, readonly) NSString *actionTitle;

@property (nonatomic, copy, readonly) WDActionAlertViewActionHandler handler;
```

#### Method

```objc
// 初始化方法
+ (instancetype)actionWithTitle:(NSString *)title handler:(WDActionAlertViewActionHandler)handler;

// 这个方法调用方不需要关心，内部用来break retain cycle
- (void)clearHandler;
```

### WDActionAlertViewInputViewConfig

#### Property

```objc
// 输入控件的唯一标识，会自动生成一串随机数，也可由外部指定
// 用于存在多输入控件时获取指定控件内输入的文字
@property (nonatomic, copy) NSString *identifier;

@property (nonatomic, copy) NSString *placeholder;
@property (nonatomic, copy) NSString *text;

// 指定输入控件可输入的文字最大长度
@property (nonatomic, assign) NSUInteger maxLength;
// 当达到最大长度时，弹出的提示信息，默认为nil，不弹
@property (nonatomic, copy) NSString *exceedMaxErrorMessage;

@property (nonatomic, assign) UIKeyboardType keyboardType;
@property (nonatomic, assign) UIReturnKeyType returnKeyType;

@property (nonatomic, strong) UIFont *font;

// 当actionAlertView显示出来时，该输入控件是否自动获取焦点，弹出键盘
// 当设置多个时，最后一个有效
@property (nonatomic, assign) BOOL autoFocus;
// 是否在输入秘密，文字密文显示
@property (nonatomic, assign) BOOL secureTextEntry;
```

#### Method

```objc

// 建议使用这两个方法获取对应的config
// 可以自动生成identifier
// 字号指定为15
// placeholder指定为 "请输入"

+ (instancetype)defaultTextFieldConfig;

+ (instancetype)defaultTextViewConfig;
```

### Demo

#### 一般用法

![](http://img.jokinryou.online/action-alert-view-general-usage.png)

#### 有一个输入控件时

![](http://img.jokinryou.online/action-alert-view-single-input.png)

#### 有多个输入控件时

![](http://img.jokinryou.online/action-alert-view-multi-inputs.png)

## What's more

规范里还有一类样式目前没有实现，毕竟目前好像还没有需求，就是这种

![](http://img.jokinryou.online/action-alert-view-to-be-implemented.png)
![](http://img.jokinryou.online/action-alert-view-to-be-implemented-02.png)

需要的话，你们跟我(xujl)说，我再去写