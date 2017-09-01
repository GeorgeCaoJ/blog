---
layout: pageshow
title: 算法实践-全排列
category: Cpp
description: 全排列的的递归和循环实现，及存在重复元素时的全排列
---

## 全排列
全排列问题即例如把**ABC**可以排列成6种方式
```
ABC
ACB
BAC
BCA
CAB
CBA
```
数组长度为**n**时，共有**n!**种，当然这个是数组元素都不重复的情况。典型的**八皇后**问题就是用全排列来解决。   
首先讨论数组元素都不重复的情况，全排列的实现可以分为两种：递归和循环。递归的方法可见**剑指offer**，循环的方法可见**STL源码剖析**中对于STL函数`next_permutation`和`prev_permutation`实现方法。  
### 
