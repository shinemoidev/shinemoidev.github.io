---
title: 工具—SHMAaceHooker
date: 2022-02-24 09:41:08
tags:
---

# 背景

当前项目中，非常多的请求都是由 XXXClient 类发起，通过长链接获取数据之后，再由 ProtocolPacker 进行自定义解包，最终获取到我们需要的数据格式供业务使用。

在这个数据交互的过程中，测试无法通过抓包获取到请求的参数及返回结果，当发生问题时，无法快速定位问题，只能由客户端开发人员进行调试，才能得到请求时具体的数据往来，导致问题解决的效率比较低。

这个工具旨在能将数据的交互可视化，使问题快速暴露，并得以解决。

<!--more-->

# 成果展示

[SHMAaceHooker 源码](https://git.shinemo.com/projects/SHMPODSPECS/repos/SHMAaceHooker)

{% asset_img acehook.jpg 这是一张图片 %}

# 期望目标

* 获取 ace 请求时的参数
* 获取 ace 请求之后的结果
* 数据可视化。

# 解决方案
* 通过 Aspect 对同步请求方法进行 Hook，得到请求参数及结果。
* 结合 Aspect & BlockHook 对异步方法进行 Hook，得到请求参数及结果。

# 具体实施

## 准备知识

#### AOP
在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。

有了这个思想作为方向之后，接下来我们就需要解决 **切入点** 和 **切片逻辑** 这两个问题了。

* **切入点**
  
XXXClient 及 XXXPack 都是由工具 AutoGen 生成的模板代码：

1. XXXClient：处理所有的请求逻辑。都继承于 AaceCaller 类。
2. XXXPack：自定义的数据解包工具。

在 Client 类的请求方法结束之后，数据也得到了 Pack 的解包，这个时候，我们就可以获取到请求的参数及返回的数据。

* **切片逻辑**

具体的获取数据的逻辑，需要分为 同步请求 和 异步请求两种处理。后面会展开介绍。

## 准备工作

1. 利用 CocoaPods 接入 Aspect 及 BlockHook。
2. 获取一个类的所有子类工具方法。
3. 获取一个类的所有方法。

[Type Encodings 类型编码](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)


## 实施

1. 先找到 **AaceCaller** 的所有子类，也就是项目中所有的 XxxClient。
2. 遍历所有 XxxClient 类，并找出其所有的方法。
3. 删选过滤。找到其 同步请求 和 异步请求 的方法。
4. 针对不同的方法，执行不同的 Hook 逻辑。

### 同步请求

请求返回的结果，是通过指针赋值的方式，直接传递给业务层，所以，在获取请求参数时，能获取到的是一个内部结构 NSConcreateValue 的对象，其实这就是一个对象的指针。我们需要做的就是根据这个指针将对应的数据对象取出。

```
+ (void)_forSyncRequest:(Class)cls selName:(NSString *)selName {
    SEL sel = NSSelectorFromString(selName);
    
    [cls aspect_hookSelector:sel withOptions:AspectPositionAfter usingBlock:^(id<AspectInfo> aspectInfo) {
        NSObject *instance = aspectInfo.instance;
        NSInvocation *invocation = aspectInfo.originalInvocation;
        NSArray *arguments = aspectInfo.arguments;
        
        NSMethodSignature *methodSignature = invocation.methodSignature;
        
        NSMutableArray *requestArguments = [[NSMutableArray alloc] init];
        NSMutableArray *responseArray = [[NSMutableArray alloc] init];
        
        for (int i = 0; i < arguments.count; i++) {
            NSObject *argument = arguments[i];
            
            // 前两个参数分别是 实例对象 和 Sel，为默认参数，这里取得时候直接跳过。
            // 后面的参数则是调用方法时，传入的参数。
            const char *type = [methodSignature getArgumentTypeAtIndex:2 + i];
            NSString *argumentType = [NSString stringWithCString:type encoding:NSUTF8StringEncoding];;
                        
            // TODO: 判断参数类型，存储请求参数。
                
            if ([argumentType isEqualToString:@"^@"]) {
                // 指向对象的指针。 针对client，这个一般都是我们传入用来获取返回值的对象指针
                NSValue *value = (NSValue *)argument;
                
                void **aaa = value.pointerValue;
                
                NSObject *use = (__bridge NSObject *)(*aaa);
                
                NSArray *array = [self findIvars:use.class];
                
                if (use) {
                    [responseArray addObject:use];
                } else {
                    [responseArray addObject:@"instance empty"];
                }
            } 
            ...省略判断逻辑代码
        }
        
        // 自定义请求对象
        SHMAaceHookModel *result = [[SHMAaceHookModel alloc] init];
        result.clientName = NSStringFromClass(cls);
        result.selName = selName;
        result.arguments = requestArguments;
        result.responses = responseArray;
        
        [[SHMAaceHookCache sharedCache] saveHookResult:result];
        
    } error:NULL];
}

```

### 异步请求

Aspect hook 方法之后，获取到 block 对象，接着用 BlockHook 对 block 进行拦截，在 block 执行完成之后，我们获取其中的参数，也即为我们需要的请求数据结果。

根据获取到的参数的类型编码确定参数的类型，并根据类型作相应的解析，得到我们需要的结果数据。

```
+ (void)_forAsyncRequest:(Class)cls selName:(NSString *)selName {
    SEL sel = NSSelectorFromString(selName);
    
    [cls aspect_hookSelector:sel withOptions:AspectPositionBefore usingBlock:^(id<AspectInfo> aspectInfo) {
        NSObject *instance = aspectInfo.instance;
        NSInvocation *invocation = aspectInfo.originalInvocation;
        NSArray *arguments = aspectInfo.arguments;
        
        NSMethodSignature *methodSignature = invocation.methodSignature;
        
        NSMutableArray *requestArguments = [[NSMutableArray alloc] init];
        NSMutableArray *responseArray = [[NSMutableArray alloc] init];
        
        [aspectInfo.originalInvocation retainArguments];
        
        for (int i = 0; i < arguments.count; i++) {
            NSObject *argument = arguments[i];
            
            // 前两个参数分别是 实例对象 和 Sel，为默认参数，这里取得时候直接跳过。
            // 后面的参数则是调用方法时，传入的参数。
            const char *type = [methodSignature getArgumentTypeAtIndex:2 + i];
            NSString *argumentType = [NSString stringWithUTF8String:type];
            
            Class blockCls = NSClassFromString(@"NSBlock");
            if ([argument isKindOfClass:blockCls]) {
                __unsafe_unretained id block = argument;
                
                [block block_interceptor:^(BHInvocation * _Nonnull invocation, IntercepterCompletion  _Nonnull completion) {
                    completion();
                    
                    for (int i = 1; i < invocation.methodSignature.numberOfArguments; i++) {

                        const char *argType = [invocation.methodSignature getArgumentTypeAtIndex:i];
                        NSString *argumentType = [NSString stringWithCString:argType encoding:NSUTF8StringEncoding];

                        NSLog(@"aace async hook ======= argumentType = %@", argumentType);

                        ...省略判断逻辑代码
                        if ([argumentType hasPrefix:@"@\"NS"]) {
                            // oc 对象
                            void *obj;
                            [invocation getArgument:&obj atIndex:i];

                            NSObject *object = (__bridge NSObject *)obj;

                            if (object) {
                                [responseArray addObject:object];
                            }
                        }
                    }
                    
                    // 异步请求需要等待 block 结束后存储
                    SHMAaceHookModel *result = [[SHMAaceHookModel alloc] init];
                    result.clientName = NSStringFromClass(cls);
                    result.selName = selName;
                    result.arguments = requestArguments;
                    result.responses = responseArray;
                    
                    [[SHMAaceHookCache sharedCache] saveHookResult:result];
                }];
                
            } else {
                if (argument) {
                    [requestArguments addObject:argument];
                }
            }
        }
        
    } error:NULL];
}
```

### 数据收集

**SHMAaceHookModel：请求的模型** 

```
@interface SHMAaceHookModel : NSObject

@property (nonatomic, copy) NSString *clientName;
@property (nonatomic, copy) NSString *selName;
@property (nonatomic, copy) NSArray *arguments;
@property (nonatomic, copy) NSArray *responses;

@end
```

**SHMAaceHookCache：收集请求模型的管理者**





