---
layout: pageshow
category: python
title: python中求余操作符的不同之处
description: python中负数取余的异常情况
---

## python中%操作符同其他语言的不同点

python中`%`操作符号同其他C#或者java相同，都是取余运算，如  
python:  
```python
->> 12 % 10
2
```  
C#:  
```csharp
->> int i = 12 % 10
i = 2
```
但是，python中当对负数取余的时候，结果就不同了  
python:
```python
->> -12 % 10
8
->> -12 % 7
2
```
C#  
```csharp
->> int i = -12 % 10
i = -2
->> int j = -12 % 7
j = -5
```  
c#中`%`操作符还是如预料的一样，但是python中却返回了不同的值。其实，python的mod()实现对正数是相同的，但是对负数，返回操作相当于c#中： 
```csharp
int result = -12 % 7;
return result < 0 ？result + 7: result;
```
这种返回结果涉及python的C底层实现，有时候还是比较方便的，比如今天是周一，那3天前是周几时，可以：  
```python
->> (1 - 3) % 7
5
```
返回是周五。
