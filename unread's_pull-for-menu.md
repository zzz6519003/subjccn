#Unread's pull-for-menu

##Background
在2013中期，RSS世界发生很大变化。Google宣布Google Reader关闭。因此，无数声音在恐惧中呼喊出来然后安静下来。

##Landscape
如果我们观察
##Evolution
##Deconstructing
这些年WWDC给我们带来很多新鲜的东西玩：UIKit Dynaics, Text Kit, Sprite Kit, UIViewcontroller transition等。我们将用其中的两个来recreate Unread的菜单，UIViewcontroller transition和UIKit Dynamics，尽管我们不会直接处理后者。
图
首先我们注意到得失当我们拉内容的时候，首先像弹簧般展现出来pull indicator。。。

#That 7 parameter method
Objective-C的一个很棒的特性是有命名的变量。与语言的冗长同时而来的，他给我们一种很自然的方式来说明一个方法的目的。尽管有些方法名的长度会吓跑菜鸟。有一个这样的新的UIView block based animation方法，animateWithDuration:delay:usingSpringWithDamping:initialSpringVelocity:options:animations:completion: 尽管不是Cocoa Touch最长的方法名，但是肯定是在排行榜上的.
不管他雄伟的存在，这个方法相当简单而且有威力的可以增加动态动画效果给你的界面而无需弄那些UIKit Dynamics的一坨。有些有观察力的读者发现之前我们介绍的http://subjc.com/castro-playback-scrubber/可以用这个方法来轻易实现，因而看起来这是个极好的机会来花点功夫在这个pull-for-menu spring behaviour。
#Stretttch
如果你想像你在拉皮条，它伸展得越长，这个皮条就会越细。这个物理表现在Unread的pull interaction被复制。尽管这是个小细节一个有可能你不仔细看都不会注意到的东西，他加深了我们的错觉，在拉这个scroll view超过了他的contentSize。我们遇到了阻力。
为了模仿这个效果，我们提供了一个view(SCSPringExpandingView)，它会在两个不同的frame中动动动。这些view对于我们憋掉的，收缩状态的frame将会占据父view的

`- (CGRect)frameForCollapsedState
{
    return CGRectMake(0.f, CGRectGetMidY(self.bounds) - (CGRectGetWidth(self.bounds) / 2.f), 
                      CGRectGetWidth(self.bounds), CGRectGetWidth(self.bounds));    
}
`
当我们拉伸view到他伸展的状态，我们用一个frame，高为superview height，一半宽。我们也将变化水平上的origin这样我们的view呆在superview center.

`- (CGRect)frameForExpandedState
{
    return CGRectMake(CGRectGetWidth(self.bounds) / 4.f, 0.f, 
                      CGRectGetWidth(self.bounds) / 2.f, CGRectGetHeight(self.bounds));
}`
为了让view的角落变圆，我们设置我们的"拉伸view"的corderRadius为view的half width，给他一个圆形的感觉（当合起来），一个近似圆的边缘（当拉伸时候）。我们也将需要更新当我们变化view时候frame的center。不然有时候会有一个rounded edge er

`- (void)layoutSubviews
{
    [super layoutSubviews];
    self.stretchingView.layer.cornerRadius = CGRectGetMidX(self.stretchingView.bounds);
}
`