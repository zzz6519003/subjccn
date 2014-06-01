[原文地址](http://www.subjc.com/unread-overlay-menu)
#Unread's pull-for-menu
向你展示革命的进化

[原文地址](subjc.com/unread-overlay-menu/)


##Background
在2013中期，RSS世界发生很大变化。Google宣布Google Reader关闭。因此，无数声音在恐惧中呼喊出来然后安静下来。
下降的使用率被称作是关闭的主要原因，然而来自Google Reader用户巨大的反应表明这项服务还是有很多人在用。对于RSS的未来和开放的web的未来，网络上一阵嘘声。然而同时也有乐观，这是一个机会，对我们没法跟Google这样的巨头竞争的开发者，这是一个机会，来做一个Google Reader的替代品。
尽管)
－－
这篇文章是关于[Unread](http://jaredsinclair.com/unread/)的pull-for-menu的交互，但是他也是关于历史，我们是走了多久，如何到这里的。

##Landscape
If we were to plot the landscape of news and content aggregation apps on iOS, we might plot apps like Flipboard and Pulse (now LinkedIn Pulse) at one end of the scale, where the experience drives not only content consumption but content discovery. 这些应用是你可以在周天早晨和一杯咖啡一起享受的（如果是澳大利亚和新西兰就是茶），沉迷于阅读中。
##Evolution
如果我们看Tweetie，一个被视为iOS dev的商标的应用，他为我们带来了现在常有的下拉刷新的模式。下拉刷新变得被接受，甚至被期待有这个功能，他被苹果认可，并且被运用在了系统自带的Mail App。
然后就是[Facebook iOS app](https://itunes.apple.com/au/app/facebook/id284882215?mt=8%E2%80%8E)让导航抽屉变得流行(也就是"God Burger", "Burger Basement")。尽管他们已经一移除了这个效果

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

```
- (CGRect)frameForCollapsedState
{
	return CGRectMake(0.f, CGRectGetMidY(self.bounds) - (CGRectGetWidth(self.bounds) / 2.f), 
					  CGRectGetWidth(self.bounds), CGRectGetWidth(self.bounds));	
}
```
当我们拉伸view到他伸展的状态，我们用一个frame，高为superview height，一半宽。我们也将变化水平上的origin这样我们的view呆在superview center.

```
- (CGRect)frameForExpandedState
{
	return CGRectMake(CGRectGetWidth(self.bounds) / 4.f, 0.f, 
					  CGRectGetWidth(self.bounds) / 2.f, CGRectGetHeight(self.bounds));
}
```
为了让view的角落变圆，我们设置我们的"拉伸view"的corderRadius为view的half width，给他一个圆形的感觉（当合起来），一个近似圆的边缘（当拉伸时候）。我们也将需要更新当我们变化view时候frame的center。不然有时候会有一个rounded edge er

```
- (void)layoutSubviews
{
	[super layoutSubviews];
	self.stretchingView.layer.cornerRadius = CGRectGetMidX(self.stretchingView.bounds);
}
```
剩下的就是用我们那个有长名字的方法来在两种状态中动动动了。
大多数变量我们之前都见过，我们来看看那两个比较重要的usingSpringWithDamping和initialSpringVelocity。
usingSpringWithDamping接受一个0.0 ~ 1.0的值来确定弹性的振幅，物理上得感觉，弹性的力度。越接近1，弹得越大，反之越小。
initialSpringVelocity还接受一个CGFloat然而这个传入值将会和动画移动距离有关。 A value of 1.0 translates to the total animation distance traversed in 1 second while a value of 0.5 translates to half the animation distance traversed in 1 second.(你自己调调看吧我翻译不了。。)
/*While these parameters correspond to physical properties, for the most part it’s a case of if it feels good, do it.*/

```
[UIView animateWithDuration:0.5f
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
        
```
- (void)layoutSubviews
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

```
- (void)setProgress:(CGFloat)progress
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
对于我们的实例内容，我们将展示一些Lorem lpsum用UITextView，一个收到了一些Text Kit的爱在ios7下。尽管我们不将覆盖任何新API在这章，任何有兴趣的人应该看看[很棒的文章](http://www.objc.io/issue-5/getting-to-know-textkit.html)。Instead，我们需要的就是记住UITextView是牛逼哄哄的UIScrollView的子类。
我们希望我们的SCDragAffordanceView总是在手上，准备展现菜单。一个选择可以考虑是把他作为subview加到UITextView，并且修改他的垂直origin基于UITextView的contentOffset，但是这让UITextView干得太多了，他的责任只是展示文字。
相反我们来创建一个独立的UIScrollView的实例，我们的UITextView和SCDragAffordanceView将被添加为subviews。

```
self.enclosingScrollView = [[UIScrollView alloc] initWithFrame:self.view.bounds];
self.enclosingScrollView.alwaysBounceHorizontal = YES;
self.enclosingScrollView.delegate = self;
[self.view addSubview:self.enclosingScrollView];
```
关键一行是alwaysBounceHorizontal为YES，不管contentSize，水平拖拽总是能继续超过bounds进行拖拽。
如果我们的嵌套UITextView的水平content size不超过他的bounds，那么我们将完成

#UIViewControllerTransitioningDelegate
一个版本带来了巨大变化。如果这篇文章在iOS7之前写的，它将变得更长，而且充满了caveat。之前如果你想有像Unread的这种pull-for-menu，你必须将你的view插在当前的view controller上面，window，或者其他类似的不干净的做法。尽管这样可以给你所需要的效果，感觉这么做总是有点不合理。
很感激的是在ios7，Apple注意到了这种模式的出现，从开发者社区又做了一次榜样，提供了一种干净，准许的方式来实现这个，用的是一系列很简单的protocol。你现在可以定义自定义动画和交互式过度效果在vc之间，通过实现UIViewcontroller TransitionDelegate协议。
这个协议生命了一些脸方法，允许你返回animator对象，定义了3个view transition阶段之一：presenting， dismissing和interacting。我们的自定义过度将会被定义在presenting和dismissing阶段。
在我们的view controller，我们将定义我们实现了UIViewcontrollerTansitionDelgate协议，并且实现我们关心的两个方法`animationControllerForPresentedController:presentingController:sourceController`, `animationControllerForDismissedController:`.
既然我们提供了回调，我们需要一个viewcontroller来展现他们。Undread的menu item动画超过了本篇文章的讨论范围，所以我们就创建一个view controller(SCMenuViewController)，当被触发时展示。

```
self.menuViewController = [[SCMenuViewController alloc] initWithNibName:nil bundle:nil];

```
一旦我们创建了一个这个类的实例，我们需要设置他的`transitionDelegate`为我们的view controller并且设置他的modalPresentationStyle为UIModalPresentationCustom这样他就会产生回调。

```
self.menuViewController.modalPresentationStyle = UIModalPresentationCustom;
self.menuViewController.transitioningDelegate = self;

```
现在我们来展示我们的menu view controller，他将回调到它的transitioningDelegate（我们的view controller）来获取当前正在展示的`UIViewControllerAnimatedTransitioning`animator对象。
#UIViewControllerAnimatedTransitioning
为了提供我们的animator对象给我们的menu view controller，我们将通过创建一个简单的旧的NSObject子类，叫做`SCOverlayPresentTransition`，然后声明他遵循`UIViewControllerAnimatedTrasnitioning`协议。在我们的`animationControllerForPresentedController:presentingController:sourceController`回调中，我们将创建一个SCOverlayPresentTransition对象并返回他。

```
- (id<UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source
{
    return [[SCOverlayPresentTransition alloc] init];
}

```
为了产生消失的动画，我们将创建一个其他的NSObject子类叫做`SCOverlayDismissTransition`并且提供一个实例，当我们收到了`animationControllerForDismissedController:`回调.

```
- (id<UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed
{
    return [[SCOverlayDismissTransition alloc] init];
}

```
我们展现和消失过渡对象包含了两个方法，`transitionDuration:`和`animateTransition:`是过渡真实发生的地方。

```
UIViewController *presentingViewController = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
UIViewController *overlayViewController = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];

```
一旦我们有了展示的和被展示的view controller，我们需要把它们的view作为我们transition的container view的subview这样他们才会在动画期间都展示出来。

```
UIView *containerView = [transitionContext containerView];
[containerView addSubview:presentingViewController.view];
[containerView addSubview:overlayViewController.view];
```
`The final piece of the presenting transition is to simply animate the views however we fancy, then notify the transitionContext object whether we’ve completed our transition successfully.
`

```
overlayViewController.view.alpha = 0.f;
NSTimeInterval transitionDuration = [self transitionDuration:transitionContext];
[UIView animateWithDuration:transitionDuration
                  animations:^{
                     overlayViewController.view.alpha = 0.9f;
                 } completion:^(BOOL finished) {
                     BOOL transitionWasCancelled = [transitionContext transitionWasCancelled];
                     [transitionContext completeTransition:transitionWasCancelled == NO];
                 }];

```
这个`SCOverlayDismissTransition`将会是个差不多一样的过程，尽管是相反的。
现在当我们的view controller被展示，它将使用我们的自定义transition，保持展示的view controller的view层级。
#Closing
在我们正在靠近iOS App Store的6周年纪念日its amazing how far the app landscape has come。The idea that we can consider apps as classics is an indication of just how fast its moving。每一年开发者都被给了一堆新的玩具玩，然而总有空间给古老的令人敬重的UIScrollView。
你可以在Github上checkout this project。[https://github.com/subjc/SubjectiveCUnreadMenu]
1.
如果你感觉有些怀旧于那ios6的没好日子，这里有一个很好的克隆iOS6 pull-to-refresh https://github.com/Sephiroth87/ODRefreshControl。
如果我们的嵌套UITextView的水平content size不超过他的bounds
