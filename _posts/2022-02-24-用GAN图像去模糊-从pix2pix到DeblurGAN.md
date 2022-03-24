---
layout: post
title: Image Deblur using GAN: from pix2pix to DeblurGAN
categories: [Deep learning, Imaging process]
description: 记录一下
keywords: deblur, deep learning, GAN
---

目录

[TOC]



# 什么是图像去模糊

图像去模糊是为了将一张模糊的图片变为清晰的图片

![美工繪圖](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/1323434098-2957267295.png)

模糊有很多种类型，根据模糊的特点来分类的，均一的模糊（uniform blur）还有非均一性的模糊，比如空间变化的模糊。传统有很多方法，比如已知PSF的迭代去卷积，还有未知PSF的盲去卷积。

传统方法，比如Richard-lucy的迭代去卷积方法，维纳去卷积等方法，都是需要迭代，时间比较久，计算也比较耗时。

现在深度学习的方法开始应用在去模糊上面。大部分的方法可以归结为三种：

1. 非盲去模糊，已知模糊核和模糊图像，去卷积网络+去噪网络结合
2. 盲去模糊：
   - 2.1估计模糊核，然后用传统迭代方法来去模糊
   - 2.2 清晰图像和模糊图像之间的回归网络

本篇文章就是第三种，利用GAN这个模型来对模糊图片和清晰图片之间进行回归。神经网络里面GAN我就默认大家都已经知道，不知道的话可以看一下[GAN的文献](https://proceedings.neurips.cc/paper/2014/hash/5ca3e9b122f61f8f06494c97b1afccf3-Abstract.html)

# pix2pix

我们来介绍一下对图片于图片之间回归重要的网络模型cycleGAN.就是风格迁移的那个网络。他的整个project叫做pix2pix。

先展示一下pix2pix的效果：

![Input image translated into a corresponding output image by pix2pix](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/pix2pix-input-image-and-output-image-1024x390.jpg)

pix2pix可以将图像分割好的map变成原图，将灰度图变成彩色图上色；可以将卫星图变为地图一样的交通图；将图像的风格由白天变为黑夜；将图像的边缘变为原图。

还有通过这个方法将低分辨率的图像转变为高分辨率的图像。

## What is a Pix2Pix GAN?

### pix2pix的源头：Conditional GAN

Pix2Pix是原来Conditinal GAN (CGAN)启发的，CGAN是GAN的一个延申。

> 生成对抗网络GAN能够通过训练学习到数据分布，进而生成新的样本。可是GAN的缺点是生成的图像是随机的，不能控制生成图像属于何种类别。比如数据集包含飞机、汽车和房屋等类别，原始GAN并不能在测试阶段控制输出属于哪一类。

<img src="https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/1.PNG" style="zoom:38%;" />

CGAN的网络结构和GAN的没有太大的差别，就是D(Discriminator) 和 G(Generator)的输入都会带上label（y），所以损失函数变化为所有输入都是在y的条件下

GAN的损失函数：
$$
\min_{G} \max_{D} V(D,G) = \mathbb{E}_{x \sim p_{data}(x)}[log \, D(x)] \,+\, \mathbb{E}_{z \sim p_z(z)}[log(1-D(G(z)))]
$$
CGAN的损失函数：
$$
\min_{G} \max_{D} V(D,G) = \mathbb{E}_{x \sim p_data(x)}[log \, D(x|y)] \, +\, \mathbb{E}_{z\sim p_z(z)}[log(1-D(G(z|y)))]
$$
其他的没有太大的变化。

**下面讲讲pix2pix在CGAN的基础上做出的改变**

### 随机性上的改变

早期的GAN一般是输入噪声，而不是和真实图像y有对应的x。在pix2pix中，没有输入随机噪声来使得generator改变$G(x)$。作者发现使用Dropout来添加生成器的随机性。

> Past conditional GANs have acknowledged this and provided Gaussian noise z as an input to the generator, in addition to x (e.g., [55]). In initial experiments, we did not find this strategy effective – the generator simply learned to ignore the noise – which is consistent with Mathieu et al. [40]. Instead, for our final models, we provide noise only in the form of dropout, applied on several layers of our generator at both training and test time. Despite the dropout noise, we observe only minor stochasticity in the output of our nets. Designing conditional GANs that produce highly stochastic output, and thereby capture the full entropy of the conditional distributions they model, is an important question left open by the present work.

### pix2pix的generator

![Pix2Pix employs a UNET Generator, an encoder-decoder with skip connections between mirrored layers in the encoder and decoder stacks.](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/Pix2Pix-employs-a-UNET-Generator-an-encoder-decoder-1024x366.jpg)

CGAN里面用的是Encoder-decoder的网络类型，因为输入的是一个随机噪声。相当于是从噪声里面解码出图像。

但是因为pix2pix里面，输入的是一对图像。图像和图像之间不适用编码解码的网络类型，应该是自编码(Autoencoder)的结构。作者在这里用的是Unet,是一种兼容编码解码的结构，加上skip，可以自编码。【关于什么是自编码？请参考[Autoencoder](https://learnopencv.com/autoencoder-in-tensorflow-2-beginners-guide/), [什么是自编码 Autoencoder](https://zhuanlan.zhihu.com/p/24813602)】

### pix2pix的Discriminator

![image-20220219225738311](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/image-20220219225738311.png)

和前面提到的CGAN一样，判别器需要学习两个，一个是生成器生成的图像$G(x)$和原数据$x$的fake loss, 一个是真实图像$y$和输入数据$x$的real loss。实际上这是一个分类器，希望fake_loss全为0， real_loss全为1，如上图所示。

![Pix2Pix GAN Discriminator Showing similarity with GAN discriminator](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/Pix2Pix-Discriminator-working-as-patch-discriminator-1.jpg)

上图中，可以看到，输入的图像特征分别与生成的图像（generated image）和真实图像(real image)，串起来输入到判别器中。同时，**要注意到，判别器判别出来的结果不是简单的0或1.而是一个feature map一样，有一定大小的矩阵0或1，这种作者称作PatchGAN的方式**。如下图所示，

 ![Patch GAN Discriminator has Conditional discriminator and Patch](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/patchgan-matrix.jpg)

比如说，上述图中一张图取了一个感受野。然后经过判别器的网络之后，得到一个输出是4*4大小的分类块。这个块里面要么是0，要么是1.是0表示这个感受野的区域属于fake label，是1表示这个感受野的区域属于real label。

### 最后讲讲pix2pix的loss改变

#### Discriminator loss

![\begin{equation*}\text{Discriminator Loss} = \frac{( \text{BCE Loss for Real Images} + \text{BCE  Loss for Generated Images} )}{2}\end{equation*}](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/quicklatex.com-987265cc4202b9f182f80c8aac23a9ee_l3.png)

D loss就是输入$(x,y) \, \rightarrow Real\,label\,  $和$(x, G(x))$计算的loss加起来求平均。

#### Generator loss

![\begin{equation*}\text{L1-Loss} = \sum_{i=1}^{n}{|\text{Generated Output} - \text{Target Output}|}$\end{equation*}](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/quicklatex.com-c16671b733b0cae19dd37adc96c460c7_l3.png)

![\begin{equation*}$\text{Generator-Loss} = \text{BCE Loss with Real Labels} + {\lambda} \cdot \text{L1-Loss}$\end{equation*}](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/quicklatex.com-20e9ebc1b4ed0d28ea569f580481bf56_l3.png)

G loss一般是adversarial loss + content loss

Adversarial loss是输入$(x, G(x))\rightarrow Real\,label$到判别器的loss

Content loss 是生成图像和真实图像之间的差异，可以是L1, L2, SSIM之类的。

# DeblurGAN

DeblurGAN也是基于CGAN的架构，然后在上面进行了一些改变。

## DeblurGAN的Generator

![image-20220220155609447](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/image-20220220155609447.png)

generator和DCGAN的模型一样，包含两个1/2间隔的卷积单元(strided convolution blocks)、9个残差块(ResBlocks)和两个反卷积单元(Transposed convolution layer)。每个ResBlock由一个卷积层、实例归一化层和ReLU激活组成。每个残差快之后都有一个0.5的Dropout正则项，模型最后还加上了全局链接（global skip connection）。这样卷积神经网络可以学习到矫正残差$I_R$。 $I_S=I_R+I_B$。S,R,B分别表示清晰的图像，残差，模糊图像。

## DeblurGAN的Discriminator

Discriminator和CGAN的一样，用PatchGAN的方法，Deblur的判别器，细节上：

> All the convolution layers except the last are followed by InstanceNorm layer and LeakyReLU with $\alpha$ = 0:2.

作者在论文中用的判别器就是卷积层加上InstanceNorm和LeakyRelu层的模型。

## DeblurGAN的Loss

1. Discriminator loss：是生成的一个patch。里面要么是0，要么是1.

2. Generator loss, 重点介绍:

![image-20220220155141302](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/image-20220220155141302.png)

如上图所示包含了两部分，一部分是advesarial loss，是用WGAN-GP的loss， 还有一部分是content loss，用的是Perceptual loss。
$$
\mathcal L = \underbrace{\underbrace{\mathcal L_{GAN}}_{adv loss} \,+\, \underbrace{\lambda*\mathcal L_{X}}_{content loss}}_{total loss} 
$$

### Adversarial loss

CGAN一般用GAN原来的方法，还有least aquare GAN, 以及Wasserstein GAN+ gradient penalty（WGAN-GP）的方法。因为这种方法的有效性比较需要深入数学理论，这里就不解释了。简单来说就是损失函数做出改变，不需要log。因为原来的是KL散度组成的JS散度，修改成Wasserstein散度方便收敛。这样就可以防止原来GAN中，判别器训练得太好，在实验中生成器会完全学不动（loss降不下去）的情况。

原来的GAN, adversarial loss:

<img src="../../../../AppData/Roaming/Typora/typora-user-images/image-20220220174835950.png" alt="image-20220220174835950" style="zoom: 50%;" />

回到DeblurGAN，作者用的是WGAN loss的方法，这里的对抗loss就是用判别器对生成器产生的结果（fake image）进行判断，没有了原来GAN的部分，产生的图像更加平滑。定义如下:
$$
\mathcal L_{GAN} = \sum_{n=1}^N-D_{\theta _D}(G_{\theta_G}(I^B))
$$

所以在图里面，作者把loss写为Perpetual loss + WGAN loss。

### Content loss

一般用L1,L2等损失函数。但是本文采用perceptual loss的方法。

>Perceptual loss is a simple L2-loss, but based on the difference of the generated and target image CNN feature maps.

定义如下：
$$
\mathcal L_X = \frac{1}{{W_{i,j}H_{i,j}}} \sum_{x=1}^{W_{i,j}} \sum_{y=1}^{H_{i,j}}(\phi_{i,j}(I^S)_{x,y}-\phi_{i,j}(G_{\theta_G}(I^B)_{x,y}))^2
$$
$\phi_{i,j}$是第j层卷积层（after activation）,第i层最大池化（maxpooling）层之后的feature map。这个特征图是在ImageNet数据集预训练下的VGG19网络。

话有一个需要注意的点是，本篇文章没有用正则项。因为作者试了TV regularization效果不好。所以在实际上训练的时候是没有加上正则项的。

# 参考

1. [Pix2Pix:Image-to-Image Translation in PyTorch & TensorFlow](https://learnopencv.com/paired-image-to-image-translation-pix2pix/#what)

2. [令人拍案叫绝的Wasserstein GAN](https://zhuanlan.zhihu.com/p/25071913)

3. [更快更稳定：这就是Wasserstein GAN](https://www.jiqizhixin.com/articles/2018-11-20-6)
4. [Image-to-Image Translation with Conditional Adversarial Networks](https://arxiv.org/pdf/1611.07004.pdf)
5. https://arxiv.org/pdf/1704.00028.pdf
6. [DeblurGAN](http://arxiv.org/abs/1711.07064)

