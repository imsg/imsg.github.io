---
layout: post
title: "值对象"
date: 2013-12-21 13:45
comments: true
categories: [iOS, 翻译]
---

翻译自[Value Objects](http://www.objc.io/issue-7/value-objects.html)

在这篇文章中，我们将看看如何用Objective-C语言编写值对象。在编写时，我们将会接触到Objective-C中的重要协议和方法。一个值对象是一个包含一些值的对象，并且可以进行相等比较。通常值对象可以被用作模型对象。例如，考虑一个简单的*Person*对象：

<!--more-->

```objc
@interface Person : NSObject

@property (nonatomic,copy) NSString* name;
@property (nonatomic,strong) NSDate* birthDate;
@property (nonatomic) NSUInteger numberOfKids;

@end
```
创建这些类型的对象是我们工作的面包和黄油（译者注：基本元素），虽然这些对象看上去很简单，但是仍然包含许多微妙之处。

有一件事，我们很多人硬性的认为这些对象应该是一成不变的。一旦你创建了一个*Person*对象，它不可能被改变。我们将在稍后涉及到可变性这个问题。


#属性

首先要注意的是我们使用属性来定义一个*Person*的特征。创建属性是想当机械的：对于普通对象的属性，你设置它们为`nonatomic`和`strong`，而对于标量属性你只需要设置`nonatomic`。默认情况下，它们也是`assign`。有一个例外，对于具有可变副本的属性，你想将他们定义为`copy`。例如，name属性的类型是`NSString`，有可能出现的情况是，有人创建了一个*Person*对象，并指定类型为`NSMutableString`的值。然后一段时间后，他或她可能会改变这个可变的字符串。如果我们的属性是`strong`而不是`copy`，我们的*Person*对象会随之改变，这不是我们想要的。对于容器类型也是一样的，例如数组或者字典。

请注意，这个拷贝是浅拷贝，容器可能还包含可变对象。例如，如果你有一个`NSMutableArray *a`包含有`NSMutableDictionary`元素，则`[a copy]`将会给你一个不可变数组，但是元素是相同的`NSMutableDictionary`对象。正如我们稍后将看到的，不可变对象的拷贝是无成本的，但是它增加了引用计数。

在旧的代码中，你可能看不到属性，因为他们是相对近期才加入到Objective-C语言的。代替现有属性，有可能会看到自定义的getter和setter方法，或纯实例变量。对于现在的代码，似乎似乎大多数人都同意使用属性，这也是我们所推荐的。

###更多阅读

[NSString:copy or retian](http://stackoverflow.com/questions/387959/nsstring-property-copy-or-retain)


#初始化方法

如果我们想要不可变对象，我们应该确保他们被创建后不能进行修改。我们可以通过添加一个初始化方法和在接口里使我们的属性只读来做到这一点。我们的接口将如下所示：

```objc
@interface Person : NSObject

@property (nonatomic,copy,readonly) NSString* name;
@property (nonatomic,strong,readonly) NSDate* birthDate;
@property (nonatomic,readonly) NSUInteger numberOfKids;

- (instancetype)initWithName:(NSString*)name
                   birthDate:(NSDate*)birthDate
                numberOfKids:(NSUInteger)numberOfKids;

@end
```

然后，在我们的实现中，我们必须使我们的属性`readwrite`，从而生成实例变量：

```objc
@interface Person ()

@property (nonatomic,copy) NSString* name;
@property (nonatomic,strong) NSDate* birthDate;
@property (nonatomic) NSUInteger numberOfKids;


@end

@implementation Person

- (instancetype)initWithName:(NSString*)name
                   birthDate:(NSDate*)birthDate
                numberOfKids:(NSUInteger)numberOfKids
{
    self = [super init];
    if (self) {
        self.name = name;
        self.birthDate = birthDate;
        self.numberOfKids = numberOfKids;
    }
    return self;
}

@end
```

现在我们可以构造新的*Person*对象，但不能修改它们了。这是非常有帮助的，当编写与*Person*对象工作的其他类时，我们知道我们正在工作的值不能改变。


#相等比较

要比较是否相等，我们必须实现`isEqual:`方法。我们希望`isEqual:`返回true当且仅当所有的属性都相等。由Mike Ash（[实现相等和散列](http://www.mikeash.com/pyblog/friday-qa-2010-06-18-implementing-equality-and-hashing.html)）和NSHipster（[相等](http://nshipster.com/equality/)）写的两篇很好的文章解释了如何做到这点。首先，让我们写`isEqual:`：

```objc
- (BOOL)isEqual:(id)obj
{
    if(![obj isKindOfClass:[Person class]]) return NO;
    
    Person* other = (Person*)obj;

    BOOL nameIsEqual = self.name == other.name || [self.name isEqual:other.name];
    BOOL birthDateIsEqual = self.birthDate == other.birthDate || [self.birthDate isEqual:other.birthDate];
    BOOL numberOfKidsIsEqual = self.numberOfKids == other.numberOfKids;
    return nameIsEqual && birthDateIsEqual && numberOfKidsIsEqual;
}
```

现在，我们检查是否我们是相同类型的类。如果不是，我们肯定不相等。然后对每个对象的属性，我们检查是否指针是相等的。||左侧的运算数似乎是多余的，但如果两个属性都为`nil`则返回`YES`。为了比较标量值相等像`NSUInteger`，我们可以只使用`==`。

有一件事值得注意：这里我们分成不同的属性到他们自己的布尔值里。在实践中，可能将它们合成一个大的条件更有意义，因为这样你直接得到惰性求值。在上面的例子中，如果名字不相等，我们就不需要检查任何其他的属性。通过把所有组合成一个if语句，我们直接得到优化。

下一步，按照[这个文档](https://developer.apple.com/library/mac/documentation/cocoa/reference/foundation/Protocols/NSObject_Protocol/Reference/NSObject.html#//apple_ref/occ/intfm/NSObject/isEqual:)，我们需要实现一个哈希函数也是如此。Apple说：

>如果两个对象相等，他们必须有相同的哈希值。如果你在子类中定义了`isEqual:`，并且打算把该子类的实例放入集合中，这最后一点就特别重要了。请确保你在你的子类中也定义了哈希。

首先，我们可以尝试运行下面没有实现哈希函数的代码：

```objc
Person* p1 = [[Person alloc] initWithName:name birthDate:start numberOfKids:0];
Person* p2 = [[Person alloc] initWithName:name birthDate:start numberOfKids:0];
NSDictionary* dict = @{p1: @"one", p2: @"two"};
NSLog(@"%@", dict);
```

我第一次跑了上面的代码，一切都很好，在字典中有两个项目。第二次，只有一个了。事情变得非常不可预测了，所以我们照着文档说的来做了。

正如你可能还记得您的计算机科学课程中，写一个好的哈希函数不是很容易的。一个好的哈希函数必须是确定性的和均匀的。确定性意味着，在相同的输入下需要生成相同的哈希值。均匀表示哈希函数的结果应该均匀地将输入映射在输出范围内。你的输出越均匀，你在集合中使用这些对象的性能越好。

首先，为了弄清楚，让我们来看看当我们没有一个哈希函数发生了什么，我们尝试使用*Person*对象作为字典的键：

```objc
NSMutableDictionary* dictionary = [NSMutableDictionary dictionary];

NSDate* start = [NSDate date];
for (int i = 0; i < 50000; i++) {
    NSString* name = randomString();
    Person* p = [[Person alloc] initWithName:name birthDate:[NSDate date] numberOfKids:i++];
    [dictionary setObject:@"value" forKey:p];
}
NSLog(@"%f", [[NSDate date] timeIntervalSinceDate:start]);
```

这在我的机器上运行需要29秒。相比之下，当我们实现一个基本的哈希函数，相同的代码运行只需要0.4秒。这不是合适的基准，但也给出了一个好的迹象，为什么要实现一个适当的哈希函数是很重要的。 对于*Person*类，我们可以用这样的哈希函数开始：

```objc
- (NSUInteger)hash
{
    return self.name.hash ^ self.birthDate.hash ^ self.numberOfKids;
}
```

这将从我们的属性中产生三个哈希值并且XOR他们。在这种情况下，对我们来说已经足够了，因为NSString的哈希函数对于短字符串来说很好（过去表现良好的字符串[最多96个字符](http://www.abakia.de/blog/2012/12/05/nsstring-hash-is-bad/)，但是现在已经改变了。见[CFString.c](http://www.opensource.apple.com/source/CF/CF-855.11/CFString.c)，寻找哈希）。对于严重的散列，你的哈希函数取决于你拥有的数据。这被[Mike Ash的文章](http://www.mikeash.com/pyblog/friday-qa-2010-06-18-implementing-equality-and-hashing.html)和[其他地方](http://www.burtleburtle.net/bob/hash/spooky.html)所提及。

在哈希的[文档](https://developer.apple.com/library/mac/documentation/cocoa/Reference/Foundation/Protocols/NSObject_Protocol/Reference/NSObject.html#//apple_ref/occ/intfm/NSObject/hash)里，有如下的段落：

>如果一个可变对象被添加到使用哈希值来确定集合中对象位置的集合中，当对象在集合中，对象的哈希方法返回的值必须不能改变。因此，无论是哈希方法必须不依赖于任何对象的内部状态信息，还是当对象在集合中你必须确保该对象的内部状态信息不会改变。因此，例如，一个可变字典可以放入一个哈希表中，但是当它在那里你不能改变它。（请注意，可能很难知道给定的对象是否在一个集合中。）

这是为了确保你的对象是不可变的另一个非常重要的原因。然后，你甚至不必担心这个问题了。

###更多阅读

- [A hash function for CGRect](https://gist.github.com/steipete/6133152)
- [A Hash Function for Hash Table Lookup](http://www.burtleburtle.net/bob/hash/doobs.html)
- [SpookyHash: a 128-bit noncryptographic hash](http://www.burtleburtle.net/bob/hash/spooky.html)
- [Why do hash functions use prime numbers?](http://computinglife.wordpress.com/2008/11/20/why-do-hash-functions-use-prime-numbers/)


#NSCopying

为了确保我们的对象是有用的，可以方便的实现`NSCopying`协议。让我们举例来说，在容器类中使用它们。对于我们类中的一个可变的变量，`NSCopying`可以被这样实现：

```objc
- (id)copyWithZone:(NSZone *)zone
{
    Person* p = [[Person allocWithZone:zone] initWithName:self.name
                                                birthDate:self.birthDate
                                             numberOfKids:self.numberOfKids];
    return p;
}
```

然而，在协议文档中，他们提到另一种方式来实现`NSCopying`：

>当类和它的内容是不可变的，通过保留原有的实现NSCopying，而不是穿件一个新的副本。

因此，对于我们不可变的版本，我们只要这样做：

```objc
- (id)copyWithZone:(NSZone *)zone
{
    return self;
}
```


#NSCoding

如果我们要序列化我们的对象，我们可以通过实现`NSCoding`来做到这一点。该协议存在两个必需的方法：

```objc
- (id)initWithCoder:(NSCoder *)decoder
- (void)encodeWithCoder:(NSCoder *)encoder
```

实现这个和实现相等方法同样简单，也比较机械：

```objc
- (id)initWithCoder:(NSCoder *)aDecoder
{
    self = [super init];
    if (self) {
        self.name = [aDecoder decodeObjectForKey:@"name"];
        self.birthDate = [aDecoder decodeObjectForKey:@"birthDate"];
        self.numberOfKids = [aDecoder decodeIntegerForKey:@"numberOfKids"];
    }
    return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder
{
    [aCoder encodeObject:self.name forKey:@"name"];
    [aCoder encodeObject:self.birthDate forKey:@"birthDate"];
    [aCoder encodeInteger:self.numberOfKids forKey:@"numberOfKids"];
}
```

关于它可以从[NSHipster](http://nshipster.com/nscoding/)和[Mike Ash的博客](http://www.mikeash.com/pyblog/friday-qa-2013-08-30-model-serialization-with-property-lists.html)中阅读更多。顺便说一句，当处理不受信任的来源，如数据来自网络，不要使用`NSCoding`。因为数据可能被篡改。通过[修改存档的数据](https://developer.apple.com/library/mac/documentation/security/conceptual/securecodingguide/Articles/ValidatingInput.html#//apple_ref/doc/uid/TP40007246-SW9)，它很可能要执行远程代码进行攻击。取而代之，使用[NSSecureCoding](http://nshipster.com/nssecurecoding/)或像JSON的自定义格式。


#Mantle

现在我们留下了一个问题：我们可以自动化它吗？事实证明，我们可以做到。一种方法是代码生成，但幸运的是有一个更好的选择：[Mantle](https://github.com/github/Mantle)。Mantle使用内省(introspection)来产生`isEqual:`和哈希。此外，它提供了一些方法来帮助你创建字典，然后可以用于写入和读取JSON。当然，一般运行时这样做将不会像自己写的代码一样有效率，但在另一方面，自动执行是一个更不容易出错的过程。


#可变性

在C语言和Objective-C语言中，可变的值是默认值。在某种程度上，它们是非常方便的，因为你可以在任何时候改变任何东西。当建立较小的系统，这应该是没有问题的。然而，正如我们许多人了解的方法，建立规模更大的系统时，事情是不可变时会相当容易。在Objective-C中，我们已经使用不可变对象很长时间了，并且现在其他语言也开始添加。

我们来看看可变对象的两个问题。一个是当你不希望它改变时它们可能会改变，另一个是在多线程环境中使用可变对象。


##意想不到的变化
假设我们有一个表视图控制器，其中有一个*People*属性：

```objc
@interface ViewController : UITableViewController

@property (nonatomic, strong) NSArray* people;

@end
```

而在我们的实现里，我们只是映射每个数组元素到一个单元格：

```objc
- (NSInteger)numberOfSectionsInTableView:(UITableView*)tableView
{
    return 1;
}

- (NSInteger)tableView:(UITableView*)tableView numberOfRowsInSection:(NSInteger)section
{
    return self.people.count;
}
```

现在，在设置了以上视图控制器的代码中，我们可能有这样的代码：

```objc
self.items = [NSMutableArray array];
[self loadItems]; // Add 100 items to the array
tableVC.people = self.items;
[self.navigationController pushViewController:tableVC animated:YES];
```

表视图将开始调用方法，如`tableView:numberOfRowsInSection:`，开始一切都很好，但是假设在某些时候，我们执行以下操作：

```objc
[self.items removeObjectAtIndex:1];
```

这改变了我们的*items*数组，但是它也改变了我们表视图控制器里的*People*数组。如果我们这样做而没有和表视图控制器有任何进一步的沟通，表视图将仍然认为有100个项目，而我们的数组只包含99个。不好的事情将会发生。取而代之，我们应该做的是以`copy`声明我们的属性：

```objc
 @interface ViewController : UITableViewController
 
 @property (nonatomic, copy) NSArray* items;
 
 @end
```

现在，无论什么时候我们分配一个可变的数组给*items*，一个不可变的副本将会创建。如果我们分配一个常规（不可变）的数组的值，拷贝操作是无害的，它仅仅增加了引用计数。


##多线程

假设我们有一个可变对象，*Account*，代表一个银行账户，它有一个方法`transfer:to:`：

```objc
- (void)transfer:(double)amount to:(Account*)otherAccount
{
    self.balance = self.balance - amount;
    otherAccount.balance = otherAccount.balance + amount;
}
```

多线程的代码可以在许多方面产生错误。例如，如果线程A读取`self.balance`，线程B可能会在线程A继续之前修改它。对于所有涉及到的危险的一个很好的解释，请参阅我们的[第二个问题](http://www.objc.io/issue-2/)。

如果我们将它替换为不可变对象，事情就容易多了。我们不能对其进行修改，这迫使我们在一个完全不同的层次上提供可变性，产生更简单的代码。


##缓存

另一件事，不可变性可以帮助的是在缓存值的时候。例如，假设你已经解析了一个markdown文档为一个包含所有不同元素节点的树形结构。如果你想生成的另外的HTML，你可以缓存这个值，因为你知道没有任何子节点会改变。如果你有可变对象，你则需要每次从零开始生成HTML，或构建优化并观察每一个单独的对象。和不变性相比，你不必担心无效的缓存。当然，这可能会带来性能损失。在几乎所有情况下，然而，简单将超过在性能上的略有下降。


##在其他语言里的不可变性

不可变对象是灵感来自于像[Haskell](http://www.haskell.org/)的函数式编程语言的概念之一。在Haskell中，值默认是不可变的。Haskell程序通常有一个[单纯功能](http://en.wikipedia.org/wiki/Purely_functional)的核心，里面没有可变对象，没有状态，而且没有副作用，像I/O。

我们可以在Objective-C编程中借鉴这个。在可能的情况下使用不可变对象，我们的项目将变得更容易测试。[Gary Bernhardt有一个很棒的讨论](https://www.destroyallsoftware.com/talks/boundaries)，显示了如何使用不可变对象来帮助我们写出更好的软件。在这个讨论中，他使用的是Ruby，但是其概念也同样适用于Objective-C语言。


###进一步阅读

- [Cocoa Encyclopedia: Object Mutability](https://developer.apple.com/library/mac/documentation/General/Conceptual/CocoaEncyclopedia/ObjectMutability/ObjectMutability.html#//apple_ref/doc/uid/TP40010810-CH5-SW1)
- [Mutability and Caching](http://garbagecollective.quora.com/Mutability-aliasing-and-the-caches-you-didnt-know-you-had)

