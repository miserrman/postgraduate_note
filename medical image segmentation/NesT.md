# Aggregating Nested Transformers

## Abstract

即使层级结构在目前的视觉Transformer已经十分的流行，但是他们需要更精巧的设计和更大的数据量来取得更好的效果，在这篇工作中，我们探索了一种不合并图像块，并且以分层的方式聚合他们，我们发现block aggretion在跨块交流中充当着重要的作用



## Introduction

**缺乏局部性信息和平移等方差等归纳偏差是ViT模型数据效率低下的原因之一。**

在另一方面，全局的注意力计算又十分耗费计算资源

所以为了解决这个问题，目前的工作在于在局部计算注意力机制，这些方法在局部图像块上执行注意，而不是整体的全局自注意力机制。

我们移除了这些模块，并进行一个更简单的结构。



## Method



![](D:\document\postgraduate\note\pic\NesT.PNG)

1. 首先把图片进行patch分块
2. patch分块后维持很多block,每个block中有固定的patch数量
3. 每一次先在block中计算注意力机制
4. 计算后对block进行合并，合并以后用步长为2的最大池化层进行池化
5. 然后重新化为block, block的数量变为原来的1/4

![](D:\document\postgraduate\note\pic\NesT1.PNG)

不断的合并block达到这种程度
$$
X \in R^{b \times T_n \times n \times d}
$$

$$
T_n \times n = H \times W / S^2
$$

![](D:\document\postgraduate\note\pic\Nest2.PNG)

最后达到层级合并的效果

![](D:\document\postgraduate\note\pic\NesT3.PNG)

