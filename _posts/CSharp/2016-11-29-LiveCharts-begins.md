---
layout: pageshow
title: 使用LiveCharts中的一些心得
category: CSharp
description: 初次使用LiveCharts遇到的一些问题和解决办法
---

## LiveCharts简介

[LiveCharts](https://lvcharts.net/)是**.NET**下强大可视化库，感谢作者[@Alberto Rodriguez](https://github.com/beto-rodriguez)的对开源的贡献。作者在[gitter](https://gitter.im/beto-rodriguez/Live-Charts)相当活跃，给大家答疑，而且作者的回复速度很快。我在gitter上问了一个问题，作者隔天就回答了，非常感谢作者。

## LiveCharts和oxyplot的比较

[oxyplot](http://www.oxyplot.org/)也是一个强大的多平台graphics库，之前就写了它的[初次使用](https://georgecaoj.github.io/blog/csharp/oxyplot-begin)，这次改用**LiveCharts**是对比了它们两个官方文档，**oxylot**的官方文档由于还在建设中，所以基本只能通过查看**github**上的example来使用。

而[LiveCharts](https://lvcharts.net/)的官方文档比较详细，而且作者相当活跃，其可视化效果更现代，风格像前端的**SVG**和**canvas**的效果。

## 添加LiveCharts的控件

[官方教程](https://lvcharts.net/App/examples/v1/wf/Install)
中已经给出了操作方法，但是可能会出现添加失败的情况

![LiveCharts1]({{site.baseurl}}/img/CSharp/LiveCharts1.jpg)

我的解决办法是：避免在中文路径下添加控件。如，在桌面新建一个项目，再进行教程中的操作流程添加成功。

![LiveCharts2]({{site.baseurl}}/img/CSharp/LiveCharts2.jpg)

## 使用中问题和解决方法

### 问题一：在同一个ColumnsSeries中改变某些Columns的颜色

这个问题的解决方法是我在[gitter](https://gitter.im/beto-rodriguez/Live-Charts)上询问的作者，感谢作者的回答。其实是一个很简单的问题，由于没有仔细参阅官方文档就急着用，这其实很不推荐这么做。

