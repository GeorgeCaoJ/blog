---
layout: pageshow
category: jekyll
title: 手把手jekyll自建博客
description: 总结一步步使用jekyll搭建个人github主页作为个人博客
---

## jekyll参考教程

我的建立博客主要是参考了两个教程

* [Ubuntu下安装jekyll和创建简单模板](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-jekyll-development-site-on-ubuntu-16-04)，此教程简单介绍了Ubuntu安装jekyll和创建简单模板

* [youtube视频教程](https://www.youtube.com/watch?v=ra8r2VymK3c&index=7&list=PLWjCJDeWfDdfVEcLGAfdJn_HXyM4Y7_k-)共34个小视频，3-5min每个，全程大约要使用1个小时36分钟，基本讲解了jekyll的网页逻辑，文件结构，是我主要的参考。（强烈建议[xx-net](https://github.com/XX-net/XX-Net)来科学上网）

这次搭建博客主要是参考了这两个教程，youtube的视频教程为主，基本看完就了解jekyll的文件结构和基本逻辑，然后根据自己想法，简单搭建了个人博客，push到github，初步完成了这个面包。

## why not use jekyll themes

[jekyll themes](http://jekyllthemes.org/)上面有很多现有的博客模板，可以按需选取，但是想自己接触一下前端框架和基本的网页，主要还是自己带大的有感情，可以具体了解整个博客的结构，不爽哪里了就马上改哪里，看腻了可以换个排版，所以，简单学习一下jekyll的文件系统和bootstap框架，就自己动手搭了一个。


### ubuntu 16.04下安装jekyll

参照[Ubuntu下安装jekyll和创建简单模板](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-jekyll-development-site-on-ubuntu-16-04)来安装jekyll包

### 一步步新建jekyll文件系统

以下主要参照[youtube视频教程](https://www.youtube.com/watch?v=ra8r2VymK3c&index=7&list=PLWjCJDeWfDdfVEcLGAfdJn_HXyM4Y7_k-)
先整体看一下此博客的jekyll文件系统

```
├── allblog.html  //所有文章列表页面
├── categories.html //文章分类导航页面
├── category
│   ├── BreadTalk.html
│   └── jekyll.html
├── _config.yml
├── css
│   ├── bootstrap.min.css
│   ├── bootstrap.min.js
│   ├── common.css
│   ├── font-awesome.min.css
│   ├── jquery-3.1.1.min.js
│   └── syntax.css
├── fonts  //图标库
│   ├── FontAwesome.otf
│   ├── fontawesome-webfont.eot
│   ├── fontawesome-webfont.svg
│   ├── fontawesome-webfont.ttf
│   ├── fontawesome-webfont.woff
│   └── fontawesome-webfont.woff2
├── img //博客引用的图片
│   └── home.jpg
├── _includes
│   ├── footer.html  //页面底栏
│   ├── nav.html     //导航栏
│   └── pagelist.html  
├── index.html //主页面
├── _layouts  //页面布局
│   ├── category.html  
│   ├── default.html
│   └── pageshow.html
├── _posts  //markdwn博客
│   ├── BreadTalk
│   │   └── 2016-10-08-blog-created.md  //jekyll要求博客的命名格式必须是year-month-day-title.md
│   └── jekyll
│       └── 2016-10-12-jekyll-tutorial.md
├── README.md

```

#### 新建jekyll配置文件

新建文本文件`_config.yml`,这是jeyll的基本配置文件：

```
makrdown: redcarpet //markdown格式
highlighter: rouge //代码高亮格式
baseurl: "https://georgecaoj.github.io/blog" //github个人主页地址
exclude: ['README.md'] // 将readme.md文件从jekyll文件系统去除
permalink: /:categories/:title // 采用更友好的网页地址格式： baseurl/categories/ttitle
```

#### index.html

新建`index.html`，jekyll默认将次文件解释为主页，即baseurl对应的页面，也就是[本博客主页](https://georgecaoj.github.io/blog)，一般主页的主要内容就是显示几篇最近的博客链接和摘要

```html
---
layout: default
---

<div class="home">
    <div class="post_list">
        {% for post in site.posts limit: 5 %}
            {% include pagelist.html %}
        {% endfor %}
    </div>
</div>

```

jekyll采用了`Liquid`模板系统，每个html页面的头必须包含`---`信息，其中的`layout: default`引用了文件夹`_layouts`中的`default.html`模板
