---
title: 做一个自己的私有pod
date: 2018-11-13 15:34:22
tags:
toc : true
---

## 前言

在代码优化的过程中，难免要把某一个功能模块或者一个通用组件做成 pod。而这些 pod 只适用于我们自己的工程，放到 CocoaPods 的官方库里显然是不太合适的，所以我们需要做成私有的 pod，以便于我们自己的项目使用。

什么都不要管，先添加我们自己的 repo

```bash
pod repo add SHMPods ssh://git@git.shinemo.com:7999/shmpodspecs/shmpods.git
```

查看是否添加成功
```bash
ls ~/.cocoapods/repos
```

## 如何创建一个私有pod

CocoaPods 官方提供了非常方便的命令让我们来创建 pod

```bash
pod lib create SHMPrivatePodDemo
```

执行命令之后，等待模板工程 clone 完毕，脚本会问你几个问题

> What platform do you want to use?? [ iOS / macOS ]
> 根据实际情况回答，一般是 iOS

> What language do you want to use?? [ Swift / ObjC ]
> 根据实际情况回答，一般是 ObjC

> Would you like to include a demo application with your library? [ Yes / No ]
> Yes，需要这个 demo 工程，做 pod 引用测试

> Which testing frameworks will you use? [ Specta / Kiwi / None ]
> 需要哪个测试框架，根据个人需要选定

> Would you like to do view based testing? [ Yes / No ]
> 是否引入 [_FBSnapshotTestCase_](https://github.com/facebookarchive/ios-snapshot-test-case) 测试框架，一般可能不太需要

> What is your class prefix?
> 类前缀，如果 pod 是公司级别的，用SHM/SMT，如果是项目级别的，用各自项目的前缀

问题回答完了，脚本会使用 Xcode 自动打开你的 demo 工程。

更多细节请参考 [Using Pod Lib Create](https://guides.cocoapods.org/making/using-pod-lib-create.html)，一定要看！！！

脚本创建的目录结构如图：

![](http://img.jokinryou.online/shm_private-dir-structure.png)

pod 库创建完成后，去 [SHMPodSpecs](https://git.shinemo.com/projects/SHMPODSPECS) 创建一个同名的project

## Pod 开发

注意，我们**不使用**脚本自动创建的 Example 工程进行 Pod 的开发，原因如下：

![](http://img.jokinryou.online/shm-private-pod-01.png)

当我们在如图所示的目录下创建目录或者文件时，目录或文件所在的物理位置，与我们的预期并不一致。这是由于该目录指向的是  _./SHMPrivatePodDemo_，而非 _./SHMPrivatePodDemo/SHMPrivatePodDemo_ （参考目录结构图）。而我们希望我们新建的源码相关的目录和文件在目录 _./SHMPrivatePodDemo/SHMPrivatePodDemo/Classes_ 下，资源文件在 _./SHMPrivatePodDemo/SHMPrivatePodDemo/Assets_ 下。（一般来说是这样的，也不一定非要在这两个目录下，只要 **podspec** 文件里配置正确就可以）

关掉脚本自动打开的 Example 工程，我们再新建一个工程。

工程名字命名为 _SHMPrivatePodDemoDevProj_ (命名规则：libNameDevProj)

![](http://img.jokinryou.online/shm-private-pod-create-proj-01.png)

路径为 _./SHMPrivatePodDemo_
![](http://img.jokinryou.online/shm-private-pod-create-proj-02.png)

DevProj 工程可能需要引入其他的 pod，所以我们直接以集成 CocoaPods 的形式使用 DevProj
```bash
cd SHMPrivatePodDemo/SHMPrivatePodDemoDevProj

pod init
pod install

open SHMPrivatePodDemoDevProj.xcworkspace
```

然后，我们把目录 _Assets_ 和 _Classes_ 两个目录拖到 **DevProj** 工程里
注意！ **不要** 勾选 _Copy items if needed_ ！！！
![](http://img.jokinryou.online/shm-private-pod-add-dirs.png)

之后，就可以愉快地添加你想要的 groups 和 files 了。资源文件要放在 _Assets_ 目录。

代码修改完成后，编辑 _README.md_ 和 _SHMPrivatePodDemo.podspec_ 两个文件

### README.md

我们自己的 pod 库，没有那么严格，只保留关键部分即可，如图
![](http://img.jokinryou.online/shm-private-pod-readme-1.png)

一些细节需要根据自己的私有pod情况进行调整，比如说需要 import 哪些头文件、有哪些 subspec 之类的

### SHMPrivatePodDemo.podspec

对于一个 pod 来说，这个文件相当重要。这个文件里的参数，定义了这个 pod 所需的一切。

具体设置请参考[官方文档](https://guides.cocoapods.org/syntax/podspec.html)

相关的 URL 可在 git.shinemo 的工程页面获取

注意，在开发过程中，所有用到的 pod, framework 和 lib，都要在podspec文件里写明（DevProj 里用到的）

`s.version`修改为此次发布想要的版本号（[版本号规则](https://semver.org/lang/zh-CN/)）

两个文件修改完成后，进入 Example 目录，执行 `pod install`，做 pod 引入测试，测试通过后，准备发布

## 发布

### 验证 podspec 文件是否正确

```bash
pod lib lint SHMPrivatePodDemo.podspec
```

验证结果若有任何 error，则验证不通过，需要修改直至验证通过
允许有 warning，可以使用`--allow-warnings`参数忽略警告，但是不建议这么做，尽量保证没有 warning

```bash
pod lib lint SHMPrivatePodDemo.podspec --allow-warnings
```

### 更新 SHMPods

#### 更新git代码库

##### 直接提交

当 gerrit 里没有加过当前这个工程，或者是首次提交时

```bash
git add .
git commit -s
git pull --rebase origin master (git remote add origin remote_repo.git, 如果是首次提交)
git push origin master
```

##### Code Review

```bash
git add .
git commit -s
git pull --rebase origin branchName
git review branchName
```

以下动作需要等 Code Review 通过之后再做

#### 打tag 

```bash
git tag 1.0.0 (podspec里s.version定义的版本号)
git push origin 1.0.0
```
参考[版本号规则](https://semver.org/lang/zh-CN/)

#### 上传 podspec 到 SHMPods

```bash
pod repo push SHMPods SHMPrivatePodDemo.podspec
```
注意，这个过程中 pod 会重新验证 podspec 文件的正确性，所以如果在`pod lib lint`的时候使用了`--allow-warnings`参数才验证通过，这里仍然需要`--allow-warnings`参数


注意， `pod lib lint`验证 podspec 文件的时候，podspec 文件中定义的版本号是不起作用的。但是做`pod repo push`的时候，pod 会先验证 podspec 文件里指定的 git 库里是否有`s.version`指定的 tag，所以要先打 tag

## 代码修改/版本更新

1. 打开 SHMPrivatePodDevProj.xcworkspace 修改代码
2. 修改 podspec 文件 (version, source, dependency, lib, framework等)
3. 打开 Example 工程，做 pod 引入测试
4. 重新走一遍 [**发布**](#发布) 流程
5. 到引用 pod 的工程里（一般是项目主工程），做`pod update SHMPrivatePodDemo`

## 私有库怎么用

用法主要参考各 pod 的 README，一般有以下两种

### 指定git路径

```ruby
pod 'SHMPrivatePodDemo', :git => 'git_remote_repo.git'
```

这种用法**只能**用在 Podfile 里，始终使用远端代码库里 master 分支的最新代码

### 通过指定source

在 Podfile 的最顶部增加两行

```ruby
source 'https://github.com/CocoaPods/Specs.git'
source 'ssh://git@git.shinemo.com:7999/shmpodspecs/shmpods.git'
```

然后，我们就可以像使用官方 pod 一样，直接指定版本号即可

```ruby
pod 'SHMPrivatePodDemo', '~> 1.0.0'
```

这种用法要求该私有 pod 的 podspec 文件已经被 push 到 SHMPods 这个 repo 里
这个方法可以在 Podfile 里使用，也可以在 podspec 文件里作为`s.dependency`被使用

更多 Podfile 配置请参考[Podfile Syntax Reference](https://guides.cocoapods.org/syntax/podfile.html)

## 私有 pod 引用私有 pod

podspec 文件中只能使用`s.dependency`的方式引用其他 pod
```ruby
s.dependency 'SHMPrivatePodDemo', '~> 1.0.0'
s.dependency 'SHMUtils/NSString', '~> 1.0.3'
```



所以在 DevProj 的 Podfile 里，也建议使用指定版本号的方式引用其他 pod，这样可以保证开发过程和最终提交当前私有 pod 代码时，他们依赖的其他 pod 的版本是一致的

在 [**发布**](#发布) 过程中，我们需要对podspec文件进行`pod lib lint`和`pod repo push`，然而，在这两个过程中，pod只会去官方库里找我们定义的`s.dependency`是否存在，当我们引用了其他私有pod时，podspec文件验证就无法通过。而 podspec 文件中无法指定`repo source`，所以，我们只能在命令里指定`sources`

```bash
pod lib lint SHMPrivatePodDemo.podspec --sources=https://github.com/CocoaPods/Specs.git,ssh://git@git.shinemo.com:7999/shmpodspecs/shmpods.git --allow-warnings

pod repo push SHMPods SHMPrivatePodDemo.podspec --sources=https://github.com/CocoaPods/Specs.git,ssh://git@git.shinemo.com:7999/shmpodspecs/shmpods.git --allow-warnings
```