---
layout:     post
title:      "为什么你应该开始使用@weakify和@strongify 宏定义"
subtitle:   "相对于传统的打破循环引用的‘Weak/Strong圆舞曲’,@weakify和@strongify能够保证后续扩展维护的安全性"
date:       2015-06-04 21:00:00
author:     "Shen Quan"
header-img: "img/post-bg-02.jpg"

---
# 为什么你应该开始使用@weakify和@strongify 宏定义
> 本文译自 [Why you should start using@weakify and @strongify macros](http://www.holko.pl/2015/05/31/weakify-strongify/)

我必须得承认，我还没在我的工程中使用```@weakify```和```@strongify```,最近在组内的交流和twitter上的交流引起了我对这两个宏的兴趣,我想让大家学习下我所发现的以及为什么它们是不错的选择
	
## 避免循环引用的标准做法
假设我们的类有一个属性叫做 model, 我们想要当 model 中的 data 变化的时候，有一个 label 的 text 会随之改变，为了达到这目的，我们设置 model :

```objective-c 
- (void)setUpModel
{
	Model *model = [Model new];
	model.dataChanged = ^(NSString *title) {
		self.label.text = title;
	}
	self.model = model;
} 
```
通过这几行代码，我们引入了一个循环引用，我们的 ViewController 持有了 model , model 持有了一个 Block , 这个 Block 又持有了 ViewController 。我们能够通过添加 __weak 和 __strong 修饰的变量来打破循环引用

```objective-c
	Model *model = [Model new];
	__weak typeof(self) weakSelf = self;
	model.dataChanged = ^(NSString *title) {
		__strong typeof(self) strongSelf = weakSelf；
		strongSelf.label.text = title;
	}
	self.model = model;
```
这就是我们在大多数 Objective-C 代码中看到的 *weak/strong dance* , 这样的写法运行起来没有问题，但是还是有不足之处。当有了新的需求，添加新的特性，block的定义会变得越来越长，越来越复杂，仍然会有可能在 block 中使用 **self** . 我们对此无法发觉 - 编译器的提示帮助只在一些简单的例子下。这就是 weakify 和 strongify 宏发挥作用的时候了。

## weakify and strongify

@weakify 和 @strongify 的 [Original implementation](https://github.com/jspahrsummers/libextobjc#features) 比较复杂，因为他们接受多个参数。为了使分析简单一点，我们引用自己的版本，只接受一个参数:

```objective-c
#define weakify(var) __weak typeof(var) AHKWeak_##var = var;
#define strongify(var) \
_Pragma("clang diagnostic push") \
_Pragma("calng diagnostic ignored \"-Wshadow\"") \
__strong typeof(var) var = AHKWeak_##var; \
_Pragma("clang diagnostic pop")
```
看下我们对这个宏的使用:

```objective-c
Model *model = [Model new];
weakify(self);
model.dataChanged = ^(NSString *title) {
	strongify(self);
	self.label.text = tile;
};
self.model = model;
```
以上代码可以翻译为:

```objective-c 
Model *model = [Model new];
__weak typeof(self) AHKWeak_self = self;
model.dataChanged = ^(NSString *title) {
	__strong typeof(self) self = AHKWeak_self;
	self.label.text = tile;
};
self.model = model;
```
在这个 block 中，**self** 被一个同名的本地变量所覆盖，然后 **self** 就能够安全得在 block 中使用，因为它引用的是本地变量，这个本地变量是被强引用，但是它的生命周期在 block 执行完成之后就结束了。即使到了后期，维护这块代码的人更换了，或者 block 使用了他人写的代码，也能够保证 **self** 的安全。

不过现在仍然有可能通过使用 ivar 引用到 *real* **self** , 这会导致一个警告，因此，这是一个容易辨认的错误

>	*Block implicitly retains 'self'; eplicitly mention 'self' to indicate this is intended behavior*

现在，你可能会问: 假如我忘记使用 strongify 会怎么样？ 这是一个有趣的地方: weakify 创建了一个新的本地变量，所以如果它没有被使用，我们就会得到一个“未使用”的警告:

> *Unused variable 'AHKWeak_self'*

不过当一个作用域中有多个 block 时，上面的警告也显得不够智能了。

当你忘记使用 weakify , 但是却使用了 strongify ，同样会得到一个警告:

> *Use of undeclared idetifier 'AHKWeak_self'*

## Final thoughts

当我们穿件一个新的 Block 时，我们必须要去考虑是否会硬气一个 retain cycle，不能完全依赖宏。不过，一旦使用了 weakify 和 strongify 宏，任何对于 block 的未来的修改，都会比使用 标准的 **weak/strong dance** 来的更加安全。

> **(注: ReactiveCocoa 中使用了 weakify 和 strongify 的原始实现)**