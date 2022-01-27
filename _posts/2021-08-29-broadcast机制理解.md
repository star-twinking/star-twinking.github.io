---
layout: post
title: Numpy 中的 Braodcast 机制理解
categories: [python]
description: 大小不同的array在element-wise的运算上Braodcast 机制理解
keywords: broadcast, numpy

---

# Numpy 中的 Braodcast 机制理解

[TOC]

原文见
>https://numpy.org/doc/stable/user/basics.broadcasting.html

## 简介
简单来说，broadcast就是两个维度不同的输入数组在element-wise的运算基础上，运算后输出的数组会有传播(broadcast)机制。
有两点需要注意：
1. 需要是element-wise运算,具体什么是element-wise的运算，可以搜索。
2. 维度不同的两个数组，有一定的规则，不能是随便两个维度不同的数组。

下面讲解以下具体的规则

-----

## 四个原则
这里直接放原文：
>1. All input arrays with ndim smaller than the input array of largest ndim, have 1’s prepended to their shapes.
>2. The size in each dimension of the output shape is the maximum of all the input sizes in that dimension.
>3. An input can be used in the calculation if its size in a particular dimension either matches the output size in that dimension, or has value exactly 1.
>4. If an input has a dimension size of 1 in its shape, the first data entry in that dimension will be used for all 
>calculations along that dimension. In other words, the stepping machinery of the ufunc will simply not step along that dimension (the stride will be 0 for that dimension).

用我们看得懂的解释来说：
1. 所有输入的数组，会向维度最大的数组看齐，其他维度更小的数组，没有的维度用1补齐。就是说A（2，3，4，1）和B（3, 4, 5）相加，那么A是维度最大的数组，B是维度小的数组，
那么B就会向A看齐，第一个维度会变成(1, 3, 4, 5),然后和A相加。
2. 运算后的输出数组的大小（size）,是所有输入数组最大的维度值。这个如下图所示

![](/images/Daily-Learning/broadcast1.png "规则1，2解释")
3. 要满足broadcast的运算要求，输入的数组的维度要么是和另外的数组匹配，要么是在该维度上大小为1。比如上述的A和B相加，A的第四个维度为1，B看齐之后，第四维度为5，所以A可以和B相加
是因为在该维度上大小为1.对于第二和第三个维度，因为大小匹配/相等，所以也可以相加。
4. 维度为1的输入数组，该维度上的后续运算都用第一组值。

![](/images/Daily-Learning/broadcast2.gif "规则4的解释")

来看更为一般的broadcasting rules：

当操作两个array时，numpy会逐个比较它们的shape（构成的元组tuple），只有在下述情况下，两arrays才算符合broadcast的机制：

   1. 相等
   2. 其中一个为1，（进而可进行拷贝拓展已至，shape匹配）

对于输出的数组的shape，则是两个array的shape在每个维度上最大值的组合。

---
## 运用例子
```python
A      (4d array):  8 x 1 x 6 x 1
B      (3d array):      7 x 1 x 5
Result (4d array):  8 x 7 x 6 x 5

A      (2d array):  5 x 4
B      (1d array):      1
Result (2d array):  5 x 4

A      (2d array):  5 x 4
B      (1d array):      4
Result (2d array):  5 x 4

A      (3d array):  15 x 3 x 5
B      (3d array):  15 x 1 x 5
Result (3d array):  15 x 3 x 5

A      (3d array):  15 x 3 x 5
B      (2d array):       3 x 5
Result (3d array):  15 x 3 x 5

A      (3d array):  15 x 3 x 5
B      (2d array):       3 x 1
Result (3d array):  15 x 3 x 5
```