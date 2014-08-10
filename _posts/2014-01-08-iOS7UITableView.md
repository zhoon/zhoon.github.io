---
layout: post
title: 模仿iOS7下的UITableView样式
categores: [iOS]
---

iOS7出来之前，好像很少人觉得UITableView很丑，或者接受不了；iOS7出来之后，各种喷，什么什么好丑阿，什么什么不能接受阿，绝对不升级iOS7阿之类的；现在，估计很多人看到iOS6，都会觉得：这UI怎么这么丑阿，很奇怪，这就是Apple厉害的地方。就我自己而言，也已经接受了iOS7的风格，看到iOS6也会觉得很丑，所以下面我想分享如何实现UITableView在iOS7以下的系统模仿iOS7的风格，这个实现思路是我们组其他人想的，我只是稍作了修改和整理。

相信iOS6的UITableView大家肯定见过，主要有两种类型：Group和Plain，Group类型是圆角并且没有打通，整个屏幕的宽度；Plain类型则是打通屏幕宽度，两种类型表格的边框都是2px。相比iOS6，iOS7的改变不少阿，首先Group变成打通屏幕宽度了，中间的边框有有相应的缩进（跟文字左对齐），而Plain保持原有的打通样式，只是跟Group一样有缩进；然后两种类型的边框都变成了1px了。

主要的思路是这样的：利用UITableViewCell的backgroundView和selectedBackgroundView属性，在normal状态和选择状态给cell设置一个自定义样式的view。

##UITableView

首先需要写一个UITableView子类CommonTableView，CommonTableView需要做的一件很重要的事，就是判断当前cell的位置，像Group类型，需要知道cell是当前section的第一个、中间或者是最后一个，甚至有可能section只有一个cell；而Plain类型也是需要知道当前cell的位置，Plain类型的cell位置主要有两种，一种是非最后一个section的最后一个cell（这种情况下是不需要画边框的），而另一种就是除了这一种的其他cell位置，需要画一个下边框。所以就有了以下这个枚举：

    typedef NS_ENUM(NSInteger, CommonTableViewCellPosition) {
        //group
	    CommonTableViewCellPositionFirstInSection = 0,
	    CommonTableViewCellPositionMiddleInSection,
	    CommonTableViewCellPositionLastInSection,
	    CommonTableViewCellPositionSingleInSection,
	    // plain
	    CommonTableViewCellPositionNormal,
	    CommonTableViewCellPositionNotInLastSectionAndLastInSection
    };

然后需要有一个函数来判断当前的cell所处的位置，这个函数是在tableView:cellForRowAtIndexPath:的时候调用的，然后返回一个position的信息，如下：

    - (CommonTableViewCellPosition)tableView:(UITableView *)tableView positionForRowAtIndexPath:(NSIndexPath *)indexPath {
	    if (self.style == UITableViewStyleGrouped) {
            NSInteger numberOfRowsInSection = [tableView.dataSource tableView:tableView numberOfRowsInSection:indexPath.section];
	 	    if (numberOfRowsInSection == 1) {
	    		return CommonTableViewCellPositionSingleInSection;
		    }
		    if (indexPath.row == 0) {
		        return CommonTableViewCellPositionFirstInSection;
		    }
            if (indexPath.row == numberOfRowsInSection - 1) {
                return CommonTableViewCellPositionLastInSection;
		    }
		    return CommonTableViewCellPositionMiddleInSection;
	    } else {
            NSInteger numberOfSectionInTableView = [tableView.dataSource numberOfSectionsInTableView:tableView];
            NSInteger numberOfRowsInSection = [tableView.dataSource tableView:tableView numberOfRowsInSection:indexPath.section];
            if (indexPath.section != numberOfSectionInTableView - 1 && numberOfRowsInSection - 1 == indexPath.row) {
                return CommonTableViewCellPositionNotInLastSectionAndLastInSection;
            } else {
                return CommonTableViewCellPositionNormal;
            }
        }
	    return CommonTableViewCellPositionNormal;
    }

这样就可以知道每个cell所处的位置了，或许有人会问，知道位置了有什么用。那当然是用来画边框的阿，像Group类型，如果cell处于section的第一个，那么它需要画上下两个边（上面不用缩进，下面需要缩进），如果是处于中间的cell，需要画一个缩进的边，最后的cell则需要画一个不缩进的边。Plain类型也是类似的原理拉。

ps：注意才初始化的时候，需要做一些样式的统一，比如说需要设置tableView的separatorColor，因为在绘制的时候边框的color是取自这个颜色值的；还有一点就是需要统一设置tableView的backgroundView，因为iOS6和7下的背景是不一样的。

##UITableViewCell

UITableViewCell也需要重写一个子类CommonTableViewCell，这个子类的所有方法如下：

    // 1、2
    - (id)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier;
    - (id)initWithReuseIdentifier:(NSString *)reuseIdentifier;
    // 3、4
    - (id)initForTableView:(UITableView *)tableView withStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier;
    - (id)initForTableView:(UITableView *)tableView withReuseIdentifier:(NSString *)reuseIdentifier;// 首选初始化方法
    // 5
    - (void)setCellUIByPosition:(CommonTableViewCellPosition)position;

首先看3和4，所有的cell初始化都必须使用这两个方法来初始化，原因是需要得到tableView，也就是cell的parentTableView，这个在绘制的时候需要用到，比如上面说的获取tableView的separatorColor；

然后是1和2，这两个是重写了系统的初始化接口，默认不用来作为cell的初始化函数，而是在3和4中会调用到这两个函数。也就是说，3和4是用来初始化的，而1和2是用来CommonTableViewCell的子类重写的，CommonTableViewCell的子类不需要重写3和4。

最后一个是5，这个是最关键的一个函数，用来获取绘制模仿iOS7的backgroundView和selectedBackgroundView。

CommonTableViewCell中有个比较重要的变量是borderEdgeInsets，这是一个UIEdgeInsets类型的变量，用来设置边框的起始位置，因为默认绘制出来的水平边框是从contentView的最小x开始画的，但是有些特殊的需求是需要所有边框都是从边缘开始画的，也就是没有缩进，这个时候就可以用borderEdgeInsets来设置了，其实这个变量就是模仿了iOS7的separatorInset，后面可以考虑把这两个合并起来，就不用每次都要设置两个了。

##CommonCellBackgroundViewHelper

这个类就是用来生产上面所说的iOS6的cell的backgroundView和selectedBackgroundView。具体原理就不说了，大家自己去看下原文件，大概说下思路：就是利用这个类来返回一个UIView，然后这个UIView会根据当前的cell的位置来添加不同的边框，所以所有的这些UIView拼起来就是有边框的cell了，这就实现了iOS7的tableView了。

##总结

相信整个过程大家大概了解了（我自己都觉得有点啰嗦），这只是一种思路，如果有其他的思路，也期待大家的分享，如果这里面有什么错误，也欢迎大家来交流哈。最后附上这个例子的[源码(Github)](https://github.com/zhoon/iOS7UITableViewStyle)。
