---
layout:     post
title:      "ReacativeCocoa教程 1/2"
subtitle:   "译自【ReactiveCocoa Tutorial Part 1/2】by Colin Eberhardt -- Raywenderlish"
date:       2015-02-05 12:00:00
author:     "Shen Quan"
header-img: "img/post-bg-02.jpg"

---
# 【译】ReacativeCocoa教程 1/2
> 本文译自```ReactiveCocoa Tutorial – The Definitive Introduction: Part 1/2```**Colin Eberhardt** [原文链接](http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1)

作为一名iOS开发者，基本上你写的每一行代码都是在响应一些事件：一个按钮的点击，一条接收到的网络信息，一个属性的变化（通过KVO）或者用户地理位置的变化（通过```CoreLocation```）,但是，这些事件使用了不同的编写方式比如actions，delegates，KVO，callbacks等。[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)为这些事件定义了一种统一的标准接口，使得事件能够更加容易的通过一个基本的工具集来联接，过滤和组合。

听起来有点疑惑?感兴趣?...有点小激动?那么，继续往下读吧 :]

ReactiveCocoa 结合了2种编程方式：

- [Functional Programming](http://en.wikipedia.org/wiki/Functional_programming) 使用高阶函数，将其他函数作为自己的参数
- [Reactive Programming](http://en.wikipedia.org/wiki/Reactive_programming) 集中处理数据流和变化的传递

因为这个，你可能听过 **ReactiveCocoa** 被描述为Functional Reactive Programming (or FRP) framework

这就是本教程要说的学术内容，编程范式是一个有趣的话题，但本教程仅着眼于实际价值，用实际的例子来替换学术理论讨论

## The Reactive Playground
这篇ReactiveCocoa教程，会通过一个非常简单的示例应用来向你介绍响应式编程（Reactive Programming），ReactivePlayground。下载[starter project](http://cdn2.raywenderlich.com/wp-content/uploads/2014/01/ReactivePlayground-Starter.zip)，然后编译运行，确保一切正常

ReactivePlayground 是一个简单的app，向用户展示了一个登录界面。支持账户密码鉴权，通过鉴权就会进入到一个"猫咪欢迎界面"
![](https://github.com/devshen/devshen.github.io/raw/master/img/2015-02-05-ReactiveCocoaTutorial-1/1-1.png)

未完待续...

