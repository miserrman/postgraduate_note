# MICCAI 2021 Transformer

##  Convolution-Free Medical Image Segmentation using Transformers

### Abstract

探索使用全transformer结构，在这篇文章证明，全transformer结构基于子注意力机制可以达到更有效的结果，我们从一张3D的特征图中提取3D的patch,并将它计算为1D的patch,我们展示了提出的模型可以达到更好的准确率，并在三个数据集上达到了表现sota

### Introduction

CNN的成功可以归结为以下几点：

* 区域连接
* 参数权重共享
* 平移不变性

赐予了CNN很大的归纳偏置

做出的贡献有以下几点：

1. 我们第一个提出了全transformer分割网络
2. 我们提出了没有CNN的结构分割效率可以好于CNN,我们的网络可以在3D图像上进行有效训练
3. 我们提出了预训练方法，可以提高我们网络的分割精度

### Method

![](D:\document\postgraduate\note\pic\BraTS1.PNG)

#### A 提出的网络

> 就连全卷积神经网络也取消了？

上图展示了我们提出的网络，输入的网络是一个3D的block B,将block分成n*3的无重叠patch,我们将n选定为一个奇数

![](D:\document\postgraduate\note\pic\BraTS2.PNG)

![](D:\document\postgraduate\note\pic\BraTS3.PNG)

Y是预测Block中心的类别

怎么划分Block

### B预训练

手动分割是一项复杂艰巨的任务，要花专业的医疗专家几个小时，因此一个方法可以使用更少的数据达到更高的准确性是非常有益的，我们提出了在无标签数据上进行预训练来对数据进行去噪和修复

为了进行去噪预训练，我们在输入模块上加了SNR=10dB的高斯噪声，对于修复预训练我们把块的中心部分设置为0

在所有的块中，目标都是无噪声的中心块，我们使用一个不同的输出层，训练网络最小化loss2从预测到输出

> 预训练方法很有趣

### 实验

![](D:\document\postgraduate\note\pic\BraTS4.PNG)

* W：Block的大小
* K：decoder的层数
* n: 每一个block切分的patch数，取奇数

![](D:\document\postgraduate\note\pic\BraTS5.PNG)



## TransBTS: Multimodal Brain Tumor Segmentation Using Transformer

### Abstract

Transformer,可以从全局信息中获益，在自然语言处理和2D图像上都取得了一定的成功，然而局部和全局信息都对密集预测任务非常重要，尤其是3D医疗分割，在这篇文章里，我们探索Transformer在3D CNN上的应用

### Research Motivation

我们被启发了这样一个idea:

怎样设计一个网络，可以利用transformer在空间数据中高效的提取local和global特征

### Method

![](D:\document\postgraduate\note\pic\BraTS6.png)

给定一个MRI扫描特征图为C * H * W * D，通过transformer在全局特征编码长距离信息