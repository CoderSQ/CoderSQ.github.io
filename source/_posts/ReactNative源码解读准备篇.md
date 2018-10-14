---
title: ReactNative启动流程
date: 2018-04-03 16:13:14
tags: ReactNative
categories: ReactNative
---

最近在学习ReactNative,很好奇其中的实现原理，并且在网上基本上是启动和通讯原理的介绍，通讯原理并没有理顺，所以想这段时间打算把ReactNative原理给整清楚，出一个针对iOS专题系列

[ReactNative源码解读准备篇]()

[ReactNative启动流程]()

[ReactNative通信原理]()

[ReactNative事件传播机制]()

[ReactNative布局原理]()

[ReactNative渲染流程]()


## 在iOS中如何导出类给RN使用
    
在这里我斗胆借用一下【ReactNative](https://reactnative.cn/docs/0.51/native-modules-ios.html#content)中的话

> 在React Native中，一个“原生模块”就是一个实现了“RCTBridgeModule”协议的Objective-C类，其中RCT是ReaCT的缩写。

```objc
@interface TestBridgeClass : NSObject <RCTBridgeModule>

@end
```

>为了实现RCTBridgeModule协议，你的类需要包含RCT_EXPORT_MODULE()宏。这个宏也可以添加一个参数用来指定在Javascript中访问这个模块的名字。如果你不指定，默认就会使用这个Objective-C类的名字。

```objc
@implementation TestBridgeClass

RCT_EXPORT_MODULE();

- (void)loveFoods:(NSString * )name {
  NSLog(@"我需要被导出");
}
@end
```
这样我们在RN中就能访问`TestBridgeClass`这个类了，那么要访问他的 ` - (void)loveFoods:(NSString * )name ` 方法呢,这时我们需要通过`RCT_EXPORT_METHOD()`宏来导出该方法.方式如下

```objc
@implementation TestBridgeClass

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(loveFoods:(NSString*) name) {
  NSLog(@"我是导出的方法");
}

- (void)test {
  NSLog(@"在RN中无法访问我");
}

@end
```

这样在RN中就可以访问`TestBridgeClass`了，需要注意的是，只有native端导出的方法才能被RN端访问

## 在RN中如何使用iOS导出的类
在RN中使用native端导出的类很简单，代码如下
```javascript
import { NativeModules } from 'react-native';

var TestBridgeClass = NativeModules.TestBridgeClass;

CalendarManager.loveFoods("banana");
```

重新复述一遍上面的内容，主要是为了分析里面的两个重要的宏的作用，`RCT_EXPORT_MODULE()` 和`RCT_EXPORT_METHOD()`


### 理解RCT_EXPORT_MODULE
我们一个一个来理解。下面，我们把该宏一步步展开，展开代码如下：

```objc
#define RCT_EXPORT_MODULE(js_name) \
RCT_EXTERN void RCTRegisterModule(Class); \
+ (NSString *)moduleName { return @#js_name; } \
+ (void)load { RCTRegisterModule(self); }

//RCT_EXTERN定义如下
#if defined(__cplusplus)
#define RCT_EXTERN extern "C" __attribute__((visibility("default")))
#define RCT_EXTERN_C_BEGIN extern "C" {
#define RCT_EXTERN_C_END }
#else
#define RCT_EXTERN extern __attribute__((visibility("default")))
#define RCT_EXTERN_C_BEGIN
#define RCT_EXTERN_C_END
#endif
```
关于RCT_EXTER的宏定义，我们只关心if和else中的第一行。以下解析来自[MJRefresh源码阅读](https://www.aliyun.com/jiaocheng/369170.html)

>其中__cplusplus 是cpp中的自定义宏,那么定义了这个宏的话表示这是一段cpp的代码,也就是说,上面的代码的含义是:如果这是一段cpp的代码,那么加入extern"C"和其中的代码。

>extern是C/C++语言中表明函数和全局变量作用范围(可见性)的关键字,该关键字告诉编译器,其声明的函数和变量可以在本模块或其它模块中使用。 

>那extern "C"呢?C++之父在设计C++之时,考虑到当时已经存在了大量的C代码,为了支持原来的C代码和已经写好C库,需要在C++中尽可能的支持C,而 extern "C"就是其中的一个策略。

>attribute是GNU C的一种机制,用法为attribute_ ((attribute-list))。当项目需要作为一个库被外包引用的时候通常在编译时可以用参数-fvisibility指定所有符号的可见性。在编译命令中加入 -fvisibility=hidden参数,会将所有默认的public的属性变为hidden。此时,如果对函数设置attribute((visibility ("default")))参数,使特定的函数仍然按默认的public属性处理,则-fvisibility=hidden参数不会对该函数起作用。所以,设置了-fvisibility=hidden参数之后,只有设置了attribute((visibility ("default")))的函数才是对外可见的。

关于attribute_更详细的用法请参考[GCC系列: __attribute__((visibility("")))](https://blog.csdn.net/veryitman/article/details/46756683)

那么`RCT_EXTERN void RCTRegisterModule(Class);`的含义其实就是往当前类中导入`RCTRegisterModule(Class)这个C方法`，该方法的定义在`RCTBridge.m`中。


`+ (NSString *)moduleName { return @#js_name; }` 这行是生命一个类方法，该类方法返回在RN中该类的名称。 #在C中的语法，作用是把宏参数变为一个字符，如果js_name="aaa"， 则`@#js_name` 的结果是`@"aaa"` ，则在RN中可以按一下方式访问该类

```javascript
    var TestBridgeClass = NativeModules.aaa;
```

`+ (void)load { RCTRegisterModule(self); }`则是在类中重写了Load方法，该方法调用之前导入的`RCTRegisterModule`函数, `RCTRegisterModule`方法很简单，只是在`RCTModuleClasses`数组中保存了当前导出到RN的类

```objc

static NSMutableArray<Class> *RCTModuleClasses; // RCTBridge中的全局静态变量

void RCTRegisterModule(Class moduleClass)
{
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    RCTModuleClasses = [NSMutableArray new];
  });

  RCTAssert([moduleClass conformsToProtocol:@protocol(RCTBridgeModule)],
            @"%@ does not conform to the RCTBridgeModule protocol",
            moduleClass);

  // Register module
  [RCTModuleClasses addObject:moduleClass];
}
```

### 理解RCT_EXPORT_METHOD()
`RCT_EXPORT_METHOD`源码展开如下

```objc
#define RCT_EXPORT_METHOD(method) \
  RCT_REMAP_METHOD(, method)

#define RCT_REMAP_METHOD(js_name, method) \
  _RCT_EXTERN_REMAP_METHOD(js_name, method, NO) \
  - (void)method; // 在类中声明导出的方法

#define _RCT_EXTERN_REMAP_METHOD(js_name, method, is_blocking_synchronous_method) \
  + (const RCTMethodInfo *)RCT_CONCAT(__rct_export__, RCT_CONCAT(js_name, RCT_CONCAT(__LINE__, __COUNTER__))) { \
    static RCTMethodInfo config = {#js_name, #method, is_blocking_synchronous_method}; \
    return &config; \
  }
```
`_RCT_EXTERN_REMAP_METHOD` 要知道这个宏定义的作用，我们先了解下`RCT_CONCAT`这个宏定义

```objc
#define RCT_CONCAT2(A, B) A ## B
#define RCT_CONCAT(A, B) RCT_CONCAT2(A, B)
```
显然，这个宏将被替换为 `A ## B`, `##`在C语言中是作为连接字符的意思，如果A="aa", B="BB",则 `A ## B`的结果为`aabb`, `RCT_CONCAT`在`_RCT_EXTERN_REMAP_METHOD`方法中的作用主要是生成一个类方法名。 `__LINE__` 是获取当前的line, `__COUTER__`是在当前文件中的计数，从0开始，每使用一次，计数加1 如我们的`loveFoods`方法，在这个宏定义后会生成一个名为 `+ [TestBridgeClass __rct_export__150]`的类方法， 其中的数字15由`__LINE__`提供， 数字0由`__COUNTER__`提供.

我们可以通过app的[linkmap](http://blog.cnbang.net/tech/2296/)来验证上面的内容: 
1.XCode开启编译选项Write Link Map File
XCode -> Project -> Build Settings -> 搜map -> 把Write Link Map File选项设为yes，并指定好linkMap的存储位置.![](https://ws4.sinaimg.cn/large/006tNc79gy1fpziy6kdv8j31kg0fyn1a.jpg)

编译后，到指定位置找到该文件，打开该文件，搜索`Symbols`,第一个后面就能找到你要的信息。我的内容如下

![](https://ws1.sinaimg.cn/large/006tNc79gy1fpzj1uyemnj30iy03c77q.jpg)

嗯，到这里我们就知道了iOS端这些宏定义具体做了啥，下一篇，我们来看启动流程篇吧...

if you have any question, leave a message!