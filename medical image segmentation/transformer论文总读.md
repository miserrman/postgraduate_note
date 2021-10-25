# Transformer论文总读

##  Medical Transformer: Gated Axial-Attention for Medical Image Segmentation

### abstract

1. 卷积结构由于归纳偏置导致效果不佳，近年来大多使用Transformer中的注意力机制来解决问题
2. 现有的Transformer网络需要大量的数据作为基础，但是在医学图像中可以用来训练的数据比较少
3. 提出了**门控轴向注意力机制**，在自注意力机制中引入了一种额外的控制结构
4. 提出局部-全局训练方法

### Introduction

**长范围的依赖，对医疗图像很重要**

图片的背景是散开的，学到长距离依赖可以减少分割时候带来的错误

当分割的图案特别大的时候

![](D:\document\postgraduate\note\pic\md_trans1.PNG)

unet和res-unet使用的是卷积，只能看到局部范围内的东西，而transformer可以关注到长距离依赖，减少分割带来的错误，白白的那一块是背景，就被识别出来了



### Medical Transformer

如下图所示，目标位于图像的正中间位置，当我们将图像进行裁剪，使得目标位于图像的右侧时，卷积神经网络的检测点（原文为detecting saliency）从图像的正中间移动到了图像的右侧，作者因此做出了一个假设：卷积神经网络可以编码目标在图像中的绝对位置，即feature vector\feature map中含有目标的绝对位置信息。

不像卷积神经网络，自注意力结构不使用任何位置信息，位置信息在视觉模型中对于捕获一个东西相当重要

网络结构如下：

![](D:\document\postgraduate\note\pic\med_trans2.PNG)

#### 轴向注意力

自注意力机制被压缩成两个自注意力模块，第一个作用于width,第二个作用于height

轴向注意在高度轴和宽度轴上的应用有效地模拟了原始的自我注意机制，具有更好的计算效率。这些位置编码通常是可以通过训练学习的，并且已经被证明具有编码图像空间结构的能力。将轴向注意机制和位置编码结合起来，提出了一种基于注意的图像分割模型。

![](D:\document\postgraduate\note\pic\trans_md3.PNG)

![](D:\document\postgraduate\note\pic\trans_md.jpg)



r应该是位置编码

![](D:\document\postgraduate\note\pic\md_trans5.PNG)

特别的，如果相对位置编码学的准确的话，门控会分配一个比不准确的更高的权重



##  [MASK-RCNN](https://blog.csdn.net/u010901792/article/details/100044200)

后面看一下这个是什么意思



## Swin Transformer

### Abstract

我们提出了通过移动窗口计算的分层transformer



### Introduction

1. 不像NLP以单词为基本单位，CNN在图片的尺寸上有很大的不同(**一个研究方向**)
2. 有两种解决方法，第一种是统一图片的尺寸，另外一种是将图片中的每一个pixel看成nlp的一个单词
3. swin transformer运行在一个patch上，在深度的transformer中逐渐合并patch
4. swin transformer的关键设计是在自注意力层之间的滑动窗口，滑动窗口合并了其他窗口

每一个window变成一个维度

![](D:\document\postgraduate\note\pic\swin_trans1.PNG)

### Method

先将整个图片分割为一个一个不重叠的patch,每个patch相当于单词里面的一个token,作者使用的patch size是4*4

feature数量为4 * 4 * 3 = 48，通过一个线性层将它处理成任意的维度，我们把维度定义为C

为了产生一个分层表达，token的数量随着transformer网络的深度逐渐减少

第一个patch merging层将2*2的块连接起来，并且经过一个线性层，将token数量下降两倍，经过一个线性层，将维度变成原来的两倍2C，，token数量
$$
\frac{H}{4} * \frac{W}{4}块，块中的特征为48，经过一个线性层特征为c,到\frac{H}{8} * \frac{H}{8}
$$

![](D:\document\postgraduate\note\pic\swin_trans2.PNG)

#### Swin Transformer block

通过替换有多头注意力机制的块成为移动窗口

Swin Transformer Block = shift window based MSA(使用多头注意力机制的Transformer),后买你跟着两层，全连接神经网络



#### 在shifted window内的自注意力机制

在shifted window内计算patch的自注意力

但是在shift window内计算注意力限制了模型的能力，引入交叉窗口的连接会提高计算窗口的效率

![](D:\document\postgraduate\note\pic\swin_trans1.PNG)

shift window机制是通过一开始是一个大窗口，然后partition成小窗口，一开始是一个8*8的特征图，

将他partition成为2*2的窗口

每个窗口的大小为4*4，然后下一个模块采取和上一个模块移动的方法，将窗口变成

2*2

作者也许是通过 滑动窗口减小width和height数量的

![](D:\document\postgraduate\note\pic\swin_trans6.PNG)

> 每个窗口就是一个patch

移位窗口划分方法在前一层引入相邻非重叠窗口之间的连接，在图像分类、目标检测和语义分割等方面具有较好的效果

> 移动窗口的partion的解释，不停的partion窗口，上次连在一起的给他断开，没连在一起的连在一起



## Pyramid Vision Transformer: A Versatile Backbone for Dense Prediction without Convolutions

> 密集预测任务：指的是语义分割，实例分割都属于密集预测任务

### abstract

不像最近提出的Vision Transformer,我们引入了pyramid vision transformer(镜子他视觉Transformer),克服了transformer转移到密集预测任务上的问题，与目前的技术水平相比，PVT有几个优点：

1. vision transformer一般用于图像分类，这种少特征值的的输出，且占用的内存较大，PVT不仅可以在密集预测上，也使用了收缩金字塔结构减少特征图的数量
2. PVT继承了CNN和Transformer的特点，将它统一成了不同的视觉任务，它可以直接替换CNN成为backbone
3. 我们通过扩展实验验证了PVT,它提升了目标检测，目标分割

### introduction

![](D:\document\postgraduate\note\pic\pyraid_trans1.PNG)

我们的主要贡献如下：

* 提出了金字塔视觉Transformer结构，是一个为了像素级别预测的纯transformer结构，结合我们的PVT和DETR,我们可以建立一个端到端的系统
* 我们克服了transformer到密集预测任务的困难，设计了一个收缩金字塔，和空间收缩注意力机制，可以减少transformer的消耗



![](D:\document\postgraduate\note\pic\pymraid_trans.PNG)

### Method

#### PVT的总体结构

![](D:\document\postgraduate\note\pic\pyraid_trans3.PNG)



我们的目标是将金字塔结构引入transformer结构

在第一阶段，我们先将他分成HW/4**2的块，每个块的大小为4×4×3，并且将它放到嵌入层获得
$$
\frac{HW}{4^2} * C_1块的大小
$$
用相同的方法，我们获得了如下的特征向量

{F1,F2,F3,F4}

我们表示块的大小为P_i

**金字塔模型是如何减小块的数量的，通过这个公式**

![](D:\document\postgraduate\note\pic\pyraid_trans4.PNG)

reshape



## 自监督学习

![](D:\document\postgraduate\note\pic\self-supervise1.PNG)、

自监督学习，就是提取图片的一部分，去预测图片的另外一部分

（感觉对图像分割的意义不大）
