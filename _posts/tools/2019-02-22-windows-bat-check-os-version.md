---
layout: pageshow
category: tools
title: windows bat脚本判断当前操作系统的方法
description: windows bat 当前操作系统
---

# windows bat脚本判断当前操作系统的方法

## 目标

获取当前操作系统的版本和架构，如win7 x64、server2008 x64等

## 命令行工具

### WMIC

通过命令行工具WMIC可以获取当前系统的详细信息，但是需要使用管理员权限执行

1. 获取当前操作系统的版本：

```bat
>> wmic os get version
Version
6.1.7601
```

此版本号可通过查表获取对应的操作系统版本

| 客户端系统名称 | 版本     |
| -------------- | -------- |
| Windows Vista SP2| 6.0.6002|
| Windows 7      | 6.1.7600 |
| Windows 7 SP1 | 6.1.7601|
| Windows 8 RTM | 6.2.9200.16384|
| Windows 10 Gold| 10.0.10240|

|服务器系统名称|版本|
|--------------|----------|
| Server 2008 | 6.0.6001|
| Server 2008 R2 RTM| 6.1.7600|
| Server 2012|6.2.9200|
| Server 2012 R2| 6.3.9600|
| Server 2016 RTM 2016-09-26| 10.0.14393|
详细信息表格可以参阅[ver](https://ss64.com/nt/ver.html)

***

2. 获取系统架构（x86 or x64)
```bat
>>wmic os get osarchitecture /format:list
OSArchitecture=64-bit
```
可以通过`/format:list`格式化输入；这里可能在某些系统语言为中文的情况下，输出结果为**64 位**，当分割字符提取64的时候需要考虑加入空格的分割符。

3. 判断是客户端系统还是服务器端
通过version获取的信息可能无法分辨是win7SP1还是Server2008R2，通过构建版本号来区分不是很可靠，我就遇到Server2008R2获取的version为6.1.7601。  

```bat
rem check server or client system
wmic os get caption /format:list | find "Server"
if errorlevel 1 (
	set IS_SERVER=0
) else (
	set IS_SERVER=1
)
```

## 判断对应操作系统

通过以上获取的信息查表，可以通过判断得到当前的系统信息

```bat
rem get os version and architecture info （分隔符为=，-， ，共三个）
for /f "tokens=2 delims==- " %%a in ('wmic os get osarchitecture /format:list') do (
	set OS_ARCH=%%a
)
for /f "tokens=2,3,4 delims==." %%a in ('wmic os get version /format:list') do (
	set OS_MAJOR=%%a
	set OS_MINOR=%%b
	set OS_BUILD=%%c
)

if %OS_MAJOR% equ 6 (
	rem 6.0 winVista or server2008
	if %OS_MINOR% equ 0 (
		if %IS_SERVER% equ 1 (
			if %OS_ARCH% equ 64 (
				rem Server 2008 x64				
			) else (
				rem Server 2008 x32
			)
		) else (
			if %OS_ARCH% equ 64 (
				rem Win vista x64
			) else (
				rem Win vista x32
			)
		)
	)
	
	rem 6.2 win8 or server2012
	if %OS_MINOR% equ 2 (		
		if %IS_SERVER% equ 1 (
			if %OS_ARCH% equ 64 (
				rem Server 2012 x64
			) else (
				rem Server 2012 x32
			)
		) else (
			if %OS_ARCH% equ 64 (
				rem Win8 x64
			) else (
				rem Win8 x32
			)
		)		
	)
	
	rem 6.3 win8.1 or server2012R2
	if %OS_MINOR% equ 3 (		
		if %IS_SERVER% equ 1 (
			if %OS_ARCH% equ 64 (
				rem Server 2012R2 x64
			) else (
				rem Server 2012R2 x32
			)
		) else (
			if %OS_ARCH% equ 64 (
				rem Win8.1 x64
			) else (
				rem Win8.1 x32
			)
		)		
	)
	
	rem 6.1 win7 or server2008R2
	if %OS_MINOR% equ 1 (		
		if %IS_SERVER% equ 1 (
			if %OS_ARCH% equ 64 (
				rem Server 2008R2 x64
			) else (
				rem Server 2008R2 x32
			)
		) else (
			if %OS_ARCH% equ 64 (
				rem Win7sp1 x64
			) else (
				rem Win7sp1 x32
			)
		)		
	)	
)
```



