---
layout: post
title: "Core data/MagicalRecord concurrency reseach"
date: 2013-11-26 09:17
comments: true
categories: [iOS]
---

###Core data research


* 每个线程必须有自己的NSManagedObjectContext
* NSManagedObjects不是线程安全的，但是NSManagedObjectIDs是线程安全的
* 如果在background保存，则需要通过core data Notification将changes同步到其他contexts

![CORE DATA STACK](http://www.objc.io/images/issue-4/stack-complex.png)

参考：  
[Concurrency with Core Data](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/CoreData/Articles/cdConcurrency.html)  
[Core Data Overview](http://www.objc.io/issue-4/core-data-overview.html)


### How MagicalRecord resolve multi-threading concurrency issue

以下为异步保存的code

```objc
Person *person = ...;
[MagicalRecord saveWithBlock:^(NSManagedObjectContext *localContext){

    Person *localPerson = [person MR_inContext:localContext];

    localPerson.firstName = @"John";
    localPerson.lastName = @"Appleseed";

} completion:^(BOOL success, NSError *error) {

    self.everyoneInTheDepartment = [Person findAll];

}];
```


如以上例子，在block里的save操作不会在main thread里，以独立的NSManagedObjectContext来工作。任务完成后，completion block将会被main thread调用，可以安全的更新UI部分。
操作对象NSManagedObject是通过NSManagedObjectID来得到的，在当前context中安全使用。


### saveWithBlock如何工作


* 创建一个NSPrivateQueueConcurrencyType类型的NSManagedObjectContext，parentConext为rootSavingContext
* 设置notification
* 回调block，返回新建立的context
* 如果有任何改动，保存当前context
* rootSavingContext同样会被保存
* Main thread的default context将会通过notification机制merge此改动
* main thread中回调completion，返回是否成功和error code


参考：  
[CORE DATA AND THREADS, WITHOUT THE HEADACHE](http://www.cimgf.com/2011/05/04/core-data-and-threads-without-the-headache/)  
[Performing Core Data operations on Threads](https://github.com/magicalpanda/MagicalRecord/blob/release/2.2/Docs/Threads.md)


###Conclusion


MagicalRecord非常好的解决了core data多线程并发问题，我们只需要使用好MagicalRecord就能解决UI界面freeze等的问题。