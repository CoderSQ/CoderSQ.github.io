---
title: ReactNative启动流程
date: 2018-04-03 09:57:14
tags: ReactNative
categories: ReactNative
---

最近在学习ReactNative,很好奇其中的实现原理，并且在网上基本上是启动和通讯原理的介绍，通讯原理并没有理顺，所以想这段时间打算把ReactNative原理给整清楚，出一个针对iOS专题系列

[ReactNative源码解读准备篇]()

[ReactNative启动流程]()

[ReactNative通信原理]()

[ReactNative事件传播机制]()

[ReactNative布局原理]()

[ReactNative渲染流程]()

## 启动流程
<span id="inline-blue">注意: 本系列粘贴的所有代码，会忽略非重要的代码 </span>

新建一个ReactNative项目，打开其中的iOS项目，我们就可以分析app的启动流程了。首先在`AppDelegate`中，有如下两行代码
```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  NSURL *jsCodeLocation;
  
  // 这个方法主要是根据环境来确定当前js文件的具体路径
  jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];

  // 这个是我们app的根视图
  RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                      moduleName:@"LoveFoods"
                                               initialProperties:nil
                                                   launchOptions:launchOptions];
  return YES;
}
```

在创建app根视图的时候创建了bridge对象,该对象负责native和js的交互，js运行环境的准备等工作
```objc
RCTBridge *bridge = [[RCTBridge alloc] initWithBundleURL:bundleURL
                                            moduleProvider:nil
                                             launchOptions:launchOptions];
return [self initWithBridge:bridge moduleName:moduleName initialProperties:initialProperties];
```

在`RCTBridge`的初始化方法`nitWithBundleURL: moduleProvider: launchOptions:`中调用了setUp方法, 我们直接来看setUp方法

```objc
- (void)setUp
{
  Class bridgeClass = self.bridgeClass;

  #if RCT_DEV // dev的情况下注册监听command+r的键盘事件，在这里我们暂时不需要关心
  RCTExecuteOnMainQueue(^{
    RCTRegisterReloadCommandListener(self);
  });
  #endif

  self.batchedBridge = [[bridgeClass alloc] initWithParentBridge:self];
  [self.batchedBridge start];
}
```
setup方法中创建了`batchedBridge`对象，该对象为`RCTCxxBridge`的实例，`RCTBridge`的工作，实际都转交给了`batchedBridge`对象


我们来下，创建`batchedBridge`对象时，做了什么事情
```
- (instancetype)initWithParentBridge:(RCTBridge *)bridge
{
  RCTAssertParam(bridge);

  if ((self = [super initWithDelegate:bridge.delegate
                            bundleURL:bridge.bundleURL
                       moduleProvider:bridge.moduleProvider
                        launchOptions:bridge.launchOptions])) {
    _parentBridge = bridge; //保存bridge
    _valid = YES;
    _loading = YES;
    _pendingCalls = [NSMutableArray new];
    _displayLink = [RCTDisplayLink new]; // 创建定时器，该定时器会定时调用_jsThreadUpdate方法


    // 创建用来保存所有导出给js使用的模块信息的数组
    _moduleDataByName = [NSMutableDictionary new];
    _moduleClassesByID = [NSMutableArray new];
    _moduleDataByID = [NSMutableArray new];

    //设置RCTBridge.m文件中的静态对象RCTCurrentBridgeInstance==batchedBridge
    [RCTBridge setCurrentBridge:self];
  }
  return self;
}
```


