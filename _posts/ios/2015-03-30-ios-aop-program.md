---
layout: post
title: objective-c 面向切面编程（AOP）
category: iOS
keywords: AOP objective-c 面向切面编程（AOP）
description: 面向切面编程（AOP）
date: 2015-03-31 00:56:01
---

## 浅谈objective-c 面向切面变成（注释：学习他人代码）

原文请参考：[技术哥](http://suenblog.duapp.com/blog/100010/iOS%E4%B8%AD%E7%9A%84%E2%80%9C%E9%9D%A2%E5%90%91%E5%88%87%E9%9D%A2%E2%80%9D%E5%BC%8F%E7%BC%96%E7%A8%8B) && [csdn博客](http://blog.csdn.net/garychow520/article/details/20499483)
## 方案一(个人觉得合适的方式)
通过使用类别(category)对运行是的方法，进行替换，实现面向切面编程的思想。
    1、创建一个类别的文件
    2、替换运行时方法的方法
    3、编写要替换的方法
    4、执行方法

## 具体方式～
2、替换运行时方法的方法(注：要引入运行时库文件#import <objc/objc-runtime.h>)
    {% highlight objective-c %}
        // 运行时，交换执行的方法
        void swizzleMethod(Class class, SEL originalSelector, SEL swizzledSelector)
        {
            // 原始方法
            Method originalMethod = class_getInstanceMethod(class, originalSelector);
            // 替换方法
            Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

            // 对一个类新增一个方法，返回新增方法是否成功, 目的是为后面替换方法。
            BOOL didAddMethod = class_addMethod(class, swizzledSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));

            if (didAddMethod) {
                // 如果添加成功，则替换实现方法体
                class_replaceMethod(class, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
            } else {
                // 添加失败，还原替换
                method_exchangeImplementations(originalMethod, swizzledMethod);
            }
        }
    {% endhighlight %}

3、编写要替换的方法
{% highlight objective-c %}
// 替换的方法，一般用于log日志等功能
- (void)swizzled_viewDidAppear:(BOOL)animated
{
    // 运行时的代码，不会是递归方式调用的～
    [self swizzled_viewDidAppear:animated];

    if (![self isKindOfClass:[UINavigationController class]]) {
    NSLog(@"%@~~~~%@", NSStringFromClass([self class]), self.title);
    }
}
{% endhighlight %}

4、执行方法
{% highlight objective-c %}
+ (void)load
{
    swizzleMethod([self class], @selector(viewDidLoad), @selector(swizzled_viewDidAppear:));
}
{% endhighlight %}

最后，在需要的地方，引入类别的头文件即可实现，一次导入，多出执行，真正的实现面向切面变成，至于为什么，只需要引入头文件就可以实现函数的替换，还在探索中～

##  方案二（这种也有硬编码的嫌疑～）
    主要是通过代理协议实现的～,实现原理还在琢磨中～，代码先记录学习～
1. 创建一个协议

{% highlight objective-c %}
#import <Foundation/Foundation.h>

@protocol DPDynamicProtocol <NSObject>

@required
- (void)doSomthing;
- (void)doOtherthing;

@end
{% endhighlight %}

2.创建实现协议的NSProxy类，在此类中拦截，捕捉异常、日志等事件

{% highlight objective-c %}
#import <Foundation/Foundation.h>
#import "DPDynamicProtocol.h"

@interface DPDynamicProxy : NSProxy <DPDynamicProtocol>
{
    @private
    id<DPDynamicProtocol> _obj; // 协议的对象
}

- (id)initWithObject:(id<DPDynamicProtocol>)obj;

@end

#import "DPDynamicProxy.h"

@implementation DPDynamicProxy

- (id)initWithObject:(id<DPDynamicProtocol>)obj
{
    _obj = obj;

    return self;
}

- (void)forwardInvocation:(NSInvocation *)invocation
{
    if (_obj) {
        NSLog(@"proxy invocation obj method: %s", __func__);

        [invocation setTarget:_obj];
        [invocation invoke];
    }
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel
{
    if ([_obj isKindOfClass:[NSObject class]]) {
        return [(NSObject *)_obj methodSignatureForSelector:sel];
    }

    return [super methodSignatureForSelector:sel];
}

- (void)doSomthing
{
    NSLog(@"proxy do something");

    [_obj doSomthing];
}

- (void)doOtherthing
{
    NSLog(@"proxy do doOtherthing");

    [_obj doOtherthing];
}
@end

{% endhighlight %}

3. 对应需要功能的类
{% highlight objective-c %}

#import <Foundation/Foundation.h>
#import "DPDynamicProtocol.h"

@interface DPNormalObject : NSObject <DPDynamicProtocol>

@end

#import "DPNormalObject.h"

@implementation DPNormalObject

- (void)doOtherthing
{
//    NSLog(@"normal object do otherthing");
}

- (void)doSomthing
{
//    NSLog(@"normal object do something");
}

@end

{% endhighlight %}
