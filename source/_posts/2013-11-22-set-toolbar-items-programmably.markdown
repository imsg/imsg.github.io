---
layout: post
title: "Set toolbar items programmably"
date: 2013-11-22 12:17
comments: true
categories: [iOS, dev_tips]
---

今天在做动态设置UIToolbar中的items的时候发现一个问题，要求是在UITableView在editing状态和非editing状态下切换toolbar的items。

以下是之前实现的方式
```objc
// toolbar
UIBarButtonItem *spaceItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemFlexibleSpace target:nil action:nil];
self.navigationController.toolbar.items = @[self.markButton, spaceItem, self.moveButton, spaceItem, self.trashButton];
```

在此代码运行的时候，如果当前的view controller已经不是代码的view controller时，这段代码依然会执行，从而把当前view controller的toolbar给更新了，导致toolbar乱了。

查了一下UIViewController的API，发现其自带设置toolbar item的方法：`- (void)setToolbarItems:(NSArray *)toolbarItems animated:(BOOL)animated`

以下为修改后的代码，只在代码所属的UIViewController中设置toolbar items。
```objc
    // toolbar
    UIBarButtonItem *spaceItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemFlexibleSpace target:nil action:nil];
    [self setToolbarItems:@[self.markButton, spaceItem, self.moveButton, spaceItem, self.trashButton] animated:YES];
```
