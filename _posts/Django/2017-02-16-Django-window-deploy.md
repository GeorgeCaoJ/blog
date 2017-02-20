---
layout: pageshow
title: Windows下成功部署Django步骤
category: Django
description: Windows7下部署Django方案Apache2.4.23+mod_wsgi4.4.6+python 3.4.3
---

## 总结在前

经过了两天的google，才成功在**windows7**系统上部署成功**Django**,过程中遇到的问题和解决方法都一一记录了下来。总的体会是，确实麻烦而且困难，但是还是要感谢**google**。

## Django VS Flask

[Django](https://www.djangoproject.com/)是基于**python**语言的web框架，其功能完备，内置了数据库，管理系统等，更重要的是其社区活跃文档丰富，其在[stackoverflow](http://stackoverflow.com/questions/tagged/django)的相关问题截至今天(2017-02-16)有**136785**个。  

[Flask](http://flask.pocoo.org/)是另一个基于ptyhon的web框架，更轻量级，更自由，只有基本的功能，其他通过手动扩展同样强大，国内的[豆瓣](https://www.douban.com/)就是基于这一框架搭建的。其相关问题在[stackoverflow](http://stackoverflow.com/questions/tagged/flask)截至今天有**15199**个。  

而我选择先用django的理由，主要的看重文档丰富和社区更活跃，在使用过程中遇到其他人遇到的问题更容易得到解答，加上初学python的时候接着就学习了django一段时间。

## Django在Windows上的部署方案

根据[Django官方推荐](https://docs.djangoproject.com/en/1.10/howto/deployment/wsgi/modwsgi/)的方案：[Apache](https://httpd.apache.org/) + [mod_wsgi](https://modwsgi.readthedocs.io/)来部署。但是在Windows平台部署会增加许多麻烦，而且**mod_wsgi**无法开启[daemon mode](http://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html#delegation-to-daemon-process)。

## python是什么

这里需要明确一个概念，python是什么？  
python是一门编程语言，并不设计实现，它只定义怎么写代码，按照什么语法写；具体怎么跑程序，怎么实现代码的功能，就要靠编译器，将python代码编译成能在机器上跑的字节码。可以见[stackoverflow](http://stackoverflow.com/a/17131014/7051837)的解答。  
涉及到具体实现，就有多种实现方式，**Jpyhton**可以将python代码编译成能在JVM里运行的字节码，**IronPython**可以编译成Microsoft CLR（即**.NET**环境）上运行的字节码，而**CPython**则将python代码编译成C环境的字节码。一般的在windows平台，都是使用**CPython**来编译python代码，而**CPython**就需要用到微软的C++编译环境。  
微软的Visual C++编译器是和**Visual Studio**捆绑在一起的， VS2010对应的就是VC10环境，不同版本的python可能是用不同版本的**Visual Studio**编译。

## 部署步骤

1. 确认VC版本和方案  
2. 安装Apache2.4-VC10-x64  
3. 将mod_wsgi添加进apache  
4. 在python中安装mod_wsgi包  

### 1.确认VC版本和方案

mod_wsgi在Windows环境安装需要满足许多要求才能保证正常运行，具体可以看[mod_wsgi的作者在github上的说明](https://github.com/GrahamDumpleton/mod_wsgi/blob/develop/win32/README.rst)，具体来说就要满足4点：

1. apache、mod_wsgi、python的版本必须同时是64位或者32位

2. python版本是为所有用户安装的（这在安装python是一般是默认的）

3. apache服务器的预编译二进制文件必须和安装的python版本使用相同的Microsoft Visual C++编译器版本编译（等同于VC10编译的python要对应安装VC10编译的apache）

4. 使用经过相同版本VC编译器编译的mod_wsgi和python

[python官方说明](https://wiki.python.org/moin/WindowsCompilers)中，python和对应的VC版本是：

Visual C++ | CPython
--- | --- 
14.0 | 3.5, 3.6
10.0 | 3.3, 3.4
9.0 | 2.6, 2.7, 3.0, 3.1, 3.2

具体VC选择时，考虑到VC14的**mod_wsgi**需要手动编译，需要从github上下载了源码通过**Visual Studio 2014**编译，所以选择了编译好的VC10版本的[mod_wsgi.so](http://grapevine.dyndns-ip.com/download/folder.asp?eid=33&folder=%2Fdownload%2Fapache%2Fmod_wsgi-windows-4%2E4%2E6%2Fmod_wsgi-windows-4%2E4%2E6%2Fapache24-win64-vc10%2Fmodules%2F)。确定了VC版本且windows系统是64位后，方案就确定了
> **python3.4 x64** + **Apache2.4 VC10 x64** + **mod-wsgi-py34-VC10**

### 2.安装Apache2.4-VC10-x64

Apache Windows版本需要手动从源码编译，或者使用编译好的[各种发行版](https://httpd.apache.org/docs/2.4/platform/windows.html),我选择了[Apache Lounge distribution Apache 2.4 VC10](https://www.apachelounge.com/download/VC10/)下载。  
安装Apache步骤：  
1. 解压`httpd-2.4.23-win64.zip`  
2. 参照`ReadMe.txt`操作  
3. 将文件夹`Apache24`移动到C盘根目录  
4. 环境变量PATH中添加`C:\Apache24\bin`  
5. 打开Apache配置文件`C:\Apache24\conf\httpd.conf`并添加: ServerName localhost:80(部署方案是在本机运行，如果是生产环境则需要改为特定IP地址)  
6. 打开命令行，输入`httpd.exe -k install`将apache添加进**Service**  
7. 可以打开windows的**服务**,找到*Apache24*来打开apache;或者通过apache自带的**ApacheMonitor.exe**来打开，位于`C:\Apache24\bin`  

### 3. 将mod_wsgi添加进apache

下载[mod_wsgi.so](http://grapevine.dyndns-ip.com/download/folder.asp?eid=33&folder=%2Fdownload%2Fapache%2Fmod_wsgi-windows-4%2E4%2E6%2Fmod_wsgi-windows-4%2E4%2E6%2Fapache24-win64-vc10%2Fmodules%2F)并解压  
1. 将解压得到的`.so`文件改名为`mod_wsgo.so`  
2.复制文件到apache24目录下的`modules`文件夹内  
3. `httpd.conf`引入模块  

> LoadModule wsgi_module modules/mod_wsig.so

### 4.在python中安装mod_wsgi包

> pip install mod_wsgi  

安装mod_wsgi包，但是我安装时遇到错误:`error: Microsoft Visual C++ 10.0 is required (Unable to find vcvarsall.bat).`，可能是我电脑安装的是**Visual Studio 2015**的原因，默认不是用VC10编译环境。参照[stackoverflow上的解答](http://stackoverflow.com/a/32006750/7051837)，成功安装，具体步骤是：  
1. 打开**VS2015 命令提示行**，输入`set VS100COMNTOOLS=%VS140COMNTOOLS%`
2. 然后使用pip安装， `pip instal mod_wsgi`  

成功安装后，按照[github](https://github.com/GrahamDumpleton/mod_wsgi)上提示在命令行使用`mod_wsgi-express start-server`打开apache服务器，但是在windows上，是没有`start-server`这条命令的，只限于在**linux**平台。  
作者在[Issues](https://github.com/GrahamDumpleton/mod_wsgi/issues/189)中给出了解答

> On Windows mod_wsgi-express is only used to work out the configuration you need to add into your normal Apache instance. You then need to configure the normal Apache instance for your specific WSGI application and use normal method to start Apache. You can't use mod_wsgi-express start-server on Windows. 

因此在windows上还是使用常规方法启动apahce服务器。


### 5.创建Django项目来测试部署是否成功

按照[Django官方教程](https://docs.djangoproject.com/en/1.10/intro/tutorial01/)创建一个简单的项目`django-admin startproject mysite`，然后创建一个简单的app`python manage.py startapp polls`，在根据[官方的教程](https://docs.djangoproject.com/en/1.10/howto/deployment/wsgi/modwsgi/)在`httpd.conf中添加配置信息
```
WSGIScriptAlias / "C:/Apache24/djangoTest/mysite/mysite/wsgi.py"

WSGIPythonPath "C:/Apache24/djangoTest/mysite" 

WSGIPythonHome "C:/Python34"

<Directory "C:/Apache24/djangoTest/mysite/mysite">
<Files wsgi.py>
Require all granted
</Files>
</Directory>

```

`WSGIScriptAlias`的`/`为url的根地址，然后是使用双引号引用django项目的`wsgi.py`。  
`WSGIPythonPath`链接django的项目根目录  
`WSGIPythonHome`链接对应python根目录  

最后打开apache24服务，访问localhost是否成功，到此基本应该是成功的，出现打开失败，可以去`Apache24/logs/error.log`查看错误信息然后google解决

## 使用Virtualenv环境运行django app

django推荐使用`virtualenv`来创建web应用，好处就是不依赖外部python环境，减少其他模块对其的影响，而要配置在apache中沙箱模式又遇到了问题。  
首先创建一个virtualenv,在project的父文件运行命令行

```
python -m venv myvenv
```

然后在激活venv(windows和linux的激活方式稍有不同)

```
myvenv\Scripts\activate
```

然后安装django。  

在`httpd.conf`中将`WSGIPythonHome`替换为

```
WSGIPythonHome "C:/Apache24/djangoTest/myvenv"
```

此时可能会启动失败，查看`error.log`  
> [Fri Feb 17 18:29:03.378723 2017] [wsgi:warn] [pid 6216:tid 160] mod_wsgi: Compiled for Python/3.4.2.
[Fri Feb 17 18:29:03.378723 2017] [wsgi:warn] [pid 6216:tid 160] mod_wsgi: Runtime using Python/3.4.3.
Fatal Python error: Py_Initialize: unable to load the file system codec
ImportError: No module named 'encodings'
[Fri Feb 17 18:29:03.472323 2017] [mpm_winnt:crit] [pid 3480:tid 288] AH00419: master_main: create child process failed. Exiting.

参照[简书上有人写的解决方案](http://www.jianshu.com/p/a6e908ed199c)，在环境变量中新建一个变量`PYTHONPATH`，值设置为`C:\Python34\DLLs;C:\Python34\Lib;C:\Python34\Lib\site-packages;`，重新启动apache即可成功启动。

## 设置Apache访问静态文件

当访问`localhost/admin`页面时，发现只有纯文本的html，并没有django默认生成的**js + css + fonts + img**,查看页面确实也没有访问到这些静态文件。  
[Django官方](https://docs.djangoproject.com/en/1.10/howto/static-files/deployment/)给出的解决方案是不推荐开发模式下的静态文件访问方式使用在生产环境下。具体操作为：  
1. 在`Project/Project/settings.py`文件中添加`STATIC_ROOT = 'static'`(注意不要为`/static`, 否则会将静态文件夹移动到根目录，如C盘根目录)  
2. 在命令行中运行`python manage.py collectstatic`  

这样就将静态文件移动到了Project目录下，如`C:/Apache24/djangoTest/mysite/static`，然后需要将次静态目录添加到Apache的配置文件中，在`httpd.conf`中添加  
```
Alias /static "C:/Apache24/djangoTest/mysite/static"

<Directory "C:/Apache24/djangoTest/mysite/static">
Require all granted
</Directory>
```

值得注意的是[Django官方](https://docs.djangoproject.com/en/1.10/howto/deployment/wsgi/modwsgi/#serving-files)给的配置是`Alias /static/ "C:/Apache24/djangoTest/mysite/static"`,然而并没有起作用，要删除`/`后才成功访问。

## 总结在后

流水账的配置过程记完了，中间遇到了各种各样的问题，但通过强大的google和官方文档的支持，一步一步解决了，python web的部署果然还是有些麻烦。不得不感叹
> PHP是最好的语言



