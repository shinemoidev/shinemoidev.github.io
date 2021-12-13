---
title: SwiftUI 与 UIKit 混合开发简述
date: 2021-12-13 14:46:28
tags:
---

SwiftUI 中有很多官方的组件已经非常好用，但是在一些特殊的场景中还是需要使用 UIKit 的组件提供支持，这篇文章简单介绍一些如何让 SwiftUI 和 UIKit 的组件混合开发的方法。

在真正使用 SwiftUI 开发时，还是推荐尽量使用原有组件来完成功能。

<!--more-->

<-- more -->

## UIKit in SwiftUI

#### UIView

要在 SwiftUI 中使用 UIKit 的话，需要用 **UIViewRepresentable** 对 UIView 进行一层包装。

且 UIView 需要使用 **typealias** 指定一个需要包装的 UIView 类型，并实现两个方法：

* func makeUIView(context: Self.Context) -> Self.UIViewType
  
创建一个 View 并返回。可在这里设置默认的一些属性及状态。

* func updateUIView(_ uiView: Self.UIViewType, context: Self.Context)

在 SwiftUI 中可改变的属性在这里进行设置及刷新 View 的状态。

需要暴露的、可设置的属性，就在 View 中声明一个属性，供 SwiftUI 使用时进行调整。

* 例子1。常规显示的 View。

```
// Mark :- UIkit Define
struct AAView: UIViewRepresentable {
    typealias UIViewType = UIProgressView
    
    @Binding var progress: Float // SwiftUI 中设置
    
    func makeUIView(context: Context) -> UIProgressView {
        let view = UIProgressView()
        return view
    }
    
    func updateUIView(_ uiView: UIViewType, context: Context) {
        uiView.progress = progress
    }
}

// Mark :- SwiftUI
struct ContentView: View {
    @State var progress: Float = 0.5
    
    var body: some View {
        AAView(progress: $progress)
    }
}
```

* 例子2。包装的 View 中有 Target/Action 事件。
```
// Mark :- UIkit Define
struct AASwitchView: UIViewRepresentable {
    typealias UIViewType = UISwitch
    
    @Binding var isOn: Bool
    
    func makeUIView(context: Context) -> UISwitch {
        let view = UISwitch()
        view.addTarget(self, action: #selector(Coordinator.valueChanged(sender:)), for: .valueChanged)
        return view
    }
    
    func updateUIView(_ uiView: UIViewType, context: Context) {
        uiView.isOn = isOn
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject {
        var view: AASwitchView
        
        init(_ view: AASwitchView) {
            self.view = view
        }
        
        @objc func valueChanged(sender: UISwitch) {
            view.isOn = sender.isOn
        }
    }
}

// Mark :- SwiftUI
struct ContentView: View {    
    @State var isOn = false
    
    var body: some View {        
        AASwitchView(isOn: $isOn)
    }
}
```

#### UIViewController

要在 SwiftUI 中使用 UIViewController 的话，需要用 **UIViewControllerRepresentable** 对 UIViewController 进行一层包装。

且 UIView 需要使用 **typealias** 指定一个需要包装的 UIView 类型，并实现两个方法：

* func makeUIViewController(context: Self.Context) -> Self.UIViewControllerType
  
创建一个 UIViewController 并返回。可在这里设置默认的一些属性及状态。

* func updateUIViewController(_ uiViewController: Self.UIViewControllerType, context: Self.Context)

在 SwiftUI 中可改变的属性在这里进行设置及刷新 UIViewController 的状态。

需要暴露的、可设置的属性，就在 View 中声明一个属性，供 SwiftUI 使用时进行调整。

```
struct AANavViewController: UIViewControllerRepresentable {
    typealias UIViewControllerType = UINavigationController
    
    var root: UIViewController
    var title: String
    
    func makeUIViewController(context: Context) -> UINavigationController {
        let vc = UINavigationController()
        return vc
    }
    
    func updateUIViewController(_ uiViewController: UINavigationController, context: Context) {
        uiViewController.topViewController?.title = title
    }
}

struct ContentView: View {
    var body: some View {
        AANavViewController(root: UIViewController(), title: "Title")
    }
}
```

## SwiftUI in UIkit

SwiftUI 中的控件想在 UIKit 中使用的话相对来说比较简单。只需要通过 **UIHostingController** 包装一次即可。

```
let vc = UIHostingController(rootView: Text("Text"))
```

这边需要注意的是，取到的是一个 UIViewController 对象。