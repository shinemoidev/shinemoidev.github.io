---
title: Autorotate - 让你的应用支持旋转
date: 2018-07-02 11:53:00
tags:
- iOS
---

iOS App大多数情况下都是只支持竖屏的，少部分页面才支持旋转，甚至有些页面需要强制横屏。本文将介绍应用如何支持旋转，以及旋转后我们页面需要如何处理的。

### UIDeviceOrientation和UIInterfaceOrientation

iOS旋转包括设备旋转和界面旋转，设备方向我们控制不了，看用户怎么握住他的手机，界面方向一般跟着设备方向旋转而旋转（如果支持的话），通常我们说的APP支持旋转，就是界面旋转。

![屏幕快照 2018-07-02 上午9.56.41.png](https://upload-images.jianshu.io/upload_images/4315285-c5b43339e3aacfe9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--more-->

**UIDeviceOrientation：设备方向**

iOS设备方向是通过加速计来获取的，总共有七个方向：

```
typedef NS_ENUM(NSInteger, UIDeviceOrientation) {
    UIDeviceOrientationUnknown,
    UIDeviceOrientationPortrait,            // Device oriented vertically, home button on the bottom
    UIDeviceOrientationPortraitUpsideDown,  // Device oriented vertically, home button on the top
    UIDeviceOrientationLandscapeLeft,       // Device oriented horizontally, home button on the right
    UIDeviceOrientationLandscapeRight,      // Device oriented horizontally, home button on the left
    UIDeviceOrientationFaceUp,              // Device oriented flat, face up
    UIDeviceOrientationFaceDown             // Device oriented flat, face down
} __TVOS_PROHIBITED;
```

当设备方向变化时候，会发出通知`UIDeviceOrientationDidChangeNotification `，注册监听该通知来处理页面。

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    //开启和监听 设备旋转的通知（不开启的话，设备方向一直是UIInterfaceOrientationUnknown）
    if (![UIDevice currentDevice].generatesDeviceOrientationNotifications) {
        [[UIDevice currentDevice] beginGeneratingDeviceOrientationNotifications];
    }
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleDeviceOrientationChange:) name: UIDeviceOrientationDidChangeNotification object:nil];
}

//设备方向改变的处理
- (void)handleDeviceOrientationChange:(NSNotification *)notification{
    UIDeviceOrientation deviceOrientation = [UIDevice currentDevice].orientation;
    switch (ddeviceOrientation) {
        case UIDeviceOrientationFaceUp:
            NSLog(@"屏幕朝上平躺");
            break;
        case UIDeviceOrientationFaceDown:
            NSLog(@"屏幕朝下平躺");
            break;
        case UIDeviceOrientationUnknown:
            NSLog(@"未知方向");
            break;
        case UIDeviceOrientationLandscapeLeft:
            NSLog(@"屏幕向左横置");
            break;
        case UIDeviceOrientationLandscapeRight:
            NSLog(@"屏幕向右橫置");
            break;
        case UIDeviceOrientationPortrait:
            NSLog(@"屏幕直立");
            break;
        case UIDeviceOrientationPortraitUpsideDown:
            NSLog(@"屏幕直立，上下顛倒");
            break;
        default:
            NSLog(@"无法辨识");
            break;
    }
}

//最后在dealloc中移除通知，并结束设备旋转的通知
- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self];
    [[UIDevice currentDevice] endGeneratingDeviceOrientationNotifications];
}
```

**UIInterfaceOrientation：界面方向**

iOS界面方向和Home的按钮方向是一致的，总共有五个方向：

```
typedef NS_ENUM(NSInteger, UIInterfaceOrientation) {
    UIInterfaceOrientationUnknown            = UIDeviceOrientationUnknown,
    UIInterfaceOrientationPortrait           = UIDeviceOrientationPortrait,
    UIInterfaceOrientationPortraitUpsideDown = UIDeviceOrientationPortraitUpsideDown,
    UIInterfaceOrientationLandscapeLeft      = UIDeviceOrientationLandscapeRight,
    UIInterfaceOrientationLandscapeRight     = UIDeviceOrientationLandscapeLeft
} __TVOS_PROHIBITED;
```

当界面方向变化时候，会发出通知`UIApplicationWillChangeStatusBarOrientationNotification `、`UIApplicationDidChangeStatusBarOrientationNotification `，注册监听该通知来处理页面。

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleStatusBarOrientationChange:) name: UIApplicationDidChangeStatusBarOrientationNotification object:nil];
}

//界面方向改变的处理
- (void) handleStatusBarOrientationChange:(NSNotification *)notification{
    UIInterfaceOrientation interfaceOrientation = [[UIApplication sharedApplication] statusBarOrientation];
    switch (interfaceOrientation) {
        case UIInterfaceOrientationUnknown:
            NSLog(@"未知方向");
            break;
        case UIInterfaceOrientationPortrait:
            NSLog(@"界面直立");
            break;
        case UIInterfaceOrientationPortraitUpsideDown:
            NSLog(@"界面直立，上下颠倒");
            break;
        case UIInterfaceOrientationLandscapeLeft:
            NSLog(@"界面朝左");
            break;
        case UIInterfaceOrientationLandscapeRight:
            NSLog(@"界面朝右");
            break;
        default:
            break;
    }
}

//最后在dealloc中移除通知
- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}
```

### 视图控制器中旋转方向的设置

**支持旋转（不建议）：**

在项目的General-->Deployment Info-->Device Orientation中，勾选支持的方向，一般Upside Down不勾选，没几个APP会支持颠倒。

![屏幕快照 2018-07-02 上午9.43.32.png](https://upload-images.jianshu.io/upload_images/4315285-5e8f9359bf3e9890.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>备注：为什么不建议呢？因为这样设置的话，如果你的启动页使用的是LaunchScreen.storyboard，那么当你横屏启动APP时，启动页也会横屏启动，而不是竖屏启动，这很怪异（一般app启动页面都是竖屏的）；然而，貌似使用LaunchImage设置启动页，有不受影响，当你横屏启动APP时，启动页依然是竖屏，有点没搞懂。

**在AppDelegate中实现代理：（建议）**

```
- (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window {
    return UIInterfaceOrientationMaskPortrait | UIInterfaceOrientationMaskLandscape;
}
```

>备注：注意，如果APP有多个window，判断下window回更合理些，这里返回的是keyWindow；当然，你所有window都支持该旋转方向，也不是不可以。


### 支持ViewController屏幕方向设置

我们需要通过重载ViewController方法，来控制界面是否支持旋转，以及旋转方向。

**WDMainTabBarController**

```
#pragma mark - Orientation

- (BOOL)shouldAutorotate {
    return [self.selectedViewController shouldAutorotate];
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations{
    return [self.selectedViewController supportedInterfaceOrientations];
}
```


**WDBaseNavigationController**

```
#pragma mark - Orientation

- (BOOL)shouldAutorotate {
    return [[self.viewControllers lastObject] shouldAutorotate];
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return [[self.viewControllers lastObject] supportedInterfaceOrientations];
}
```

**WDBaseViewController**

由于APP默认仅支持竖屏，所有我写了个基类BaseViewController，在基类中实现如下方法：

```
#pragma mark - Orientation

- (BOOL)shouldAutorotate {
    return NO;
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskPortrait;
}
```

**WDWebViewController**

WDWebViewController支持制动旋转

```
#pragma mark - Orientation

- (BOOL)shouldAutorotate {
    return YES;
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskAll;
}
```

**WDThirdViewController**

WDThirdViewController强制横屏

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    self.navigationItem.title = @"Third";
    
    self.oldOrientation = [UIDevice currentDevice].orientation;
    if (self.oldOrientation == UIDeviceOrientationLandscapeLeft) {
        // 如果设备本身是横屏的，但界面不是横屏，这时候直接设置LandscapeLeft不生效，需要设置Unknown欺骗系统后，再设置横屏
        [[UIDevice currentDevice] setValue:[NSNumber numberWithInteger:UIDeviceOrientationUnknown] forKey:@"orientation"];
    }
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    
    UIInterfaceOrientation interfaceOrientation = [[UIApplication sharedApplication] statusBarOrientation];
    switch (interfaceOrientation) {
        case UIInterfaceOrientationLandscapeLeft:
        case UIInterfaceOrientationLandscapeRight: {
            // 本身是横屏，直接break
            break;
        }
            
        default: {
            // 强制横屏
            [[UIDevice currentDevice] setValue:[NSNumber numberWithInteger:UIDeviceOrientationLandscapeLeft] forKey:@"orientation"];
            break;
        }
    }
}

- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];
    
    // 还原设备方向
    [[UIDevice currentDevice] setValue:[NSNumber numberWithInteger:self.oldOrientation] forKey:@"orientation"];
}

#pragma mark - Orientation

- (BOOL)shouldAutorotate {
    return YES;
}

- (UIInterfaceOrientationMask)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskLandscape;
}
```

至于旋转页面的处理，我在WDBaseViewController中添加了通知和方法：

```
//监听UIApplicationDidChangeStatusBarOrientationNotification的处理
- (void)handleStatusBarOrientationChange: (NSNotification *)notification {
    UIInterfaceOrientation interfaceOrientation = [[UIApplication sharedApplication] statusBarOrientation];
    BOOL isLandscape = NO;
    switch (interfaceOrientation) {
        case UIInterfaceOrientationUnknown: {
            //NSLog(@"未知方向");
            break;
        }
        case UIInterfaceOrientationPortrait:
        case UIInterfaceOrientationPortraitUpsideDown: {
            isLandscape = NO;
            break;
        }
        case UIInterfaceOrientationLandscapeLeft:
        case UIInterfaceOrientationLandscapeRight: {
            isLandscape = YES;
            break;
        }
            
        default:
            break;
    }
    
    CGFloat navBarHeight = WD_NAV_BAR_HEIGHT;
    if (isLandscape) {
        navBarHeight = WD_NAV_BAR_HEIGHT_LANDSCAPE;
    }
    
    [self updateLayoutNavBarWhenAutorotate:navBarHeight isLandscape:isLandscape];
}

- (void)updateLayoutNavBarWhenAutorotate:(CGFloat)navBarHeight isLandscape:(BOOL)isLandscape {
    self.navigationBarBackImageView.frame = CGRectMake(0, 0, WD_SCREEN_WIDTH, navBarHeight);
    self.navigationBarLineView.frame = CGRectMake(0, navBarHeight - 0.5, WD_SCREEN_WIDTH, 0.5);
}
```

子类WDBaseWebViewController重载

```
- (void)updateLayoutNavBarWhenAutorotate:(CGFloat)navBarHeight isLandscape:(BOOL)isLandscape {
    [super updateLayoutNavBarWhenAutorotate:navBarHeight isLandscape:isLandscape];
    
    CGRect frame =CGRectMake(0, navBarHeight, WD_SCREEN_WIDTH, WD_SCREEN_HEIGHT - navBarHeight);
    self.webView.frame = frame;
}
```

### 最后

这里有几个地方需要注意的，竖屏和横屏之后的navigationBar和tabBar的宽高都发生了变化，而且不同设备之间大小有差异，比如：

- iphone 7竖屏时，navigationBar的宽高是(375, 44)，statusBar的宽高是(375, 20)，横屏时，navigationBar宽高是(667, 32)，statusBar的宽高是(667, 0)
- iphone x竖屏时，navigationBar的宽高是(375, 44)，statusBar的宽高是(375, 44)，横屏时，navigationBar宽高是(812, 32)，statusBar的宽高是(812, 0)

还有一个有意思的是：当你横屏的时候，状态栏会被隐藏不见了（可能是横屏为了更多的展示内容），如果我们需要将状态栏显示出来，ViewController需要重载`prefersStatusBarHidden`方法，因此我在WDBaseViewController中添加如下方法：

```
#pragma mark - Status Bar

- (BOOL)prefersStatusBarHidden {
    return NO;
}
```

在WDBaseNavigationController中添加如下方法：

```
#pragma mark - Status Bar

- (UIViewController *)childViewControllerForStatusBarHidden {
    return self.topViewController;
}

- (UIStatusBarStyle)preferredStatusBarStyle {
    return UIStatusBarStyleLightContent;
}
```

but 在iphone x中，无论我如何尝试，状态栏都不会出现，始终隐藏，不得其解。

>备注：通过`CGRectGetWidth([UIScreen mainScreen].bounds)`，获取的宽高都是旋转后的值。

 [Demo地址](https://github.com/qq510304723/DemoTest/tree/master/Autorotate)
 
