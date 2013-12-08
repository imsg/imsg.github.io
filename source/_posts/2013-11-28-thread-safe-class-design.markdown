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


<!--more-->
### 为什么UIKit不是线程安全的？

像UIKit这样大的框架上确保线程安全是一个重大的任务，会带来巨大的成本。改变非原子property为原子property只是所需要改变的一小部分。通常你想要一次改变多个property，然后才能看到更改的结果。对于这一点，Apple不得不暴露一个方法，像CoreData的`performBlock:`和同步的方法`performBlockAndWait:`。如果你考虑大多数调用UIKit类是有关配置(configuration)，使他们线程安全更没有意义。

然而，即使调用不是关于配置(configuration)来共享内部状态，因此它们不是线程安全的。如果你已经写回到黑暗时代iOS3.2及以前的应用程序，你一定经历过当准备背景图像时使用`NSString`的`drawInRect:withFont:`随时崩溃。值得庆幸的是随着iOS4的到来，Apple提供了大部分绘图的方法和类，例如`UIColor`和`UIFont`在后台线程中的使用。

不幸的是，Apple的文档目前还缺乏有关线程安全的主题。他们建议只在主线程访问，甚至连绘画方法他们都不能保证线程安全。所以阅读iOS的版本说明总是一个好主意。

在大多数情况下，UIKit类只应该在程序的主线程使用。无论是从`UIResponder`派生的类，还是那些涉及以任何方式操作你的应用程序的用户界面。

###解除分配问题

另一个在后台使用UIKit对象的风险是“解除分配问题”。Apple在TN2109里概括了这个问题，并提出了多种解决方案。这个问题是UI对象应该在主线程中释放，因为一部分对象有可能在dealloc中对视图层次结构进行更改。正如我们所知，这种对UIKit的调用需要发生在主线程上。

由于它常见于次线程，操作或块保留调用者，这很容易出错，并且很难找到并修复。这也是在AFNetworking中长期存在的一个bug，只是因为不是很多人知道这个问题，照例，显然它很罕见，并且很难重现崩溃。在异步块操作里一贯使用`__weak`和不访问ivars会有所帮助。

###集合类

Apple有一个很好的概述文档，对iOS和Mac上列出线程安全最常见的基础类。一般情况下，不可变类，像NSArray是线程安全的，而它们的可变的变体，像NSMutableArray则不是。事实上，当在一个队列中序列化的访问时，是可以在不同线程中使用它们的。请记住，方法可能返回一个集合对象的可变变体，即使它们生命它们的返回类型是不可变的。好的做法是写一些像`return [array copy]`来确保返回的对象实际上是不可变的。

不同于像Java语言，Foundation框架不提供框架外的线程安全的集合类。其实这是非常合理的，因为在大多数情况下，你想在更高层使用你的锁去避免过多的锁操作。一个值得注意的例外是缓存，其中一个可变的字典可能会保存不变的数据-在这里Apple在iOS4中增加了NSCache，它不仅能锁定访问，还可以在低内存情况下清除它的内容。

这就是说，在你的程序中，这也许是有效的情况，其中一个线程安全的可变的字典可以很轻便的。而这要归功于类簇(class cluster)的解决方案，它可以很容易的写一个。

#原子属性(properties)

有没有想过Apple如何处理原子设置/获取属性？现在你可能已经听说过spinlocks, semaphores, locks, @synchronized - 那Apple使用什么？幸运的是，Objective-C运行是公开的，所以我们可以看看幕后发生了什么。

一个非原子属性的setter方法可能看起来像这样：

```objc
- (void)setUserName:(NSString *)userName {
      if (userName != _userName) {
          [userName retain];
          [_userName release];
          _userName = userName;
      }
}
```

这是手动retain/release变量，然而用ARC生成的代码看起来类似。让我们看看这段代码，很显然当`setUserName:`被同时调用就遇到了麻烦。我们最终可能会释放`_userName`两次，这会破坏内存，并且导致难以发现的bug。

对于任意一个非手工实现的property内部发生的是，编译器生成一个调用` objc_setProperty_non_gc(id self, SEL _cmd, ptrdiff_t offset, id newValue, BOOL atomic, signed char shouldCopy)`。在我们的例子中，调用参数是这样的：

```objc
objc_setProperty_non_gc(self, _cmd, 
  (ptrdiff_t)(&_userName) - (ptrdiff_t)(self), userName, NO, NO);
```

`ptrdiff_t`你可能看起来很怪异，但最终它是一个简单的指针算法，因为一个Objective-C类正是另一个C结构。

`objc_setProperty`调用下面的方法：

```objc
static inline void reallySetProperty(id self, SEL _cmd, id newValue, 
  ptrdiff_t offset, bool atomic, bool copy, bool mutableCopy) 
{
    id oldValue;
    id *slot = (id*) ((char*)self + offset);

    if (copy) {
        newValue = [newValue copyWithZone:NULL];
    } else if (mutableCopy) {
        newValue = [newValue mutableCopyWithZone:NULL];
    } else {
        if (*slot == newValue) return;
        newValue = objc_retain(newValue);
    }

    if (!atomic) {
        oldValue = *slot;
        *slot = newValue;
    } else {
        spin_lock_t *slotlock = &PropertyLocks[GOODHASH(slot)];
        _spin_lock(slotlock);
        oldValue = *slot;
        *slot = newValue;        
        _spin_unlock(slotlock);
    }

    objc_release(oldValue);
}
```

除了相当有趣的名字，这种方法其实是相当简单，并使用128个在PropertyLocks可用的spinlocks其中之一。这是一个务实的和快速的解决方案 - 最坏的情况是，因为一个哈希冲突，一个setter不得不等待一个不相关的setter结束。

虽然这些方法在任何公共头文件都没有声明，但可以手动调用它们。我并不是说这是一个好主意，但如果你想要原子属性和想要同时实现setter，知道这些是很有趣的并且可能会相当有用。

```objc
// Manually declare runtime methods.
extern void objc_setProperty(id self, SEL _cmd, ptrdiff_t offset, 
  id newValue, BOOL atomic, BOOL shouldCopy);
extern id objc_getProperty(id self, SEL _cmd, ptrdiff_t offset, 
  BOOL atomic);

#define PSTAtomicRetainedSet(dest, src) objc_setProperty(self, _cmd, 
  (ptrdiff_t)(&dest) - (ptrdiff_t)(self), src, YES, NO) 
#define PSTAtomicAutoreleasedGet(src) objc_getProperty(self, _cmd, 
  (ptrdiff_t)(&src) - (ptrdiff_t)(self), YES)
```

[参考这个gist](https://gist.github.com/steipete/5928690)全部片段包括处理结构的代码。但是请记住我们不建议使用这个。

###@synchronized如何？

你可能很好奇为什么Apple不使用一个已有的运行时特性`@synchronized(self)`来做属性锁。一旦你看了源代码，你将明白这还有很多事要做。Apple采用最多三个上锁/解锁序列，部分原因是他们还增加了异常展开(exception unwinding)。比起更加快速的spinlock方案，这个会慢一些。由于设置属性通常是相当快的，spinlocks是最完美的选择。当你需要确保没有代码死锁而抛出异常，`@synchronized(self)`是个好的选择。

#你自己的类

单独使用原子属性不会让你的类线程安全的。它只会保护你在setter中免受竞态条件(race conditions)，但不会保护你的应用程序逻辑。请考虑以下代码片段：

```objc
if (self.contents) {
    CFAttributedStringRef stringRef = CFAttributedStringCreate(NULL, 
      (__bridge CFStringRef)self.contents, NULL);
    // draw string
}
```

我在PSPDFKit早早就犯了这个错误。偶尔，当`contents`属性检查后被设置为`nil`，该应用程序以EXC_BAD_ACCESS崩溃了。对这个问题简单的解决办法是捕获变量：

```objc
NSString *contents = self.contents;
if (contents) {
    CFAttributedStringRef stringRef = CFAttributedStringCreate(NULL, 
      (__bridge CFStringRef)contents, NULL);
    // draw string
}
```

这样就解决了问题，但在大多数情况下，它不是那么简单的。试想一下，我们也有一个`textColor`属性，我们在一个线程中改变两次属性。那么，我们的渲染线程可能最终会使用有旧颜色值的新内容，我们得到一个奇怪的组合。这就是为什么Core Data在一个线程或队列中绑定模型对象。

对于这个问题没有一个统一标准的解决方案。使用不可变的模型是一个解决方案，但它有它自己的问题。另一种方法是限制在主线程或一个特定的队列更改现有对象，而在工作线程中使用之前生成的副本。我推荐Jonathan Sterling在<Lightweight Immutability in Objective-C>文章中为解决这个问题更多的想法。

简单的解决方法是使用`@synchronize`。其他的是非常，非常有可能让你陷入困境。更聪明的人一次又一次地在其他方法上失败了。

###实用的线程安全设计

在试图做线程安全之前，认真考虑是否是必要的。请确保它不是过早的优化。如果它像是一个配置类，考虑线程安全是没有意义的。更好的方法是抛出一些断言来确保它的正确使用：

```objc
void PSPDFAssertIfNotMainThread(void) {
    NSAssert(NSThread.isMainThread, 
      @"Error: Method needs to be called on the main thread. %@", 
      [NSThread callStackSymbols]);
}
```

现在肯定有线程安全的代码，一个很好的例子就是缓存类。一个好的方法是使用一个并行dispatch_queue为读/写锁，以最大限度地提高性能，并尝试只锁定那些真正需要的地方。一旦你开始使用多个队列用于锁定不同部位，事情将很快变得棘手。

有时候，你也可以重写你的代码，使特殊的锁不是必需的。考虑这个代码片段，是一个多播委托的形式。 （在许多情况下，使用NSNotifications会更好，但也有有效的多路广播委托用例。）

```objc
// header
@property (nonatomic, strong) NSMutableSet *delegates;

// in init
_delegateQueue = dispatch_queue_create("com.PSPDFKit.cacheDelegateQueue", 
  DISPATCH_QUEUE_CONCURRENT);

- (void)addDelegate:(id<PSPDFCacheDelegate>)delegate {
    dispatch_barrier_async(_delegateQueue, ^{
        [self.delegates addObject:delegate];
    });
}

- (void)removeAllDelegates {
    dispatch_barrier_async(_delegateQueue, ^{
        self.delegates removeAllObjects];
    });
}

- (void)callDelegateForX {
    dispatch_sync(_delegateQueue, ^{
        [self.delegates enumerateObjectsUsingBlock:^(id<PSPDFCacheDelegate> delegate, NSUInteger idx, BOOL *stop) {
            // Call delegate
        }];
    });
}
```

除非`addDelegate:`或`removeDelegate:`每秒被调用上千次，否则下面是更简洁的方法：

```objc
// header
@property (atomic, copy) NSSet *delegates;

- (void)addDelegate:(id<PSPDFCacheDelegate>)delegate {
    @synchronized(self) {
        self.delegates = [self.delegates setByAddingObject:delegate];
    }
}

- (void)removeAllDelegates {
    self.delegates = nil;
}

- (void)callDelegateForX {
    [self.delegates enumerateObjectsUsingBlock:^(id<PSPDFCacheDelegate> delegate, NSUInteger idx, BOOL *stop) {
        // Call delegate
    }];
}
```

当然，这个例子有点儿认为构造的，它可以简单的局限于在主线程更改。但对于许多数据结构，在修改方法中创建不可变的副本是值得的，让广大的应用程序逻辑并不需要处理过多的锁定。注意，我们仍然要在`addDelegate:`申请锁，否则如果委托对象被来自不同的线程同时调用，它可能会迷失。

#GCD的陷阱

对于大部分的锁定需求，GCD是完美的。这很简单，很快速，并且它的基于块的API使得它更难偶然做出不平衡锁。不过，也有不少缺陷，我们将要在这里探索其中一些。

###使用GCD作为递归锁

GCD是一个队列来序列化访问共享资源。这可以被用于锁定，但它比`@synchronized`大不相同。 GCD队列是不可重入的 - 这将打破队列特性。许多人试图使用`dispatch_get_current_queue()`来作为替代方案，这是一个坏主意。Apple在iOS6中废弃此方法自然有它的原因。

```objc
// This is a bad idea.
inline void pst_dispatch_sync_reentrant(dispatch_queue_t queue, 
  dispatch_block_t block) 
{
    dispatch_get_current_queue() == queue ? block() 
                                          : dispatch_sync(queue, block);
}
```

未完待续。。。
