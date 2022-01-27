---
layout: post
title: 将numpy格式的图像转换为tensor几种方式比较
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2

---

# 将numpy格式的图像转换为tensor几种方式比较

##  1.ToTensor()

`torchvision.transforms`  on [`ToTensor()`](https://pytorch.org/docs/stable/_modules/torchvision/transforms/transforms.html#ToTensor)

需要注意的地方：

- numpy的RGB是和tensor以及PIL的不同，对于灰度图像没有什么关系。

> Converts a PIL Image or numpy.ndarray (H x W x C) in the range [0, 255] to a torch.FloatTensor of shape (C x H x W) in the range [0.0, 1.0] if the PIL Image belongs to one of the modes (L, LA, P, I, F, RGB, YCbCr, RGBA, CMYK, 1) or if the numpy.ndarray has dtype = np.uint8
>
> **In the other cases, tensors are returned without scaling.**

解决方法：

[Pytorch中Tensor与各种图像格式的相互转换、读取和展示](https://blog.csdn.net/weixin_40244676/article/details/117336356)

- 如果是二维数据，toTensor会增加数据的一个维度



## 2.torch.Tensor()

## 3.torch.from_numpy()

注意使用这个的时候，可以用value.copy().这样不会影响数据本身。

___

下面是一些例子，可以更加深入理解

```python
import numpy as np
import torch

arr = np.arange(12, dtype=np.uint8).reshape(3, 4)
trans = transforms.Compose([transforms.ToTensor()])
t1, t2, t3 = torch.Tensor(arr), torch.from_numpy(arr), trans(arr)
print(t1)
print(t2)
print(t3)

# 注意如果不是uint8数据类型，如果是float数据类型，还是会不变的
arr = np.arange(12, dtype=np.float32).reshape(3, 4)
trans = transforms.Compose([transforms.ToTensor()])
t1, t2, t3 = torch.Tensor(arr), torch.from_numpy(arr), trans(arr)
print(t1)
print(t2)
print(t3)
```

