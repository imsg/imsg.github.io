---
layout: post
title: "handle the return key in UITextView"
date: 2013-12-12 09:30
comments: true
categories: [iOS]
---

今天QA同事提出希望模仿iOS native email client中写一封邮件时，在To/Cc/Bcc中填写email地址后按return键可以自动加上“, ”，不需要用户手动添加各个email地址之间的分隔符。如果写完收件人后再次return，则自动跳到下一个文本框中。
研究了一下，发现UITextFieldDelegate中有`- (BOOL)textFieldShouldReturn:(UITextField *)textField`回调，可以处理return事件，但是UITextViewDelegate没有此类方法。看了一下UITextViewDelegate类中其他方法，发现`- (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text`同样可以实现此功能。


<!--more-->


```objc
- (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text
{
    BOOL shouldChangeText = YES;
    switch (textView.tag) {
        case TextViewTagTo:
        case TextViewTagCc:
        case TextViewTagBcc:
        {
            if ([text isEqualToString:@"\n"]) {
                NSString *oldText = [textView.text stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]];
                NSUInteger oldLen = oldText.length;
                if (oldLen == 0 || (oldLen > 0 && [[oldText substringFromIndex:oldLen-1] isEqualToString:@","])) {
                    [textView setText:[oldText stringByTrimmingCharactersInSet:[NSCharacterSet characterSetWithCharactersInString:@","]]];
                    // Jump to the next field
                    [textView resignFirstResponder];
                    [[self.view viewWithTag:textView.tag+1] becomeFirstResponder];
                }
                else if (oldLen > 0) {
                    // Append ", " to the end
                    NSString *newText = [NSString stringWithFormat:@"%@%@", oldText, @", "];
                    [textView setText:newText];
                }
                shouldChangeText = NO;
            }
            break;
        }
            
        default:
            break;
    }
    return shouldChangeText;
}
```
