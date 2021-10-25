#  Image Segmentation Transformer

> ​	我们建议沿着探索位置与其上下文之间的关系的路线来研究语义分割的方案。主要使用的是每个像素与其他类别的信息，然后在每个像素上使用softmax函数（计算要通过像素信息和相应的x信息），得到每个像素相应类别的权重

## 摘要

我们提出了目标上下文表示

1. 先学习目标区域基于ground truth的分割
2. 计算目标区域的每一个像素的组合
3. 计算每个像素和目标之间的关系，并且通过区域上下文增强每个像素的表现



## 主要贡献

提出了一种新的关系上下文方法，该方法根据粗分割结果学习像素与对象区域特征之间的关系来增强像素特征的描述。

特点：

1.与之前的关系上下文方法（如non-local、dual attention、ocnet等）不同的是对对象进行了区分，学习了像素-对象区域之间的关系

2.可以理解为一种对粗分割的后操作处理

## 主要步骤

1. 将上下文像素划分为一组软对象区域，每个软对象区域对应一个类别，即从深层网络计算出的粗分割。
2. 通过聚合相应对象区域中的像素表示作为每个对象区域的表示。
3. 使用对象上下文表示扩展每个像素的表示。 OCR是所有对象区域表示的加权聚合，其加权根据像素和对象区域之间的关系计算。

Backbone：ResNet  or HRNet

Pixel Representations：对backboe得到的特征图，先进行了上采样，然后3*3卷积改变了通道数，每个像素对应的特征为

Soft Object Regions：对backboe得到的特征图，先进行了上采样，然后进行语义分割得到粗分割结果（通道数为NUM_CLASS的特征图，每层表示该像素属于该类的概率）

Object Region Representations：每类对象的特征表示为

![](D:\document\postgraduate\note\pic\segtrans1.PNG)

Pixel-Region Relation：每个像素与每个对象区域之间的关系



![](D:\document\postgraduate\note\pic\segtrans2.PNG)

> 这个关系不是就用的是softmax函数吗

其中， ![是](D:\document\postgraduate\note\pic\segtrans4.PNG)关系函数（参考transformer) ，和是两个转换换函数，表示为1×1conv→BN→ReLU

Object Contextual Representations粉块：



其中，和是变换函数，同上

Augmented Representations:将对象上下文模块与初始特征图concat得到Object Contextual Representations，再进行通道数改变

![image-20211012001836709](C:\Users\26583\AppData\Roaming\Typora\typora-user-images\image-20211012001836709.png)

使用transformation函数

### transformation OCR方法

transformer首先是编码和解码结构

分心模型vs注意力机制

![](D:\document\postgraduate\note\pic\transeg.png)

z

