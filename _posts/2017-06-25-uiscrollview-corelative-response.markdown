---
layout:     post
title:      "UIScrollView的联动反馈效果实现"
date:       2017-06-24 12:12:12
author:     "DesmondDAI"
header-img: "img/in-post/corelative_scrollview/banner.jpg"
tags:
    - iOS
    - Swift
---

<br>

> UX的核心是在帮助用户理解应用的同时提供更愉悦的使用体验

<br>

**环境：**
- Xcode 8.3.1
- Swift 3


## 目录
- [demo](#demo)
- [关键点](#keys)
- [Storyboard的设置](#storyboard)
- [代码](#code)
- [总结](#conclusion)

<p id="demo"></p>
## demo
![demo](/img/in-post/corelative_scrollview/demo_corelative_tableview.gif)


`UIScrollView`及其子类`UITableView`, `UICollectionView`是app里经常用到的视图组件。
iOS自带的反弹特性提供了不错的反馈信息。结合实现滑动相关的delegate，我们可以构建出多种交互反馈，
给用户提供更明白又眼前一亮的效果。

以下展示一个通过滑动scroll view，与标题图片进行联动反馈的例子。这个反馈效果在很多App里都很普遍。
其实在大半年前我就写过一篇[*博客*](http://desmonddai.me/2016/08/27/elastic-uiimageview/)分享同样效果的实现方法。
后面积累了些项目经验后，发现有更简洁且稳妥的实现方式，同时开始转用Swift。往后的博客也会用Swift来写。
在此推荐一波Swift，相对OC而言，Swift提供了更多实用的语言属性，写法也比OC简洁不少。


<p id="keys"></p>
## 关键点
- **UIImageView**:
  - `height`属性跟随`UIScrollView`的滑动而线性变化
  - `contentMode`属性设定为`UIViewContentModeScaleAspectFill`
- **UIScrollView**:
  - 初始`contentInset`属性的`height`设定为`UIImageView`对象的`height`


<p id="storyboard"></p>
## Storyboard的设置
虽然标题图片与scroll view孰上孰下都会是同样的视觉效果，不过要想标题图片可以点击，建议将其image view放在上层，如图：

![视图层级](/img/in-post/corelative_scrollview/view_hierachy.png)

为了使例子更清楚，这里使用table view，scroll view和其他子类同理

**设定Auto Layout的约束：**
- image view: 上、左、右贴边，并添加高度约束，数值随意，反正需要在代码初始化并联动更改
- table view: 上、下、左、右贴边

具体如图:
![image view的constraints](/img/in-post/corelative_scrollview/image_view_constraints.png)
<small class="img-hint">image view的约束</small>

![table view的constraints](/img/in-post/corelative_scrollview/table_view_constraints.png)
<small class="img-hint">table view的约束</small>

![整体效果](/img/in-post/corelative_scrollview/vc_views.png)
<small class="img-hint">整体效果</small>

需要注意的是，image view需要设定：
- `contentMode = .scaleAspectFill`，让image view根据最短一边来等比放大和缩放
- `isUserInteractionEnabled = true`
- `clipsToBounds = true` (不设定的话会出现即使image view高度为0但image依旧会展示的效果)
- 这里image view会挡住下面table view的cell的设计。可以在设计时先把table view放在上层，
完了再放回下面，拖一下鼠标的事情


<p id="code"></p>
## 代码
最开心的进入代码的时刻来了~惯常把image view和table view的引用拉到view controller里来。同时，
把image view的height约束也拉过来：

```swift
var bannerImageViewDefaultHeight: CGFloat {
    return UIScreen.main.bounds.width * 1 / 2
}

@IBOutlet weak var bannerImageView: UIImageView!

@IBOutlet weak var tableView: UITableView! {
    didSet {
        tableView.delegate = self
        tableView.dataSource = self
        tableView.estimatedRowHeight = 40.0
        tableView.rowHeight = UITableViewAutomaticDimension
        tableView.contentInset = UIEdgeInsetsMake(bannerImageViewDefaultHeight, 0.0, 0.0, 0.0)
    }
}

@IBOutlet weak var bannerImageViewHeightConstraint: NSLayoutConstraint! {
    didSet {
        bannerImageViewHeightConstraint.constant = bannerImageViewDefaultHeight
    }
}
```

**以上代码的几个要点：**
- 这里想要标题图片的高度是屏幕宽度的一半，于是利用computed属性`bannerImageViewDefaultHeight`返回
- 设定标题图片的高度约束`bannerImageViewHeightConstraint`。为什么是使用约束更改高度？下文会详解说明
- 设定table view的`contentInset`为上端inset为初始标题图片的高度，好让table view的内容刚好在标题图片下方开始展示
- `didSet`里的代码在对象引用设定后会调用，在`didSet`之前，我们一般在`viewDidLoad`里设定视图组件的属性。
使用`didSet`可以使设定代码与对应的组件更紧凑，便于阅读。关于Swift的不同属性的解释请[*戳此*](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html)
- `estimatedRowHeight`的值影响table view对content高度的初步计算，从而影响右侧scrolling indicator的精确度；
`rowHeight = UITableViewAutomaticDimension`是让table view开启利用Auto Layout计算layout的功能，
据说苹果在iOS 11里对table view进行了优化，`rowHeight`不再需要人手设定而隐式开启使用Auto Layout。具体有待考证。


**设定滑动的联动反馈效果：**
```swift
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    let offsetY = scrollView.contentOffset.y
    bannerImageViewHeightConstraint.constant = offsetY < 0.0 ? -offsetY : 0.0
}
```

超级实用的`scrollViewDidScroll(_:)`排上用场啦~scroll view往下滑动的`contentOffset.y`是减小，
当添加大于0的上端inset，长度为`bannerImageViewDefaultHeight`时，初始的`contentOffset.y`会是`-bannerImageViewDefaultHeight`。

这里说下为什么改约束而不是直接改image view的frame呢？我曾经也习惯直接改frame，简单快捷，
不需要额外拉一条约束的引用进入view controller。但在我看完realm的一篇[*分享*](https://news.realm.io/news/gotocph-marin-todorov-auto-layout-animations-ios/)之后，
关于之前遇到的view的一些奇怪行为恍然大悟。简单来说就是，在iOS的定义里，Auto Layout相当于一系列视图组件的布局规则。
当我们在代码里直接更改frame时，我们是在Auto Layout之外进行操作。那么问题来了，系统的一些特定事件会触发布局系统重新读取我们设定在视图组件上的一系列约束，
这就意味着我们更改的frame会被重置，而这也许不是我们想要的结果。因此我现在的偏好是，如果用了Auto Layout做布局，
代码里针对view的布局更改甚至一些动画都通过修改约束来进行，这样当特定事件出现时，视图组件的布局会维持原样。
那么，有哪些系统特定事件呢？如下：
- 键盘的弹出与收回
- 手机竖屏与横屏的切换
- 视图组件或其父视图组件调用`layoutIfNeeded()`时 (其实这是根本)


<p id="conclusion"></p>
## 总结
`scrollViewDidScroll(_:)`是可以实现很多有趣的滑动反馈的开始。对不同应用，应该根据实际来实现合适的用户体验反馈。
完整代码已上传至个人Github。传送请[*戳此*](https://github.com/DesmondDAI/MasterTableView)
