---
layout: post
title: "Thread-Safe Class Design"
date: 2013-11-28 22:39
comments: true
categories: [iOS, 翻译]
---

翻译自[Thread-Safe Class Design](http://www.objc.io/issue-2/thread-safe-class-design.html)

#线程安全类的设计

此文章将侧重于编写线程安全类和使用Grand Central Displatch(GCD)时的实用的技巧，设计模式，以及反模式。


#线程安全  



### Apple的框架

首先让我们来看一下Apple的框架。一般情况下，除非提前声明，否则大多数类默认不是线程安全的。一些是我们所期望的，但是另一些却会相当有趣。

其中甚至有经验的iOS/Mac开发人员常会犯的错误是在后台线程中访问部分UIKit/AppKit。最容易犯的错误是在后台线程中对property赋值，比如图片，因为他们的内容是在后台从网络上获取的。Apple的代码是性能优化过的，如果你从不同线程去改动property，它是不会警告你的。

例如图片这种情况，一个常见的问题是你的改动会产生延迟。但是如果两个线程同时设置图片，很可能你的程序将直接崩溃，因为当前设置的图片可能会被释放两次。由于这是和时机相关的，因此崩溃通常发生在客户使用时，而并不是在开发过程中。

虽然没有官方的工具来发现这样的错误，但是有一些技巧可以避免这种错误发生。[The UIKit Main Thread Guard](https://gist.github.com/steipete/5664345)是一小段代码，可以修补任何调用UIView的setNeedsLayout和setNeedsDisplay，以及在发送调用之前检查是否执行在主线程。由于这两种方法被许多UIKit的setters方法调用（包括图片），这将会捕获许多线程相关的错误。虽然这个不使用私有API，但是我们不建议在产品程序中使用，而是最好在开发过程是使用。

UIKit非线程安全是Apple有意的设计决定。从性能方面来说线程安全没有太多好处，它实际上会使很多事情变慢。而事实上UIKit和主线程捆绑使它很容易编写并发程序和使用UIKit。你所需要做的就是确保总是在主线程上调用UIKit。



### 为什么UIKit不是线程安全的？

未完待续。。。
