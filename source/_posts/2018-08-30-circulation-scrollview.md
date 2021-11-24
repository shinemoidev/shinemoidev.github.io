---
title: UI组件-WDCirculationScrollview
date: 2018-08-30 15:14:17
tags:
- 组件
- UI组件
---
轮播图片显示控件

### 使用
```
WDCirculationScrollView *circuScrollView = [WDCirculationScrollView alloc] init];
[self.contentView addSubview:circuScrollView];
circuScrollView.frame = ...
circuScrollView.imageUrls = ...
```
支持initWithFrame,setFrame,autolayout布局

### 属性
```
@property (nonatomic, strong) NSArray<NSString *> *imageUrls;

//刷新pageControl等
@property (nonatomic, copy) void(^pageChangedBlock)(NSInteger currentPage, NSInteger numOfPages);
//点击图片
@property (nonatomic, copy) void(^selectAImageBlock)(NSInteger index);

//个性化滚动imageview
@property (nonatomic, copy) void(^customerAImageFrame)(UIImageView *imageView, NSInteger index);

//自动滚动时使用
@property (nonatomic, copy) void(^startOrPauseTimerBlock)(BOOL canStart);
```


#### imageUrls
当然是指要显示的图片的网络地址列表

#### pageChangedBlock
回调告诉外部，图片个数或者当前图片变化。外部界面进行处理，例如更新pagecontrol的当前页和页面个数。

#### selectAImageBlock
点击某张图片回调，外部进行页面跳转等。

#### customerAImageFrame
个性化图片展示，比如 设置图片展示的大小，圆角，contentMode等(默认 UIViewContentModeScaleAspectFill --铺满裁剪)

#### startOrPauseTimerBlock
停止和启动定时器回调。由于在图片onDrag的时候需要暂停，onDragEnd的时候继续timer,所以需要自动轮播的需要设置该回调去启动和暂停timer

#### 方法:scrollToNext
外部调用该方法表示滚动到下一个。比如自动轮播时，定时调用来自动翻到下一张。

## 关于属性注入的感想
以需要(frame, imageUrls, pageChangedBlock, selectAImageBlock)四个参数为例：
如果用构造函数来注入这些参数。因为都是可选的，我们要写以下构造函数来实现：
initWithFrame:
initWithImageUrls:
initWithPageChangedBlock:
initWithSelectAImageBlock:
initWithFrame: imageUrls: pageChangedBlock: selectAImageBlock:
initWithFrame: imageUrls: pageChangedBlock:
initWithFrame: imageUrls: 
initWithFrame: pageChangedBlock: 
initWithFrame: pageChangedBlock: selectAImageBlock:
initWithFrame: imageUrls: selectAImageBlock:
...
等 C(4,0) + C(4,1) + C(4,2) + C(4,3) + C(4,4) = 1 + 4 + 6 + 4 + 1 = 16 个方法满足。4参数个就需要16个之多。

*但是用属性方式 这16个都是可以省的。*

这里不得不提下swift构造函数：
```
init(frame:CGRect = .zero, imageUrls:Array<String>? = nil, pageChangedBlock:((_ currentPage:Int, _ numOfPages:Int) -> Void)? = nil, selectAImageBlock:((_ index:Int) -> Void)? = nil) {
        super.init()
```
只要一个就够了。

可以这样用：
```
let circuScrollView = WDCirculationScrollView(imageUrls: nil)
```
也可以这样用：
```
let circuScrollView = WDCirculationScrollView(imageUrls: nil, selectAImageBlock: nil)
```
只是不能换着顺序用：
比如
```
let circuScrollView = WDCirculationScrollView(selectAImageBlock: nil, imageUrls: nil)
```
是不行的。
