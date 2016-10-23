---
layout: pageshow
category: jekyll
title: jekyll自建博客，简化创建category(tag)页
description: jekyll博客category（tag）标签页的简化创建
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
.
├── allblog.html
├── categories.html
├── category
│   ├── BreadTalk.html
│   ├── C#.html
│   └── jekyll.html
├── _config.yml
├── css
│   ├── bootstrap.min.css
│   ├── bootstrap.min.js
│   ├── common.css
│   ├── font-awesome.min.css
│   ├── jquery-3.1.1.min.js
│   └── syntax.css
├── fonts
│   ├── FontAwesome.otf
│   ├── fontawesome-webfont.eot
│   ├── fontawesome-webfont.svg
│   ├── fontawesome-webfont.ttf
│   ├── fontawesome-webfont.woff
│   └── fontawesome-webfont.woff2
├── img
│   ├── home.jpg
│   └── oxyplot.png
├── _includes
│   ├── footer.html
│   ├── nav.html
│   └── pagelist.html
├── index.html
├── _layouts
│   ├── category.html
│   ├── default.html
│   └── pageshow.html
├── _posts
│   ├── BreadTalk
│   │   └── 2016-10-08-blog-created.md
│   ├── C#
│   │   └── 2016-10-22-oxyplot-begin.md
│   └── jekyll
│       └── 2016-10-12-jekyll-tutorial.md
├── README.md
```

## 简化创建标签导航页（category or tag.html)的方法

本来想把创建博客每个步骤慢慢记录下来，但是真正想写下来的时候，发现其实是光以markdown的形式，记录复杂的文件创建和引用是一件复杂的事，因此，[youtube视频教程](https://www.youtube.com/watch?v=ra8r2VymK3c&index=7&list=PLWjCJDeWfDdfVEcLGAfdJn_HXyM4Y7_k-)其实是最好的教程，基本看完就可以动手搭建了。
而视频教程中提及的标签导航页创建方法对于每个标签，都需要创建一个对应的specified_category.html来显示此标签下的所有文章，方法十分繁琐，但jekyll生成的是静态网页，所以无法动态生成。因为找了很久有没有比较简便的方法，终于在google的帮助下[找到了教程](https://christianspecht.de/2014/10/25/separate-pages-per-tag-category-with-jekyll-without-plugins/)，感谢这为德国大牛的帮助。
具体方法为：

1.在**_layouts**中新建一个**category.html**

```
---
layout: default
---

	{% for post in site.categories[page.category] %}
		{% include pagelist.html %}				
    {% endfor %}
```

新建一个layout，以后所有的specified_catgegoty.html都引用这个category.html

2.新建一个category文件夹，在文件夹中创建specified_catgegoty.html

```
├── category
│   ├── BreadTalk.html
│   ├── C#.html
│   └── jekyll.html
```

而每个specified_catgegoty.html中，只需要引用**category.html**，如jekyll.html如下：

```
---
layout: category
category: jekyll
---
```

每次要使用新的标签时，只需要在category文件夹中，新建对应的specified_catgegoty.html，只需要包含*layout*和*category*信息即可，便可以在写新博客中直接使用对应的category标签。

这种方法依然要为每个标签创建独立的html文件，但是从将重复的代码提出来放进**category.html**来简化每次创建新的标签。
