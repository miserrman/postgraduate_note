# 空洞卷积在图像语义分割的作用

## Abstract

为了处理多尺度分割对象的问题

## 解决问题

第一个挑战：连续池化和下采样，让高层特征具有局部图像变换的内在不变性，这允许DCNN学习越来越抽象的特征表示。但同时引起的**特征分辨率下降**，会妨碍密集的定位预测任务，因为这需要详细的空间信息。DeepLabv3系列解决这一问题的办法是使用空洞卷积(前两个版本会使用CRF细化分割结果)，这允许我们可以保持参数量和计算量的同时提升计算特征响应的分辨率，从而获得更多的上下文。

> 怎么使用空洞卷积解决这个问题

第二个挑战：多尺度目标的存在。现有多种处理多尺度目标的方法，我们主要考虑4种，如下图：
![image-20211013172054468](C:\Users\26583\AppData\Roaming\Typora\typora-user-images\image-20211013172054468.png)

多尺度目标检测方法



## DeepLab v1,v2

### 空洞卷积的应用

Input stride 也就是`空洞因子`或者`膨胀因子`，在`同样的卷积核大小`下，通过增加Input stride可以增大卷积核的`感受野`。更好的示意图：

![img](https://upload-images.jianshu.io/upload_images/4688102-0dde7c653b4526ac.gif?imageMogr2/auto-orient/strip|imageView2/2/w/395/format/webp)



可以发现感受野从`3`变成 了`5`，近似的扩大了`2`倍，卷积核大小仍为`3x3`，Input stride为`2`，现在都叫`dilate rate`。

**注意是近似扩大两倍**

重温VGG-16结构图：



![img](https://upload-images.jianshu.io/upload_images/4688102-5c5b402bd956300d.png?imageMogr2/auto-orient/strip|imageView2/2/w/470/format/webp)



作者为了加载预先在ImageNet训练好的VGG-16模型，并保证图片仅缩放了8倍做了如下修改

> 这里用到了一步预训练

般利用到条件随机场`（CRFs）`来处理分割中不光滑问题，它只考虑到目标像素点的附近点，是一个短距离的CRFs。由于网络中得到的结果已经比较光滑了，更希望的是修复一些小的结构，所以用到了`全连接`的CRF模型。它的能量函数：



## 主要贡献

本文重新讨论了空洞卷积的使用，这让我们在级联模块和空间金字塔池化的框架下，能够获取更大的感受野从而获取多尺度信息。

改进了ASPP模块：由不同采样率的空洞卷积和BN层组成，我们尝试以级联或并行的方式布局模块。

讨论了一个重要问题：使用大采样率的3 × 3 3×33×3的空洞卷积，因为图像边界响应无法捕捉远距离信息，会退化为1×1的卷积, 我们建议将图像级特征融合到ASPP模块中。

阐述了训练细节并分享了训练经验，论文提出的"DeepLabv3"改进了以前的工作，获得了很好的结果


## 方法

### 1. 空洞卷积

![](D:\document\postgraduate\note\pic\aton1.PNG)

空洞卷积公式，r代表rate，逐渐放大空洞卷积的rate

> output stride代表最后的结果和原图的尺寸缩小差异，最后的结果是32倍小，则out_strider=32

![](D:\document\postgraduate\note\pic\aton2.PNG)

unit rate和单位

rate = unit_rate * 2

output_strider = 16 multi_grid = (1,2,4)

rate = (2,4,8)

对于在DeepLabv2中提出的ASPP模块，其在特征顶部映射图并行使用了四种不同采样率的空洞卷积。这表明以不同尺度采样是有效的，我们在DeepLabv3中向ASPP中添加了BN层。不同采样率的空洞卷积可以有效的捕获多尺度信息，但是，我们发现随着采样率的增加，滤波器的有效权重(权重有效的应用在特征区域，而不是填充0)逐渐变小。

### 2. 金字塔池化空洞卷积

当我们不同采样率的3 × 3 3×33×3卷积核应用在65 × 65 65×6565×65的特征映射上，当采样率接近特征映射大小时，3 × 3 3×33×3的滤波器不是捕捉全图像的上下文，而是退化为简单的1 × 1 1×11×1滤波器，只有滤波器中心点的权重起了作用。

为了克服这个问题，我们考虑使用图片级特征。具体来说，我们在模型最后的特征映射上应用全局平均，将结果经过1 × 1 1×11×1的卷积，再双线性上采样得到所需的空间维度。最终，我们改进的ASPP包括：
![image-20211013195308388](C:\Users\26583\AppData\Roaming\Typora\typora-user-images\image-20211013195308388.png)

使用resnet做backbone



## 将金字塔结构和编码解码网络结合

```python
#without bn version
class ASPP(nn.Module):
    def __init__(self, in_channel=512, depth=256):
        super(ASPP,self).__init__()
        self.mean = nn.AdaptiveAvgPool2d((1, 1)) #(1,1)means ouput_dim
        self.conv = nn.Conv2d(in_channel, depth, 1, 1)
        self.atrous_block1 = nn.Conv2d(in_channel, depth, 1, 1)
        self.atrous_block6 = nn.Conv2d(in_channel, depth, 3, 1, padding=6, dilation=6)
        self.atrous_block12 = nn.Conv2d(in_channel, depth, 3, 1, padding=12, dilation=12)
        self.atrous_block18 = nn.Conv2d(in_channel, depth, 3, 1, padding=18, dilation=18)
        self.conv_1x1_output = nn.Conv2d(depth * 5, depth, 1, 1)
 
    def forward(self, x):
        size = x.shape[2:]
 
        image_features = self.mean(x)
        image_features = self.conv(image_features)
        image_features = F.upsample(image_features, size=size, mode='bilinear')
 
        atrous_block1 = self.atrous_block1(x)
        atrous_block6 = self.atrous_block6(x)
        atrous_block12 = self.atrous_block12(x)
        atrous_block18 = self.atrous_block18(x)
 
        net = self.conv_1x1_output(torch.cat([image_features, atrous_block1, atrous_block6,
                                              atrous_block12, atrous_block18], dim=1))
        return net

```

空洞卷积计算公式

这里引入了一个新的超参数 d，(d - 1) 的值则为塞入的空格数，假定原来的卷积核大小为 k，那么塞入了 (d - 1) 个空格后的卷积核大小 n 为：

![img](https://img-blog.csdnimg.cn/20190214094330187.png)

进而，假定输入空洞卷积的大小为 i，步长 为 s ,空洞卷积后特征图大小 o 的计算公式为：

![img](https://img-blog.csdnimg.cn/20190214094330195.png)

