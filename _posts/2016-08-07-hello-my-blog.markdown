---
layout:     post
title:      "菜鸟Desmond开了个博客"
subtitle:   "\"Hello World\""
date:       2016-08-07 12:34:56
author:     "DesmondDAI"
header-img: "img/post-bg-hello-my-blog.jpg"
tags:
    - 生活
    - Web
---

<br>

> “On my way.”

<br>

## 目录

- [前言](#foreword)
- [搭建过程](#build)
  - [介绍](#build-intro)
  - [安装过程](#build-process)
- [后记](#postscript)

<p id="foreword"></p>
---

## 前言

跌跌撞撞终于搭起了自己的博客:tada:。

多亏[Github Pages](https://pages.github.com/)提供了免费host服务以及非常适合搭建博客的[Jekyll](http://jekyll.bootcss.com/)模板引擎，也感谢[Hux](https://github.com/Huxpro)的模板，让一个接触Web开发才一个月的菜鸟不怎么费力就拥有了属于自己的一片小空间，还是挺有成就感的~

在去年就有建立个人博客的设想，因为平时也会把一些经验或想法记录在本地的**Markdown**文档里，如果放到网上不仅可以满足小小的虚荣心，还能供自己随时查阅。不过那时因为在忙mobile app的开发，而且想到从搭建服务器环境到设计模板等都需要自己去做会很耗时间，于是就一直搁置直到最近在工作里开始参与Web开发。

Web前端开发的多样性和趣味性极大地引起了我的兴趣，平时上网看到些UI/UX设计美轮美奂的页面时都很惊叹，想着自己要能做出这样酷的网页那就太帅了~

其实这些动机还都不足以让我在刚开始的几天里进入“吃饭上班睡觉搭博客”的生活模式，真正“导火索”是上周投出的七份简历没有一个回复的挫败感= =。也许有不同的原因，但觉得有个订制的博客来展示自己的话也许招聘公司会有兴趣些，自己也能积累些Web开发的基础，又可以满足当初搭建博客的愿望，于是在某个下班回家的晚上就马上开始了。

<p id="build"></p>
---

## 搭建过程

<p id="build-intro"></p>

### 介绍

之前就听说Github可以托管网站，于是找到了[Github Pages](https://pages.github.com/)，这项服务提供每个Github帐号一个帐号对应的以及无限制的项目对应的网站托管。服务使用方法也相当方便，源代码直接`git push`到远端仓库，网站就能重新加载上线，省掉了system admin的繁琐。官方推荐的一个项目方案是[Jekyll](http://jekyll.bootcss.com/)。Jekyll是特别适合项目文档和个人博客的静态模板生成器，从我使用一周多的时间来看，觉得有以下好处：

- 支持**Markdown**，方便专注于写作内容以及在主流网站都拥有相当不错的渲染效果
- 项目结构简单清晰
- 支持使用**Liquid**语言在模板里读取项目参数以及添加逻辑
- 自带轻量级Web服务器，方便本地调试完再`push`到远端仓库

在一个**Jekyll**项目里，包括了以下的基本组成部分：

- `_config.yml`：项目范围的配置信息，如域名、SEO标题等，语法是一种叫**YAML**(Yet Another Markdown Language)的标记语言。
- `_includes`：摆放可以复用的模板组件，例如我的项目里是放着`head.html`等在多个页面都用到的组件。具体的引用方式请参考[Jekyll](http://jekyll.bootcss.com/)或者更详细的[Liquid](https://shopify.github.io/liquid/)文档。
- `_layouts`：摆放主页面的模板。
- `_posts`：摆放博客文章，推荐使用**Markdown**

<p id="build-process"></p>

### 安装过程

**_首先在本地搭建好Jekyll所需环境_**：**Jekyll**是基于**Ruby**开发的，建议使用**gem**来安装，步骤不介绍了，具体点[这里](http://jekyllcn.com/docs/installation/)。

**_在Github上创建项目_**：因为是与帐号对应，因此项目名建议是：`yourGithubName.github.io`，例如我的是`desmonddai.github.io`。

![创建Github项目](/img/in-post/post-hello-my-blog/github-pages-new-setup.png)
<small class="img-hint">创建Github博客项目</small>

**_克隆项目到本地，创建Jekyll目录_**：先`git clone`项目到本地，接着有两种方式创建**Jekyll**目录：

1. 全新**Jekyll**目录
2. 在别人**Jekyll**模板基础上修改

我比较懒，直接用了[Hux](https://github.com/Huxpro/huxblog-boilerplate)的样板模板，操作是把模板项目`git clone`下来后，把根目录下所有文件（除了`.git`）复制到自己项目根目录下。如果是方式1，可以用**shell**执行`jekyll new myBlog`，之后就会看到多了一个**myBlog**的文件夹，把里面的所有东西复制到你项目根目录即可。

**_开启本地Web服务器调试_**：方式1新建的**Jekyll**项目已经可以运行。在你的项目根目录下打`jekyll serve`开启服务器，可以看到4000端口被监听，浏览器输入`localhost:4000`，接着就是见证奇迹的时刻：

![Jekyll初始项目的外观](/img/in-post/post-hello-my-blog/jekyll-new-project-web-page.png)
<small class="img-hint">Jekyll初始项目已经提供了个基本模板</small>

**_提交更改，推送到远端仓库_**：本地调试没有问题的话，就可以`git commit`提交更改，`git push`到远端仓库后一般马上就能看到新更改的版本，域名是`yourGithubName.github.io`。也可以自定义域名，我是在[NameCheap](https://www.namecheap.com/)上购买，6.88刀1年，不算贵，需要设置**CNAME**，在项目根目录下创建**CNAME**文件，里面填上你的个人域名即可。

![CNAME配置](/img/in-post/post-hello-my-blog/github-pages-cname.png)
<small class="img-hint">项目CNAME配置，用于自定义域名</small>

<p id="postscript"></p>
---

## 后记

每次看完大牛们的博客有所收获的时候都满怀感激之情，期望自己未来也可以像他们一样，把学到的东西反馈给社区，哪怕只是鼓舞到一个读者的学习兴趣，那这个博客就是有价值的。目前的我无论在技术还是写作表达上还有太多需要学习的地方，期望能与更多小伙伴们交流学习，请不要吝啬表达内心的想法，在评论里留下你们的意见吧~:muscle:

参考:

- [使用Github Pages和Jekyll搭建个人博客](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/)
- [Jekyll中文网](http://jekyll.bootcss.com/)
