---
layout: pageshow
title: ss-libev+obfs插件部署安装流程
category: linux
description: 记录ss+obfs插件部署流程及守护进程监控服务
---

## 开始前
[shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)是现在依然维护更新的ss项目，用于科学上网，但是在某些场景如学校、公司，对网络严格控制的情况下，直接ss无法进行访问，需要加上obfs的插件来伪装，具体策略后面有时间深入研究下，此处针对安装流程进行记录。

## 环境搭建
本次安装基于centos8，x86_64系统进行
### shadowsocks-libev服务端编译
安装[shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)，按照readme脚本示意，下载源码进行编译，注意点：libev中引用了其他项目，git clone时需要循环clone，使用命令`git submodule update --init --recursive`,编译intall后得到ss-server。  
启动ss采用配置文件的形式`ss-server -c <file_path>,配置文件示例：
```json
{
        "server": "0.0.0.0",
        "server_port": xxx,
        "password": "xxxxx",
        "method": "xxxx",
        "timeout": 600,
        "fast_open": false,
        "reuse_port": true,
        "no_delay": true,
}
```
注意点：服务器防火墙设置需要把ss端口开放出来：
```shell
$firewall-cmd --add-port=5060/tcp  --permanent
$firewall-cmd --add-port=5060/udp  --permanent
$firewall-cmd --reload
```
### simple-obfs服务端编译
按照[simple-obfs](https://github.com/shadowsocks/simple-obfs)教程clone源码进行编译，install后得到obfs-server,以插件形式启动ss,并在后台运行
```shell
$nohup ss-server -c /home/config.json --plugin obfs-server --plugin-opts "obfs=http" -f /tmp/ss-server.pid -u > /dev/null 2>&1 &
```

注解：

/dev/null ：代表空设备文件  

*>*  ：代表重定向到哪里，例如：echo “123” > /home/123.txt  

*1*  ：表示stdout标准输出 (the handle for standard output or STDOUT)，系统默认值是1，所以”>/dev/null”等同于”1>/dev/null”

*2*  ：表示stderr标准错误 (the handle for standard error or STDERR)

*&* ：m>&n 表示 文件描述符 m 重定向到文件描述符 n 所指向的文件。&n 也表明 n 是一个描述符（descriptor）而不是文件。

*> /dev/null 2>&1 &* 语句含义：

*> /dev/null* ： 首先标准输出（stdout) 重定向到空设备文件，也就是不输出任何信息到终端，说白了就是不显示任何信息。

*2>&1* ：接着，标准错误输出 （stderr） 重定向到标准输出的指向文件，因为之前标准输出已经重定向到了空设备文件，所以标准错误输出也重定向到空设备文件。

第二个&：在结尾加上“&”来将命令同时放入后台运行。
关于重定向可以参考[https://wiki.bash-hackers.org/howto/redirection_tutorial](https://wiki.bash-hackers.org/howto/redirection_tutorial)

### simple-obfs客户端
按照simple-obfs可以从源码进行编译成windows客户端，[simple-obfs-Cygwin](https://github.com/imgk/simple-obfs-Cygwin/releases)已有人编译了现成的客户端，直接拿来用即可(感谢！感谢！)。解压得到**obfs-local.exe**客户端，放在shadowsocks-windows桌面程序相同路径下，在服务器设置中增加
* 插件程序：obfs-local
* 插件选项：obfs=http;obfs-host=www.bing.com  

启动客户端，即可成功绕过限制，科学上网。

## 守护进程

以上服务端启动ss并启用了obfs插件，但是服务端程序会因为程序崩溃，机器故障异常，不可能每次收到以命令行启动ss，因此需要配置守护进程，守护ss程序正常运行。   
**systemctl**是当前最为使用的守护进程工具,因此采用systemctl工具将ss-server添加到severice。具体教程参考[廖老师的systemctl介绍](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)以及[官方文档](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)
### 增加systemctl配置文件
进入`/usr/lib/systemd/system`增加*shadowsocks.service*文件：
```
[Unit]
Description=Shadowsocks-libev
After=systemd-sysctl.service

[Service]
Type=forking
ExecStart=ss-server -c /home/config.json --plugin obfs-server --plugin-opts "obfs=http" -f /tmp/ss-server.pid -u > /dev/null 2>&1
Restart=always

[Install]
WantedBy=multi-user.target

```
**Restart**参数是让systemctl总是自动重启服务，除非服务是通过systemctl停止的  
使用命令`service shadowsocks start`启动服务，之后服务器或程序崩溃都会自动重启服务。  

注意：*ExecStart*如果要执行shell脚本时，需要将shell程序作为主程序，而不只是配置脚本路径，如`ExecStart=/usr/bin/bash /home/test.sh`

