# DenseNet

## densenet优点

1. 减轻了vanishing-gradient（梯度消失）
2. 加强了feature的传递
3. 更有效地利用了feature
4. 一定程度上较少了参数数量**

在深度学习网络中，随着网络深度的加深，梯度消失问题会愈加明显，目前很多论文都针对这个问题提出了解决方案，比如ResNet，Highway Networks，Stochastic depth，FractalNets等，尽管这些算法的网络结构有差别，但是核心都在于：create short paths from early layers to later layers。那么作者是怎么做呢？延续这个思路，那就是在保证网络中层与层之间最大程度的信息传输的前提下，直接将所有层连接起来！

## densenet的结构

![](D:\document\postgraduate\note\pic\dense1.PNG)

在传统的卷积神经网络中，如果你有L层，那么就会有L个连接，但是在DenseNet中，会有L(L+1)/2个连接。**简单讲，就是每一层的输入来自前面所有层的输出。**如下图：x0是input，H1的输入是x0（input），H2的输入是x0和x1（x1是H1的输出）……

假设输入为一个图片 ![[公式]](https://www.zhihu.com/equation?tex=X_%7B0%7D) , 经过一个L层的神经网络, 其中第i层的非线性变换记为 ![[公式]](https://www.zhihu.com/equation?tex=H_%7Bi%7D) (*), ![[公式]](https://www.zhihu.com/equation?tex=H_%7Bi%7D) (*)可以是多种函数操作的累加如BN、ReLU、Pooling或Conv等. 第i层的特征输出记作 ![[公式]](https://www.zhihu.com/equation?tex=X_%7Bi%7D) .

> ResNet

传统卷积前馈神经网络将第i层的输出 ![[公式]](https://www.zhihu.com/equation?tex=X_%7Bi%7D) 作为i+1层的输入,可以写作![[公式]](https://www.zhihu.com/equation?tex=X_%7Bi%7D) = ![[公式]](https://www.zhihu.com/equation?tex=H_%7Bi%7D) ( ![[公式]](https://www.zhihu.com/equation?tex=X_%7Bi-1%7D) ). ResNet增加了旁路连接,可以写作

![[公式]](https://www.zhihu.com/equation?tex=X_%7Bl%7D) = ![[公式]](https://www.zhihu.com/equation?tex=X_%7Bl%7D) ( ![[公式]](https://www.zhihu.com/equation?tex=X_%7Bl-1%7D) )+ ![[公式]](https://www.zhihu.com/equation?tex=X_%7Bl-1%7D)

ResNet的一个最主要的优势便是梯度可以流经恒等函数来到达靠前的层.但恒等映射和非线性变换输出的叠加方式是相加, 这在一定程度上破坏了网络中的信息流.

> Dense Connectivity