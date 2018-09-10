---
layout: pageshow
category: tools
title: CapsLock和Esc按键互换方法
description: VIM输入模式下，ESC键位太远，通过键位互换解决
---


>方法参考于[博客](https://www.cnblogs.com/zhahw/p/5344290.html)  

**windows7**下CapsLock按键的使用率很低，与vi/vim编程时常用的<esc>按键互换可以很方便了vim党，具体互换方法为修改注册表信息：  
1. 找到注册表位置[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout],Win10下在HKEY_CURRENT_USER目录中
2. 新建一个二进制值的key，名为**Scancode Map**
3. 输以下值  
> 00, 00, 00, 00
> 
> 00, 00, 00, 00
> 
> 03, 00, 00, 00
> 
> 3A, 00, 01, 00
> 
> 01, 00, 3A, 00
> 
> 00, 00, 00, 00

 前两行和最后一行，都是固定的，全部为0。第三行，表示你修改了几个键，其实我们只是改了两个键，不过最后那一行也要算进去，所以是3。
4. 重启电脑生效
下面附上各个键位值的参考：

> Escape 01 00
>
> Tab 0F 00
>
>Caps Lock 3A 00
>
>Left Alt 38 00
>
>Left Ctrl 1D 00
>
>Left Shift 2A 00
>
>Left Windows 5B E0
>
>Right Alt 38 E0
>
>Right Ctr l1D E0
>
>Right Shift 36 00
>
>Right Windows 5C E0
>
>Backspace 0E 00
>
>Delete 53 E0
>
>Enter 1C 00
>
>Space 39 00
>
>Insert 52 E0
>
>HOME 47 E0
>
>End 4F E0
>
>Num Lock 45 00
>
>Page Down 51 E0
>
>Page Up 49 E0
>
>Scroll Lock 46 00

