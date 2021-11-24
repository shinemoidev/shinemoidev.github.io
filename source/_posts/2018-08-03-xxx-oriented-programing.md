---
title: 面向xxx写代码
date: 2018-08-03 19:12:11
tags:
toc: true
---

这篇文章比较抽象，大家在看的时候多搜一下相关概念尽量去理解。

<!--more-->

## 基本概念

[维基百科-面向对象程序设计](https://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1)

## 面向对象的主要特征

### 封装

封装将对象有关的数据和行为封装成整体来处理，可以避免外部对象修改对象内部的状态，从而保持对象本身的稳定性，因此在代码编写过程中，要尽量**考虑到对象的每个属性，不需要暴露的尽量不要暴露**

### 继承

使子类具有父类的属性和方法，这样可以最大程度地重用代码

### 多态

当两个类具有继承关系时，子类可以通过重写父类的方法，以实现调用同一个方法时产生不同运行的结果

## 面向对象有多好

1. 对象易于理解和抽象，面向对象很容易把现实世界反映到计算机领域，从而方便设计

2. 能够非常方便地进行代码复用

3. 能够非常方便地应对复杂代码（扩展性、开放性）

面向对象这么好，为什么我们用起来却感觉没有那么爽？

## SOLID (面向对象设计原则)

### S (单一职责原则)

> SRP, Single Responsibility Principle

一个类有且仅有一个职责，只有一个引起它变化的原因

简单来说，一个类只做好一件事儿就行了，不要管跟自己不相关的。核心是高内聚与低耦合。

这个原则非常容易理解，也非常容易违背。最初在创建一个类的时候，我们都不自觉的会遵守这个原则，让这个类的功能相对单一。随着逻辑开始变得复杂，原来功能单一类需要被细化成粒度更细的两个或者多个职责，这个时候我们应该重新梳理，重构之前写的代码，将不同的职责封装到不同的类中。但是，由于时间关系或者是其他什么原因，就继续在原来的类里写新的代码，“一不小心”就违背了这个原则。

比如说

```objc
@interface WDLoginManager : NSObject

- (void)updateUnreadMessageNumber:(long)unreadNumber; // 同步消息未读数到服务端
- (void)asyncFetchFreeInfoWithUserId:(NSString *)userId; // 查询短信/商务电话余量

@end
```

两个完全不相关的功能在一个类里。

- 同步未读数逻辑变更，导致WDLoginManager变更
- 查询余量接口变更，导致WDLoginManager变更

这就违反了单一职责原则。所以，应该将不同的功能拆分到不同的类中，来做各自的事儿。

当然，拆分的粒度要具体情况具体分析，不要为了设计而设计

### O (开放关闭原则，开闭原则)

> OCP, Open Closed Principle

一个软件的实体比如类，模块和函数应该对扩展开放，而对修改关闭。我们应该通过扩展来实现变化，而不是通过修改原有的代码来实现变化，该原则是面向对象设计最基本的原则。

目前来看，在我们的工程中，每当需求变更，通常情况下我们需要修改大量的代码来满足需求，很大程度上就是因为我们对这个原则理解的不够透彻。

开闭原则的关键在于抽象，我们需要抽象出那些不会变化或者基本不变的东西，这部分相对稳定，也就是对修改关闭的地方；对于容易变化的部分，也进行适度的封装，但是这部分是可以相对自由修改的，通过面向对象的继承和多态机制，可以实现对抽象的继承，通过重写其方法改变固有行为，实现新的扩展方法，这就是对扩展开放的地方。

实现开闭原则的重点是：

- 充分利用抽象和封装，抽象出相对稳定的接口，封装变化
- 使用面向接口编程

比如说

```objc
// in file WDToolBarButton.m
- (void)updateConstraints {
    id <WDToolBarBtnViewLayout> layout = nil;
    if (!self.toolbarItem.iconName.length) {
        layout = [[WDToolBarBtnViewIconPosNoneLayout alloc] init];
    } else {
        switch (self.toolbarItem.iconPos) {
            case WDToolBarItemIconPosLeft: {
                layout = [[WDToolBarBtnViewIconPosLeftLayout alloc] init];
                break;
            }
            case WDToolBarItemIconPosUp: {
                layout = [[WDToolBarBtnViewIconPosUpLayout alloc] init];
                break;
            }
            case WDToolBarItemIconPosDown: {
                layout = [[WDToolBarBtnViewIconPosDownLayout alloc] init];
                break;
            }
            case WDToolBarItemIconPosRight: {
                layout = [[WDToolBarBtnViewIconPosRightLayout alloc] init];
                break;
            }
            case WDToolBarItemIconPosBack: {
                layout = [[WDToolBarBtnViewIconPosBackLayout alloc] init];
                break;
            }
            default: {
                layout = [[WDToolBarBtnViewIconPosNoneLayout alloc] init];
                break;
            }
        }
    }

    [layout layoutView:self.elementsView withColorScheme:self.colorScheme sizeScheme:self.sizeScheme];

    [super updateConstraints];
}

```

我们需要根据当前 **self.toolbarItem.iconPos** 来确定Button里的图片和文字要怎么布局。此时，对于WDToolBarButton来说，不需要知道具体要怎么对图片和文字进行排版，只要给一个合适的遵循 **WDToolBarBtnViewLayout** 协议的对象，让该对象去布局即可。

布局的行为是要做的，而这个行为是相对稳定的，而具体怎么做却是变化的（UI调整）。当我们要改变布局方式时，只要去对应的layout对象里修改即可。

### L (里氏替换原则)

> LSP, Liskov Substitution Principle

所有使用基类代码的地方，如果换成子类对象的时候还能够正常运行，则满足这个原则，否则就是继承关系有问题，应该废除两者的继承关系，这个原则可以用来判断我们的对象继承关系是否合理。

里氏替换原则包含以下4层含义：

- 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法
- 子类中可以增加自己特有的方法
- 当子类的方法重载父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松
- 当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格

两个问题：

- WDSelctConfig及其子类，是否满足里氏替换原则？

- [是否与多态违背？](https://www.zhihu.com/question/27191817?rf=27191795)

在实际编程中，我们常常会通过重写父类的方法来完成新的功能，这样写起来虽然简单，但是整个继承体系的可复用性会比较差，特别是运用多态比较频繁时，程序运行出错的几率非常大。如果非要重写父类的方法，比较通用的做法是：原来的父类和子类都继承一个更通俗的基类，原有的继承关系去掉，采用依赖、聚合、组合等关系代替。

现实是，我们在自己的编程中常常会违反里氏替换原则，程序照样跑的好好的。所以，你说要遵循里氏替换原则，我不，我就不，会咋样？

### I (接口隔离原则)

> ISP, Interface Segregation Principle

客户端不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上。

看个例子

![shm-xxxop-isp-01](http://img.jokinryou.online/shm-oop-isp-01.png)

最开始，*Class A* 依赖 _Interface I_ 提供的接口 `abs_func_1` `abs_func_2` `abs_func_3`，我们建立了 *Class C* 实现了这三个方法，完美。

有一天，我们有了另外一个类 *Class B*，它也依赖接口 `abs_func_1`，同时，它又会依赖 `abs_func_4` 和 `abs_func_5`，为了写得快一点，我们就把 `abs_func_4` 和 `abs_func_5` 顺手写在了 *Interface I* 里，同时用了新的 *Class D* 来实现 `abs_func_1` `abs_func_4` `abs_func_5`。此时 *Class D* 也不得不实现 `abs_func_2` 和 `abs_func_3`，或者简单地写一个空实现； *Class C* 也不得不实现 `abs_func_4` 和 `abs_func_5`，或者也是简单的空实现。也就是说 *Class C* 和 *Class D* 中，都存在了他们用不到的方法。

所以当接口过于臃肿时，只要接口中出现的方法，不管对依赖于它的类有没有用处，实现类中都必须去实现这些方法，这显然不是好的设计。

OC里有 **@option** 关键字可以标识选择性实现，但是会给客户端造成困扰，我到底该不该实现这个 **@option** 标记的接口。

为了遵循接口隔离原则，我们需要对 *Interface I* 进行拆分，设计如下：

![shm-xxxop-isp-02](http://img.jokinryou.online/shm-oop-isp-02.png)

接口隔离原则的含义是：建立单一接口，不要建立庞大臃肿的接口，尽量细化接口，接口中的方法尽量少。也就是说，我们要为各个类建立专用的接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用。在程序设计中，依赖几个专用的接口要比依赖一个综合的接口更灵活。接口是设计时对外部设定的“契约”，通过分散定义多个接口，可以预防外来变更的扩散，提高系统的灵活性和可维护性。

要注意：

- 虽然说接口尽量小，但也要有限度。对接口进行细化可以提高程序设计的灵活性是事实，但是如果过小，会造成接口数量过多，使设计复杂化。所以一定要适度

- 为依赖接口的类定制服务，只暴露给调用方需要的方法，调用方不需要的方法隐藏起来。只有专注地为一个模块提供服务，才能建立最小的依赖

### D (依赖倒置原则)

> DIP, Dependence Inversion Principle

问题由来：

类A直接依赖于类B，假如要将类A修改为依赖类C，则必须通过修改类A的代码来达成。这种场景下，类A一般是高层模块，负责复杂的业务逻辑。类B和C是底层模块，负责基本的原子操作。假如修改类A，将会给程序带来不必要的风险。

什么是依赖倒置：

- 高层模块不应该依赖低层模块，两者都应该依赖其抽象

- 抽象不应该依赖细节

- 细节应该依赖抽象

抽象：即抽象类或接口（OC里就是protocol），都是不能被实例化的

细节：即具体的实现类，遵循协议并实现协议定义的接口的类，可以通过 `[[xxx alloc] init]` 直接被实例化

依赖倒置原则基于这样一个事实：相对于细节的多变性，抽象的东西要稳定的多。以抽象为基础搭建起来的架构比以细节为基础搭建起来的架构要稳定的多。

所以，依赖倒置原则的核心思想是 **面向接口编程**。在OC中， 也就是 **面向协议编程**。使用协议制定好规范和契约，而不去涉及任何具体的操作，把展示细节的任务交给协议的实现类去做。

举个例子：

```objc
@interface Ultron : NSObject

- (NSString *)name;

@end

@implementation Ultron

- (NSString *)name {
    return @"Ultron";
}

@end

@interface IronMan : NSObject

- (void)fightWithUltron:(Ultron *)Ultron;

@end

@implementation IronMan

- (void)fightWithUltron:(Ultron *)ultron {
    NSLog(@"IronMan suit up and fight with %@", ultron.name);
}

@end

IronMan *ironMan = [[IronMan alloc] init];
Ultron *ultron = [[Ultron alloc] init];
[ironMan fightWithUltron:ultron];

```

运行结果： IronMan suit up and fight with Ultron

代码很简单，但实际上这是一个非常脆弱的设计。现在钢铁侠可以跟奥创打，但是，现在灭霸来了，作为漫威英雄，不能只打奥创不打灭霸吧，不管结果怎么样，出于角色设定，打还是要打的。但是现在打不了，怎么办？因为上面的代码，钢铁侠依赖奥创，所以导致如果要让钢铁侠能打灭霸，只能去修改钢铁侠的代码。那下次，又来一个反派呢，再修改钢铁侠的代码吗？改着改着就会发现，钢铁侠最后连小辣椒都打。这种处理方式显然是不可取的，频繁修改会带来很大的系统风险。

问题的根源，就是钢铁侠依赖于奥创，两者是紧耦合关系，其导致的结果就是系统的可维护性大大降低。

根据依赖倒置原则，我们对以上代码进行修改，提取抽象部分。钢铁侠是漫威英雄，奥创是漫威反派，所以我们提取出两个协议 **MarvelHero** 和 **MarvelVillain**，都提供各自必须的抽象方法，这样以后美队、雷神这样的漫威英雄都可以和奥创、灭霸这样的反派打，到底怎么打，则由具体的实现类负责。由于遵循依赖倒置原则，只依赖于抽象，不依赖于细节，所以增加类无需修改其他类。

修改后的代码：

```objc
// 漫威英雄
@protocol MarvelHero <NSObject>

// 出于角色设定，都要跟反派打
- (void)fightWithVillain:(MarvelVillain)marvelVillain;

@end

// 漫威反派
@protocol MarvelVillain <NSObject>

- (NSString)name; // 反派都有名字

@end

// I'm IronMan
@interface IronMan : NSObject <MarvelHero>

@end

@implementation IronMan

- (void)fightWithVillain:(MarvelVillain)marvelVillain {
    NSLog(@"IronMan suit up and fight with %@", marvelVillain.name);
}

@end

// 奥创
@interface Ultron : NSObject <MarvelVillian>

@end

@implementation Ultron

- (NSString *)name {
    return @"Ultron";
}

@end

// 灭霸
@interface Thanos : NSObject <MarvelVillian>

@end

@implementation Thanos

- (NSString *)name {
    return @"Thanos";
}

@end

id <MarvelHero> ironMan = [[IronMan alloc] init];
id <MarvelVillian> ultron = [[Ultron alloc] init];
id <MarvelVillian> thanos = [[Thanos alloc] init]; // 这里符合里氏替换原则
[ironMan fightWithVillian:ultron];
[ironMan fightWithVillian:thanos];

```

运行结果：

IronMan suit up and fight with Ultron

IronMan suit up and fight with Thanos

- **MarvelHero** 是复杂的业务逻辑，属于高层模块，而 **MarvelVillian** 是原子模块，属于低层模块。MarvelHero 依赖于抽象的 MarvelVillian 接口，这就做到了高层模块不应该依赖低层模块，两者都应该依赖于抽象

- **MarvelHero** 和 **MarvelVillian** 接口与各自的实现都没有关系，增加实现类不会影响接口，这就做到了抽象不应该依赖于细节

- **IronMan**、 **Ultron**、 **Thanos** 实现类都要去实现各自协议定义的方法，所以是依赖于接口的。这就做到了细节应该依赖抽象

来看看什么是倒置

最开始，我们按照正常人的思维，钢铁侠要打奥创，打就打，要打灭霸，打就打，编程也是这样，都是按照面向实现的思维方式来设计。而现在要倒置思维，提取公共的抽象，面向协议（接口）编程。不再依赖于具体实现了，而是依赖于协议，这就是依赖的思维方式“倒置”了。

实现的方式

- 接口定义中使用抽象类

- 构造方法传递抽象类

- setter方法中传递抽象类

说到底，依赖倒置原则的核心就是面向接口（协议）编程的思想，尽量对每个实现类都提取抽象和公共接口形成接口或抽象类，依赖于抽象而不要依赖于具体实现。依赖倒置原则的本质其实就是通过抽象（抽象类或接口）使各个类或模块的实现彼此独立，不相互影响，实现模块间的松耦合。但是这个原则也是5个设计原则中最难以实现的了，如果没有实现这个原则，那么也就意味着开闭原则也无法实现。

## 最后

个人认为，面向对象和面向接口（协议）是不可分割的。我们在平时写代码的时候，要有意识地按照以上5个原则做设计做编码，看到不爽的地方，适当进行重构，这样，在需求不断变化的情况下，我们也能最大限度地保持代码的可维护性、可扩展性。
