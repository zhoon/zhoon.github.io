---
layout: post
title: 理解NSCopying和NSMutableCopying
categories: [iOS]
---

NSCopying本质上就是一个协议，这个协议只有一个方法：copyWithZone:zone，NSMutableCopying也是类似的，只有一个方法是：mutableCopyWithZone:zone，想要理解NSCopying的具体作用，我们可以先看看下面一个例子。

假如我们有个Person的类，这个类有一个username的属性，现在我们先初始化一个Person的实例person1，在某些时候，我们定义了一个Person的实例person2并把person1赋值给它，此时再修改person2的username属性，即下面这样：

Person *person1 = [[Person alloc] init];<br>
person1.username = @"zhoon";<br>
......<br>
Person *person2 = person1;<br>
person2.username = @"chen";

这个时候我们会发现，person1的username也变成了“chen”，也就是改变person的属性同时也会作用在person1上面。之所以会出现这样的问题，我们可以打log看看，会发现person2的内存地址跟person1是一样的，也就是说person1和person2其实指向的是同一个对象，所以改变person2的username其实也就是改变了person1。显然这样的情况比不是我们希望发生的。

我们希望的是得到一个person对象的副本，所以我们会用copy：Person *person2 = []person1 copy]，这样看似没有什么问题，一运行发现crash了，悲剧。看log发现错误是说Person类没有找到copyWithZone:zone方法。

先说说copy，只要是调了copy来复制一个对象的副本，他就会去找这个类中的copyWithZone:zone方法，然后返回一个Person的实例，也就是副本。所以若想使某个类支持拷贝的功能，只需要声明该类遵从NSCopying协议，并实现其中的copyWithZone:zone方法即可。做了上面这些工作，重新按照上面的代码运行，这个时候我们打log会发现，person1和person2的内存地址不一样了，也就是一个Person的副本产生了，此时无论这么修改person2，也不会影响到person1了。

类似的例子，我们可以看看系统的NSString，在xcode中查看他的头文件可以发现，NSString遵从了NSCopying协议，我们看下下面的代码：

NSString *s1 = @"zhoon";<br>
NSString *s2 = [s1 copy];

按照上面的说法，这里s2是copy过来的，所以修改s2是不会影响到s1的，但是问题是，我们打log看到s1和s2的内存地址是一样的，奇怪，上面不是说copy两个对象的内存地址会不一样的吗？嗯，理论确实是这样的，只不过苹果对NSString的copy做了优化，还是指向同一个内存地址，但是也不会互相影响。因为这里跟上面还是有点差别的，想想看，修改s2的值，不就是直接把s2指向新的地址了吗？是不会影响到s1指向的值的，那何必多占用一个内存呢？而上面的person2我们只是修改他的一个属性，所以是不会自动创建一个新的person对象的。所以上面的代码就跟下面的效果是一样的：

NSString *s1 = @"zhoon";<br>
NSString *s2 = s1;

接下来看看NSMutableCopying，这个其实跟NSCopying是同一个原理，在调用mutableCopy的时候会走到这个方法来索取一个新对象。至于他们两个的区别主要是用来区分浅拷贝和深拷贝。

<strong>浅拷贝:只复制对象本身,对象里的属性,包含的对象不做拷贝;</strong><br>
<strong>深拷贝:即复制对象本身,对象里的包含对象,属性也都复制.</strong>

即你需要像下面这样实现这两个方法；

<pre><code>-(id)copyWithZone:(NSZone *)zone  
{  
    //浅拷贝的实现  
    Person *person = [[[self class] allocWithZone:zone] init];  
    person.username = _username;
}</code></pre>

<pre><code>-(id)mutableCopyWithZone:(NSZone *)zone  
{  
    //深拷贝的实现  
    Person *person = [[[self class] allocWithZone:zone] init];  
    person.username = [_username copy];
}</code></pre>

注意：<br>
<strong>1、Foundation类支持copy和mutableCopy，默认为浅拷贝，这也就解释了刚才NSString为啥copy之后还是跟原来的NSString内存地址一样<strong><br>
<strong>2、自己定义的类默认为深拷贝，也就是本身和其属性都使用了新的对象，而不是指向原来的对象，除非自己在方法里面指定为浅拷贝，像上面的copyWithZone:zoon方法<strong><br>
<strong>3、非NSObject类或者子类直接赋值即可，无需考虑浅拷贝或者深拷贝<strong>







