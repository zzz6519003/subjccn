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
剩下的就是用我们那个有长名字的方法来在两种状态中动动动了。
大多数变量我们之前都见过，我们来看看那两个比较重要的usingSpringWithDamping和initialSpringVelocity。
usingSpringWithDamping接受一个0.0 ~ 1.0的值来确定弹性的振幅，物理上得感觉，弹性的力度。越接近1，弹得越大，反之越小。
initialSpringVelocity还接受一个CGFloat然而这个传入值将会和动画移动距离有关。你自己调调看吧我翻译不了。。
/*While these parameters correspond to physical properties, for the most part it’s a case of if it feels good, do it.*/

```[UIView animateWithDuration:0.5f
                      delay:0.0f
     usingSpringWithDamping:0.4f
      initialSpringVelocity:0.5f
                    options:UIViewAnimationOptionBeginFromCurrentState
                 animations:^{
                     self.stretchingView.frame = [self frameForExpandedState];
                 } completion:NULL];
```
就是这样。只用一个方法调用和一些魔法数字，我们就可以利用ios7动态的内涵了。
#Threes a crowd

现在我们创建了SCSpringExpandingView，我们还需要创建一个view来装下SCSpringExpandingView。我们叫他SCDragAffordanceView。
SCDragAffordanceView的基本工作就是放置三个SCSpringExpandingView同时提供一个接口，我们可以传入pull-for-menu交互的进度。
为了SCSpringExpandingView的layout，我们覆盖layoutSubviews，并且把每一个frames排列齐，间距相等，位于我们的bounds中间。

```- (void)layoutSubviews
{
    [super layoutSubviews];    
    CGFloat interItemSpace = CGRectGetWidth(self.bounds) / self.springExpandViews.count;
    NSInteger index = 0;
    for (SCSpringExpandView *springExpandView in self.springExpandViews)
    {
        springExpandView.frame = CGRectMake(interItemSpace * index, 0.f, 4.f, 
                                 CGRectGetHeight(self.bounds));
        index++;
    }
}
```
既然我们的views被铺放了，我们需要更新他们当有人调用setProgress:方法。如果我们看回Unread，我们可以看到三个不同的形态：一个倒塌的，扩展的和完成的状态。开始的两个我们说过，但是最后一个代表一个状态，就是我们释放就会导致菜单被显现。
为了实现这个，我们遍历我们的三个SCSpringExpandingView并更新颜色根据progress的值是否大于或者等于1.0，跟着的是progress是否足够大以至view被展开。
```- (void)setProgress:(CGFloat)progress
{
    _progress = progress;
    
    CGFloat progressInterval = 1.0f / self.springExpandViews.count;
    
    NSInteger index = 0;
    for (SCSpringExpandView *springExpandView in self.springExpandViews)
    {
        BOOL expanded = ((index * progressInterval) + progressInterval < progress);
    
        if (progress >= 1.f)
        {
            [springExpandView setColor:[UIColor redColor]];
        }
        else if (expanded)
        {
            [springExpandView setColor:[UIColor blackColor]];
        }
        else
        {
            [springExpandView setColor:[UIColor grayColor]];
        }
        
        [springExpandView setExpanded:expanded animated:YES];
        index++;
    }
}
```
既然我们已经讲了一些新鲜的东西，//let’s take a detour down a well travelled road.

#Nested UIScrollView

问任何一个iOS开发者，他们都会告诉你，嵌套scroll view是UI元素，多到以至于Apple有一个章节https://developer.apple.com/library/ios/documentation/windowsviews/conceptual/UIScrollView_pg/NestedScrollViews/NestedScrollViews.html 在他们的UIScrollView Programming Guide里。我们学这么多革命的IOS界面不提到他们简直就是犯罪有木有。
对于我们的市里内容，我们将展示一些Lorem lpsum用UITextView，一个收到了一些Text Kit的爱在ios7下。尽管我们不将覆盖任何新API在这章，任何有兴趣的人应该看看很棒的文章在objc.io上http://www.objc.io/issue-5/getting-to-know-textkit.html。Instead，我们需要的就是记住UITextView是牛逼哄哄的UIScrollView的子类。
我们希望我们的SCDragAffordanceView总是在手上，准备展现菜单。一个选择可以考虑是把他作为subview加到UITextView，并且修改他的垂直origin基于UITextView的contentOffset，但是这让UITextView干得太多了，他的责任只是展示文字。
相反我们来创建一个独立的UIScrollView的实例，我们的UITextView和SCDragAffordanceView将被添加为subviews。
```self.enclosingScrollView = [[UIScrollView alloc] initWithFrame:self.view.bounds];
self.enclosingScrollView.alwaysBounceHorizontal = YES;
self.enclosingScrollView.delegate = self;
[self.view addSubview:self.enclosingScrollView];
```
关键一行是alwaysBou...为YES，不管contentSize，水平拖拽总是能继续超过bounds进行拖拽。
如果我们的嵌套UITextView的水平content size不超过他的bounds，那么我们将完成