---
layout: post
title: "从UIColor生成UIImage，以及对UIImage进行高斯模糊"
date: 2013-12-28 22:39
comments: true
categories: [iOS，工作随笔]
---

最近UI Designer设计的UI底色为高斯模糊的效果，因此研究了一下Core Image，写了一个UIImage的扩展类，方便使用。

以下为radius为75.0f的效果：

![Gaussian Blur UIImage from UIColor](http://imsg.github.com/images/2013/blurImage.png)

<!--more-->

UIImage扩展类头文件：
```objc

@interface UIImage (Extension)

+ (UIImage *)imageWithColor:(UIColor *)color andSize:(CGSize)size;
+ (UIImage *)gaussianBlurImage:(UIImage *)image andInputRadius:(CGFloat)radius;
+ (UIImage *)gaussianBlurImageWithColor:(UIColor *)color andSize:(CGSize)size andInputRadius:(CGFloat)radius;

@end

```

UIImage扩展类的实现.m文件：
```objc
@implementation UIImage (Extension)

+ (UIImage *)imageWithColor:(UIColor *)color andSize:(CGSize)size
{
    CGRect rect = CGRectMake(0, 0, size.width, size.height);
    UIGraphicsBeginImageContextWithOptions(rect.size, NO, 0);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, color.CGColor);
    CGContextFillRect(context, rect);
    
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
    
    UIGraphicsEndImageContext();
    
    return img;
}

+ (UIImage *)gaussianBlurImage:(UIImage *)image andInputRadius:(CGFloat)radius
{
    if (!image) {
        return nil;
    }
    
    CIContext *context = [CIContext contextWithOptions:nil];
    CIImage *inputImage = [CIImage imageWithCGImage:image.CGImage];
    
    CIFilter *filter = [CIFilter filterWithName:@"CIGaussianBlur"];
    [filter setValue:inputImage forKey:kCIInputImageKey];
    [filter setValue:[NSNumber numberWithFloat:radius] forKey:@"inputRadius"];
    CIImage *result = [filter valueForKey:kCIOutputImageKey];
    CGImageRef cgImage = [context createCGImage:result fromRect:[inputImage extent]];
    
    return [UIImage imageWithCGImage:cgImage];
}

+ (UIImage *)gaussianBlurImageWithColor:(UIColor *)color andSize:(CGSize)size andInputRadius:(CGFloat)radius
{
    UIImage *image = [UIImage imageWithColor:color andSize:size];
    if (image) {
        return [UIImage gaussianBlurImage:image andInputRadius:radius];
    }
    else {
        return nil;
    }
}

@end

```
