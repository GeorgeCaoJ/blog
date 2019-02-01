---
layout: pageshow
category: tools
title: UCRT及VC2015环境安装记
description: VC2015依赖于UCRT组件
---


# UCRT及VC2015环境安装记

## 什么是VC运行时库

Visaul C++运行时库就是一组动态库（dll)文件，用对应版本的Visual Studio开发编译的C++程序都会依赖对应版本的VC运行时库，其对应关系基本就是VS20xx-VC20xx，参考 [The latest supported Visual C++ downloads](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads)

| Visual Studio 版本 | Visual C++ Rumtime 版本 |
| ------------------ | ----------------------- |
| VS2008             | VC2008                  |
| VS2010             | VC2010                  |
|  VS2012|VC2012|
|VS2013|VC2013|
|VS2015|VC2015|
|VS2017|VC2017|
这些发布的运行时库也会有问题，所以官方提供了更新包来修复。
以下就是我安装VC2015时遇到的踩坑的经历分享。

## VC2015的不同点在于UCRT组件
### UCRT组件
> 在 Visual Studio 2015 中，重构了 Microsoft C 运行时库 (CRT)。 将标准 C 库、POSIX 扩展和 Microsoft 特定的函数、宏和全局变量移动到了新库，即通用 C 运行时库（通用 CRT 或 UCRT）。 将 CRT 特定于编译器的组件移动到了新的 vcruntime 库中。
UCRT 现为 Windows 组件，并作为 Windows 10 的一部分提供。 UCRT 支持基于 C 调用约定的稳定 ABI，且谨遵 ISO C99 标准（仅有少数例外）。 它将不再绑定到特定版本的编译器。 可以在 Visual Studio 2015 或 Visual Studio 2017 支持的任何 Windows 版本上使用 UCRT。 其好处是使用者不再需要在每次升级 Visual Studio 时都以新版本的 CRT 为目标来更新自己的版本。
使用此重构时，许多 CRT 头文件、库文件和可再发行组件的名称或位置以及代码所需的部署方法都有所改变。 此外，还在 UCRT 中添加或更改了许多功能和宏以提高对标准的一致性。 若要利用这些更改，必须更新现有代码和项目生成系统。

详情可参见[将代码升级到通用 CRT](https://docs.microsoft.com/zh-cn/cpp/porting/upgrade-your-code-to-the-universal-crt?view=vs-2017)和[UCRT官方团队介绍](https://blogs.msdn.microsoft.com/vcblog/2015/03/03/introducing-the-universal-crt/)

### UCRT部署
UCRT现在是Windows操作系统的一个组件，被包含在了Windows10系统中，但之前版本的系统只有通过额外安装才能支持部署，微软推荐方式也是默认方式是使用Windows Update的方式，对Win10之前的操作系统安装[KB2999226](https://support.microsoft.com/en-us/help/2999226/update-for-universal-c-runtime-in-windows)补丁更新，这也是微软官方建议的集中部署方式。  
但是Windows操作系统本身也存在很多坑，会出现在客户机上安装补丁失败的情况，这样会出现发布的软件在客户机上跑不起来的情况，所以可以采用本地部署方案，
> 支持 UCRT 的本地应用部署（尽管由于性能和安全原因不推荐）。 用于本地应用部署的 DLL 作为 Windows SDK 的一部分包含在 redist 子目录下。 所需的 DLL 包括 ucrtbase.dll 和名为 api-ms-win-subset.dll. 的一组 APISet forwarder DLL。 每个操作系统所需的 DLL 集各不相同，因此建议在使用应用本地部署时包括所有 DLL。 有关应用本地部署的其他详细信息和注意事项，请参阅 [通用CRT部署](https://docs.microsoft.com/zh-cn/cpp/ide/universal-crt-deployment?view=vs-2017)。

因而开发者Phillip Hellewell在[UCRT官方团队介绍](https://blogs.msdn.microsoft.com/vcblog/2015/03/03/introducing-the-universal-crt/)博客下吐槽微软
> Just wanted to say, we’ve decided to go with app-local deployment, as we feel like we have no other choice.
>
> Essentially, by inventing the Universal CRT and refusing to create a merge module for it, Microsoft has punished us:
>
> * Punished developers by wasting our time figuring out a solution to ensure end users get it.
> * If we do nothing => Punish all end users with Windows 8.1 or older who aren’t up-to-date on Windows updates. (Our product won’t run.)
> * If we try to install VCRedist or the MSU => Punish some end users with Windows 8.1 or older who are having trouble with Windows updates (note: some people got stuck on 8.1 w/out Update); in scenarios like this, the update can fail or hang. We have seen it.
> * If we do app-local deployment => Punish end users with Windows 8.1 or older by making it so our product will not get any security updates to UCRT.
> * Thanks for punishing us Microsoft. We deserve it for not embracing Windows 10 like you wanted us to.

大意就是微软尼玛坑呀：
* 既然重构了CRT，也不向前兼容，非要通过Winows更新来支持旧系统
* 但尼玛Widows Update也是个坑呀，老系统用户有太多人关闭了系统更新（所以现在Win10进行强制更新，用户没得选咯）
* 那么用户通过补丁包更新好了吧，又可能会踩坑，补丁包在某些操作系统安装是会挂起，补丁包装不上了，程序跑不起来了
* 好吧，只能通过本地部署UCRT了，软件目录带上一大堆UCRT的动态库文件，Windows要是对UCRT修复了bug，也无法及时更新给客户，会带来安装问题。


## VC2015运行时库安装避坑

### 问题现象
目前最新的VC2015运行时库[Microsoft Visual C++ 2015 Redistributable Update 3 RC](https://www.microsoft.com/en-us/download/details.aspx?id=52685)中包含了[KB2999226](https://support.microsoft.com/en-us/help/2999226/update-for-universal-c-runtime-in-windows)，当检测到系统未不满足其UCRT依赖时会先进行补丁安装，然后安装VC运行时库。  
那么就会遇到在某些操作系统，安装补丁包挂起的现象，导致VC运行时库也无法安装上。为了解决这个问题，经过多次试验和排查，可以认定为Windows Update程序影响导致，具体现象为：
*补丁包MSU开始安装后，会长时间停留在checking for updates的状态，Windows Update一直在网络上搜索更新程序，而不进行此补丁包的安装。*

### 解决过程
通过Windows Update命令行重复安装卸载补丁来测试正常执行安装的情况，在命令行中直接输入`wusa.exe`可查看到命令行帮助文档
```batch
wusa.exe /uninstall /kb:2999226
wusa.exe C:\Users\Administrator\Desktop\Windows8.1-KB2999226-x64.msu /quiet /norestart
```
在测试过程中出现长时间安装程序不退出的情况，则表示安装在Checking for Updates的状态，通过命令行`net stop wuauserv`可直接停止Windows Update Service，来终止Windows Update安装程序。

### 解决方案
[SUCCESS]实际测试发现的解决方案只有通过**手动关闭Windows Update**才能正确安装上补丁包。  
[FAIL]想尝试使用修改注册表参数的方式避免用户手动修改设置的方案，均未正确安装，脚本记录如下：

 ```
reg add HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU /v NoAutoUpdate /t REG_DWORD /d 1

reg delete HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU /v NoAutoUpdate /t REG_DWORD 

reg add HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate /v DisableWindowsUpdateAccess /t REG_DWORD /d 0

reg delete HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate /v DisableWindowsUpdateAccess

 ```
[注册表wiki](https://gpsearch.azurewebsites.net/#2791)上课查询所有注册信息及其参数列表的意义


