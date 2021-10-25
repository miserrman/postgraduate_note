# unet学习

## 上采样的方法

当我们用神经网络生成图片的时候，经常需要将一些低分辨率的图片转换为高分辨率的图片。

> 图像分辨率指图像中存储的信息量，是每[英寸](https://baike.baidu.com/item/英寸/4733911)图像内有多少个像素点，分辨率的单位为PPI(Pixels Per Inch)，通常叫做像素每英寸。图像分辨率一般被用于ps中，用来改变图像的清晰度。

* 线性插值
* 转置卷积

## 线性插值
> 线性插值用的比较多的主要有三种：最近邻插值算法、双线性插值、双三次插值（BiCubic），当然还有各种其改进型。

### 1. 最近临近插值法
最近邻插值算法是最简单的一种插值算法，当图片放大时，缺少的像素通过直接使用与之最近原有颜色生成，也就是说照搬旁边的像素这样做结果产生了明显可见的锯齿。在待求像素的四邻像素中，将距离待求像素最近的邻灰度赋给待求像素。

![](D:\document\postgraduate\note\pic\1.png)

### 2. 双线性插值法

双线性插值就是做两次线性变换，先在X轴上做一次线性变换，求出每一行的R点：

![](D:\document\postgraduate\note\pic\2.png)

![](D:\document\postgraduate\note\pic\3.png)

可以把上式汇成所要计算的f（x,y）

![](D:\document\postgraduate\note\pic\4.png)



![](D:\document\postgraduate\note\pic\20191023132938572.png)

## 反卷积

转置卷积操作是一对多（也叫反卷积）现在，假设我们想要反过来操作。我们想要将输入矩阵中的一个值映射到输出矩阵的9个值，这将是一个一对多(one-to-many)的映射关系。这个就像是卷积操作的反操作，其核心观点就是用转置卷积。举个例子，我们对一个2 × 2 2 \times 22×2的矩阵进行上采样为4 × 4 4 \times 44×4的矩阵。这个操作将会维护一个1对应9的映射关系。


![](D:\document\postgraduate\note\pic\6.png)

我们可以将一个卷积操作用一个矩阵表示。这个表示很简单，无非就是将卷积核重新排列到我们可以用普通的矩阵乘法进行矩阵卷积操作。如下图就是原始的卷积核：

![](D:\document\postgraduate\note\pic\10.png)

我们对这个3 × 3 3 \times 33×3的卷积核进行重新排列，得到了下面这个4 * 16的卷积矩阵：
这个便是卷积矩阵了，这个矩阵的每一行都定义了一个卷积操作。下图将会更加直观地告诉你这个重排列是怎么进行的。每一个卷积矩阵的行都是通过重新排列卷积核的元素，并且添加0补充(zero padding)进行的。



![](D:\document\postgraduate\note\pic\7.png)

为了将卷积操作表示为卷积矩阵和输入矩阵的向量乘法，我们将输入矩阵4 × 4 4 \times 44×4摊平(flatten)为一个列向量，形状为16 × 1 16 \times 116×1，如下图所示。




![](D:\document\postgraduate\note\pic\8.png)

我们可以将这个4 × 16的卷积矩阵和1 × 16的输入列向量进行矩阵乘法，这样我们就得到了输出列向量。


![](D:\document\postgraduate\note\pic\9.png)



这个输出的4 × 1的矩阵可以重新塑性为一个2 × 2 的矩阵，而这个矩阵正是和我们一开始通过传统的卷积操作得到的一模一样。
所以各位发现了吗，关键点就在于这个卷积矩阵，你可以从16(4 × 4)到4(2 × 2)因为这个卷积矩阵尺寸正是4 × 16的，然后呢，如果你有一个16 × 4 的矩阵，你就可以从4到16了，这不就是一个上采样的操作吗？

啊哈哈哈哈哈哈哈哈



## skip connection

### 残差网络

随着网络层数的增加，网络发生了退化（degradation）的现象：随着网络层数的增多，训练集loss逐渐下降，然后趋于饱和，当你再增加网络深度的话，训练集loss反而会增大。注意这并不是过拟合，因为在过拟合中训练loss是一直减小的。

当网络退化时，浅层网络能够达到比深层网络更好的训练效果，这时如果我们把低层的特征传到高层，那么效果应该至少不比浅层的网络效果差。

* 主要方法：增加一条直接映射

#### 1. 残差块

差网络是由一系列残差块组成的（图1）。一个残差块可以用表示为：

![](D:\document\postgraduate\note\pic\11.PNG)

残差块分成两部分直接映射部分和残差部分。h(x)是直接映射（左边部分），F是残差部分（右边部分），一般由两个到三个卷积操作组成



![](D:\document\postgraduate\note\pic\12.png)

图1中的Weight在卷积网络中是指卷积操作，addition是指单位加操作。

在卷积网络中， ![[公式]](https://www.zhihu.com/equation?tex=x_l) 可能和 ![[公式]](https://www.zhihu.com/equation?tex=x_%7Bl%2B1%7D) 的Feature Map的数量不一样，这时候就需要使用 ![[公式]](https://www.zhihu.com/equation?tex=1%5Ctimes1) 卷积进行升维或者降维（图2)。这时，残差块表示为：

![](D:\document\postgraduate\note\pic\13.PNG)

#### 2. 残差网络背后的原理

根据BP中使用的导数的链式法则，损失函数 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvarepsilon) 关于 ![[公式]](https://www.zhihu.com/equation?tex=x_l) 的梯度可以表示为

![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_l%7D+%3D+%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_L%7D%5Cfrac%7B%5Cpartial+x_L%7D%7B%5Cpartial+x_l%7D+%3D+%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_L%7D%281%2B%5Cfrac%7B%5Cpartial+%7D%7B%5Cpartial+x_l%7D%5Csum_%7Bi%3Dl%7D%5E%7BL-1%7D%5Cmathcal%7BF%7D%28x_i%2C+%7BW_i%7D%29%29+%3D+%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_L%7D%2B%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_L%7D+%5Cfrac%7B%5Cpartial+%7D%7B%5Cpartial+x_l%7D%5Csum_%7Bi%3Dl%7D%5E%7BL-1%7D%5Cmathcal%7BF%7D%28x_i%2C+%7BW_i%7D%29+%5Ctag%7B7%7D)

上面公式反映了残差网络的两个属性：

1. 在整个训练过程中， ![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+%7D%7B%5Cpartial+x_l%7D%5Csum_%7Bi%3Dl%7D%5E%7BL-1%7D%5Cmathcal%7BF%7D%28x_i%2C+%7BW_i%7D%29+) 不可能一直为 ![[公式]](https://www.zhihu.com/equation?tex=-1) ，也就是说在残差网络中不会出现梯度消失的问题。
2. ![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial+%5Cvarepsilon%7D%7B%5Cpartial+x_L%7D) 表示 ![[公式]](https://www.zhihu.com/equation?tex=L) 层的梯度可以直接传递到任何一个比它浅的 ![[公式]](https://www.zhihu.com/equation?tex=l) 层。



#### 3. 直接映射是最好的选择

对于假设1，我们采用反证法，假设 ![[公式]](https://www.zhihu.com/equation?tex=h%28x_l%29+%3D+%5Clambda_l+x_l) ，那么这时候，残差块（图3.b）表示为

![[公式]](https://www.zhihu.com/equation?tex=x_%7Bl%2B1%7D+%3D+%5Clambda_lx_l+%2B+%5Cmathcal%7BF%7D%28x_l%2C+%7BW_l%7D%29%5Ctag%7B8%7D)

对于更深的L层

![[公式]](https://www.zhihu.com/equation?tex=x_%7BL%7D+%3D+%28%5Cprod_%7Bi%3Dl%7D%5E%7BL-1%7D%5Clambda_i%29x_l+%2B+%5Csum_%7Bi%3Dl%7D%5E%7BL-1%7D%5Cmathcal%7BF%7D%28x_i%2C+%7BW_i%7D%29%5Ctag%7B9%7D)

为了简化问题，我们只考虑公式的左半部分 ![[公式]](https://www.zhihu.com/equation?tex=x_%7BL%7D+%3D+%28%5Cprod_%7Bi%3Dl%7D%5E%7BL-1%7D%5Clambda_l%29x_l) ，损失函数 ![[公式]](https://www.zhihu.com/equation?tex=%5Cvarepsilon) 对 ![[公式]](https://www.zhihu.com/equation?tex=x_l) 求偏微分得

![[公式]](https://www.zhihu.com/equation?tex=%5Cfrac%7B%5Cpartial%5Cvarepsilon%7D%7B%5Cpartial+x_l%7D+%3D+%5Cfrac%7B%5Cpartial%5Cvarepsilon%7D%7B%5Cpartial+x_L%7D+%5Cleft%28+%28%5Cprod_%7Bi%3Dl%7D%5E%7BL-1%7D%5Clambda_i%29+%2B+%5Cfrac%7B%5Cpartial%7D%7B%5Cpartial+x_l%7D+%5Chat%7B%5Cmathcal%7BF%7D%7D%28x_i%2C+%5Cmathcal%7BW%7D_i%29%5Cright%29%5Ctag%7B10%7D+)



上面公式反映了两个属性：

1. 当 ![[公式]](https://www.zhihu.com/equation?tex=%5Clambda%3E1) 时，很有可能发生梯度爆炸；
2. 当 ![[公式]](https://www.zhihu.com/equation?tex=%5Clambda%3C1) 时，梯度变成0，会阻碍残差网络信息的反向传递，从而影响残差网络的训练。

所以 ![[公式]](https://www.zhihu.com/equation?tex=%5Clambda) 必须等1。同理，其他常见的激活函数都会产生和上面的例子类似的阻碍信息反向传播的问题。



#### 4. 激活函数适合移动到残差部分



## 代码理解

```python
class Up(nn.Module):
    """Upscaling then double conv"""

    def __init__(self, in_channels, out_channels, bilinear=True):
        super().__init__()
        # bilinear双线性插值
        # if bilinear, use the normal convolutions to reduce the number of channels
        if bilinear:
            self.up = nn.Upsample(scale_factor=2, mode='bilinear', align_corners=True)
            self.conv = DoubleConv(in_channels, out_channels, in_channels // 2)
        else:
            self.up = nn.ConvTranspose2d(in_channels, in_channels // 2, kernel_size=2, stride=2)
            self.conv = DoubleConv(in_channels, out_channels)

    def forward(self, x1, x2): #x2是下采样的时候的矩阵，要将上下两个采样的矩阵对齐
        x1 = self.up(x1) #啥意思
        # input is CHW
        diffY = x2.size()[2] - x1.size()[2]
        diffX = x2.size()[3] - x1.size()[3]

        x1 = F.pad(x1, [diffX // 2, diffX - diffX // 2,
                        diffY // 2, diffY - diffY // 2])   左边扩充一半，右边扩充左边剩下的一半
        x = torch.cat([x2, x1], dim=1)
        return self.conv(x)
```

> 在PyTorch里面，对于计算机视觉任务，如果图像数据是3维张量，那么张量维数的意义是 **通道数x高x宽**；
>
> 如果是4维张量，那么张量维数的意义是 **样本数x通道数x高x宽**。

F.pad是pytorch内置的tensor扩充函数，便于对数据集图像或中间层特征进行维度扩充，下面是pytorch官方给出的函数定义。(图片中是二维扩充)



[pad的详细理解](https://blog.csdn.net/binbinczsohu/article/details/106359426)



- input
  需要扩充的tensor，可以是图像数据，抑或是特征矩阵数据
- pad
  扩充维度，用于预先定义出某维度上的扩充参数
- mode
  扩充方法，’constant‘, ‘reflect’ or ‘replicate’三种模式，分别表示常量，反射，复制
- value
  扩充时指定补充值，但是value只在mode='constant’有效，即使用value填充在扩充出的新维度位置，而在’reflect’和’replicate’模式下，value不可赋值

![](D:\document\postgraduate\note\pic\扩充.png)



```python
def train_net(net,
              device,
              epochs: int = 5,
              batch_size: int = 1,
              learning_rate: float = 0.001,
              val_percent: float = 0.1,
              save_checkpoint: bool = True,
              img_scale: float = 0.5,
              amp: bool = False):
    # 1. Create dataset
    try:
        dataset = CarvanaDataset(dir_img, dir_mask, img_scale)
    except (AssertionError, RuntimeError):
        dataset = BasicDataset(dir_img, dir_mask, img_scale)

    # 2. Split into train / validation partitions
    n_val = int(len(dataset) * val_percent)
    n_train = len(dataset) - n_val
    train_set, val_set = random_split(dataset, [n_train, n_val], generator=torch.Generator().manual_seed(0))

    # 3. Create data loaders
    loader_args = dict(batch_size=batch_size, num_workers=4, pin_memory=True)
    train_loader = DataLoader(train_set, shuffle=True, **loader_args)
    val_loader = DataLoader(val_set, shuffle=False, drop_last=True, **loader_args)

    # (Initialize logging)
    experiment = wandb.init(project='U-Net', resume='allow', anonymous='must')
    experiment.config.update(dict(epochs=epochs, batch_size=batch_size, learning_rate=learning_rate,
                                  val_percent=val_percent, save_checkpoint=save_checkpoint, img_scale=img_scale,
                                  amp=amp))

    logging.info(f'''Starting training:
        Epochs:          {epochs}
        Batch size:      {batch_size}
        Learning rate:   {learning_rate}
        Training size:   {n_train}
        Validation size: {n_val}
        Checkpoints:     {save_checkpoint}
        Device:          {device.type}
        Images scaling:  {img_scale}
        Mixed Precision: {amp}
    ''')

    # 4. Set up the optimizer, the loss, the learning rate scheduler and the loss scaling for AMP
    optimizer = optim.RMSprop(net.parameters(), lr=learning_rate, weight_decay=1e-8, momentum=0.9)
    scheduler = optim.lr_scheduler.ReduceLROnPlateau(optimizer, 'max', patience=2)  # goal: maximize Dice score
    grad_scaler = torch.cuda.amp.GradScaler(enabled=amp)
    criterion = nn.CrossEntropyLoss()
    global_step = 0

    # 5. Begin training
    for epoch in range(epochs):
        net.train()
        epoch_loss = 0
        with tqdm(total=n_train, desc=f'Epoch {epoch + 1}/{epochs}', unit='img') as pbar:
            for batch in train_loader:
                images = batch['image']
                true_masks = batch['mask']

                assert images.shape[1] == net.n_channels, \
                    f'Network has been defined with {net.n_channels} input channels, ' \
                    f'but loaded images have {images.shape[1]} channels. Please check that ' \
                    'the images are loaded correctly.'

                images = images.to(device=device, dtype=torch.float32)
                true_masks = true_masks.to(device=device, dtype=torch.long)

                with torch.cuda.amp.autocast(enabled=amp):
                    masks_pred = net(images)
                    loss = criterion(masks_pred, true_masks) \
                           + dice_loss(F.softmax(masks_pred, dim=1).float(),
                                       F.one_hot(true_masks, net.n_classes).permute(0, 3, 1, 2).float(),
                                       multiclass=True)

                optimizer.zero_grad(set_to_none=True)
                grad_scaler.scale(loss).backward()
                grad_scaler.step(optimizer)
                grad_scaler.update()

                pbar.update(images.shape[0])
                global_step += 1
                epoch_loss += loss.item()
                experiment.log({
                    'train loss': loss.item(),
                    'step': global_step,
                    'epoch': epoch
                })
                pbar.set_postfix(**{'loss (batch)': loss.item()})

                # Evaluation round
                if global_step % (n_train // (10 * batch_size)) == 0:
                    histograms = {}
                    for tag, value in net.named_parameters():
                        tag = tag.replace('/', '.')
                        histograms['Weights/' + tag] = wandb.Histogram(value.data.cpu())
                        histograms['Gradients/' + tag] = wandb.Histogram(value.grad.data.cpu())

                    val_score = evaluate(net, val_loader, device)
                    scheduler.step(val_score)

                    logging.info('Validation Dice score: {}'.format(val_score))
                    experiment.log({
                        'learning rate': optimizer.param_groups[0]['lr'],
                        'validation Dice': val_score,
                        'images': wandb.Image(images[0].cpu()),
                        'masks': {
                            'true': wandb.Image(true_masks[0].float().cpu()),
                            'pred': wandb.Image(torch.softmax(masks_pred, dim=1)[0].float().cpu()),
                        },
                        'step': global_step,
                        'epoch': epoch,
                        **histograms
                    })

        if save_checkpoint:
            Path(dir_checkpoint).mkdir(parents=True, exist_ok=True)
            torch.save(net.state_dict(), str(dir_checkpoint / 'checkpoint_epoch{}.pth'.format(epoch + 1)))
            logging.info(f'Checkpoint {epoch + 1} saved!')
```

 前面已介绍，AMP其实就是Float32与Float16的混合，那为什么不单独使用Float32或Float16，而是两种类型混合呢？原因是：在某些情况下Float32有优势，而在另外一些情况下Float16有优势。这里先介绍下FP16：

优势有三个：

　１．减少显存占用；

　２．加快训练和推断的计算，能带来多一倍速的体验；

　３．张量核心的普及（NVIDIA　Tensor Core）,低精度计算是未来深度学习的一个重要趋势。
