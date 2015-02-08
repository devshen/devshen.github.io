---
layout:     post
title:      "ReacativeCocoa 教程 1/2"
subtitle:   "本文通过一个示例，一步步构建分析代码，解析 ReactiveCocoa 的编程方式和思想精髓，熟悉其最常用的属性和方法，开启函数响应式编程的大门"
date:       2015-02-08 23:00:00
author:     "Shen Quan"
header-img: "img/post-bg-02.jpg"

---
# 【译】ReacativeCocoa教程 1/2
> 本文译自```ReactiveCocoa Tutorial – The Definitive Introduction: Part 1/2``` post by **Colin Eberhardt** [原文链接](http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1)

作为一名iOS开发者，基本上你写的每一行代码都是在响应一些事件：一个按钮的点击，一条接收到的网络信息，一个属性的变化（通过KVO）或者用户地理位置的变化（通过```CoreLocation```）,但是，这些事件使用了不同的编写方式比如actions，delegates，KVO，callbacks等。[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)为这些事件定义了一种统一的标准接口，使得事件能够更加容易的通过一个基本的工具集来联结，过滤和组合。

听起来有点疑惑?感兴趣?...有点小激动?那么，继续往下读吧 :]

ReactiveCocoa 结合了2种编程方式：

- [Functional Programming](http://en.wikipedia.org/wiki/Functional_programming) 使用高阶函数，将其他函数作为自己的参数
- [Reactive Programming](http://en.wikipedia.org/wiki/Reactive_programming) 集中处理数据流和变化的传递

因为这个，你可能听过 **ReactiveCocoa** 被描述为Functional Reactive Programming (or FRP) framework

这就是本教程要说的学术内容，编程范式是一个有趣的话题，但本教程仅着眼于实际价值，用实际的例子来替换学术理论讨论

## The Reactive Playground
这篇ReactiveCocoa教程，会通过一个非常简单的示例应用来向你介绍响应式编程（Reactive Programming），ReactivePlayground。下载[starter project](http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/ReactivePlayground-Starter.zip)，然后编译运行，确保一切正常

ReactivePlayground 是一个简单的app，向用户展示了一个登录界面。支持账户密码鉴权，鉴权成功就会进入到一个"猫咪欢迎界面"
![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-1.png)

现在我们来看下这个初始工程，很简单，不会花太长时间

打开 ```RWViewController.m```，你能很快分辨出 SignIn 按钮的 enable 条件吗?隐藏和显示 signInFailure label的规则是什么?在这个例子中，你也许只需要1、2分钟来回答上面的问题，但是在一个复杂的工程中，你很可能会花比较长的时间。通过使用ReactiveCocoa，应用的逻辑会显示的更加清楚，是时候开始了！

## 添加ReactiveCocoa框架


添加ReactiveCocoa框架最简单的方式是通过[CocoaPods](http://cocoapods.org).如果你从未使用过CocoaPods,可以看下[Introduction To CocoaPods](http://www.raywenderlich.com/64546/introduction-to-cocoapods-2)教程，通过教程中的最少步骤，你就能将安装好本教程所需的必备条件。
> 如果由于某些原因你不想使用CocoaPods，你也能够使用ReactiveCocoa，只要按照Github上的文档步骤来就行[Importing ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa#importing-reactivecocoa)

如果你还打开着 ReactivePlayground，关掉它，CocoaPods能够创建一个Xcode Workspace，来代替原始的工程文件。

打开Terminal，进入到你的工程目录，输入以下：

```vim
touch Podfile
open -e Podfile
```
这样会创建一个空的Podfile文件，并用TextEdit打开，复制黏贴下面的内容：

```vim
platform :ios, '7.0'
 
pod 'ReactiveCocoa', '2.1.8'
```
> 译者注：你也可以将7.0换成8.0，也可以删除'2.1.8',这样会使用最新的ReactiveCocoa版本

保存文件，回到Terminal窗口，输入以下

```vim
pod install
```
你会看到类似如下输出：

```vim
Analyzing dependencies
Downloading dependencies
Installing ReactiveCocoa (2.1.8)
Generating Pods project
Integrating client project
 
[!] From now on use `RWReactivePlayground.xcworkspace`.
```
这表明ReactiveCocoa框架已被下载，CocoaPods创建了Xcode Workspace将框架合入你当前的工程

打开新生成的workspace，**RWReactivePlayground.xcworkspace**，看下CocoaPods创建的工程结构:

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-2.png)

你会看到CocoaPods创建了一个新的workspace，并添加了我们的原始工程，**RWReactivePlayground**，以及一个包含了ReactiveCocoa的**Pods**工程，CocoaPods真的将库依赖变得简单。

## 试一试，玩一玩
之前介绍中说过，ReactiveCocoa提供了一个标准的接口去处理应用内不同的事件流。在ReactiveCocoa的术语中被称为 **signals** ，由**RACSignal**类来声明。

打开**RWViewController.m**，导入ReactiveCocoa头文件

```
#import <ReactiveCocoa/ReactiveCocoa.h>
```
现在还不急着替换代码，我们先简单玩一玩这个框架，在`ViewDidLoad`下添加如下代码：

```objective-c
[self.usernameTextField.rac_textSignal subscribeNext:^(id x) {
  NSLog(@"%@", x);
}];
```
编译运行，在 username text field中输入```is this magic?```,得到如下输出：

```
2013-12-24 14:48:50.359 RWReactivePlayground[9193:a0b] i
2013-12-24 14:48:50.436 RWReactivePlayground[9193:a0b] is
2013-12-24 14:48:50.541 RWReactivePlayground[9193:a0b] is 
2013-12-24 14:48:50.695 RWReactivePlayground[9193:a0b] is t
2013-12-24 14:48:50.831 RWReactivePlayground[9193:a0b] is th
2013-12-24 14:48:50.878 RWReactivePlayground[9193:a0b] is thi
2013-12-24 14:48:50.901 RWReactivePlayground[9193:a0b] is this
2013-12-24 14:48:51.009 RWReactivePlayground[9193:a0b] is this 
2013-12-24 14:48:51.142 RWReactivePlayground[9193:a0b] is this m
2013-12-24 14:48:51.236 RWReactivePlayground[9193:a0b] is this ma
2013-12-24 14:48:51.335 RWReactivePlayground[9193:a0b] is this mag
2013-12-24 14:48:51.439 RWReactivePlayground[9193:a0b] is this magi
2013-12-24 14:48:51.535 RWReactivePlayground[9193:a0b] is this magic
2013-12-24 14:48:51.774 RWReactivePlayground[9193:a0b] is this magic?
```
你会看到，每次你改变了 text Field 的值之后，block中的代码就会执行，没有target-action，没有delegates，只有 signals 和 blocks. 是不是挺兴奋的

ReactiveCocoa signals 发送了events给它的订阅者，有三种events类型需要知道：**next**,**error**,**completed**。一个 signal 可以发送任意数量的 next events，在它由于 error 或者 completes 而关闭之前。在这篇教程中，你会关注这个 **next** event. 记得阅读教程的第二部分，学习error和completed events。

RACSignal 有一些方法，你可以用来订阅这些不同的 event 类型。每个方法使用一个或者更多的 blocks， 当一个 event 发生的时候，你 block 中的逻辑就会被执行。在上面的例子中，你会看到 ```subscribeNext:```方法，在每一个 next event 中执行block。

ReactiveCocoa框架使用category给数量众多的 UIKit 添加了 signals，这样你就能添加 subscriptions 给这些 events, 这就是 **rac_textSignal** 属性的由来。

> 译者注：如果你对代码中 signal 和 subscriber 这个说法感到不好理解，你也可以这样想，当你创建一个 signal 时, 就像创建了一个入口处有开关的管道，管道的开关是关着的，外部的 value 是不会传递到管道当中去的，当你对这个 signal 执行了```subscribeNext:```, 相当于将管道的开关给打开了，vlaue 就会进入管道，往 block 中传递，block 其实就是订阅者。

ReactiveCocoa有不少的方法来操作 events stream。举个栗子，假设你只需要打印长度大于3的 username , 你可以通过 **filter** 操作，更新你刚才添加`ViewDidLoad`的代码：

```objective-c
[[self.usernameTextField.rac_textSignal
  filter:^BOOL(id value) {
    NSString *text = value;
    return text.length > 3;
  }]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  }];
```
编译运行，你会看到如下输出：

```
2013-12-26 08:17:51.335 RWReactivePlayground[9654:a0b] is t
2013-12-26 08:17:51.478 RWReactivePlayground[9654:a0b] is th
2013-12-26 08:17:51.526 RWReactivePlayground[9654:a0b] is thi
2013-12-26 08:17:51.548 RWReactivePlayground[9654:a0b] is this
2013-12-26 08:17:51.676 RWReactivePlayground[9654:a0b] is this 
2013-12-26 08:17:51.798 RWReactivePlayground[9654:a0b] is this m
2013-12-26 08:17:51.926 RWReactivePlayground[9654:a0b] is this ma
2013-12-26 08:17:51.987 RWReactivePlayground[9654:a0b] is this mag
2013-12-26 08:17:52.141 RWReactivePlayground[9654:a0b] is this magi
2013-12-26 08:17:52.229 RWReactivePlayground[9654:a0b] is this magic
2013-12-26 08:17:52.486 RWReactivePlayground[9654:a0b] is this magic?
```
这里你创建了一个很简单的 pipeline (管道)，这是 Reactive Programming (响应式编程)的精髓，你使用 data flows(数据流)的形式来描述你的应用功能。

把这些 flow 图像化也许更好理解：

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-3.png)

图中你能看到，**rac_textSignal** 是事件的源，数据流通过一个 filter(只有string长度大于3的数据才能通过)，将 username 的值传到管道末端的 block 处。

这里应该注意到，filter 的输出也是一个 RACSignal。你能够像下面这样重写代码解构 pipeline 步骤：

```objective-c
RACSignal *usernameSourceSignal = 
    self.usernameTextField.rac_textSignal;
 
RACSignal *filteredUsername = [usernameSourceSignal  
  filter:^BOOL(id value) {
    NSString *text = value;
    return text.length > 3;
  }];
 
[filteredUsername subscribeNext:^(id x) {
  NSLog(@"%@", x);
}];
```
因为对 RACSignal 的每一个操作，都返回一个 RACSignal， 它被称作 [fluent interface](http://en.wikipedia.org/wiki/Fluent_interface), 这个特性允许你能够不使用变量，去组合构建 pipelines。

> ReactiveCocoa 大量使用 blocks，如果你对 blocks 还不是很了解，你可以阅读下 Apple 的 [Blocks Programming Topic](https://developer.apple.com/library/ios/documentation/cocoa/Conceptual/Blocks/Articles/00_Introduction.html), 或者，你对 blocks 比较熟悉，但是对其写法结构有疑惑，记不住，你可以看下 [f*****gblocksyntax.com](http://fuckingblocksyntax.com)，挺有用的。

## 小转换
还原代码

```objective-c
[[self.usernameTextField.rac_textSignal
  filter:^BOOL(id value) {
    NSString *text = value; // implicit cast
    return text.length > 3;
  }]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  }];
```
有注释的那一行中，id 类型被隐式转换成 NSString，这不够优雅，幸运的是，传递给 block 的值始终都是 NSString，你可以自己改变参数类型，更新代码如下：

```objective-c
[[self.usernameTextField.rac_textSignal
  filter:^BOOL(NSString *text) {
    return text.length > 3;
  }]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  }];
```
编译允许，运行结果还是一样的。

## 什么是 Event ?
到现在为止，本教程已经描述了不用的 event 类型，但是还没有详细说过这些 events 的结构，有趣的是，一个 event 能够包含任何东西

举个栗子，你准备对这个 pipeline 添加另一个操作，在`ViewDidLoad`中添加如下代码：

```objective-c
[[[self.usernameTextField.rac_textSignal
  map:^id(NSString *text) {
    return @(text.length);
  }]
  filter:^BOOL(NSNumber *length) {
    return [length integerValue] > 3;
  }]
  subscribeNext:^(id x) {
    NSLog(@"%@", x);
  }];
```
如果你编译运行，你会发现应用现在打印出的是不再是输入的文字内容，而是其长度

```
2013-12-26 12:06:54.566 RWReactivePlayground[10079:a0b] 4
2013-12-26 12:06:54.725 RWReactivePlayground[10079:a0b] 5
2013-12-26 12:06:54.853 RWReactivePlayground[10079:a0b] 6
2013-12-26 12:06:55.061 RWReactivePlayground[10079:a0b] 7
2013-12-26 12:06:55.197 RWReactivePlayground[10079:a0b] 8
2013-12-26 12:06:55.300 RWReactivePlayground[10079:a0b] 9
2013-12-26 12:06:55.462 RWReactivePlayground[10079:a0b] 10
2013-12-26 12:06:55.558 RWReactivePlayground[10079:a0b] 11
2013-12-26 12:06:55.646 RWReactivePlayground[10079:a0b] 12
```
新添加的 map 操作把 event 数据通过给定的 block 给转化为其他数据了。对每一个接收到的 **next** event，程序运行给定的 block ，并将返回值作为新的 **next** event。在上面的代码中，map 操作获取 NSString 输入的长度，将长度返回了。同样来看下图形解析：

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-4.png)

在 map 之后的所有步骤，都是接收 NSNumber 数据。你能够使用 map 操作来转换接收到的数据变成任何你想要的数据，不过必须是个对象类型。

> 在上面的例子中，**text.length** 的返回类型是 NSUInteger，不属于对象，为了使用它作为一个 event 的值，必须对它进行封装，[Objective-C literal syntax](https://www.mikeash.com/pyblog/friday-qa-2012-06-22-objective-c-literals.html) 提供了简单的方法```– @(text.length)```

## 创建有效状态的Signal
首先你要创建2个 signals，用来指代 username 和 passowrd text field 是否是有效的。添加如下代码到 `ViewDidLoad`:

```objective-c
RACSignal *validUsernameSignal =
  [self.usernameTextField.rac_textSignal
    map:^id(NSString *text) {
      return @([self isValidUsername:text]);
    }];
 
RACSignal *validPasswordSignal =
  [self.passwordTextField.rac_textSignal
    map:^id(NSString *text) {
      return @([self isValidPassword:text]);
    }];
```
上面的代码，map 将 rac_textSignal 的流转化成了 boolean 流(通过 NSNumber 类型)，第二步，是将 boolean 流转化成 color 流，就是说，你 subscribe 这个 signal，使用 signal 的值来更新 text field 的背景色，一种可行的方法是如下：

```objective-c
[[validPasswordSignal
  map:^id(NSNumber *passwordValid) {
    return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
  }]
  subscribeNext:^(UIColor *color) {
    self.passwordTextField.backgroundColor = color;
  }];
```
(但是，请不要添加上面的代码，因为还有更加优雅的方法)

上面的代码，获取到转化后的颜色之后，就将颜色值赋值给 text field，但是，这样的代码表述并不清晰，这是种退步。

幸运的是，ReactiveCocoa 有一个宏，让你能够优雅的描述这种场景。添加如下代码到你创建的两个 signals 下方：

```objective-c
RAC(self.passwordTextField, backgroundColor) =
  [validPasswordSignal
    map:^id(NSNumber *passwordValid) {
      return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
    }];
 
RAC(self.usernameTextField, backgroundColor) =
  [validUsernameSignal
    map:^id(NSNumber *passwordValid) {
     return [passwordValid boolValue] ? [UIColor clearColor] : [UIColor yellowColor];
    }];
```
**RAC** 宏将 signal 的输出赋值给了一个对象的属性，这个宏需要2个参数，第一个是包含你要赋值的属性的对象，第二个是你要赋值的属性，每次 signal 发出一个 next event，event 的值就会赋值给给定的属性。

这是一种非常优雅的解决方法，不是吗

最后一件事，找到 ```updateUIState```方法，去掉第1、2行

```objective-c
self.usernameTextField.backgroundColor = self.usernameIsValid ? [UIColor clearColor] : [UIColor yellowColor];
self.passwordTextField.backgroundColor = self.passwordIsValid ? [UIColor clearColor] : [UIColor yellowColor];
```
这样就清除了 non-reactive 的代码

编译运行，你会发现 text field 在无效的时候会高亮，有效的时候会 clean。

看着还可以，这里我们有2条简单的 pipelines，用来传递 text signals，map 它们变成"有效状态"的 booleans，然后第二次 map 成 UIColor，将这个 UIColor 和 text field 的 background Color 绑定。

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-5.png)

你是不是有疑问，为什么要创建2个分开的```validPasswordSignal```和```validUsernameSignal``` signals，而不是只创建一个 fluent pipeline，后面我们就会知道了，:]

## 组合 signals
**Sign In** 按钮只在 username 和 passoword text field 都有有效输入的时候，才能正常工作，现在是时候做这个场景的 reactive-style 了。

目前的代码已经的 signals 是发出一个 boolean 值，来指示 username 和 password field 是否有有效输入：```validUsernameSignal```和```validPasswordSignal```。你的任务是组合这两个 signals，来指示什么时候 enable 登录按钮。

在`ViewDidLoad`的最后添加：

```objective-c
RACSignal *signUpActiveSignal =
  [RACSignal combineLatest:@[validUsernameSignal, validPasswordSignal]
                    reduce:^id(NSNumber *usernameValid, NSNumber *passwordValid) {
                      return @([usernameValid boolValue] && [passwordValid boolValue]);
                    }];
```
上面的代码使用```combineLatest:reduce: ```方法联结了```validUsernameSignal```和```validPasswordSignal```的值，转变为一个新的 signal。 每次两个 signal 源中的任何一个发出一个新的 value，reduce block 都会被执行，并且 block 的返回值会被当做联结后的 signal 的 next value。

现在我们有了一个新 signal，添加如下代码到`ViewDidLoad`的最后：

```objective-c
[signUpActiveSignal subscribeNext:^(NSNumber *signupActive) {
   self.signInButton.enabled = [signupActive boolValue];
 }];
```
运行之前，前去掉旧的实现，删除下面两个属性

```objective-c
@property (nonatomic) BOOL passwordIsValid;
@property (nonatomic) BOOL usernameIsValid;
```
删除`ViewDidLoad`中的如下代码：

```objective-c
// handle text changes for both text fields
[self.usernameTextField addTarget:self
                           action:@selector(usernameTextFieldChanged)
                 forControlEvents:UIControlEventEditingChanged];
[self.passwordTextField addTarget:self 
                           action:@selector(passwordTextFieldChanged)
                 forControlEvents:UIControlEventEditingChanged];
```
同时再删除```updateUIState```,```usernameTextFieldChanged```和```passwordTextFieldChanged```方法，以及`ViewDidLoad`中对```updateUIState```的调用。

如果你编译运行，留心 Sign In 按钮，当 username 和 password text fields 内容有效的花，按钮就会 enable。

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-6.png)

上面的图有两个重要的概念，能让你完成强大的功能：

- 过滤 - signals 能够有多个订阅者，能够为多个后面的 pipeline 做源，在上面的图表中，注意到表述 username 和 password 有效性的boolean signals 被分开用于不同的目的

- 联结 – 多个 signals 能够组合成新的 signal，在这次的例子中，2个 boolean signal 被联结，你能够联结任意类型的 signal。

这些变化的结果是，应用不再需要私有属性去表示当前两个 text fields 的有效状态。这是一个重要的不同，当你接受一个 reactive style -- 你不需要使用实例变量去追踪临时状态

## 响应式 Sign-in
应用目前使用了响应式 pipeline (上面举例的图) 来管理 text fields 和 按钮的状态。但是，按钮点击处理仍然使用了 actions，那么下一步就是替换遗留下的应用逻辑，来使得他们都成为 Reactive。

**Touch Up Inside** 点击事件被封装为 signInButtonTouched 方法，通过 storyboard action 连接。你即将用 Reactive 来替换它，那么首先，先去除当前的方法连接。

打开 Main.storyboard，找到 Sign In 按钮，ctrl-click 打开连接面板，点击 x 去除连接，如果不会的话，可以看下图：

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-7.png)

目前为止，你使用过 rac_textSignal (当 text 变化的时候，发送 event)，为了处理按钮事件，你要使用 rac_signalorControlEvents.

来到 RWViewController.m，添加如下代码到 `ViewDidLoad`的最后：

```
[[self.signInButton
   rac_signalForControlEvents:UIControlEventTouchUpInside]
   subscribeNext:^(id x) {
     NSLog(@"button clicked");
   }];
```
上面的代码创建了一个基于 `UIControlEventTouchUpInside` 事件的 signal，添加了一个 subscription 来打印一个消息当 event 发生的时候。

编译运行，每当你点击一次按钮，就会出现如下输出：

```
2013-12-28 08:05:10.816 RWReactivePlayground[18203:a0b] button clicked
2013-12-28 08:05:11.675 RWReactivePlayground[18203:a0b] button clicked
2013-12-28 08:05:12.605 RWReactivePlayground[18203:a0b] button clicked
2013-12-28 08:05:12.766 RWReactivePlayground[18203:a0b] button clicked
2013-12-28 08:05:12.917 RWReactivePlayground[18203:a0b] button clicked
```
那么现在按钮有了一个点击事件 signal，下一步就是将 signal 和 登录处理 连接起来。打开 RWDummySignInService.h 看下接口：

```objective-c
typedef void (^RWSignInResponse)(BOOL);
 
@interface RWDummySignInService : NSObject
 
- (void)signInWithUsername:(NSString *)username
                  password:(NSString *)password 
                  complete:(RWSignInResponse)completeBlock;
 
@end
```
这个服务使用 username，password，completion block 作为参数，给定的 block 会在登录成功或者失败之后执行，你可以直接在 `subscribeNext:` 中使用这个接口，但是干嘛要这么做呢? 这种场景是一种异步事件驱动类型场景，对于 ReactiveCocoa 来说很简单，有更优雅的方式。

## 创建 Signals
幸运的是，用已有的异步 APIs 改编成 signals 比较简单。首先，删除 RWViewController.m 中的 `signInButtonTouched:` 方法，你不需要这个逻辑，因为它会被 响应式方法 替换。

在 RWViewController.m 中添加如下方法：

```objective-c
-(RACSignal *)signInSignal {
  return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
    [self.signInService
     signInWithUsername:self.usernameTextField.text
     password:self.passwordTextField.text
     complete:^(BOOL success) {
       [subscriber sendNext:@(success)];
       [subscriber sendCompleted];
     }];
    return nil;
  }];
}
```
上面的方法创建了一个 signal，作用是使用当前的 username 和 password 登录，我们来看看它的组成部分。

代码中使用 `createSignal:` 方法创建一个 signal。描述这个 signal 的 block 在是一个单入参的闭包，被传递给这个方法，当 signal 有一个订阅者的时候，block 中的代码就会被执行。

block 的入参是一个 subscribe 实例，遵循 RACSubscriber 协议，其有方法能够发送 events，你可能也需要发送任意数目的 next events，以 error 或者 complete 结束，这里它发送一个 next event 用来表示登录是否成功

这个 block 的返回类型是 RACDisposable，它允许你在 subscription 被取消或者结束的时候，做一些清理工作。这个 signal 不需要清理，所以返回 nil。

你看，封装一个异步 API 是不是很简单！

现在要开始使用这个新的 signal。在 `viewDidLoad` 中更新代码：

```objective-c
[[[self.signInButton
   rac_signalForControlEvents:UIControlEventTouchUpInside]
   map:^id(id x) {
     return [self signInSignal];
   }]
   subscribeNext:^(id x) {
     NSLog(@"Sign in result: %@", x);
   }];
```
上面的代码使用 map 方法将按钮点击 signal 转变为 sign-in signal，订阅者简单的打印下结果。

如果你编译运行，点击 Sign In 按钮，你会看到如下结果...感觉不像是我们想要的

```
2014-01-08 21:00:25.919 RWReactivePlayground[33818:a0b] Sign in result:<RACDynamicSignal: 0xa068a00> name: +createSignal:
```
`subscribeNext:` block 被传递了一个 signal，但是不是 sign-in signal 的结果，是时候用图形来看下发生了什么：

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-8.png)

`rac_signalForControlEvents` 发送了一个 next event 当你点击按钮时，map 操作创建并返回 sign-in signal，这意味着后面的 pipeline 接收到的是一个 RACSignal，这就是你所打印出来的那个信息。

这种情况被称为 *signal of signals*，换句话说就是一个外部 signal 包含了一个 inner signal。如果你想的话，你可以在 outer signal 的 `subscribeNext:` block 中 subscribe 这个 inner signal，但是这样的话，看起来会很杂乱，幸运的是，这是一个普遍的问题，ReactiveCocoa 有准备。

## Signal of Signals
这个问题的解决方式简单易懂，只要将 map 操作变为 flattenMap 操作：

```objective-c
[[[self.signInButton
   rac_signalForControlEvents:UIControlEventTouchUpInside]
   flattenMap:^id(id x) {
     return [self signInSignal];
   }]
   subscribeNext:^(id x) {
     NSLog(@"Sign in result: %@", x);
   }];
```
这里仍然像之前那样将按钮点击事件映射为 sign-in signal，但是同时，通过从 inner signal 发送 events 到 outer signal 来 flattens 它，编译运行，会看到如下：

```
2013-12-28 18:20:08.156 RWReactivePlayground[22993:a0b] Sign in result: 0
2013-12-28 18:25:50.927 RWReactivePlayground[22993:a0b] Sign in result: 1
```
现在 pipeline 正常输出了你想要的，最后一步是给 `subscribeNext` 添加登录成功后逻辑，用以下代码替换：

```
[[[self.signInButton
  rac_signalForControlEvents:UIControlEventTouchUpInside]
  flattenMap:^id(id x) {
    return [self signInSignal];
  }]
  subscribeNext:^(NSNumber *signedIn) {
    BOOL success = [signedIn boolValue];
    self.signInFailureText.hidden = success;
    if (success) {
      [self performSegueWithIdentifier:@"signInSuccess" sender:self];
    }
  }];
```
`subscribeNext:` block 获取了 sign-in signal 的结果，更新  `signInFailureText` text field, 如果登录成功则跳转画面

编译运行，成功！

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-9.png)

不知道你注意到没有，当前的应用有一个小的用户体验问题，当 sign-in 服务正在运行时，**Sign In** 按钮应该置为不可用，这防止用户重复登录操作，另外，如果登录失败，失败信息应该在用户再次尝试登录的时候隐藏。

但是，如何在当前的 pipeline 中添加这些逻辑?改变按钮的状态并不是转化，或者过滤，或者其他你已知的概念，它被称为 `side-effect`，或者说是一种逻辑，当一个 next event 发生时，你想要在一个 pipeline 中执行的逻辑，同时它并不改变当前 event。

## 添加 side-effects
用如下代码替换当前的 pipeline ：

```
[[[[self.signInButton
   rac_signalForControlEvents:UIControlEventTouchUpInside]
   doNext:^(id x) {
     self.signInButton.enabled = NO;
     self.signInFailureText.hidden = YES;
   }]
   flattenMap:^id(id x) {
     return [self signInSignal];
   }]
   subscribeNext:^(NSNumber *signedIn) {
     self.signInButton.enabled = YES;
     BOOL success = [signedIn boolValue];
     self.signInFailureText.hidden = success;
     if (success) {
       [self performSegueWithIdentifier:@"signInSuccess" sender:self];
     }
   }];
```
你能看到上面代码在按钮点击之后直接添加了 `doNext:` 操作，注意 `doNext：` 的 block 并不返回值，因为这是一个 `side-effect`，不改变 event。

`doNext:` block 将按钮的 enable 属性置为 NO，并隐藏 failure text，之后 `subscribeNext:` block 重置按钮的 enable 属性，并依据登录结构显示或隐藏错误信息。

![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-10.png)

编译运行，你会看到不错的效果。

如果你参照教程写代码时写乱了，代码运行不正确，你可以下载 [final project](http://cdn1.raywenderlich.com/wp-content/uploads/2014/01/ReactivePlayground-Final.zip) (已经完成CocoaPods依赖)，或者你可以在 [Github](https://github.com/ColinEberhardt/RWReactivePlayground) 上查看 commit 记录，一步步运行。

## 总结
希望这篇教程能够帮助你在你自己的应用中使用 ReactiveCocoa，对这种编程思想做些练习，但是，就像其他语言或程序，当你找到窍门之后，它就会变得相对简单。 ReactiveCocoa 的深层思想，只有 event streams，没有别的了。

我在 ReactiveCocoa 中发现的一个有趣事情是，你能有很多种方式来解决同一个问题。你也许会想要用本教程的应用来做练习，尝试用不同的 split 和 combine 来调整你的 signals 和 pipelines。

ReactiveCocoa 是为了让你的代码更加清晰，更容易理解。个人认为，如果一个应用它的业务逻辑使用 fluent 句法，用信息的 pipeline 来表述，它就更加容易理解。

在本教程的 [second Part](http://www.raywenderlich.com/62796/reactivecocoa-tutorial-pt2) 中，你会学习到进阶的 subjects 比如 error 处理，以及如何管理在不同线程上运行的代码，在这之前，have fun experimenting!

> 译者注: 工作比较忙，加班比较多，第二部分的教程还没翻译好，一旦翻译完成，我会在本文的开头和结尾附上翻译地址，在此，感谢原作者 **Colin Eberhardt** 的分享，大家有兴趣可以去他的Blog看看，更多精彩哦~，再附上 [原文链接](http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1)，[Colin Eberhardt's Blog](http://www.scottlogic.com/blog/ceberhardt/)，[Colin Eberhardt's Github](https://github.com/colineberhardt)

大家如果喜欢这篇博客，可添加feed订阅，<font color="orange">follow</font> 我的[Github](https://github.com/devshen)，我会将自己的学习心得分享出来，谢谢~ :]