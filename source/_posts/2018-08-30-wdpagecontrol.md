---
title: UI组件-WDPageControl
date: 2018-08-30 14:20:58
tags:
- 组件
- UI组件
---
分页显示图标
### 使用示例

```
    //初始化
    WDPageControl *pageControl = [[WDPageControl alloc] init];
    [self.bgView addSubview:self.pageControl];

    //autolayout:只需要设置位置
    [self.pageControl mas_makeConstraints:^(MASConstraintMaker *make) {
        make.centerX.equalTo(self.bgView);
        make.bottom.equalTo(self.bgView).offset(-8);
    }];

    //设置样式。默认WDPageControlStyleDarkContent（白色背景）
    self.pageControl.controlStype = WDPageControlStyleDarkContent;

    //设置页面个数，当前页
    self.pageControl.numberOfPages = numOfPages;
    self.pageControl.currentPage = currentPage;
```
使用起来相当方便，不需要设置frame,由于重写了 *intrinsicContentSize* 方法。所以布局推荐使用autolayout,相当不建议用frame，intrinsicContentSize自动随着numberOfPages的改变而改变，并且当 *numberOfPages <= 0* 时自动隐藏。

### 扩展
新增样式
```
typedef NS_ENUM(NSInteger, WDPageControlStyle) {
    WDPageControlStyleDarkContent,//适用背景白色
    WDPageControlStyleLightContent,//适用背景非白色
    //需要新增样式扩展该值，并在 set controlStype 中修改不可见的属性
};
```



