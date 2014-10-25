---
layout: post
title: iOS动态字体DynamicType的实现(1)
categories: [iOS]
---

动态字体是iOS7才被引入到iPhone上的，我们可以在手机的设置-通用-字体大小里面设置手机显示的字体大小，设置后我们可以发现很多系统自带的app界面字体或者布局都有所改变，如果我们的app也想要提供这样一个动态字体的体验，那就可以用iOS7引入的DynamicType来实现。

DynamicType是属于Text Kit下面的一个特性。Text Kit是建立在Core Text框架上的，CoreText.framework是一个庞大而复杂的框架，而Text Kit在继承了Core Text强大功能的同时给开发者提供了比较友好的面向对象的API。

今天这篇文章会先介绍DynamicType的简单使用，后面会再写文章介绍DynamicType的进阶使用。想要使用DynamicType来提升app的体验很容易只需要按照下面的步骤来，几分钟搞定：

#设置UIkit的font属性

iOS7的UIFont提供了一个接口preferredFontForTextStyle:返回一个UIFont的实例，用来设置UIkit的font属性。其参数是一个UIFontTextStyle的NSString对象，有下面六个值：

UIFontTextStyleHeadline<br>
UIFontTextStyleSubheadline<br>
UIFontTextStyleBody<br>
UIFontTextStyleFootnote<br>
UIFontTextStyleCaption1<br>
UIFontTextStyleCaption2

也就是说，我们无需再跟以前一样设置UIKit的font为某个固定的数值(pointSize)，而是设置为系统默认的这六个值。如下：

<pre><code>UILabel *textLabel = [[UILabel alloc] init];
textLabel.font = [UIFont preferredFontForTextStyle:UIFontTextStyleBody];
</code></pre>

#接收DynamicType的NSNotificationCenter

上一步中我们设置了UILabel的font，但是运行之后在设置里面改变系统的字体大小，发现UILabel并没有变化，这是肯定的，因为你没有在系统字体update之后去update你的font，你的程序只是单纯在初始化的时候设置了一次font是不够，我们需要在系统设置字体大小的时候动态更新我们app的字体。

iOS提供了一个UIContentSizeCategoryDidChangeNotification的消息通知，我们要做的是注册这个消息通知，当接收到通知之后，updata UIlabel的font属性就可以了。

<pre><code>- (id)initWithStyle:(UITableViewStyle)style {
	if (self = [super initWithStyle:style]) {
		if (IOS_VERSION >= 7.0) 
		{
			[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleContentSizeCategoryDidChanged:) name:UIContentSizeCategoryDidChangeNotification object:nil];
		}
	}
}
// 回调方法 handleContentSizeCategoryDidChanged:
- (void)handleContentSizeCategoryDidChanged:(NSNotification *)notification 
{
   textLabel.font = [UIFont preferredFontForTextStyle:UIFontTextStyleBody];
   // 注意这里需要调用setNeedsLayout然后在layoutSubview或者viewDidLayoutSubview里面更新textLabel的frame
   [self.view setNeedsLayout]; 
}
</code></pre>

好了，动态字体就这样完成了。总结起来就是这样的：注册一个DynamicType的消息通知，然后在系统字体大小改变的时候来update界面上需要支持DynamicType的UIKit的font，最后再layout一下界面的布局即可。实现的效果我们可以看看下面的效果图：

![](/img/artical/dynamictype2.png)

![](/img/artical/dynamictype.png)














