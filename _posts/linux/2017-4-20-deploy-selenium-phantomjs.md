---
layout: pageshow
category: linux
title: 部署selenuim + phantomjs自动化脚本
description: 将脚本部署到linux服务器涉及的操作和问题
---

## 内容概要

本地系统为*Ubuntu 16.04LTS*, 服务器系统为*Centos x86_64*  
* ssh登录远程linux系统  
* 安装依赖（phantomjs)
* 文件传输（将本地文件传输至服务器）  
* 运行脚本

### ssh登录远程linux系统

**Windows**下登录远程linux系统使用[PuTTY](http://www.putty.org/)  
**Linux**系统下使用ssh（Secure Shell)登录  
前提，需要事先知道linux服务器用户名的账号密码，如登录*root*用户，则需要知道root的密码。之前买了搬瓦工的vps翻墙，流量1000G/月，网速限制下怎么也用不完这么多，正好可以用来部署。  
首先，通过**SSH**登录linux服务器root用户, `-p`后为服务器提供的SSH端口号,
```$ ssh root@<server-IP> -p port```  
然后会要求输入root用户密码，正确输入后就能进入服务器shell

### 安装脚本依赖

因为脚本用到了[phantomJS](http://phantomjs.org/download.html),需要下载phantomJS驱动。PhantomJS其实就是没有GUI的浏览器，可以实现普通浏览器相同的功能。
通过官网找到*linux64*版本的链接下载地址（可以用过chrome调试模式，查看`<a href="url"`>),然后通过`wget`命令下载  
`$ wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2`  
等待下载完后，解压  
`$ tar -xvf $(find phantom*)`  
将`phantomjs`文件复制到`/usr/bin`目录  
```
$ cd phantomjs-2.1.1-linux-x86_64/bin
$ cp phantomjs /usr/bin
```
在shell中输入`phantomjs`查看是否能正确运行  
`$ phantomjs hello.js`  
我遇到了如下错误  

> error while loading shared libraries: libfontconfig.so.1: cannot open shared object file: No such file or directory  

通过安装`fontconfig`可以解决
`yum intall fontconfig`

### 文件传输

怎么将本地的脚本传输到linux服务器呢，可以通过**scp**，使用方式和**cp**差不多，都是`scp localfile <user>@<remote_server>:<path/to/save>`，但是这里需要注意的是要带上SSH端口号，否则
默认会用22端口进行连接，但是服务器各个服务器ssh端口不同，所以需注明具体端口。  
`$ scp -P <port> muji.py root@<server_ip>:~/Downloads`  
这里我选择将文件拷贝到`~/Dowmloads`文件目录中，端口命令注意为大写P  

### 运行脚本

准备将脚本运行在`virtualenv`环境中又遇到了问题，在python3.4中创建virtualenv时遇到了类似错误  
`Error: Command '['/some/directories/bin/python3.4', '-Im', 'ensurepip', '--upgrade', '--default-pip']' returned non-zero exit status 1`  
google到了[解决方案](https://askubuntu.com/questions/488529/pyvenv-3-4-error-returned-non-zero-exit-status-1)  

```shell
pyvenv-3.4 --without-pip myvenv
source ./myvenv/bin/activate
wget https://pypi.python.org/packages/source/s/setuptools/setuptools-3.4.4.tar.gz
tar -vzxf setuptools-3.4.4.tar.gz
cd setuptools-3.4.4
python setup.py install
cd ..
wget https://pypi.python.org/packages/source/p/pip/pip-1.5.6.tar.gz
tar -vzxf pip-1.5.6.tar.gz
cd pip-1.5.6
python setup.py install
cd ..
deactivate
source ./myvenv/bin/activate
```

创建virtualenv时不要包含**pip**包，然后手动从**pypi**上下载。






