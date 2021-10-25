# 注意力机制

> attention机制可以它认为是一种资源分配的机制，可以理解为对于原本平均分配的资源根据attention对象的重要程度重新分配资源，重要的单位就多分一点，不重要或者不好的单位就少分一点，在深度神经网络的结构设计中，attention所要分配的资源基本上就是权重了
>
> 视觉注意力分为几种，核心思想是基于原有的数据找到其之间的关联性，然后突出其某些重要特征，有通道注意力，像素注意力，多阶注意力等，也有把NLP中的自注意力引入。

## TransUnet代码阅读

```python
class VisionTransformer(nn.Module):
    def __init__(self, config, img_size=224, num_classes=21843, zero_head=False, vis=False):
        super(VisionTransformer, self).__init__()
        self.num_classes = num_classes
        self.zero_head = zero_head
        self.classifier = config.classifier
        self.transformer = Transformer(config, img_size, vis)
        self.decoder = DecoderCup(config)
        self.segmentation_head = SegmentationHead(
            in_channels=config['decoder_channels'][-1],
            out_channels=config['n_classes'],
            kernel_size=3,
        )
        self.config = config

    def forward(self, x):
        if x.size()[1] == 1:
            x = x.repeat(1,3,1,1)# 对图层*3把它变成三维图层
        x, attn_weights, features = self.transformer(x)  # (B, n_patch, hidden)
        x = self.decoder(x, features)
        logits = self.segmentation_head(x)
        return logits
```

