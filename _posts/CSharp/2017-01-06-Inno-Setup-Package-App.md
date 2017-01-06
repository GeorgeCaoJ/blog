---
layout: pageshow
title: 使用第三方工具Inno Setup打包应用程序
category: CSharp
description: Inno Setup程序打包工具
---

## Inno Setup

[Inno Setup](http://www.jrsoftware.org/isinfo.php)官方介绍

> Inno Setup is a free installer for Windows programs. First introduced in 1997, Inno Setup today rivals and even surpasses many commercial installers in feature set and stability.

应用打包工具基本使用于所有window版本的系统。**including: Windows 10, Windows 8.1, Windows 8, Windows Server 2012, Windows 7, Windows Server 2008 R2, Windows Vista, Windows Server 2008, Windows XP, Windows Server 2003, and Windows 2000. (No service packs are required.)**

## 差异

之前用VS官方的打包工具[Microsoft Visual Studio 2015 Installer Projects](https://marketplace.visualstudio.com/items?itemName=VisualStudioProductTeam.MicrosoftVisualStudio2015InstallerProjects)后，发现无法生成主程序运行时的日志txt文件，安装后除了主程序输出之外都是dll库文件。没有找到方法使其将**Release**文件打包，因为转而使用**Inno Setup**。

## 发布应用程序的方法

* 通过VS自带的**publish**功能，发布**ClickOnce Application**。具体操作是在解决方案的**Property->发布**，生成一个clickOnce应用程序（不推荐）

* 通过安装VS官方的插件进行打包，可见[之前的博客](https://georgecaoj.github.io/blog/csharp/Package-Application)，只包含dll文件和主程序文件，无法输出错误日志文件（应该没有找到合适的方法）。

* 通过[Inno Setup](http://www.jrsoftware.org/isinfo.php)第三方打包工具，将VS生成的Release文件一起打包成安装工具，可以生成桌面快捷方式。

## 简单的操作方法

1. 进入软件，新建一个**Setup Script**

![Inno 1]({{site.baseurl}}/img/CSharp/Inno1.jpg)

2. 提示操作，将VS生成的Release文件添加进来。先**Browse**主运行程序**.exe**，然后将Release文件中的其他文件通过**Add file**添加。

![Inno 2]({{site.baseurl}}/img/CSharp/Inno2.jpg)

3. 根据提示设置安装文件的Icon等信息

## 应用程序的桌面快捷方式Icon

应用程序的桌面快捷方式会继用原输出程序的图标，所以只需要在VS生成之前，将程序的ICON设置为需要的图标。  
* 对于**WinForm**的程序，只需要在主窗体的属性中找到**Icon**，然后设置图标即可。  

* 对于**WPF**程序，可以在解决方案的**Property->应用程序->图标和清单**中设置即可。



