---
layout: pageshow
title: 使用VisualStudio打包应用程序
category: CSharp
description: 通过VS官方工具包将开发的程序打包成安装程序的简单步骤
---

## 目的

(开发环境为**Visual Studio 2015**)

* 将开发好的应用程序打包成一个可安装的**.exe**文件或**.msi**文件

* 添加桌面快捷方式和图标

### 首先，安装MS官方打包工具

从[Microsoft Visual Studio 2015 Installer Projects](https://marketplace.visualstudio.com/items?itemName=VisualStudioProductTeam.MicrosoftVisualStudio2015InstallerProjects)上下载扩展并安装完成

### 开始打包

1.在VStudio中打开已完成的程序，然后**文件->添加->新建项目**选择**其他项目类型**中的**Setup Wizard**，修改名称

![VSPackage1]({{site.baseurl}}/img/CSharp/VSPackage1.jpg)

2.选择**Windows application**

![VSPackage2]({{site.baseurl}}/img/CSharp/VSPackage2.jpg)

3.选择**主输出**（若有自定义配置文件应添加其他选项）

![VSPackage3]({{site.baseurl}}/img/CSharp/VSPackage3.jpg)

4.添加额外的文件，此步额外重要，如桌面图标文件，特别是用到额外的**.dll**文件时，必须手动添加进来，比如我使用了**ControlCAN.dll**来和CAN进行通讯，所以必须添加进来

![VSPackage4]({{site.baseurl}}/img/CSharp/VSPackage4.jpg)


### 创建应用的快捷方式和桌面图标

1.右击**主输出**文件，创建快捷程序的快捷方式

![VSPackage5]({{site.baseurl}}/img/CSharp/VSPackage5.jpg)

2.将得到的**shortcut**文件拖动到**User's Desktop**文件夹中

![VSPackage6]({{site.baseurl}}/img/CSharp/VSPackage6.jpg)

3.设置**shortcut**的**Icon**属性，添加**.ico**图标。[Online Convert Image to ICO Format](http://image.online-convert.com/convert-to-ico)可以将jpg等格式图片转为ico格式，图标可以设置为`256*256`pixels

## 总结

至此，可以将应用程序简单的打包成一个安装包，但是需要主要的是，用户必须安装对应的**.NET Framework**环境才能正常运行，如果安装出现问题，会提示用户下载并安装。

