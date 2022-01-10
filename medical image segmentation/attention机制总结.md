# Attention机制总结

## External Attention

[外部注意力机制paper](https://arxiv.org/abs/2105.02358)

使用两个线性层和两个归一化层

> 首先分析自注意力机制的缺点：O（n**2）的时间复杂度，使得自注意力机制的运算在patch上进行，而不是在像素级别进行。selg attention可以看作使用线性层和self层去精炼输入特征，自注意力机制只考虑了在同一个数据样本内不同元素的关系，忽视了不同样本之间潜在的关系。

我们提出了一种新的注意力机制，通过输入特征向量和额外的记忆单元

![](D:\document\postgraduate\note\pic\attention1.png)

M是一个可学习的参数，作为一个训练数据集的记忆，A是一个Attention Map从数据集级别推出来的先验知识

![](D:\document\postgraduate\note\pic\attention2.png)

![](D:\document\postgraduate\note\pic\attention3.png)



## Squeeze-and-Excitation Attention 

![](https://raw.githubusercontent.com/xmu-xiaoma666/External-Attention-pytorch/master/model/img/SE.png)

有点像通道注意力机制



## SK Attention

在视神经科学领域在神经科学领域，视觉皮层神经元的感受野大小受到刺激的调节是众所周知的，但在神经元网络的构建中很少考虑到这一点。我们提出了一种在CNN的动态选择机制，使得每个神经元自动调整感受野



![](D:\document\postgraduate\note\pic\attention5.png)

1. Split使用kernel=3和kernel=5的卷积核分别卷积
2. Fuse:将两个特征图加在一起，并用全局平均池化层得出S
3. 使用两个可学习的向量a,b作为软注意力机制对两个不同尺度的特征图进行选择，a + b = 1

计算公式一览：

![image-20211213221215728](C:\Users\26583\AppData\Roaming\Typora\typora-user-images\image-20211213221215728.png)

![](D:\document\postgraduate\note\pic\attention6.png)



## ECA Attention

[ECA论文链接](https://arxiv.org/pdf/1910.03151.pdf)

##  PARALLEL MULTI-SCALE ATTENTION

[论文链接](https://arxiv.org/pdf/1911.09483.pdf)

1. 即使自注意力机制可以建模长距离的依赖，在深的层中注意力会过分关注单一token,导致局部信息被不充分使用，
2. 和表示长的序列的困难，
3. 在这篇工作中，我们探索了并行的多尺度序列表示学习，力图去捕获长距离和短距离的结构

![](D:\document\postgraduate\note\pic\attention7.png)

MUSE使用了编码器解码器的框架，编码器采用word embedding作为输入，最重要的部分是MUSE部分，采用了三个部分

* 自注意力捕获全局信息
* 深度可分离卷积捕获局部信息
* 用于捕获token特性的前馈神经网络

![](D:\document\postgraduate\note\pic\attention8.png)

### 深度可分离卷积

- 逐通道卷积

Depthwise Convolution的一个卷积核负责一个通道，一个通道只被一个卷积核卷积

一张5×5像素、三通道彩色输入图片（shape为5×5×3），Depthwise Convolution首先经过第一次卷积运算，DW完全是在[二维平面](https://www.zhihu.com/search?q=二维平面&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A92134485})内进行。卷积核的数量与上一层的通道数相同（通道和卷积核一一对应）。所以一个三通道的图像经过运算后生成了3个Feature map(如果有same padding则尺寸与输入层相同为5×5)，如下图所示。

![img](https://pic4.zhimg.com/80/v2-a20824492e3e8778a959ca3731dfeea3_1440w.jpg)

其中一个Filter只包含一个大小为3×3的Kernel，卷积部分的参数个数计算如下：

N_depthwise = 3 × 3 × 3 = 27

Depthwise Convolution完成后的Feature map数量与输入层的通道数相同，无法扩展[Feature map](https://www.zhihu.com/search?q=Feature+map&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A92134485})。而且这种运算对输入层的每个通道独立进行卷积运算，没有有效的利用不同通道在相同空间位置上的feature信息。因此需要Pointwise Convolution来将这些Feature map进行组合生成新的Feature map

- 逐点卷积

Pointwise Convolution的运算与常规卷积运算非常相似，它的卷积核的尺寸为 1×1×M，M为上一层的通道数。所以这里的卷积运算会将上一步的map在深度方向上进行[加权组合](https://www.zhihu.com/search?q=加权组合&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A92134485})，生成新的Feature map。有几个卷积核就有几个输出Feature map

![img](https://pic4.zhimg.com/80/v2-2cdae9b3ad2f1d07e2c738331dac6d8b_1440w.jpg)

由于采用的是1×1卷积的方式，此步中卷积涉及到的参数个数可以计算为：

N_pointwise = 1 × 1 × 3 × 4 = 12

经过Pointwise Convolution之后，同样输出了4张Feature map，与常规卷积的输出维度相同

```python
class Depth_Pointwise_Conv1d(nn.Module):
    def __init__(self,in_ch,out_ch,k):
        super().__init__()
        if(k==1):
            self.depth_conv=nn.Identity()
        else:
            self.depth_conv=nn.Conv1d(
                in_channels=in_ch,
                out_channels=in_ch,
                kernel_size=k,
                groups=in_ch,
                padding=k//2
                )
        self.pointwise_conv=nn.Conv1d(
            in_channels=in_ch,
            out_channels=out_ch,
            kernel_size=1,
            groups=1
        )
    def forward(self,x):
        out=self.pointwise_conv(self.depth_conv(x))
        return out
    


class MUSEAttention(nn.Module):

    def __init__(self, d_model, d_k, d_v, h,dropout=.1):


        super(MUSEAttention, self).__init__()
        self.fc_q = nn.Linear(d_model, h * d_k)
        self.fc_k = nn.Linear(d_model, h * d_k)
        self.fc_v = nn.Linear(d_model, h * d_v)
        self.fc_o = nn.Linear(h * d_v, d_model)
        self.dropout=nn.Dropout(dropout)

        self.conv1=Depth_Pointwise_Conv1d(h * d_v, d_model,1)
        self.conv3=Depth_Pointwise_Conv1d(h * d_v, d_model,3)
        self.conv5=Depth_Pointwise_Conv1d(h * d_v, d_model,5)
        self.dy_paras=nn.Parameter(torch.ones(3))
        self.softmax=nn.Softmax(-1)

        self.d_model = d_model
        self.d_k = d_k
        self.d_v = d_v
        self.h = h


    def forward(self, queries, keys, values, attention_mask=None, attention_weights=None):

        #Self Attention
        b_s, nq = queries.shape[:2]
        nk = keys.shape[1]

        q = self.fc_q(queries).view(b_s, nq, self.h, self.d_k).permute(0, 2, 1, 3)  # (b_s, h, nq, d_k)
        k = self.fc_k(keys).view(b_s, nk, self.h, self.d_k).permute(0, 2, 3, 1)  # (b_s, h, d_k, nk)
        v = self.fc_v(values).view(b_s, nk, self.h, self.d_v).permute(0, 2, 1, 3)  # (b_s, h, nk, d_v)

        att = torch.matmul(q, k) / np.sqrt(self.d_k)  # (b_s, h, nq, nk)
        if attention_weights is not None:
            att = att * attention_weights
        if attention_mask is not None:
            att = att.masked_fill(attention_mask, -np.inf)
        att = torch.softmax(att, -1)
        att=self.dropout(att)

        out = torch.matmul(att, v).permute(0, 2, 1, 3).contiguous().view(b_s, nq, self.h * self.d_v)  # (b_s, nq, h*d_v)
        out = self.fc_o(out)  # (b_s, nq, d_model)

        v2=v.permute(0,1,3,2).contiguous().view(b_s,-1,nk) #bs,dim,n
        self.dy_paras=nn.Parameter(self.softmax(self.dy_paras))
        out2=self.dy_paras[0]*self.conv1(v2)+self.dy_paras[1]*self.conv3(v2)+self.dy_paras[2]*self.conv5(v2)
        out2=out2.permute(0,2,1) #bs.n.dim

        out=out+out2
        return out

```

