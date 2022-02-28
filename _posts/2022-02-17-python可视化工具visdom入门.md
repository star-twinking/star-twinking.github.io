---
layout: post
title: python可视化工具visdom使用教程
categories: [python, pytorch]
description: 记录visdom的使用方法
keywords: visdom, 可视化工具
---

# python可视化工具visdom使用教程

[TOC]

## 概述

> Visdom:一个灵活的**可视化工具**，可用来对于 实时，富数据的创建，组织和共享。支持Torch和Numpy还有pytorch。
>
> visdom
> 可以实现远程数据的可视化，对科学实验有很大帮助。我们可以远程的发送图片和数据，并进行在ui界面显示出来，检查实验结果，或者debug.

## 几个基本概念：

Panes（窗格): interface刚打开是个白板，可以用将数据和图片发送到backend，生成多个对应的窗格，panes可以进行拖放，删除，panes保存在envs中，envs的状态存与绘画之间

![img](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/v2-dfd3e09af5ed0da3dba2368473653c86_1440w.jpg)

Environments(环境)：env对可视化空间进行分区，每个用户默认有个main的env, envs的状态是长期保持的，可以在代码中修改env的名字。

![img](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/v2-fc8e6dab0eb11b2a565766b84727cbfb_1440w.jpg)



## 安装和启动

要用这个先要安装，对于python模块而言，安装都是蛮简单的：

```
pip install visdom
```

安装完每次要用cmd命令行直接输入代码：

```
python -m visdom.server
```

![image-20220217224508788](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/image-20220217224508788.png)

然后根据提示在浏览器中输入相应地址即可，默认地址为：`http://localhost:8097/`

**注意使用时需要将这个命令端一直打开。**

## 使用示例

### 1. vis.text(), vis.image()

```
import visdom  # 添加visdom库
import numpy as np  # 添加numpy库
vis = visdom.Visdom(env='test')  # 设置环境窗口的名称,如果不设置名称就默认为main
vis.text('test: Hello Visdom!', win='main')  # 使用文本输出
vis.images(np.random.rand(3, 3, 64, 64), nrow=6, win='imgs', opts={'title': 'imgs'}) # 绘制3张图片随机的彩色图片

```

> 其中:
>
> visdom.Visdom(env=‘命名新环境')
> vis.text(‘文本', win=‘环境名')
> vis.image(‘图片'，win=‘环境名')

![image-20220217221312736](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/image-20220217221312736.png)

### 2. 画直线 .line() 一条

```python
import visdom
import numpy as np
vis = visdom.Visdom(env='my_windows')  # 设置环境窗口的名称,如果不设置名称就默认为main
x = list(range(10))
y = list(range(10))
# 使用line函数绘制直线 并选择显示坐标轴
vis.line(X=np.array(x), Y=np.array(y), opts=dict(showlegend=True))
```

> vis.line([x], [y], opts=dict(showlegend=True)[展示说明])



![画直线](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/1627529917200074.png)

### 两条

```
import visdom
import numpy as np
vis = visdom.Visdom(env='my_windows')
x = list(range(10))
y = list(range(10))
z = list(range(1,11))
vis.line(X=np.array(x), Y=np.column_stack((np.array(y), np.array(z))),  opts=dict(showlegend=True))
```

> vis.line([x], [y=np.column_stack((np.array(y),np.array(z),np.array(还可以增加)))])
> np.column_stack(a,b), 表示两个矩阵按列合并



![画两条直线](https://atts.w3cschool.cn/attachments/image/20210729/1627530027725594.png)

### sin(x)曲线

```
import visdom
import torch
vis = visdom.Visdom(env='sin')
x = torch.arange(0, 100, 0.1)
y = torch.sin(x)
vis.line(X=x,Y=y,win='sin(x)',opts=dict(showlegend=True))
```



![画曲线](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/1627530270222143.png)

### 持续更新图表

```python
import visdom
import numpy as np
vis = visdom.Visdom(env='my_windows')
# 利用update更新图像
x = 0
y = 0
my_win = vis.line(X=np.array([x]), Y=np.array([y]), opts=dict(title='Update'))
for i in range(10):
    x += 1
    y += i
    vis.line(X=np.array([x]), Y=np.array([y]), win=my_win, update='append')
```

**使用“append”追加数据，“replace”使用新数据，“remove”用于删除“name”中指定的跟踪。**



![更新数据](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/1627530347257017.png)

### vis.images()

```python
import visdom
import torch
# 新建一个连接客户端
# 指定env = 'test1'，默认是'main',注意在浏览器界面做环境的切换
vis = visdom.Visdom(env='test1')
# 绘制正弦函数
x = torch.arange(1, 100, 0.01)
y = torch.sin(x)
vis.line(X=x,Y=y, win='sinx',opts={'title':'y=sin(x)'})
# 绘制36张图片随机的彩色图片
vis.images(torch.randn(36,3,64,64).numpy(),nrow=6, win='imgs',opts={'title':'imgs'})
```



![绘制正弦图片](https://atts.w3cschool.cn/attachments/image/20210729/1627530439610059.png)



![绘制彩色图](https://atts.w3cschool.cn/attachments/image/20210729/1627530657333058.png)

### 绘制loss函数的变化趋势

```
#绘制loss变化趋势，参数一为Y轴的值，参数二为X轴的值，参数三为窗体名称，参数四为表格名称，参数五为更新选项，从第二个点开始可以更新
vis.line(Y=np.array([totalloss.item()]), X=np.array([traintime]),
                win=('train_loss'),
                opts=dict(title='train_loss'),
                update=None if traintime == 0 else 'append'
                )
```

## 实际代码

此代码出自CycleGAN的 utils.py 里一个实现

```
# 记录训练日志，显示生成图，画loss曲线 的类
class Logger():
    def __init__(self, n_epochs, batches_epoch):
        '''
        :param n_epochs:  跑多少个epochs
        :param batches_epoch:  一个epoch有几个batches
        '''
        self.viz = Visdom() # 默认env是main函数
        self.n_epochs = n_epochs
        self.batches_epoch = batches_epoch
        self.epoch = 1 # 当前epoch数
        self.batch = 1 # 当前batch数
        self.prev_time = time.time()
        self.mean_period = 0
        self.losses = {}
        self.loss_windows = {} # 保存loss图的字典集合
        self.image_windows = {} # 保存生成图的字典集合

    def log(self, losses=None, images=None):
        self.mean_period += (time.time() - self.prev_time)
        self.prev_time = time.time()

        sys.stdout.write('
Epoch %03d/%03d [%04d/%04d] -- ' % (self.epoch, self.n_epochs, self.batch, self.batches_epoch))

        for i, loss_name in enumerate(losses.keys()):
            if loss_name not in self.losses:
                self.losses[loss_name] = losses[loss_name].data.item() #这里losses[loss_name].data是个tensor（包在值外面的数据结构），要用item方法取值
            else:
                self.losses[loss_name] = losses[loss_name].data.item()

            if (i + 1) == len(losses.keys()):
                sys.stdout.write('%s: %.4f -- ' % (loss_name, self.losses[loss_name]/self.batch))
            else:
                sys.stdout.write('%s: %.4f | ' % (loss_name, self.losses[loss_name]/self.batch))

        batches_done = self.batches_epoch * (self.epoch - 1) + self.batch
        batches_left = self.batches_epoch * (self.n_epochs - self.epoch) + self.batches_epoch - self.batch
        sys.stdout.write('ETA: %s' % (datetime.timedelta(seconds=batches_left*self.mean_period/batches_done)))

        # 显示生成图
        for image_name, tensor in images.items(): # 字典.items()是以list形式返回键值对
            if image_name not in self.image_windows:
                self.image_windows[image_name] = self.viz.image(tensor2image(tensor.data), opts={'title':image_name})
            else:
                self.viz.image(tensor2image(tensor.data), win=self.image_windows[image_name], opts={'title':image_name})

        # End of each epoch
        if (self.batch % self.batches_epoch) == 0: # 一个epoch结束时
            # 绘制loss曲线图
            for loss_name, loss in self.losses.items():
                if loss_name not in self.loss_windows:
                    self.loss_windows[loss_name] = self.viz.line(X=np.array([self.epoch]), Y=np.array([loss/self.batch]),
                                                                 opts={'xlabel':'epochs', 'ylabel':loss_name, 'title':loss_name})
                else:
                    self.viz.line(X=np.array([self.epoch]), Y=np.array([loss/self.batch]), win=self.loss_windows[loss_name], update='append') #update='append'可以使loss图不断更新
                # 每个epoch重置一次loss
                self.losses[loss_name] = 0.0
            # 跑完一个epoch，更新一下下面参数
            self.epoch += 1
            self.batch = 1
            sys.stdout.write('
')
        else:
            self.batch += 1
```

train.py中调用代码是

```
# 绘画Loss图
logger = Logger(opt.n_epochs, len(dataloader))

for epoch in range(opt.epoch, opt.n_epochs):
    for i, batch in enumerate(dataloader):
    	
    	......
		
		# 记录训练日志
            # Progress report (http://localhost:8097) 显示visdom画图的网址
            logger.log({'loss_G': loss_G, 'loss_G_identity': (loss_identity_A + loss_identity_B),
                        'loss_G_GAN': (loss_GAN_A2B + loss_GAN_B2A),
                        'loss_G_cycle': (loss_cycle_ABA + loss_cycle_BAB), 'loss_D': (loss_D_A + loss_D_B)},
                       images={'real_A': real_A, 'real_B': real_B, 'fake_A': fake_A, 'fake_B': fake_B})
```



![运行结果](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/1627530746374566.png)

到此这篇常见的visdom使用方法的介绍就介绍到这了,更多python 可视化工具的了解和数据分析学习内容可以搜索[W3Cschool](https://www.w3cschool.cn/)以前的文章或继续浏览下面的相关文章。
