---
title: iOS函数响应式编程(一)
date: 2018-09-20 11:19:47
tags: 函数式编程 ReactiveObjc
toc: true
---

# iOS函数响应式编程

## 函数响应式编程简介
函数式编程想必您一定听过，但响应式编程的说法就不大常见了。与响应式编程对应的命令式编程就是大家所熟知的一种编程范式，我们先来看一段代码：
```
int a = 3;
int b = 4;
int c = a + b;
NSLog(@"c is %d", c); // => 7
a = 5;
b = 7;
NSLog(@"c is %d", c); // 仍然是7
```
命令式编程就是通过表达式或语句来改变状态量，例如c = a + b就是一个表达式，它创建了一个名称为c的状态量，其值为a与b的加和。下面的a = 5是另一个语句，它改变了a的值，但这时c是没有变化的。所以命令式编程中c = a + b只是一个瞬时的过程，而不是一个关系描述。在传统的开发中，想让c跟随a和b的变化而变化是比较困难的。而让c的值实时等于a与b的加和的编程方式就是响应式编程。

<!--more-->

实际上，在日常的开发中我们会经常使用响应式编程的思想来进行开发。最典型的例子就是Excel，当我们在一个B1单元格上书写一个公式“=A1+5”时，便声明了一种对应关系，每当A1单元格发生变化时，单元格B2都会随之改变。

iOS开发中也有响应式编程的典型例子，例如Autolayout。我们通过设置约束描述了各个视图的位置关系，一旦其中一个变化，另一个就会响应其变化。类似的例子还有很多。

函数响应式编程（英文Functional Reactive Programming，以下简称FRP，）正是在函数式编程的基础之上，增加了响应式的支持。

简单来讲，FRP是基于异步事件流进行编程的一种编程范式。针对离散事件序列进行有效的封装，利用函数式编程的思想，满足响应式编程的需要。

区别于面向过程编程范式以过程单元作为核心组成部分，面向对象编程范式以对象单元作为核心组成部分，函数式编程范式以函数和高阶函数作为核心组成部分。FRP则以离散有序序列作为核心组成部分，也可将其定义为信号。其特点是具备可迭代特性并且允许离散事件节点有时间联系.

## iOS项目的函数响应式编程选型
很长一段时间以来，iOS项目并没有很好的FRP支持，直到iOS 4.0 SDK中增加了Block语法才为函数式编程提供了前置条件，FRP开源库也逐步健全起来。

最先与大家见面的莫过于ReactiveCocoa这样一个库了，ReactiveCocoa是Github在制作Github客户端时开源的一个副产物，缩写为RAC。它是Objective-C语言下FRP思想的一个优秀实例，后续版本也支持了Swift语言。

gitlab地址: https://github.com/ReactiveCocoa/ReactiveObjC

iOS的项目主要以客户端项目为主，主要的业务场景就是进行页面交互和与服务器拉取数据，这里面会包含多种事件和异步处理逻辑。FRP本身就是面向事件流的一种开发方式，又擅长处理异步逻辑。所以从逻辑上是满足iOS客户端业务需要的。

然而能够把一个理念融合到实际的项目中，需要一个漫长的过程。

## ReactiveCocoa 试图解决什么问题
1. 传统 iOS 开发过程中，状态以及状态之间依赖过多的问题
2. 传统 MVC 架构的问题：Controller比较复杂，可测试性差
3. 提供统一的消息传递机制


## 一步一步进行函数响应式编程
### 统一回调

写法不统一聚焦在回调形式的不统一上，iOS中的回调方式有非常多的种类：UIKit主要进行的事件处理target-action、跨类依赖推荐的delegate模式、iOS 4.0纳入的block、利用通知中心（Notifcation Center）进行松耦合的回调、利用键值观察（Key-Value Observe，简称KVO）进行的监听。由于场景不同，选用的规则也不尽相同，并且我们没有办法很好的界定什么场景该写什么样的回调。

看下面的例子:

```
// 代替target-action
    [[self.confirmButton rac_signalForControlEvents:UIControlEventTouchUpInside]
     subscribeNext:^(id x) {
        // 回调内容写在这里
     }];
    
// 代替delegate
    [[self rac_signalForSelector:@selector(scrollViewDidScroll:) fromProtocol:@protocol(UIScrollViewDelegate)]
     subscribeNext:^(id x) {
         // 回调内容写在这里
         NSLog(@"view did scroll");
     }];
    //切记要讲下面这句话放在注册delegate的后面,否则会引起回调无法被调用.
    self.scrollView.delegate = self;
    
// 代替block
    [[self asyncProcess]
     subscribeNext:^(id x) {
        // 回调内容写在这里
     } error:^(NSError *error) {
        // 错误处理写到这里
     }];
    
// 代替notification center
    [[[NSNotificationCenter defaultCenter] rac_valuesForKeyPath:@"Some-key" observer:nil]
     subscribeNext:^(id x) {
        // 回调内容写在这里
     }];

// 代替KVO
    [RACObserve(self, userReportString)
     subscribeNext:^(id x) {
        // 回调内容写在这里
     }];
```

### 信号的使用
#### 信号的概念
作为RAC中最为核心的一个类，信号可以理解为传递数据变化信息的工具，信号会在数据发生变化时发送事件流给它的订阅者，然后订阅者执行响应方法。信号本身不具备发送信号的能力，而是交给一个订阅者去发出。

测试场景：我们要对一个用于输入用户名的UITextFiled进行检测，每次输入内容变化的时候都打出输入框的内容，使用RAC来实现此操作的关键代码如下：
```
[self.userNameTxtField.rac_textSignal subscribeNext:^(NSString * _Nullable x) {
        NSLog(@"测试：%@",x);
}];
```
#### ReactiveCocoa信号机制
我们会对上面的代码产生疑问，RAC是怎么做到上述代码功能的呢？而且我们常说的订阅者又在哪里呢？

其实RAC已经使用Category的形式为我们基本的UI控件创建了信号(如上例中的rac_textSignal)，所以这里我们才可以很方便的实现信号订阅，而且订阅者在整个过程中也是对于我们隐藏的。 现在我们使用自定义信号的方法，从创建信号到订阅信号细致的了解一下这个过程。首先上一段创建信号的测试代码如下：

```
//创建信号
RACSignal *testSignal = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
    //1.订阅者发送信号内容
    [subscriber sendNext:@"发送信号内容"];
    //2.订阅者发送信号完成的信息，不需要再发送数据时，最好发送信号完成，可以内部调起清理信号的操作。
    [subscriber sendCompleted];
    //3.创建信号的Block参数，需要返回一个RACDisposable对象 ，可以返回nil。
    //RACDisposable对象用于取消订阅信号，此block在信号完成或者错误时调用。
    RACDisposable *racDisposable = [RACDisposable disposableWithBlock:^{
       NSLog(@"信号Error或者Complete时销毁");
    }];
    return racDisposable;
}];
    
//订阅信号
[testSignal subscribeNext:^(id  _Nullable x) {
    //新变化的值
    NSLog(@"订阅信号：subscribeNext:%@",x);
} error:^(NSError * _Nullable error) {
    //信号错误，被取消订阅,被移除观察
    NSLog(@"订阅信号：Error:%@",error.description);
} completed:^{
    //信号已经完成，被取消订阅，被移除观察
    NSLog(@"订阅信号：subscribeComplete");
}];
```
```
控制台打印:
2018-09-17 14:17:31.837292+0800 FRPSamples[10409:262829] 订阅信号：subscribeNext:发送信号内容
2018-09-17 14:17:31.837462+0800 FRPSamples[10409:262829] 订阅信号：subscribeComplete
2018-09-17 14:17:31.837618+0800 FRPSamples[10409:262829] 信号Error或者Complete时销毁
```

#### 创建信号
创建信号，我们需要使用RACSignal的类方法createSignal。该方法需要一个Block作为参数。查看源码，我们就会发现RACSignal最终是通过调用自己子类RACDynamicSignal的createSignal方法，将这个Block设置给了自己的didSubscribe属性的。

```
//RACSignal.m文件
+ (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe {
 	return [RACDynamicSignal createSignal:didSubscribe];
}
```

```
//RACDynamicSignal.h文件
@interface RACDynamicSignal ()
// The block to invoke for each subscriber.
@property (nonatomic, copy, readonly) RACDisposable * (^didSubscribe)(id<RACSubscriber> subscriber);
@end
```

```
//RACDynamicSignal.m文件
+ (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe {
 	RACDynamicSignal *signal = [[self alloc] init];
 	signal->_didSubscribe = [didSubscribe copy];
 	return [signal setNameWithFormat:@"+createSignal:"];
}
```

**didSubscribe**：这是创建信号时候需要传入的一个block，它的传入参数是订阅者subscriber，而返回值是需要是一个RACDisposable对象。创建信号后的didSubscrib是一个等待执行的block。

**RACSubscriber**：表示订阅者，创建信号时订阅者发送信号，这里的订阅者是一个协议而非一个类。信号需要订阅者帮助其发送数据。查看RACSubscriber的协议，我可以看到以下几个方法：

```
//发送信息
- (void)sendNext:(nullable id)value;
//发送错误消息
- (void)sendError:(nullable NSError *)error;
//发送完成信息
- (void)sendCompleted;
//
- (void)didSubscribeWithDisposable:(RACCompoundDisposable *)disposable;
```

在创建一个信号的时候，订阅者使用sendNext发送信息。而且如果我们不再发送数据，最好在这里执行一次sendCompleted方法，这样的话，信号内部会自动调用对应的方法取消信号订阅。

RACDisposable：这个类用于取消订阅信号和清理资源，在信号出现错误或者信号完成的时候，信号会自动调起RACDisposable对象的block方法。在代码中我们也可以看到，创建RACDisposable对象是使用disposableWithBlock方法设置了一个block操作，执行block操作之后，信号就不再被订阅了。

**总结**：创建信号就是使用createSignal方法，创建一个信号，并为信号设置了一个didSubscribe属性(也就是一系列订阅者需要做的操作)。

#### 订阅信号

进入订阅信号的源码我们看到如下代码：
```
- (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock error:(void (^)(NSError *error))errorBlock completed:(void (^)(void))completedBlock {
 	NSCParameterAssert(nextBlock != NULL);
 	NSCParameterAssert(errorBlock != NULL);
 	NSCParameterAssert(completedBlock != NULL);
 	
 	RACSubscriber *o = [RACSubscriber subscriberWithNext:nextBlock error:errorBlock completed:completedBlock];
 	return [self subscribe:o];
}
```

在此方法中，我们可以看到订阅信号有两个过程：

**过程1：使用subscribeNext的方法参数，创建出一个订阅者subscriber。**

**过程2：信号对象执行了订阅操作subscribe，方法中传入参数是刚创建的订阅者。**

注：这也就解释了我们常提起却看不见的订阅者存在哪里的问题。真实开发中我们只关心订阅者需要发送的值就行了，而不需要关心其内部订阅的过程。

继续打开信号的subscribe方法，看到源码如下：

```
- (RACDisposable *)subscribe:(id<RACSubscriber>)subscriber {
 	NSCParameterAssert(subscriber != nil);
 	RACCompoundDisposable *disposable = [RACCompoundDisposable compoundDisposable];
 	subscriber = [[RACPassthroughSubscriber alloc] initWithSubscriber:subscriber signal:self disposable:disposable];
 	if (self.didSubscribe != NULL) {
 		RACDisposable *schedulingDisposable = [RACScheduler.subscriptionScheduler schedule:^{
 			RACDisposable *innerDisposable = self.didSubscribe(subscriber);
 			[disposable addDisposable:innerDisposable];
 		}];
 		[disposable addDisposable:schedulingDisposable];
 	}
 	return disposable;
}
```

上面的代码中我们不难看出：**除了对于订阅者和清理对象的再次封装外，最重要的就是创建信号时为信号设置Block(didSubscribe)被调用了，而且Block参数使用了我们创建的订阅者。**


### 信号的操作
这里我们介绍一些关于信号的最常用的操作, 高级用法后续介绍.

#### map,flattenMap
**map的操作**
1. 传入一个block,类型是返回对象，参数是value;
2. value就是源信号的内容，直接拿到源信号的内容做处理;
3. 把处理好的内容，直接返回就好了，不用包装成信号，返回的值，就是映射的值。

```
[[_textField.rac_textSignal map:^id _Nullable(NSString * _Nullable value) {
    // 当源信号发出，就会调用这个block，修改源信号的内容
    // 返回值：就是处理完源信号的内容。
    return [NSString stringWithFormat:@"hello:%@",value];
}] subscribeNext:^(id  _Nullable x) {
    NSLog(@"%@",x); // hello: "x"
}];
```

**flattenMap的操作**

1.传入一个block，block类型是返回值RACStream，参数value；
2.参数value就是源信号的内容，拿到源信号的内容做处理；
3.包装成RACReturnSignal信号，返回出去。

```
[[_textField.rac_textSignal flattenMap:^__kindof RACSignal * _Nullable(NSString * _Nullable value) {
    return  [RACSignal return:[NSString stringWithFormat:@"hello %@", value]];
}] subscribeNext:^(id  _Nullable x) {
    NSLog(@"%@",x); // hello "x"
}];
```

**flatternMap和Map的区别**

所以flattenMap和map的区别在于，flattenMap的block参数返回一个“任意类型”信号RACSignal到bind内部去做addSignal(RACSignal)操作来对RACSignal进行订阅；
而map是限定flattenMap只能返回一个RACReturnSignal信号去bind内部做addSigna(RACReturenSignal)操作来对RACReturnSignal进行订阅，而对RACReturnSignal进行订阅只能获取RACReturnSignal内部携带的value值。

**小结**
1. FlatternMap中的Block返回信号
2. Map中的Block返回对象
3. 开发中，如果信号发出的值不是信号，映射一般使用Map
4. 开发中，如果信号发出的值是信号，映射一般使用FlatternMap



#### bind
flattenMap 的底层实现是通过bind实现的

Map 的底层实现是通过 flattenMap 实现的

1. 传入一个返回值RACSignalBindBlock的block;
2. 描述一个RACSignalBindBlock类型的bindBlock作为block的返回值;
3. 描述一个返回结果的信号，作为bindBlock的返回值. 
注意：在bindBlock中做信号结果的处理

```
[[_textField.rac_textSignal bind:^RACSignalBindBlock _Nonnull{
    return ^RACSignal*(id value, BOOL *stop){
        // 做好处理，通过信号返回出去.
        return [RACSignal return:[NSString stringWithFormat:@"hello: %@",value]];
    };
}] subscribeNext:^(id  _Nullable x) {
    NSLog(@"%@",x); // hello: "x"
}];
```

#### concat
按照从左到右的顺序拼接信号,当多个信号发出的时候,有顺序的接受信号.
```
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@1];
    [subscriber sendCompleted]; //注释这句signalB不会subscribeNext
    return nil;
}];

RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@2];
    [subscriber sendCompleted]; //注释这句concat之后不会onCompleted
    return nil;
}];

// 把signalA拼接到signalB后，signalA发送完成，signalB才会被激活
[[signalA concat:signalB] subscribeNext:^(id  _Nullable x) {
    NSLog(@"%@",x);
} completed:^{
    NSLog(@"complete.....");
}];
```

#### then
用于连接两个信号，当第一个信号完成，才会连接then返回的信号
**底层实现**

1. 使用concat连接then返回的信号
2. 先过滤掉之前的信号发出的值

```
RACSignal * signle = [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
    [subscriber sendNext:@"1"];
    [subscriber sendNext:@"2"];
    [subscriber sendCompleted];
    return nil;
}];


[[signle then:^RACSignal * _Nonnull{
    return [RACSignal createSignal:^RACDisposable * _Nullable(id<RACSubscriber>  _Nonnull subscriber) {
        [subscriber sendNext:@"3"];
        return nil;
    }];
}] subscribeNext:^(id  _Nullable x) {
    //then会把之前的信号过滤掉，所以这里只会接收到第二个信号的值 3
    NSLog(@"%@",x);
}];

[signle subscribeNext:^(id  _Nullable x) {
    //第一个信号订阅了，所以顺序执行到这里才会打印1，2
    NSLog(@"%@",x);
}];
```

#### merge
把多个信号合并为一个信号，任何一个信号有新值的时候就会调用
```
//创建多个信号
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@1];
    return nil;
}];

RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@2];
    return nil;
}];

// 合并信号,任何一个信号发送数据，都能监听到.
RACSignal *mergeSignal = [signalA merge:signalB];

[mergeSignal subscribeNext:^(id x) {
    NSLog(@"%@",x); //
}];
```

#### zipWith
把两个信号压缩成一个信号，只有当两个信号同时发出信号内容时，并且把两个信号的内容合并成一个元组，才会触发压缩流的next事件
```
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@1];
    return nil;
}];

RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@2];
    return nil;
}];

// 压缩信号A，信号B
RACSignal *zipSignal = [signalA zipWith:signalB];

[zipSignal subscribeNext:^(id x) {
    // x 为元祖 RACTuple
    NSLog(@"%@",x); // (1, 2) 
}];
```

#### combineLatest
将多个信号合并起来，并且拿到各个信号的最新的值,必须每个合并的signal至少都有过一次sendNext，才会触发合并的信号
```
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@1];
    return nil;
}];

RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@2];
    return nil;
}];

// 把两个信号组合成一个信号,跟zip一样，没什么区别
RACSignal *combineSignal = [signalA combineLatestWith:signalB];

[combineSignal subscribeNext:^(id x) {
    NSLog(@"%@",x); // (1, 2)
}];
```

#### reduce
用于信号发出的内容是元组，把信号发出元组的值聚合成一个值

一般都是先组合在聚合

```
RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@1];
    return nil;
}];

RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@2];
    return nil;
}];

// reduceblcok的返回值：聚合信号之后的内容。
RACSignal *reduceSignal = [RACSignal combineLatest:@[signalA,signalB] reduce:^id(NSNumber *num1 ,NSNumber *num2){
    return [NSString stringWithFormat:@"%@ %@",num1,num2];
}];

[reduceSignal subscribeNext:^(id x) {
    NSLog(@"%@",x); // 1 2
}];
```


#### filter
过滤信号，获取满足条件的信号
```
//获取到位数大于6的值
[[self.mainText.rac_textSignal filter:^BOOL(NSString *value) {
    return value.length > 6;
}] subscribeNext:^(NSString * _Nullable x) {
    NSLog(@"%@",x); // x 值位数大于6
}];
```

#### ignore
忽略掉指定的值
```
[[self.mainText.rac_textSignal ignore:@"666"] subscribeNext:^(id x) {
    NSLog(@"ignore onSubscribed %@",x);
}];
```

#### interval
类似于 NSTimer

```
//每隔1秒发送一次信号
[[RACSignal interval:1 onScheduler:[RACScheduler mainThreadScheduler]] subscribeNext:^(id x) {
    NSLog(@"interval fired %@",x);
}];
```
#### delay
延迟执行，类似于 GCD 的 after

```
[[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@1];
    return nil;
}] delay:2] subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];
```

#### skip
跳过第几个信号,接受后面的信号
```
[[[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@1];
    [subscriber sendNext:@2];
    [subscriber sendNext:@3];
    return nil;
}] skip:1] subscribeNext:^(id x) {
    NSLog(@"skip test %@",x); //跳过1
}];
```


#### take
从第一个信号开始开始一共取N次的信号
```
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [subscriber sendNext:@1];
    [subscriber sendNext:@2];
    [subscriber sendNext:@3];
    return nil;
}];
[[signal take:2] subscribeNext:^(id x) {
    NSLog(@"take test %@",x); //只接受1,2
}];
```

### 常用UI相关信号
#### rac_signalForControlEvents(UIControl)
用于监听控件某个事件, 当事件发生时会触发回调
```
[[self.loginBtn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(UIButton *sender) {
        NSLog(@"button clicked!");
}];
```

#### rac_imageSelectedSignal (UIImagePickerController)
选择图片的信号
```
self.imagePicker = [UIImagePickerController new];
[self.imagePicker.rac_imageSelectedSignal subscribeNext:^(id x) {
    //该block回调是在照片选择完成的时候调用
    @strongify(self);
    NSLog(@"%@",x);
    NSDictionary *dic = (NSDictionary *)x;
    self.myImageView.image = dic[@"UIImagePickerControllerOriginalImage"];
    [self.imagePicker dismissViewControllerAnimated:YES completion:nil];
}];
self.imagePicker.sourceType = UIImagePickerControllerSourceTypePhotoLibrary;
[self presentViewController:self.imagePicker animated:YES completion:nil];
```

#### rac_textSignal
textSignal 顾名思义就是关于某些控件关于text属性的信号,当控件的text属性发生变化的时候回通知所有的订阅者. 
```
RACSignal *accountValidSignal =
   [self.accountTV.rac_textSignal map:^id(id value) {
       return @(self.accountTV.text.length > 5);//转化为是否合法布尔值
}];
```

#### RAC宏
RAC宏用来订阅某个实例的成员属性的变化,例如:

```
RACSignal *accountValidSignal =
   [self.accountTV.rac_textSignal map:^id(id value) {
       return @(self.accountTV.text.length > 5);//转化为是否合法布尔值
}];
//根据 self.accountTV 控件是否合法来设置背景颜色
RAC(self.accountTV, backgroundColor) =
   [accountValidSignal map:^id(NSNumber* accountValid) {
  return accountValid.boolValue ? [UIColor whiteColor] : [UIColor redColor]; //转化为UIColor
}];
```

### 其他
#### 避免循环引用，外部@weakify(self)，内部@strongify(self)
```
// @weakify() 宏定义
    @weakify(self) //相当于__weak typeof(self) weakSelf = self;
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        @strongify(self)  //相当于__strong typeof(weakSelf) strongSelf = weakSelf;
        NSLog(@"%@",self.view);
        return nil;
    }];
```

### 常用RAC对象
见下一次分享



### 参考文章
https://blog.csdn.net/a709314090/article/details/53870398
http://williamzang.com/blog/2016/06/27/ios-kai-fa-xia-de-han-shu-xiang-ying-shi-bian-cheng/




