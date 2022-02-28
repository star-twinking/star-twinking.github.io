---
layout: post
title: Torch中的transform作用于灰度图像的通道改变问题
categories: [Debug, Pyorch]
description: 读取png文件的时候遇到的，有时候存在有时候不存在。
keywords: debug, image imread
---

Content here

# Torch中的transform作用于灰度图像的维度改变问题

## 导入相应的包

```python
from PIL import Image
import torchvision.transforms as transforms
import cv2
```

## 用Image.open()读取灰度图像，是两维的


```python
image_path = r'C:\Users\kilin\Pictures\a1.png'
image = Image.open(image_path)
print('PIL format:', type(image), image.size)
```

    >>> PIL format: <class 'PIL.PngImagePlugin.PngImageFile'> (1120, 840)


## 用torch自带的transform导致灰度图像变为三个通道的图像


```python
transform_list = [transforms.ToTensor(), transforms.Normalize(mean=0.5, std=0.5)]
transform = transforms.Compose(transform_list)

image_t1 = transform(image)
print('pil1 after transform', image_t1.shape)
```

    >>> pil1 after transform torch.Size([3, 840, 1120])


## 即使将图像转换为灰度，也会出现一样的问题


```python
image_path = r'C:\Users\kilin\Pictures\Adrenal Gland.jpg'
image1 = Image.open(image_path).convert('L') # 转换为灰度图像
print('PIL format:', type(image1), image.size)
image_t2 = transform(image1)
print('pil2 after transform', image_t1.shape)
```

```
>>> PIL format: <class 'PIL.Image.Image'> (1120, 840)
pil2 after transform torch.Size([3, 840, 1120])
```


## 经过测试opencv读取不会出现这种问题

要注意Image.open()读取的和opencv读取的通道顺序需要改变


```python
image_cv = cv2.imread(image_path, cv2.IMREAD_GRAYSCALE)
print('cv2 format:', type(image_cv), image_cv.shape)

image_t2 = transform(image_cv)
print('cv2 after transform', image_t2.shape)
```

```
>>> cv2 format: <class 'numpy.ndarray'> (972, 1232)
cv2 after transform torch.Size([1, 972, 1232])
```
