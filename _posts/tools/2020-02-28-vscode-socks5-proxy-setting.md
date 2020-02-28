---
layout: pageshow
category: tools
title: VSCode如何配置SOCKS5代理
description: VSCode在工作网络中无法实现插件更新和下载，所以需要配置代理的情况
tag: VSCode | socks5
---

## 编辑器页面无法配置
第一个直接的方法是直接通过VSCode的设置进行配置，但是页面提示需要满足http开头的正则才能正确配置上，这样的方式无法配置成功
![vscode-setting-ui](../../img/tools/vscode-socks5-1.jpg)
通过[VSCode github issue](https://github.com/Microsoft/vscode/issues/58080)得知VSCode从未支持过SOCK5方式的代理，跳转到[Network Connections in VSCode](https://code.visualstudio.com/docs/setup/network)后了解到VSCode使用了Chromium网络通信组件，因此可以尝试Chromium的代理设置

## 命令行参数成功配置SOCKS5
翻阅[Chromium network settings](https://www.chromium.org/developers/design-documents/network-stack/socks-proxy),Chromium通过启动是增加以下命令行参数实现SOCKS代理设置
```shell
--proxy-server="socks5://myproxy:8080"
--host-resolver-rules="MAP * ~NOTFOUND , EXCLUDE myproxy"
```
因此在本地VSCode快捷方式中目标参数增加以上命令项，即可实现VSCode的SOCKS5代理
```shell
C://****/Code.exe --proxy-server="socks5://myproxy:8080" --host-resolver-rules="MAP * ~NOTFOUND , EXCLUDE myproxy"
```