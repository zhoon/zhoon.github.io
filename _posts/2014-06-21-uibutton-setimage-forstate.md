---
layout: post
title: UIButton中setImage:forState接口的小问题
category: iOS
---

<pre>此问题为团队项目中遇到的并由团队成员归纳总结，我这里用文章的形式记载总结并且分享给其他遇到同样问题的朋友参考。我们欢迎不同的想法和更多的讨论，有什么问题直接在下面评论回复。</pre>

我们知道UIButton的setImage:forState接口是用来为button设置图片用的，其中后面的forState参数就是按钮的不同状态，也就是说为按钮的不同状态设置图片。常见的forState参数有；UIControlStateNormal、UIControlStateHighlighted、UIControlStateSelected、UIControlStateDisabled这几种，也就是常态、高亮、选择、禁止这几种状态。

###项目中发现的问题：

1，如果只是设置了UIControlStateNormal这种状态，那么其他的状态系统会默认给你设置UIControlStateNormal状态的image；

2，UIControlStateSelected和UIControlStateDisabled状态貌似存在冲突，也就是说，如果当前是selected状态，那么如果此时你调用setEnabled接口设置为NO，那么按钮会置灰，并且image会变成normal状态的image，而不是selected状态的image，通过打断点发现，进入setEnabled的时候，图片还是selected状态的图片，但是过了super setEnabled之后，图片就变成了normal状态的图片了，所以这里不知道这种冲突是苹果本身设计就如此，还是这是一个bug。

如果此时有一个需求，就是setEnabled的时候，我不想要图片变为normal状态的图片再置灰色，而是保持selected状态的图片并且置灰色。那该怎么办呢？我的做法是在基类里重写setEnabled接口，如下：
<pre><code>- (void)setEnabled:(BOOL)enabled
{
    [super setEnabled:enabled];
    if (self.selected) {
        self.imageView.image = [self imageForState:UIControlStateSelected];
    } else {
        self.imageView.image = [self imageForState:UIControlStateNormal];
    }
}</code></pre>

tips：所有有forState参数的接口都有类似的问题，遇到同样的问题也可以参考这种方法。