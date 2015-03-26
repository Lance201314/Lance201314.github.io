---
layout: post
title: iOS测试代码
category: iOS
keywords: 测试代码是否可以高亮
description:

---

## 测试用Jekyll是否可以使iOS代码高亮

{% highlight objective-c %}

UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"BTAddress" bundle:nil];
[self.navigationController pushViewController:[storyboard instantiateViewControllerWithIdentifier:@"viewController"] animated:YES];

{% endhighlight %}