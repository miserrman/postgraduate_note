# transformer

## 经典的transformer模型架构图

![](D:\document\postgraduate\note\pic\tr1.PNG)

把transformer模型分为下面几个部分

![](D:\document\postgraduate\note\pic\tr2.jpg)

## transformer最核心的部分——注意力机制

![](D:\document\postgraduate\note\pic\tr3.jpg)

向量的内积，其几何意义是什么？

**答：表征两个向量的夹角，表征一个向量在另一个向量上的投影**

![img](https://pic2.zhimg.com/80/v2-f6973006b0ca2b67f452439698e6aacd_1440w.jpg)

**投影的值大，说明两个向量相关度高**。

**我们考虑，如果两个向量夹角是九十度，那么这两个向量线性无关，完全没有相关性！**

我们回想Softmax的公式，Softmax操作的意义是什么呢？



![img](https://pic4.zhimg.com/80/v2-cdf70f87d0d540475c21051cbdd8ac33_1440w.jpg)



**答：归一化**

我们结合上面图理解，Softmax之后，这些数字的和为1了。我们再想，Attention机制的核心是什么？

#### softmax参数的含义



![](D:\document\postgraduate\note\pic\tr5.PNG)



**加权求和**

那么权重从何而来呢？就是这些归一化之后的数字。当我们关注"早"这个字的时候，我们应当分配0.4的注意力给它本身，剩下0.4关注"上"，0.2关注"好"。当然具体到我们的Transformer，就是对应向量的运算了，这是后话。

在我们之前的例子中并没有出现`Q` `K` `V`的字眼，因为其并不是公式中最本质的内容。

`Q` `K` `V`究竟是什么？我们看下面的图

> 神经网络终究是矩阵运算，所以x不一定是严格的一维

![img](https://pic3.zhimg.com/80/v2-55d08f662a489739c3220486de095e12_1440w.jpg)

其实，许多文章中所谓的`Q` `K` `V`矩阵、查询向量之类的字眼，其来源是 ![[公式]](https://www.zhihu.com/equation?tex=X) 与矩阵的乘积，**本质上都是** ![[公式]](https://www.zhihu.com/equation?tex=X) **的线性变换**。

为什么不直接使用 ![[公式]](https://www.zhihu.com/equation?tex=X) 而要对其进行线性变换？

当然是为了提升模型的拟合能力，矩阵 ![[公式]](https://www.zhihu.com/equation?tex=W) 都是可以训练的，起到一个缓冲的效果。

如果你真正读懂了前文的内容，读懂了 ![[公式]](https://www.zhihu.com/equation?tex=Softmax%28XX%5ET%29) 这个矩阵的意义，相信你也理解了所谓查询向量一类字眼的含义。

#### d_k的意义

假设 ![[公式]](https://www.zhihu.com/equation?tex=Q%2CK) 里的元素的均值为0，方差为1，那么 ![[公式]](https://www.zhihu.com/equation?tex=A%5ET%3DQ%5ETK) 中元素的均值为0，方差为d. 当d变得很大时， ![[公式]](https://www.zhihu.com/equation?tex=A) 中的元素的方差也会变得很大，如果 ![[公式]](https://www.zhihu.com/equation?tex=A) 中的元素方差很大，那么 ![[公式]](https://www.zhihu.com/equation?tex=Softmax%28A%29) 的分布会趋于陡峭(分布的方差大，分布集中在绝对值大的区域)。总结一下就是 ![[公式]](https://www.zhihu.com/equation?tex=Softmax%28A%29) 的分布会和d有关。因此 ![[公式]](https://www.zhihu.com/equation?tex=A) 中每一个元素除以![[公式]](https://www.zhihu.com/equation?tex=%5Csqrt%7Bd_k%7D) 后，方差又变为1。这使得 ![[公式]](https://www.zhihu.com/equation?tex=Softmax%28A%29) 的分布“陡峭”程度与d解耦，从而使得训练过程中梯度值保持稳定。

```python
class Self_Attention(nn.Module):
    # input : batch_size * seq_len * input_dim
    # q : batch_size * input_dim * dim_k
    # k : batch_size * input_dim * dim_k
    # v : batch_size * input_dim * dim_v
    def __init__(self,input_dim,dim_k,dim_v):
        super(Self_Attention,self).__init__()
        self.q = nn.Linear(input_dim,dim_k)
        self.k = nn.Linear(input_dim,dim_k)#线性变换
        self.v = nn.Linear(input_dim,dim_v)
        self._norm_fact = 1 / sqrt(dim_k)
        
    
    def forward(self,x):
        Q = self.q(x) # Q: batch_size * seq_len * dim_k
        K = self.k(x) # K: batch_size * seq_len * dim_k
        V = self.v(x) # V: batch_size * seq_len * dim_v
         
        atten = nn.Softmax(dim=-1)(torch.bmm(Q,K.permute(0,2,1))) * self._norm_fact # Q * K.T() # batch_size * seq_len * seq_len
        
        output = torch.bmm(atten,V) # Q * K.T() * V # batch_size * seq_len * dim_v
        
        return output
```



#### 多头注意力机制

同样先上公式：
![[公式]](https://www.zhihu.com/equation?tex=MutiHead%28Q%2CK%2CV%29+%3D+Concat%28head_1%2Chead_2%2C...%2Chead_h%29W%5EO+%5C%5C+where%2C+head_i+%3D+Attention%28QW_i%5EQ%2CKW_i%5Ek%2CVW_i%5EV%29)

![[公式]](https://www.zhihu.com/equation?tex=W_i%5EQ%5Cin+R%5E%7Bd_%7Bmodel%7D+%5Ctimes+d_%7Bk%7D%7D%2CW_i%5Ek%5Cin+R%5E%7Bd_%7Bmodel%7D+%5Ctimes+d_%7Bk%7D%7D%2CW_i%5EV%5Cin+R%5E%7Bd_%7Bmodel%7D+%5Ctimes+d_%7Bv%7D%7D%2CW_O%5E%5Cin+R%5E%7Bhd_%7Bv%7D+%5Ctimes+d_%7Bmodel%7D%7D)

论文中采用8个头，因此 ![[公式]](https://www.zhihu.com/equation?tex=h%3D8) ![[公式]](https://www.zhihu.com/equation?tex=d_k%3Dd_v%3Dd_%7Bmodel%7D%2Fh%3D64)

如果single self-attention理解，那么这部分其实也很好理解：通过权重矩阵 ![[公式]](https://www.zhihu.com/equation?tex=W_i%5EQ%2CW_i%5Ek%2CW_i%5EV) 将![[公式]](https://www.zhihu.com/equation?tex=Q%2CK%2CV) 分割，每个头分别计算 single self-attention，因为权重矩阵 ![[公式]](https://www.zhihu.com/equation?tex=W_i%5EQ%2CW_i%5Ek%2CW_i%5EV) 各不相同， ![[公式]](https://www.zhihu.com/equation?tex=QW_i%5EQ%2CKW_i%5Ek%2CVW_i%5EV) 的结果各不相同，因此我们说每个头的关注点各有侧重。最后，将每个头计算出的 single self-attention进行concat，通过总的权重矩阵W O W^O*WO*决定对每个头的关注程度，从而能够做到在不同语境下对相同句子进行不同理解。

一句话总结：Attention是将query和key映射到同一高维空间中去计算相似度，而对应的multi-head attention把query和key映射到高维空间 ![[公式]](https://www.zhihu.com/equation?tex=%5Calpha) 的不同子空间 ![[公式]](https://www.zhihu.com/equation?tex=%28%5Calpha_1%2C%5Calpha_2%2C...%2C%5Calpha_h%29) 中去计算相似度。

```python
# Muti-head Attention 机制的实现
from math import sqrt
import torch
import torch.nn


class Self_Attention_Muti_Head(nn.Module):
    # input : batch_size * seq_len * input_dim
    # q : batch_size * input_dim * dim_k
    # k : batch_size * input_dim * dim_k
    # v : batch_size * input_dim * dim_v
    def __init__(self,input_dim,dim_k,dim_v,nums_head):
        super(Self_Attention_Muti_Head,self).__init__()
        assert dim_k % nums_head == 0
        assert dim_v % nums_head == 0
        self.q = nn.Linear(input_dim,dim_k)
        self.k = nn.Linear(input_dim,dim_k)
        self.v = nn.Linear(input_dim,dim_v)
        
        self.nums_head = nums_head
        self.dim_k = dim_k
        self.dim_v = dim_v
        self._norm_fact = 1 / sqrt(dim_k)
        
    
    def forward(self,x):
        Q = self.q(x).reshape(-1,x.shape[0],x.shape[1],self.dim_k // self.nums_head) 
        K = self.k(x).reshape(-1,x.shape[0],x.shape[1],self.dim_k // self.nums_head) 
        V = self.v(x).reshape(-1,x.shape[0],x.shape[1],self.dim_v // self.nums_head)
        print(x.shape)
        print(Q.size())

        atten = nn.Softmax(dim=-1)(torch.matmul(Q,K.permute(0,1,3,2))) # Q * K.T() # batch_size * seq_len * seq_len
        
        output = torch.matmul(atten,V).reshape(x.shape[0],x.shape[1],-1) # Q * K.T() * V # batch_size * seq_len * dim_v
        
        return output
```

#### mask机制

attention机制里涉及两种mask,padding mask和sequence mask

##### padding mask

输入的序列要进行对齐，所以有大量的填充0的运算，所以要截取左边的内容，其他的直接舍弃。

##### sequence mask


![img](https://pic1.zhimg.com/80/v2-4d82d55512d23ca63a42ac5b85b8c494_1440w.jpg)



文章前面也提到，sequence mask 是为了使得 decoder 不能看见未来的信息。也就是对于一个序列，在 time_step 为 t 的时刻，我们的解码输出应该只能依赖于 t 时刻之前的输出，而不能依赖 t 之后的输出。因此我们需要想一个办法，把 t 之后的信息给隐藏起来。



## embedding层

Embedding部分接受原始的文本输入(batch_size*seq_len,例:[[1,3,10,5],[3,4,5],[5,3,1,1]])，叠加一个普通的Embedding层以及一个Positional Embedding层，输出最后结果。

![img](https://pic3.zhimg.com/80/v2-50fc5d6c2ba6afa585de2ae5e176b53e_1440w.jpg)

在这一层中，输入的是一个list: [batch_size * seq_len]，输出的是一个tensor:[batch_size * seq_len * d_model]

需要关注论文里的mask机制，在文本输入之前需要统一长度

```python
class Embedding(nn.Module):
    def __init__(self,vocab_size):
        super(Embedding, self).__init__()
        # 一个普通的 embedding层，我们可以通过设置padding_idx=config.PAD 来实现论文中的 padding_mask
        self.embedding = nn.Embedding(vocab_size,config.d_model,padding_idx=config.PAD)


    def forward(self,x):
        # 根据每个句子的长度，进行padding，短补长截
        for i in range(len(x)):
            if len(x[i]) < config.padding_size:
                x[i].extend([config.UNK] * (config.padding_size - len(x[i]))) # 注意 UNK是你词表中用来表示oov的token索引，这里进行了简化，直接假设为6
            else:
                x[i] = x[i][:config.padding_size]
        x = self.embedding(torch.tensor(x)) # batch_size * seq_len * d_model
        return x
```

