---
layout: pageshow
title: 学习WPF和MVVM一段时间的总结
category: CSharp
description: 学习MVVM设计模式后第一次用于生产
---

## WPF的MVVM设计模式

从**winform**转变到**WPF**的过程，难点主要还是在MVVM的设计模式。当然，如果依然采用winform的涉及方式，在每个控件背后绑定事件的方式运用在wpf中，依然可行，但是假如GUI改版，其背后绑定的特别为此界面设计的事件不得不多数弃用。而MVVM最大的好处是将一切业务逻辑放在**ViewModel**中 ，将GUI的操作放在**view**中