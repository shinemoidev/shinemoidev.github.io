---
title: 讯盟 Objective-C 代码规范
date: 2018-02-06 10:01:35
toc: true
categories: 团队
tags:
- 其他
- iOS 
- 代码规范
---

对于团队，如果代码风格不统一，阅读或修改同事的代码会非常困难，造成潜在风险。

对于个人，代码规范是对自身编码习惯的一种监督，如果没有这种监督，有时候因为偷懒，会写出难看的代码，时间长了自己都看不懂。这样对于代码的维护是不利的。

每个人写代码都有自己的风格，一份统一的代码规范肯定不能让所有人都感觉非常舒服，但是当我们由个人变成团队，不再是为自己写代码时，请大家尽量遵守大家一起制定的代码规范。

<!--more-->

## 更新日志

2017-09-30

- [修改文件的组织](#fileOrganization)

2016-07-27

- [增加初始化方法中何时return的说明](#returnInInit)
- [私有方法命名](#namingMethods)
- 增加[TODO](#todo)格式说明
- 修改[命名常量](#namingConstants)规则
- 增加[代码注释](#codeComments)格式说明

2016-07-26

- 增加[switch](#switch)的相关约定
- 增加[.h文件](#headerFileFormat)格式

2016-07－20

- 增加目录
- [if-else中else的写法](#if-else-else-format)

## <span id = "reference">参考资料</span>

本文参考了以下文章，并做了适当修改，以使我们的代码规范能够具有普遍性和适当的个性

[Apple's Cocoa Coding Guidelines](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)

[Google's Open Source Objective-C Style Guid](https://google.github.io/styleguide/objcguide.xml?showone=Displaying_Hidden_Details_in_this_Guide#Displaying_Hidden_Details_in_this_Guide)

[Objective-C编码规范](http://www.jianshu.com/p/30ccbaf0f251#)


## <span id = "codeFormat">代码格式</span>

### <span id = "spaceAndTab">空格和制表符Tab</span>

只能使用空格来缩进，任何情况下都不要使用制表符Tab。在*Xcode > Preference > Text Editing*中将Tab和自动缩进都设置为4个空格。

### <span id = "lineMaxLength">行最大长度</span>

普遍来讲，代码行最大长度为80个字符，即每行代码包括缩进的空格在内不能多于80个字符。由于Objective-C特殊的语法，此处规定代码行最大长度为120个字符，可以在*Xcode > Preferences > Text Editing > Page guid at column: 120*中设置。

### <span id = "methodStyle">方法的书写</span>

在每个方法的定义前留白一行，也就是在方法和方法之间留空一行。

一个典型的Objective-C方法看起来应该是这样的：

*栗子*

```objc
- (void)doSomethingWithString:(NSString *)theString {
    //...do anything you want
}
```

`-` / `+` 和返回值 `(void)` 之间应该有一个空格。而 `(void)` 之后没有空格。第一个大括号 `{` 在函数名所在行的末尾，与最后一个参数之间有一个空格。

如果一个函数有很多参数或者名字很长，应该以 `:` 为基准多行对齐显示：

*栗子*

```objc
- (void)doSomethingWithFoo:(SMTFoo *)theFoo
                      rect:(CGRect)theRect
                  interval:(NSTimInterval)theInterval {

}
```

分行时，如果第一段名字过短，后续名称应该以4个空格为单位进行缩进：

*栗子*

```objc
- (void)shortFoo:(SMTFoo *)theFoo
    	  longKeyword:(CGRect)theRect
	evenLongerKeyword:(NSTimeInterval)theInterval
                error:(NSError **)error {
    //...
}
```

### <span id = "methodInvocation">方法调用</span>

函数调用和书写的格式差不多，可以按照函数的长短来选择写在一行或者分成多行。

写在一行：

*栗子*

```objc
[self doFooWithName:arg1 age:arg2 error:arg3];
```

分行写，按 *:* 对齐：

```objc
[self doFooWithName:arg1
                age:arg2
              error:arg3];
```

以下写法是错误的：

```objc
// some lines with >1 arg
[self doFooWithName:arg1 age:arg2
              error:arg3];

[self doFooWithName:arg1
                age:arg2 error:arg3];

// aligning keywords instead of colons
[self doFooWithName:arg1
      age:arg2
      error:arg3];
```

如果函数第一段较短，则以最长的一段左侧缩进4个空格为准，`:` 对齐：

```objc
[self shortFoo:arg1
                longKeyword:arg2
    evenLongerLongerKeyword:arg3
                      error:&arg4];
```

### <span id = "headerFileFormat">.h文件</span>

从上到下依次包括

.h文件中主要包括 _FOUNDATION_EXPORT部分_、 _@class部分_、 _typedef部分_、 _@protocol部分_、 _@interface部分_。每部分之间应该有一个空行。如果包含多个 _typedef_，则每个 _typedef_ 之间应该有一个空行。在 _@interface_ 中，有上到下依次为 _@property部分_、 _类方法部分_、 _实例方法部分_。

**注意栗子中的空行！！！**

栗子
```objc
#import "lib1.h"
#import "lib2.h"

@class WDClass1;
@class WDClass2;

// 如果有宏定义的话，在此位置
#define MACRONAME1 expression1
#define MACRONAME2 expression2

FOUNDATION_EXPORT NSString *const kWDGlobalVar1;
FOUNDATION_EXPORT NSString *const kWDGlobalVar2;

typedef NS_ENUM(NSInteger, WDEnum1) {
    WDEnum1Case0,
    WDEnum1Case1,
    WDEnum1Case2,
    WDEnum1Case3,
};

typedef NS_ENUM(NSInteger, WDEnum2) {
    WDEnum2Case0,
    WDEnum2Case1,
    WDEnum2Case2,
    WDEnum2Case3,
};

@interface WDClass : NSObject

@property (nonatomic, strong) NSString *var1;
@property (nonatomic, assign) BOOL var2;

// 这是变量的注释
@property (nonatomic, strong) NSObject *object;

+ (instancetype)classMethodName;
- (NSString *)instanceMethodName1;

// 这是方法的注释
- (CGFloat)instanceMethodName2;

@end

```


## <span id = "blocks">Blocks</span>

根据block的长度，有不同的书写规则：

* 不论block多短，就算是一行能放得下，也要分行写

* 如果分行显示的话，block的右括号 `}` 应该和调用block那行代码的第一个非空字符对齐

* Block内始终以4个空格缩进

* 如果block过于庞大，比如说多于20行，应该单独声明成一个变量来使用

* `^` 和 `(` 之间，`^` 和 `{` 之间都没有空格，参数列表的右括号 `)` 和 `{` 之间有一个空格

*栗子*

```objc
// Though the entire block can fit on one line, do a wrapping.
// The block can be put on a new line, indented four spaces, with the
// closing brace aligned with the first character of the line on which
// block was declared.
[operation doSomethingWithCompletionHandler:^{
	[self onOperationDone];
}];

// Using a block with a C API follows the same alignment and spacing
// rules as with Objective-C.
dispatch_async(myQueue, ^{
	// do what you want
});

// An example where the parameter wraps and the block declaration fits
// on the same line. Note the spacing of |^(SessionWindow *window) {|
// compared to |^{| above.
[[SessionService sharedService]
	loadWindowWithCompletionBlock:^(SessionWindow *window) {
        if (window) {
          [self windowDidLoad:window];
        } else {
          [self errorLoadingWindow];
        }
    }
];

// An example where the parameter wraps and the block declaration does
// not fit on the same line as the name.
[[SessionService sharedService]
    loadWindowWithCompletionBlock:
        ^(SessionWindow *window) {
            if (window) {
              [self windowDidLoad:window];
            } else {
              [self errorLoadingWindow];
            }
        }
];


// Large blocks can be declared out-of-line.
void (^largeBlock)(void) = ^{
    // ...
};
[_operationQueue addOperationWithBlock:largeBlock];

// An example with multiple inlined blocks in one invocation.
[myObject doSomethingWith:arg1
               firstBlock:^(Foo *a) {
                   // ...
               }
               secondBlock:^(Bar *b) {
                   // ...
               }
];
```

## <span id = "arrayAndDic">NSArray/NSDictionary语法糖</span>

应该用可读性更好的语法糖来构造 `NSArray` 和 `NSDictionary` 等数据结构，避免使用冗长的 `alloc`, `init` 方法。

如果构造代码写在一行，不需要在括号两端留空格：

*栗子*

```objc
NSArray *array = @[[foo description], @"Another String", [bar description]];
NSDictionary *dict = @{NSForegroundColorAttributeName: [NSColor redColor]};
```

*不需要留空格*

```objc
NSArray *array = @[ [foo description], @"Another String", [bar description] ];
NSDictionary *dict = @{ NSForegroundColorAttributeName : [NSColor redColor] };
```

如果构造代码不写在一行内，构造元素同样使用 *4个空格* 来进行缩进，右括号 `]` 或者 `}` 写在新的一行，并且与声明代码的第一行第一个非空字符对齐：

*栗子*

```objc
NSArray *array = @[
	@"This",
	@"is",
	@"an",
	@"array"
];

NSDictionary *dictionary = @{
	NSFontAttributeName: [NSFont fontWithName:@"Helvetica-Bold" size:12],
	NSForegroundColorAttributeName: fontColor
};


// Don't need to add spaces to align the params
NSDictionary *option2 = @{
  NSFontAttributeName :            [NSFont fontWithName:@"Arial" size:12],
  NSForegroundColorAttributeName : fontColor
};
```

严禁再使用以下方法初始化NSArray/NSDictionary

```objc
NSArray *array = [NSArray arrayWithObjects:@"obj1", @"obj2", nil];
NSDictionary *dict = [NSDictionary dictionaryWithObjectsAndKeys:...nil];
```

构造NSArray时使用 `<>` 语法表明元素类型

```objc
NSArray<WdOrgUser *> *users;
```

## <span id = "namingRules">命名规范</span>

### <span id = "namingBasic">基本原则</span>

采用[驼峰命名法](https://zh.wikipedia.org/wiki/%E9%A7%9D%E5%B3%B0%E5%BC%8F%E5%A4%A7%E5%B0%8F%E5%AF%AB)

#### <span id = "namingClear">清晰</span>

命名应该尽可能的清晰和简洁，但在Objective-C中，**清晰比简洁更重要**。由于Xcode强大的自动补全功能，我们不必担心名称过长的问题。

```objc
// 清晰
insertObject:atIndex:

// 不清晰，insert的对象类型和at的位置属性都没有说明
insert:at:

// 清晰
removeObjectAtIndex:

// 不清晰，remove的对象类型没有说明，参数的作用没有说明
remove:
```

不要使用单词的简写，拼写出完整的单词：

```objc
// 清晰
destinationSelection
setBackgroundColor:

// 不清晰，不要使用简写
destSel
setBkgdColor:
```

然而，有部分单词简写在Objective-C编码过程中非常常用，以至于成为了一种规范，这些简写可以在代码中直接使用，下面列举了部分：

| 全拼 | 简写 | 备注|
| :--- | :--- | :---: |
| Allocate | alloc | |
| Alternate| alt | |
| Application | app | |
| Calculate | calc | |
| Deallocate | dealloc | |
| Function | func | |
| Horizontal | horiz | |
| Vertical | vert | |
| Information | info | |
| Initialize | init | |
| Integer | int | |
| Maximum | max | |
| Minimum | min | |
| Message | msg | |
| Interface Builder Archive | nib | |
| Pasteboard | pboard | |
| Rectangle | rect | |
| Temporary | temp | 不要再写tmp了！ |
| Representation | Rep | used in class name such as *NSBitmapImageRep* |

命名方法或者函数时要避免歧义

```objc
// 有歧义，是返回一个sendPort还是send一个port？
sendPort

// 有歧义，是返回一个名字属性的值还是display一个name的动作？
displayName
```

#### <span id = "namingConsistency">一致性</span>

整个工程的命名风格要保持一致性，最好和苹果SDK的代码保持统一。不同类中完成相似功能的方法应该叫一样的名字，比如我们总是用 `count` 来返回集合的个数，不能在A类中使用 `count` 而在B类中使用 `getNumber`。

#### <span id = "namingPrefix">使用前缀</span>

如果代码需要打包成Framework给别的工程使用，或者工程项目非常庞大，需要拆分成不同的模块，使用命名前缀是非常有用的。

* 前缀由大写字母缩写组成，比如Cocoa中的 `NS` 前缀代表Foundation框架中的类，`IB` 则代表Interface Builder框架。

* 可以在为类、协议、函数、常量以及typedef宏命名的时候使用前缀，但注意不要为成员变量或者方法使用前缀，因为他们本身就包括在类的命名空间中。

* 命名前缀的时候不要和苹果SDK框架冲突。

* 推荐使用三个大写字母的前缀，比如SHM(ShineMo)。

### <span id = "namingClassAndProtocol">命名类和协议（Class&Protocol）</span>

类名以大写字母开头，应该包含一个 *名词* 来表示它代表的对象类型，同时可以加上必要的前缀，比如 `NSString`, `NSDate`, `NSScanner`, `NSApplication` 等等

而协议名称应该清晰地表示它所执行的行为，而且要和类名区别开来，所以通常使用 *ing* 词尾来命名一个协议， 比如 `NSCopying`, `NSLocking`。

有些协议本身包含了很多不相关的功能，主要用来为某一特定类服务，这时候可以直接用类名来命名这个协议，比如 `NSObject` 协议，它包含了id对象在生存周期内的一系列方法。

### <span id = "namingHeaders">命名头文件（Headers）</span>

源码的头文件名应该清晰地暗示它的功能和包含的内容：

* 如果头文件内定义了类或者协议，直接用类名或者协议名来命名头文件，比如 `NSLocal.h` 定义了 `NSLocal` 类

* 如果头文件内定义了一系列的类、协议、类别，使用其中最主要的类名来命名头文件，比如 `NSString.h` 定义了 `NSString` 和 `NSMutableString`

* 每一个Framework都应该有一个和框架同名的头文件，包含了框架中所有公共类头文件的引用，比如 `Foundation.h`

* Framework中有时候会实现在别的框架中类的类别扩展，这样的文件通常使用 *被扩展* + *Additions* 的方式来命名，比如 `NSBundle + Additions.h`

### <span id = "namingMethods">命名方法（Methods）</span>

Objective-C的方法名通常都比较长，这是为了让程序有更好地可读性，按照苹果的说法『好的方法名应该可以以一个句子的形式朗读出来』

方法一般以小写字母打头，每一个后续的单词首字母大写，方法名中不应该有标点符号（包括下划线），有两个例外：

* 可以用一些通用的大写字母缩写打头方法，比如 *PDF*, *TIFF* 等
* 可以用带下划线的前缀来命名类别中的方法，比如类别中的方法使用 *shm_* 或者 *wd_* 开头

私有方法 __不需要__ 加 *p_* 或者 *_* 前缀

如果方法表示让对象执行一个动作，使用动词打头的命名，注意不要使用 *do*, *does* 这种多余的关键字，动词本身的暗示就足够了：

```objc
// 动词打头的方法表示让对象执行一个动作
- (void)invokeWithTarget:(id)target;

- (void)selectTabViewItem:(NSTabViewItem *)tabViewItem;
```

如果方法是为了获取对象的一个属性值，直接用属性名称来命名这个方法，注意不要添加 *get* 或者其他的动词前缀：

```objc
// 正确，使用属性名来命名方法
- (CGFloat)cellHeight;

// 错误，添加了多余的动词前缀
- (CGFloat)calcCellHeight;

- (CGFloat)getCellHeight;
```

对于有多个参数的方法，务必在每一个参数前都添加关键词，关键词应当清晰说明参数的作用：

```objc
// 正确，保证每个参数都有关键词修饰
- (void)sendAction:(SEL)aSelector toObject:(id)anObject forAllCells:(BOOL)flag;

// 错误，遗漏关键词
- (void)sendAction:(SEL)aSelector :(id)anObject :(BOOL)flag;

// 正确
- (id)viewWithTag:(NSInteger)aTag;

// 错误，关键词的作用不清晰
- (id)taggedView:(int)aTag;
```

不要用 *and* 来连接两个参数， 通常 *and* 用来表示方法执行了两个相对独立的操作（从设计上来说，这时候应该拆分成两个独立的方法）：

```objc
// 错误，不要用『and』来连接参数
- (int)runModalForDirectory:(NSString *)path andFile:(NSString *)name andTypes:(NSArray *)fileTypes;

// 正确，使用『and』来表示两个相对独立的操作
- (int)openFile:(NSString *)fullPath withApplication:(NSString *)appName andDeactivate:(BOOL)flag;
```

方法的参数命名也有一些需要注意的地方：

* 和方法名类似，参数的第一个字母小写，后面的每一个单词首字母大写

* 不要在方法中使用类似 *pointer*, *ptr* 这样的字眼去表示指针，参数本身的类型足以说明

* 不要使用只有一两个字母的参数名

* 不要使用简写，拼出完整的单词

下面列举了一些常用参数名：

```objc
...action:(SEL)aSelector
...alignment:(int)mode
...atIndex:(int)index
...content:(CGRect)aRect
...doubleValue:(double)aDouble
...floatValue:(float)aFloat
...font:(UIFont *)font
...frame:(CGRect)frame
...intValue:(int)anInt
...keyEquivalent:(NSString *)charCode
...length:(int)length
...point:(CGPoint)point
...stringValue:(NSString *)aString
...tag:(int)tag
...target:(id)anObject
...title:(NSString *)aString
```

### <span id = "namingAccessorMethods">存取方法（Accessor Methods）</span>

存取方法是指用来获取和设置类属性值的方法，属性的不同类型，对应不同的存取方法规范：

```objc
// 属性是一个名词时的存取方法范式
- (type)noun;
- (void)setNoun:(type)aNoun;

// 栗子
- (NSString *)title;
- (void)setTitle:(NSString *)aTitle;


// 属性是一个形容词时的存取方法的范式
- (BOOL)isAdjective;
- (void)setAdjective:(BOOL)flag;

// 栗子
- (BOOL)isEditable;
- (void)setEditable:(BOOL)flag;

// 属性是一个动词时存取方法的范式
- (BOOL)verbObject;
- (void)setVerbObject:(BOOL)flag;

// 栗子
- (void)showsAlpha;
- (void)setShowsAlpha:(BOOL)flag
```

命名存取方法时不要将动词转化为被动形式来使用：

```objc
// 正确
- (void)setAcceptsGlyphInfo:(BOOL)flag;
- (BOOL)acceptsGlyphInfo;

// 错误，不要使用动词的被动形式
- (void)setGlyphInfoAccepted:(BOOL)flag;
- (BOOL)glyphInfoAccepted;
```

可以使用 *can*, *should*, *will* 等词来协助表达存取方法的意思，但是不要使用 *do* 和 *does* :

```objc
// 正确
- (void)setCanHide:(BOOL)flag;
- (BOOL)canHide;
- (void)setShouldCloseDocument:(BOOL)flag;
- (BOOL)shouldCloseDocument;

// 错误，不要使用『do』或者『does』
- (void)setDoesAcceptGlyphInfo:(BOOL)flag;
- (BOOL)doesAcceptGlyphInfo;
```

为什么Objective-C中不适用 *get* 前缀来表示属性获取方法？因为 *get* 在Objective-C中通常只用来表示从函数指针返回值的函数：

```objc
// 三个参数都是作为函数的返回值来使用，这样的函数名可以使用『get』前缀
- (void)getLineDash:(float *)pattern count:(int *)count phase:(float *)phase;
```

### <span id = "namingDelegate">命名委托（Delegate）</span>

当特定的事件发生时，对象会触发它注册的委托方法。委托是Objective-C中常用的传递消息的方式。委托有它固定的命名范式。

一个委托方法的第一个参数是触发它的对象，第一个关键词是触发对象的类名，除非委托方法只有一个名为 `sender` 的参数：

```objc
// 第一个关键词为触发委托的类名
- (BOOL)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
- (BOOL)application:(NSApplication *)sender openFile:(NSString *)filename;

// 当只有一个『sender』参数时可用省略类名
- (BOOL)applicationOpenUntitledFile:(NSApplication *)sender;
```

根据委托方法触发的时机和目的，使用 *should*, *will*, *did* 等关键词

```objc
- (void)browserDidScroll:(NSBrowser *)sender;
- (NSUndoManager *)windowWillReturnUndoManager:(NSWindow *)window;
- (BOOL)windowShouldClose:(id)sender;
```

### <span id = "namingCollectionMethods">集合类操作方法（Collection Methods）</span>

有些对象管理着一系列其它对象或者元素的集合，需要使用类似『增删改查』的方法来对集合进行操作，这些方法的命名范式一般为：

```objc
// 集合操作范式
- (void)addElement:(elementType)anObj;
- (void)removeElement:(elementType)anObj;
- (NSArray *)elements;

// 栗子
- (void)addLayoutManager:(NSLayoutManager *)obj;
- (void)removeLayoutManger:(NSLayoutManager *)obj;
- (NSArray *)layoutManagers;
```

注意，如果返回的集合是无序的，优先使用 `NSSet` 来代替 `NSArray` 。如果需要将元素插入到特定的位置，使用类似于这样的命名：

```objc
- (void)insertLayoutManager:(NSLayoutManager *)obj atIndex:(int)index;
- (void)removeLayoutMangerAtIndex:(int)index;
```

如果管理的集合元素中有指向管理对象的指针，要设置成 `weak` 类型以防止引用循环。

### <span id = "namingFunctions">~~命名函数（Functions）~~</span>

> 我也不是很清楚这一段到底说的啥，大家扫一眼就就好了

在很多场合仍然需要用到函数，比如说如果一个对象是单例，那么应该使用函数来代替类方法执行相关操作。

函数的命名和方法有一些不同，主要是：

* 函数名称一般带有缩写前缀，表示方法所在的框架

* 前缀后的单词以『驼峰』表示法显示，第一个单词首字母大写

函数名的第一个单词通常是一个动词，表示方法执行的操作：

```objc
NSHighlightRect
NSDeallocateObject
```

如果函数返回其参数的某个属性值，省略动词：

```objc
unsigned int NSEventMaskFromType(NSEventType type)
float NSHeight(NSRect aRect)
```

如果函数通过指针参数来返回值，需要在函数名中使用 *Get* ：

```objc
const char *NSGetSizeAndAlignment(const char *typePtr, unsigned int *sizep, unsigned int *alignp)
```

函数的返回类型是BOOL时的命名：

```objc
BOOL NSDecimalIsNotANumber(const NSDecimal *decimal)
```

### <span id = "namingVars">命名属性和实例变量（Properties&Instance Variables）</span>

__*__ 的位置与类型之间有一个空格，与变量名之间没有空格。即 __*__ 紧跟在变量名前。

属性和对象的存取方法相关联，属性的第一个字母小写，后续单词首字母大写，不必添加前缀。属性按功能命名成名词或者动词：

```objc
// 名词属性
@property (strong) NSString *title;
// 动词属性
@property (assign) BOOL showsAlpha;
```

属性也可以命名成形容词，这时候通常会指定一个带有 *is* 前缀的get方法来提高可读性：

```objc
@property (assign, getter=isEditable) BOOL editable;
```

命名实例变量，在变量名前加上 *_* 前缀（有些有历史的代码会将 *_* 放在后面），其它和命名属性一样：

```
@implementation MyClass {
    BOOL _showsTitle;
}
```

一般来说，类需要对使用者隐藏数据存储的细节，所以不要将实例方法定义成公共可访问的接口，可以使用 `@private`, `@protected` 前缀

如果一个变量需要暴露给外部使用，则需要在.h文件中声明为`@property`；

如果一个变量不需要暴露给外部使用，则在.m文件中声明：

```
@implementation SMTContactsDataService {
    NSString *_myUid;
    NSString *_loginMobile;
    NSArray *_loginUserOrgIds;
    NSDictionary *_orgVersion;
    int64_t _favouritContactsVersion;
    int64_t _publicServiceVersion;
}
```

按照苹果的说法，不建议在除了 `init`, `dealloc`, `setter`, `getter` 方法以外的地方直接访问实例变量，但很多人认为直接访问会让代码更加清晰可读，只在需要计算或者执行操作的时候才使用存取方法访问。但是有时候我们会通过重写 `getter` 和 `setter` 方法来做一些额外的事。所以此处建议遵循苹果的建议。

### <span id = "namingConstants">命名常量（Constants）</span>

常量名字应该以小写字母 _k_ 开头，_k_ 之后的第一个字母大写，遵循驼峰命名法。

如果常量需要提供给全局使用，需要 _k_ 之后接 _WD_ 再接 _ClassName_ 再接变量名字

栗子

```objc
int const kNumberOfFiles = 12;

WDLogin.h

FOUNDATION_EXPORT NSString *const kWDLoginDoLoginNotification;

WDLogin.m

NSString *const kWDLoginDoLoginNotification = @"doLoginNotification";

```

如果要定义一组相关的常量，尽量使用枚举类型（enumerations），枚举类型的命名规则和函数的命名规则相同。建议使用 `NS_ENUM` 和 `NS_OPTIONS` 宏来定义枚举类型：

*定义一个枚举*

一定要指定起始值

```objc
typedef NS_ENUM(NSInteger, WDMatrixMode) {
    WDMatrixModeRadio = 0,
    WDMatrixModeHighlight,
    WDMatrixModeList,
    WDMatrixModeTrack,
};
```

*定义bit map*

```objc
typedef NS_OPTIONS(NSUInteger, WDWindowMask) {
    WDWindowMaskBorderless      = 0,
    WDWindowMaskTitle           = 1 << 0,
    WDWindowMaskClosable        = 1 << 1,
    WDWindowMaskMiniaturizable  = 1 << 2,
    WDWindowMaskResizable       = 1 << 3,
};
```

尽量保持 *=* 对齐

使用 `const` 定义浮点型或者单个的整数型常量，如果要定义一组相关的整数常量，应该优先使用枚举。常量的命名规范和函数相同：

```objc
const float NSLightGray;
```

不要使用 `#define` 宏来定义常量，如果是整型常量，尽量使用枚举，浮点型常量，使用 `const` 定义。`#define` 通常用来给编译器决定是否编译某块代码，比如常用的：

```objc
#ifdef DEBUG
```

注意到一般由编译器定义的宏会在前后都有两个 `_` ，即前后都有一个 `__` , 比如 `__MACH__`

### <span id = "namingNotifications">命名通知（Notifications）</span>

通知常用于在模块间传递消息，所以通知要尽可能地表示出发生的事件，通知的命名范式是：

> [触发通知的类名] + [Did | Will] | [动作] + Notification

*栗子*

```objc
NSApplicationDidBecomeActiveNotification
NSWindowDidMiniaturizeNotification
NSTextViewDidChangeSelectionNotification
NSColorPanelColorDidChangeNotification
```

## <span id = "comments">注释</span>

读没有注释代码是很痛苦的，好的注释不仅能让人轻松读懂你的程序，还能提升代码的逼格。注意注释是为了让别人看懂，而不是仅仅让你自己看懂。

### <span id = "fileComments">文件注释</span>

每个文件都应该写文件注释，文件注释通常包含

* 文件所在模块

* 作者信息

* 历史版本信息

* 版权信息

* 文件包含的内容、作用

### <span id = "codeComments">代码注释</span>

好的代码应该是『自解释』（self-documenting）的，但仍然需要详细的注释来说明参数的意义，返回值、功能以及可能的副作用。

方法、函数、类、协议、类别的定义都需要注释，推荐采用Apple的标准注释风格，好处是可以在引用的地方 *alt+点击* 自动弹出注释，非常方便。

推荐使用[VVDocumenter](https://github.com/onevcat/VVDocumenter-Xcode)自动生成注释

协议、委托的注释要明确说明其被触发的条件：

```objc
/*
 * Delegate - Sent when failed to init connection, like p2p failed.
 */
- (void)initConnectionDidFailed:(IPCConnectionHandler *)handler;
```

如果在注释中要引用参数名或者方法函数名，使用 *| |* 将参数或者方法括起来以避免歧义：

```objc
// Somettimes we need |count| to be less than zero

// Remember to call |StringWithoutSpaces("foo bar baz")|

```

基本格式 (注意`//`之前和之后的空格)

```objc

代码与注释在同一行
- (void)setTitle:(nullable NSString *)title forState:(UIControlState)state; // default is nil. title is assumed to be single line

代码与注释分两行
// default is nil. title is assumed to be single line
- (void)setTitle:(nullable NSString *)title forState:(UIControlState)state;
```

`/*`这种形式的注释使用[VVDocumenter](https://github.com/onevcat/VVDocumenter-Xcode)自动生成的格式即可

__定义在头文件里的接口方法、属性最好要有注释！__

## <span id = "codeStyle">编码风格</span>

每个人都有自己的编码风格，这里总结了一下比较好的Cocoa编码风格和注意点

### <span id = "noNewAllowed">不要使用new方法</span>

尽管很多时候能用 `new` 代替 `alloc init` 方法，Cocoa的规范就是使用 `alloc init` 方法，使用 `new` 会让一些读者困惑。

### <span id = "publicAPIBrief">Public API要尽量简洁</span>

共有接口要设计的简洁，满足核心的功能需求就可以了。不要设计很少会用到，但是参数极其复杂的API。如果要定义复杂的方法，使用类别或者类扩展。

### <span id = "importAndInclude">#import 和 #include</span>

`#import` 是Cocoa中常用的引用头文件的方式，它能自动防止重复引用文件。什么时候使用 `#import`，什么时候使用 `#include` 呢？

* 当引用的是一个Objective-C或者Objective-C++的头文件时，使用 `#import`

* 当引用的是一个C或者C++的头文件时，使用 `#include` ，这时必须要保证被引用的文件提供了保护域（#define guard）__这是啥？__

*栗子*

```objc
#import <Cocoa/Cocoa.h>
#include <CoreFoundation/CoreFoundation.h>
#import "SMTFoo.h"
#include "base/basictypes.h"
```

为什么不全部使用 *#import* 呢？主要是为了保证代码在不同平台间共享时不出现问题

### <span id = "frameworkRootHeaderFile">引用框架的根头文件</span>

每一个框架都会有一个和框架同名的头文件，它包含了框架内接口的所有引用，在使用框架的时候，应该直接引用这个头文件，而不是其它子模块的头文件，即使是你只用到了其中的一小部分，编译器会自动完成优化的

```objc
// 正确，引用根头文件
#import <Foundation/Foundation.h>

// 错误，不要单独引用框架内的其它头文件
#import <Foundation/NSArray.h>
#import <Foundation/NSString.h>
```

### <span id = "boolUsage">BOOL的使用</span>

BOOL在Objective-C中被定义为 `signed char` 类型，这意味着一个BOOL类型的变量不仅仅可以表示 `YES(1)`和 `NO(0)`两个值，所以永远不要将BOOL类型变量直接和 `YES` 比较：

```objc
// 错误，无法确定|great|的值是否是YES(1)，不要将BOOL值直接与YES比较
BOOL great = [foo isGreat];
if (great == YES) {
	// ...be great!
}

// 正确
BOOL great = [foo isGreat];
if (great) {
	//...be great!
}
```

同样的，也不要将其它类型的值作为BOOL来返回，这种情况下，BOOL变量只会取值的最后一个字节来赋值，这样很有可能会取到`0(NO)`。但是，一些逻辑操作符比如 `&&`, `||`, `!` 的返回时可以直接赋给BOOL的：

```objc
// 错误，不要将其它类型转化为BOOL返回
- (BOOL)isBold {
	return [self fontTraits] & NSFontBoldTrait;
}

- (BOOL)isValid {
	return [self stringValue];
}

// 正确
- (BOOL)isBold {
	return ([self fontTraits] & NSFontBoldTrait) ? YES : NO;
}

// 正确，逻辑操作符可以直接转化为BOOL
- (BOOL)isValid {
	return [self stringValue] != nil;
}

- (BOOL)isEnabled {
	return [self isValid] && [self isBold];
}
```

另外BOOL类型可以和 *_Bool*, *bool* 相互转化， 但是不能和 *Boolean* 转化

### <span id = "noInstanceVarInInitAndDealloc">在init和dealloc中不要用存取方法访问实例变量</span>

当 `init` 和 `dealloc` 方法被执行时，类的运行时环境不是处于正常状态，使用存取方法访问变量可能会导致不可预料的结果，因此应该在两个方法内直接访问实例变量

```objc
// 正确，直接访问实例变量
- (instancetype)init {
	self = [super init];
	if (!self) {
        return nil;
	}
	_bar = [[SMTBar alloc] init];
	return self;
}

// 错误，不要通过存取方法访问
- (instancetype)init {
	self = [super init];
	if (!self) {
	   return nil;
	}
	self.bar = [[SMTBar alloc] init];
	return self;
}
```

### <span id = "if-else">if-else</span>

不管`if`或者`else`中的代码块有多少，都应该放在 `{}` 中

```objc
// 正确
if (!condition) {
    return;
}

// 错误
if (!condition)
    return;
```

if-else 超过4层的时候，就要考虑重构，多层的if-else结构很难维护

当需要一定条件下才执行某项操作时，最左边的应该是最重要的代码，不要将最重要的代码内嵌到`if`中

<span id = "returnInInit">此原则同样适用于`init`方法中</span>

```objc
// 良好的风格
- (void)someMethod {
    if (!flag) {
        return;
    }
    // 重要的代码写在这里
}

- (instancetype)init {
    self = [super init];
    if (!self) {
        return nil;
    }
    // do other initializing things
    return self;
}

// 不提倡的风格
- (void)someMethod {
    if (flag) {
        // 重要的代码
    }
    return;
}
```

<span id = "if-else-else-format">else</span>紧跟在if块的右 _}_ 之后，不要另起一行

```objc
// 良好的风格
- (void)someMethod {
    if (condition1) {
        // do something
    } else if (condition2) {
        // do something
    } else {
        // do something
    }
}

// 不提倡的风格
- (void)someMethod {
    if (condition1) {
        // do something
    }
    else if (condition2) {
        // do something
    }
    else {
        // do something
    }
}

```

### <span id = "NSStringCopy">保证NSString在赋值时被复制</span>

`NSString` 非常常用，在它被传递或者赋值时应该保证是以复制`（copy）`的方式进行的，这样可以防止在不知情的情况下String的值被其他对象修改

```objc
- (void)setFoo:(NSString *)aFoo {
	_foo = [aFoo copy];
}
```

### <span id = "NSNumberUsage">使用NSNumber的语法糖</span>

使用带有 *@* 符号的语法糖来生产NSNumber对象能使代码更简洁：

```objc
NSNumber *fortyTwo = @42
NSNumber *piOverTwo = @(M_PI / 2);
enum {
	kMyEnum = 2;
};
NSNumber *myNumber = @(kMyEnum);
```

### <span id = "nilCheck">nil检查</span>

因为在Objective-C中向nil对象发送命令是不会抛出异常或者导致崩溃的，只是完全的『什么都不干』，所以，只在程序中使用nil来做逻辑上的检查。

另外，不要使用诸如 `nil == Object` 或者 `Object == nil` 的形式来判断

```objc
// 正确，直接判断
if (!objc) {
	...
}

// 错误，不要使用 nil == Object 的形式
if (nil == objc) {
	...
}
```

如果参数不允许为`nil`，则使用`NSAssert`断言处理。`NSAssert`是系统定义的宏

```objc
NSAssert(param != nil, @"param参数为空");
```

### <span id = "threadSafe">属性的线程安全</span>

定义一个属性时，编译器会自动生成线程安全的存取方法`（Atomic）`，但这样会大大降低性能，特别是对于那些需要频繁存取的属性来说，是极大的浪费。所以如果定义的属性不需要线程保护，记得手动添加属性关键字 `nonatomic` 来取消编译器的优化

### <span id = "dotNotation">点分语法的使用</span>

### <span id = "delegateWeak">Delegate要使用弱引用</span>

一个类的Delegate对象通常还引用着类本身，这样很容易造成引用循环的问题，所以类的Delegate属性要设置为弱引用

```objc
@property (nonatomic, weak) id <MyDelegate> delegate;
```

### <span id = "checkBlockNil">block判空处理</span>

不论任何时候，调用bloc时一定要做判空处理

```objc
// 正确，调用block之前做了判空处理
- (void)fetchUserInfoWithCompletionHandler:(void(^)(SMTUser *user))completionHandler {
	SMTUser *user = nil;
	// fetch user ...
	if (completionHandler) {
		completionHandler(user);
	}
}

// 错误，调用block之前未做判空处理
- (void)fetchUserInfoWithCompletionHandler:(void(^)(SMTUser *user))completionHandler {
	SMTUser *user = nil;
	// fetch user ...
	completionHandler(user);
}
```

### <span id = "collectionTraverse">集合的遍历</span>

尽量避免使用 `for(int i = 0; i < count; i++)` 的形式

#### <span id = "NSArrayTraverse">NSArray的遍历</span>

```objc
// 如果不需要index，使用 for-each 进行数组遍历
for (SMTFoo *foo in foos) {
	// do anything with |foo|
	// BUT NEVER do [foos removeObject:] !
}

// 如果需要index，是用block遍历
[foos enumerateObjectsUsingBlock:^(SMTFoo *foo, NSUInteger idx, BOOL *stop) {
	// do anything with |foo|
	// BUT NEVER do [foos removeObject:] !
}];
```

#### <span id = "NSDictionaryTraverse">NSDictionary的遍历</span>

```objc
[dict enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
    // ...
}];
```

### <span id = "switch">switch</span>

1. 每一个`case`下始终都要有`{}`, 不管该`case`下有几行代码
2. `break;`要在`{}`里
3. `default`必须要有且`default`块中要有`break;`

```objc

// 推荐对写法
switch (type) {
    case type1: {
        // do something
        break;
    }
    case type2: {
        // do somthing
        break;
    }
    case type3: {
        break;
    }
    default: {
        break;
    }
}

// 不推荐对写法
switch (type) {
    case type1:
    {
        // do something
        break;
    }
    case type2:
    {
        // do somthing
        break;
    }
    case type3:
        break;
}

```

### <span id = "todo">TODO</span>

基本格式

```objc
// TODO(who): what to do
```

标记TODO的地方一定要去do！do了以后把TODO标记删掉！

### <span id = "moreTips">More tips</span>

* 尽量减少在代码中之间使用数字常量，每个数字都有含义，使用常量定义等方式表明数字含义

* 尽量减少代码中的重复计算，比如代码中多处要使用屏幕宽度，然后计算`[[UIScreen mainScreen] bounds].size.width`，很多次，显得很繁琐，代码冗长。不如直接使用宏定义：
    > #define SCREEN_WIDTH ([[UIScreen mainScreen] bounds].size.width)

* 宏定义全部字母**大写**

* 任何一个单独的 *=* 两边都要空一个空格

## <span id = "fileOrganization">文件的组织</span>

尽量保证一个类一个文件，这样可以有效控制文件大小

    - Mail
        | - Home
            | - ViewController
            | - View
                | - Cell
                | - TableView
                | - ...
            | - Model
            | - Proxy
            | - ViewModel/Service
            | - Helper/Util
        | - NewMail
        | - MailDetail
        | - Biz
            | - Manager
            | - Supporting Files
                | - Ace
                | - DB

## <span id = "codeOrganization">代码块的组织</span>

在我们平时的开发工作中，看代码的时间不比写代码的时候少，所以如何让自己写的代码文件『好看』是非常重要的。

我们应该对文件中的各种方法按照功能进行分块，同时尽量保证一个类的代码行数不超过__500__行

可以使用`#pragma mark`宏辅助进行方法分块：

```objc
#pragma mark - Life Cycle
#pragma mark - Public Interface
#pragma mark - UI Actions
#pragma mark - Business Logic
#pragma mark - UITableViewDataSource
#pragma mark - UITableViewDelegate
#pragma mark - UIScrollViewDelegate
#pragma mark - Notification Handler
#pragma mark - Private Method
```

## <span id = "methodScale">方法/函数的规模</span>

一个方法或者一个函数不应超过__50__行，最多不应超过100行。一个方法应该只做一件事，一般来讲50行是够的。如果方法行数太长，应该考虑将其中的某些地方做成子函数使用。

函数的参数不应该多余5个，如果一个函数参数多余5个，则需考虑重构

子函数内不应调用成员变量 (待定)

**严禁复制代码**

如果我们在编码的时候做了任何一个copy-paste代码的动作，证明我们paste的代码是可以抽象成子函数的。
