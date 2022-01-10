# 3D  Transformer

## 3DUnet 

### Abstract

论文引入了一种三维分割的网络，我们提出了这种三维分割网络的两个用途：

1. 在一些自动分割场景中，用户要声明一些切片，网络可以从这些稀疏的片中学习，提供一个三维的密集分割
2. 在一个完全自动化的场景中，我们假设有一个稀疏的训练集存在，网络分出新的三维图片

### Introduction

三维数据在生物分析中比较常见，因为只有二维切片能被显示到电脑上。每一张进行切割的行为过于单调，效率较低，而且每一张相邻切片几乎都是相同的信息

### Network Architecture

![](D:\document\postgraduate\note\pic\3dunet1.PNG)

使用cov3d实现，没什么新意

## 3D Transformer

### method

![](D:\document\postgraduate\note\pic\3dtransformer1.PNG)

