# CycleGAN

Clycle GAN是由两个判别器和两个生成器（G,F），为了避免所有的X都映射到同一个Y，论文采用了两个生成器的方式。

既能满足X->Y的映射，又能满足Y->X的映射，

> 是变分自编码器VAE的思想，为了适应不同图像产生不同的输出图像

![](../../pic/CycleGan1.png)



# DeepKeyGen

## Abstract

医学图像加密日益重要，为了保护患者的隐私。在这篇文章里，一个DeepKeyGen的生成器被提出去产生私有密钥，可以用来加密和解密医学图像。

## Introduction

