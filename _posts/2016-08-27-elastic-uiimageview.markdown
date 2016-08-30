---
layout:     post
title:      "UIImageView与UIScrollView的一点UX效果实现"
subtitle:   "\"下拉弹性效果 | 上拉慢速压缩效果\""
date:       2016-08-27 12:12:12
author:     "DesmondDAI"
header-img: "img/in-post/post-elastic-uiimageview/header-bg-elastic-uiimageview.jpg"
tags:
    - iOS
    - Objective-C
---

<br>

> “仅阐述简单的实现思路，希望可以激发对于自定义UI控件的想象”

<br>

## demo

![自制demo效果](/img/in-post/post-elastic-uiimageview/my_app_demo.gif)


## 重点

- **UIImageView**:
  - `height`属性跟随`UIScrollView`的滑动而线性变化
  - `contentMode`属性设定为`UIViewContentModeScaleAspectFill`
- **UIScrollView**:
  - 初始`contentInset`属性的`height`设定为`UIImageView`对象的`height`

简单说就是把**scene**里的**views**设置为**UIImageView**在下**UIScrollView**在上，利用**UIScrollView**的`scrollViewDidScroll:`来线性更改**UIImageView**对象的`height`


## Storyboard的设置

先来看完整的view hierachy：

![成品的view hierachy](/img/in-post/post-elastic-uiimageview/full_view_hierachy.png)

首先，先摆一个**UIImageView**控件在**view**的最上方，配置**top**, **leading**, **trailing**的**constraints**贴边，**height**这里设为**width**的9/16

![UIImageView的配置](/img/in-post/post-elastic-uiimageview/image_view_storyboard.png)
<small class="img-hint">UIImageView（这里的view背景色设为了深灰色）</small>

![UIImageView的constraints](/img/in-post/post-elastic-uiimageview/image_view_constraints.png)
<small class="img-hint">对应的constraints</small>

别忘了配置`contentMode`为`UIViewContentModeScaleAspectFill`:
![UIImageView的contentMode](/img/in-post/post-elastic-uiimageview/image_view_content_mode.png)

接着摆一个**UIScrollView**，覆盖整个**view**，因此**UIImageView**是在**UIScrollView**的下层。此时的**UIScrollView**是空的，需要添加并且**_仅能只有_**一个子**view**，这里的子**view**取名**container view**。如此就可以在**container view**上添加自己想要的UI控件了，为了简化demo这里只加了俩**UILabel**和[doge](https://en.wikipedia.org/wiki/Doge_(meme))的图片。:full_moon_with_face:

![UIScrollView的hierachy](/img/in-post/post-elastic-uiimageview/scroll_view_hierachy.png)
<small class="img-hint">UIScrollView的hierachy</small>

![UIScrollView的内部](/img/in-post/post-elastic-uiimageview/scroll_view_storyboard.png)
<small class="img-hint">doge is watching you</small>

![UIScrollView的constraints](/img/in-post/post-elastic-uiimageview/scroll_view_constraints.png)
<small class="img-hint">UIScrollView的constraints</small>

关于**UIScrollView**需要注意的一点是它的滑动区域由`contentSize`的大小决定，因此它的功能相当丰富，可以用作多张图片的横向paging浏览，也可以像本文的例子一样纵向页面的连续滑动。目前我知道的`contentSize`的设置方式有两种：

- Storyboard里通过**container view**与**UIScrollView**的**constraints**关系让系统自动配置；
- 代码里设置

这里使用了第一种方式，也就是把**container view**的四边与**UIScrollView**边缘贴齐。但这样还没完事，因为**container view**还不知道自己的大小是多少。这里的做法是把**container view**的**width**设定为与根**view**一致，因为**scene**的根**view**会被系统配置为与手机的显示屏大小一致，这样就可以拿到**container view**的大小了。然后因为想突出滑动效果所以把**height**固定为`600`。

![container view的设置](/img/in-post/post-elastic-uiimageview/container_view_constraints.png)
<small class="img-hint">container view的constraints</small>

## 代码

按照国际惯例，先把需要操作的**views**拖`IBOutlet`过去对应的**UIViewController**里，并且声明实现`UIScrollViewDelegate`：

```objc
@interface ViewController : UIViewController<UIScrollViewDelegate>

@property (weak, nonatomic) IBOutlet UIImageView *myImageView;
@property (weak, nonatomic) IBOutlet UIScrollView *myScrollView;
```

记得把`delegate`的实现对象告诉**UIScrollView**对象（可以放在`viewDidLoad:`里）：

```objc
self.myScrollView.delegate = self;
```

接着override`viewDidLayoutSubviews`方法，这是在scene初始化时系统在根据Storyboard的设定layout完**views**之后的callback，是开发者希望在用户将要看到画面前对**views**作修改的好地方。它的官方解释里说到：

> When the bounds change for a view controller's view, the view adjusts the positions of its subviews and then the system calls this method.

因此除了在打开scene时会被调用之外，改变**view**的`bound`后也会被调用，因为**subviews**需要根据父**view**的`bound`来定位。

```objc
- (void)viewDidLayoutSubviews {
    [super viewDidLayoutSubviews];

    self.myScrollView.contentInset = UIEdgeInsetsMake(self.myImageView.frame.size.height, 0, 0, 0);
}
```

这里提一下`contentOffset`和`contentInset`的区别：

- `contentOffset`：`CGPoint`结构体，即二维坐标系的坐标`(x, y)`，对于**UIScrollView**是指`content`左上角在其**bound**里的坐标。因为滑动**UIScrollView**实质是在移动`content`，所以`contentOffset`会随着滑动而变化，例如上滑的话`contentOffset.y`是减小，下滑就是增加。
- `contentInset`：`UIEdgeInsets`结构体，与CSS里的`padding`类似。简单说就是给`content`填充的空间，这些空间分**上下左右**四个维度且不会随着滑动而改变。

这里给`myScrollView`添加与`myImageView.frame.size.height`等高的上部`contentInset`是为了看到下层的`myImageView`。

前面的铺垫终于要发挥作用啦~实现`delegate`协议里的`scrollViewDidScroll:`方法。每次滑动`myScrollView`时，系统都会到此callback，让开发者可以根据滑动做相应的逻辑！:smiling_imp:

```objc
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    CGFloat offsetY = scrollView.contentOffset.y;

    NSLog(@"offset: %f", offsetY);
    NSLog(@"inset: %f", scrollView.contentInset.top);

    CGRect myImageViewFrame = self.myImageView.frame;

    if (offsetY < 0) {
        self.myImageView.frame = CGRectMake(myImageViewFrame.origin.x, myImageViewFrame.origin.y, myImageViewFrame.size.width, -offsetY * 1.2);
    }
}
```

这里有趣的一点是改变`contentInset`会影响`contentOffset`，因此程序在运行了`viewDidLayoutSubviews`后再运行`scrollViewDidScroll:`。由此进一步知道`contentOffset`是以**内容部分**的坐标为准。
![offset_inset](/img/in-post/post-elastic-uiimageview/contentOffset_contentInset_log.png)
<small class="img-hint">scene初始化时的log</small>

`offsetY`小于0是`myScrollView`的内容坐标低于`myScrollView.bound`上边缘的时候，此处即是低于显示屏上边缘。为了实现上滑开始阶段`myImageView`有一个缩小的效果，因此把它的`height`设为`-offsetY * 1.2`。如果只需要纵向压缩的效果，设为`-offsetY`即可。

由于前面已经把`myImageView`的`contentMode`设为`UIViewContentModeScaleAspectFill`，其作用是让照片按比例填满`myImageView`，因此`height`小于照片填满时的`height`时，图片会纵向缩窄，所以上滑看起来像是以1/2的速度缩窄，反之则等比放大，即下滑拉到尽头时的弹性效果~

## 后记

其实这UX效果很大程度参考了**Stack Overflow**的一篇[回答](http://stackoverflow.com/a/33481929/4993726)。本以为需要更复杂些的代码逻辑，没想到一个简单的`contentMode`能起到这么大作用。

觉得**UIScrollView**还有好多有意思的API，有空再去学习下~

## 参考

- [使用AutoLayout配置UIScrollView](https://www.natashatherobot.com/ios-autolayout-scrollview/)
- [了解更多UIScrollView的运作机制](https://objccn.io/issue-3-2/)
