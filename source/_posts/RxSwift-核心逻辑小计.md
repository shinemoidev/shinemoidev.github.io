---
title: RxSwift 核心逻辑小计
date: 2021-12-31 14:40:26
tags:
---

RxSwift 是函数响应式编程框架。它可以帮助我们更方便的使用系统的 Target Action / 代理 / 闭包 / 通知 / KVO,同时还提供网络、数据绑定、UI事件处理、UI的展示和更新、多线程等功能。

RxSwift 的核心逻辑：

创建序列 -> 订阅 -> 发送信号 -> 响应事件 -> 销毁

这篇文章从一个最简单的事件订阅来一步步学习在 RxSwift 中事件的流动及响应。

[RxSwift 中文文档](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/rxswift_core/observable.html)

<!--more-->

首先先看一下几个基本的定义


``` Observable
public class Observable<Element> : ObservableType {
    init() {
    }
    
    public func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
        rxAbstractMethod()
    }
    
    public func asObservable() -> Observable<Element> { self }
}
```

``` ObservableType
public protocol ObservableType: ObservableConvertibleType {
    func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element
}

extension ObservableType {
    
    /// Default implementation of converting `ObservableType` to `Observable`.
    public func asObservable() -> Observable<Element> {
        // temporary workaround
        //return Observable.create(subscribe: self.subscribe)
        Observable.create { o in self.subscribe(o) }
    }
}
```

``` ObservableConvertibleType
public protocol ObservableConvertibleType {
    /// Type of elements in sequence.
    associatedtype Element

    func asObservable() -> Observable<Element>
}
```



创建一个 Observable 实例对象，并创建一个事件，后续观察并做响应处理。

```
// 创建被观察者 Observable 
let observable = Observable<String>.create { anyObservable -> Disposable in
    anyObservable.onNext("发送事件")
    anyObservable.onCompleted()
    return Disposables.create()
}

// 订阅被观察者
observable.subscribe { element in
    print("收到响应")
}.disposed(by: disposeBag)
```

下面进入 RxSwift 源码。

1. 创建被观察者 Observable 

创建 Observable 的方法，是来自 ObservableType 的扩展方法。返回了一个 AnonymousObservable 对象，它保存了发送事件闭包。

``` Create
extension ObservableType {
    // MARK: create

    public static func create(_ subscribe: @escaping (AnyObserver<Element>) -> Disposable) -> Observable<Element> {
        AnonymousObservable(subscribe)
    }
}
```

``` AnonymousObservable
final private class AnonymousObservable<Element>: Producer<Element> {
    ...
    init(_ subscribeHandler: @escaping SubscribeHandler) {
        self._subscribeHandler = subscribeHandler
    }
}
```

到这里，事件已经创建好了，但是由于没有对事件进行订阅 subscribe，所以现在目前仅仅是将事件的闭包保存在了 AnonymousObservable 对象中。

2. 订阅被观察者

```
observable.subscribe { element in
    print("收到响应")
}
```
进行订阅时，订阅对象就是上方创建的 AnonymousObservable 对象。

但是，AnonymousObservable 并没有 subscribe 方法。这时我们往上去父类查找。

AnonymousObservable -> Producer -> Observable -> ObservableType

最终可以在 ObservableType+Extensions.swift 文件中找到需要的方法实现：

```
extension ObservableType {
    public func subscribe(_ on: @escaping (Event<Element>) -> Void) -> Disposable {
        // step 1
        let observer = AnonymousObserver { e in
            on(e)
        }
        // step 2
        return self.asObservable().subscribe(observer)
    }
}
```

```
final class AnonymousObserver<Element>: ObserverBase<Element> {
    ...
    
    init(_ eventHandler: @escaping EventHandler) {

        self.eventHandler = eventHandler // 保存了响应事件的闭包。
    }
    ...
}
```

subscribe 方法有三个，会根据创建时传的参数不用选择使用对应的方法，但是处理流程都是一样的。这里选择了最简单的一个方法作为演示。

step 1: 
    方法中，创建了一个匿名观察者 AnonymousObserver 对象，并保存了响应事件的闭包。

step 2: 
    这里的 self 其实就是前面创建的 AnonymousObservable 对象。
    1. 首先 asObservable() 方法，就是 ObservableType 默认实现的一个父类（ObservableConvertibleType）的协议方法，具体实现在上方。得到的还是一个 AnonymousObservable 对象。
    2. AnonymousObservable 调用 subscribe 方法，并将生成的 observer 对象传入。

AnonymousObservable 调用 subscribe 方法，其实调用的是其父类 Producer 的 subscribe 方法。

```
class Producer<Element>: Observable<Element> {
    ...
    override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
        if !CurrentThreadScheduler.isScheduleRequired {
            // The returned disposable needs to release all references once it was disposed.
            let disposer = SinkDisposer()
            let sinkAndSubscription = self.run(observer, cancel: disposer)
            disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)

            return disposer
        }
        else {
            return CurrentThreadScheduler.instance.schedule(()) { _ in
                let disposer = SinkDisposer()
                let sinkAndSubscription = self.run(observer, cancel: disposer)
                disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)

                return disposer
            }
        }
    }
    ...
}
```

Producer 最终的方法调用会来到 self.run(observer, cancel: disposer) 方法。而 AnonymousObservable 实现的 run 方法，于是又回到 AnonymousObservable 中来处理 run 方法。

```
final private class AnonymousObservable<Element>: Producer<Element> {
    ...
    override func run<Observer: ObserverType>(_ observer: Observer, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where Observer.Element == Element {
        let sink = AnonymousObservableSink(observer: observer, cancel: cancel)
        let subscription = sink.run(self)
        return (sink: sink, subscription: subscription)
    }
}
```

AnonymousObservable 在 run 方法中创建了 AnonymousObservableSink 对象，并保存了 observer 和 cancel 对象，再调用了 sink 的 run 方法。

```
final private class AnonymousObservableSink<Observer: ObserverType>: Sink<Observer>, ObserverType {
    ...
    func run(_ parent: Parent) -> Disposable {
        parent.subscribeHandler(AnyObserver(self))
    }
}
```

parent 即为传进来的 AnonymousObservable 对象，然后创建 AnyObserver（保存 AnonymousObservableSink.on 函数），并利用 AnonymousObservable 保存的事件闭包 subscribeHandler 发送到外面：

```
public struct AnyObserver<Element> : ObserverType {
    ...
    public init<O : ObserverType>(_ observer: O) where O.E == Element {
        self.observer = observer.on
    }
    ...
}
```

因为 AnonymousObservableSink 的 run 方法调用了我们创建事件的闭包，于是这个时候来到我们最初的地方。 开始执行 onNext 方法，闭包内的参数 anyObservable 就是 sink 对象中创建的 AnyObserver 对象。

```
let observable = Observable<String>.create { anyObservable -> Disposable in
    anyObservable.onNext("发送事件")
    anyObservable.onCompleted()
    return Disposables.create()
}
```

AnyObserver 实现了 ObserverType 协议

``` ObserverType
public protocol ObserverType {
    func on(_ event: Event<Element>)
}

/// Convenience API extensions to provide alternate next, error, completed events
extension ObserverType {
    
    public func onNext(_ element: Element) {
        self.on(.next(element))
    }
    
    public func onCompleted() {
        self.on(.completed)
    }
    
    public func onError(_ error: Swift.Error) {
        self.on(.error(error))
    }
}
```

AnyObserver.onNext => AnyObserver.on。  AnyObserver 实现了 on 方法：

```
public struct AnyObserver<Element> : ObserverType {
    ...
    public func on(_ event: Event<Element>) {
        return self.observer(event)
    }
    ...
}
```

self.observer 就是创建实例时传入的 AnonymousObservableSink.on 函数。

```
final private class AnonymousObservableSink<O: ObserverType>: Sink<O>, ObserverType {
    ...
    func on(_ event: Event<E>) {
        ...
        switch event {
        case .next:
            if load(self._isStopped) == 1 {
                return
            }
            self.forwardOn(event)
        case .error, .completed:
            if fetchOr(self._isStopped, 1) == 0 {
                self.forwardOn(event)
                self.dispose()
            }
        }
    }
    ...
}
```

根据传入的 event 类型 .next 来到对应 case 逻辑。 开始执行 self.forwardOn(event) 方法。 AnonymousObservableSink 并没有这个方法，于是去父类中查找。

```
class Sink<O : ObserverType> : Disposable {
    ...
    final func forwardOn(_ event: Event<O.E>) {
        ...
        if isFlagSet(self._disposed, 1) {
            return
        }
        self._observer.on(event)
    }
    ...
}
```

self._observer 便是 AnonymousObservableSink 保存的 AnonymousObserver（这个 AnonymousObserver 对象就是订阅时生成的保存了事件响应闭包的对象）。调用它的 on 方法，并没有对应的实现，根据套路在父类 ObserverBase 中找到：

```
class ObserverBase<Element> : Disposable, ObserverType {
    ...
    func on(_ event: Event<Element>) {
        switch event {
        case .next:
            if load(self.isStopped) == 0 {
                self.onCore(event)
            }
        case .error, .completed:
            if fetchOr(self.isStopped, 1) == 0 {
                self.onCore(event)
            }
        }
    }
    ...
}
```

self.onCore(event) 方法被调用，AnonymousObserver 实现了，于是回到 AnonymousObserver 中：

```
final class AnonymousObserver<Element>: ObserverBase<Element> {
    ...
    override func onCore(_ event: Event<Element>) {
        self.eventHandler(event)
    }
    ...
}
```

self._eventHandler 便是 AnonymousObserver 保存的事件响应的闭包。最终，事件来到我们定义的响应处理的闭包中处理。

## 总结

1. 泛型的使用

2. 用了大量协议来组织对象

3. 协议的默认实现

4. 函数是一等公民 - 函数式编程

   * 函数可以存储在变量中
   * 函数作为参数
   * 函数作为返回值

