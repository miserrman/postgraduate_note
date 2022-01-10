# Swin Transformer代码阅读



## 窗口注意力机制

torch.meshgrid()

torch.meshgrid（）的功能是生成网格，可以用于生成坐标。函数输入两个数据类型相同的一维张量，两个输出张量的行数为第一个输入张量的元素个数，列数为第二个输入张量的元素个数，当两个输入张量数据类型不同或维度不是一维时会报错。

```python

# 【1】
import torch
a = torch.tensor([1, 2, 3, 4])
print(a)
b = torch.tensor([4, 5, 6])
print(b)
x, y = torch.meshgrid(a, b)
print(x)
print(y)
 
结果显示：
tensor([1, 2, 3, 4])
tensor([4, 5, 6])
tensor([[1, 1, 1],
        [2, 2, 2],
        [3, 3, 3],
        [4, 4, 4]])
tensor([[4, 5, 6],
        [4, 5, 6],
        [4, 5, 6],
        [4, 5, 6]])
 
 
 
# 【2】
import torch
a = torch.tensor([1, 2, 3, 4, 5, 6])
print(a)
b = torch.tensor([7, 8, 9, 10])
print(b)
x, y = torch.meshgrid(a, b)
print(x)
print(y)
 
结果显示：
tensor([1, 2, 3, 4, 5, 6])
tensor([ 7,  8,  9, 10])
tensor([[1, 1, 1, 1],
        [2, 2, 2, 2],
        [3, 3, 3, 3],
        [4, 4, 4, 4],
        [5, 5, 5, 5],
        [6, 6, 6, 6]])
tensor([[ 7,  8,  9, 10],
        [ 7,  8,  9, 10],
        [ 7,  8,  9, 10],
        [ 7,  8,  9, 10],
        [ 7,  8,  9, 10],
```



在shift_transformer源代码中，也用了mesh_grid生成网格坐标

```python
 # coords是坐标的意思
 coords_h = torch.arange(self.window_size[0])
 coords_w = torch.arange(self.window_size[1])
 coords = torch.stack(torch.meshgrid([coords_h, coords_w]))
```

这样生成的坐标，上面一层表示行，下面一层表示列

shift_window的forward函数

N应该是patch的数量，C是embedding后的维度

相对位置编码的计算

**@是python矩阵乘法的运算符**

```python
    def forward(self, x, mask=None):
        """ Forward function.

        Args:
            x: input features with shape of (num_windows*B, N, C)
            mask: (0/-inf) mask with shape of (num_windows, Wh*Ww, Wh*Ww) or None
        """
        B_, N, C = x.shape
        qkv = self.qkv(x).reshape(B_, N, 3, self.num_heads, C // self.num_heads).permute(2, 0, 3, 1, 4)
        q, k, v = qkv[0], qkv[1], qkv[2]  # make torchscript happy (cannot use tensor as tuple)

        q = q * self.scale
        attn = (q @ k.transpose(-2, -1))
```



```python
class WindowAttention(nn.Module):
    """ Window based multi-head self attention (W-MSA) module with relative position bias.
    It supports both of shifted and non-shifted window.

    Args:
        dim (int): Number of input channels.
        window_size (tuple[int]): The height and width of the window.
        num_heads (int): Number of attention heads.
        qkv_bias (bool, optional):  If True, add a learnable bias to query, key, value. Default: True
        qk_scale (float | None, optional): Override default qk scale of head_dim ** -0.5 if set
        attn_drop (float, optional): Dropout ratio of attention weight. Default: 0.0
        proj_drop (float, optional): Dropout ratio of output. Default: 0.0
    """

    def __init__(self, dim, window_size, num_heads, qkv_bias=True, qk_scale=None, attn_drop=0., proj_drop=0.):

        super().__init__()
        self.dim = dim
        self.window_size = window_size  # Wh, Ww
        self.num_heads = num_heads
        head_dim = dim // num_heads
        self.scale = qk_scale or head_dim ** -0.5

        # define a parameter table of relative position bias
        self.relative_position_bias_table = nn.Parameter(
            torch.zeros((2 * window_size[0] - 1) * (2 * window_size[1] - 1), num_heads))  # 2*Wh-1 * 2*Ww-1, nH

        # get pair-wise relative position index for each token inside the window
        coords_h = torch.arange(self.window_size[0])
        coords_w = torch.arange(self.window_size[1])
        # torch.stack()拼接函数，沿着一个新维度对输入张量序列进行连接。 序列中所有的张量都应该为相同形状。
        # torch.meshgrid（）的功能是生成网格，可以用于生成坐标。函数输入两个数据类型相同的一维张量，两个输出张量的行数为第一个输入张量的元素个数，列数为第二个输入张量的元素个数，当两个输入张量数据类型不同或维度不是一维时会报错。
        coords = torch.stack(torch.meshgrid([coords_h, coords_w]))  # 2, Wh, Ww
        coords_flatten = torch.flatten(coords, 1)  # 2, Wh*Ww
        # 计算相对位置编码
        relative_coords = coords_flatten[:, :, None] - coords_flatten[:, None, :]  # 2, Wh*Ww, Wh*Ww
        relative_coords = relative_coords.permute(1, 2, 0).contiguous()  # Wh*Ww, Wh*Ww, 2
        relative_coords[:, :, 0] += self.window_size[0] - 1  # shift to start from 0
        relative_coords[:, :, 1] += self.window_size[1] - 1
        relative_coords[:, :, 0] *= 2 * self.window_size[1] - 1
        relative_position_index = relative_coords.sum(-1)  # Wh*Ww, Wh*Ww
        self.register_buffer("relative_position_index", relative_position_index)

        self.qkv = nn.Linear(dim, dim * 3, bias=qkv_bias)
        self.attn_drop = nn.Dropout(attn_drop)
        self.proj = nn.Linear(dim, dim)
        self.proj_drop = nn.Dropout(proj_drop)

        trunc_normal_(self.relative_position_bias_table, std=.02)
        self.softmax = nn.Softmax(dim=-1)

    def forward(self, x, mask=None):
        """ Forward function.

        Args:
            x: input features with shape of (num_windows*B, N, C)
            mask: (0/-inf) mask with shape of (num_windows, Wh*Ww, Wh*Ww) or None
        """
        B_, N, C = x.shape
        qkv = self.qkv(x).reshape(B_, N, 3, self.num_heads, C // self.num_heads).permute(2, 0, 3, 1, 4)
        q, k, v = qkv[0], qkv[1], qkv[2]  # make torchscript happy (cannot use tensor as tuple)

        q = q * self.scale
        attn = (q @ k.transpose(-2, -1))

        relative_position_bias = self.relative_position_bias_table[self.relative_position_index.view(-1)].view(
            self.window_size[0] * self.window_size[1], self.window_size[0] * self.window_size[1], -1)  # Wh*Ww,Wh*Ww,nH
        relative_position_bias = relative_position_bias.permute(2, 0, 1).contiguous()  # nH, Wh*Ww, Wh*Ww
        attn = attn + relative_position_bias.unsqueeze(0)

        if mask is not None:
            nW = mask.shape[0]
            attn = attn.view(B_ // nW, nW, self.num_heads, N, N) + mask.unsqueeze(1).unsqueeze(0)
            attn = attn.view(-1, self.num_heads, N, N)
            attn = self.softmax(attn)
        else:
            attn = self.softmax(attn)

        attn = self.attn_drop(attn)

        x = (attn @ v).transpose(1, 2).reshape(B_, N, C)
        x = self.proj(x)
        x = self.proj_drop(x)
        return x
```



## SwinTransformer Block

```python
 # partition windows
 x_windows = window_partition(shifted_x, self.window_size)  # nW*B, window_size, window_size, C
 x_windows = x_windows.view(-1, self.window_size * self.window_size, C)  # nW*B, window_size*window_size, C

 # W-MSA/SW-MSA
 attn_windows = self.attn(x_windows, mask=attn_mask)  # nW*B, window_size*window_size, C

 # merge windows
 attn_windows = attn_windows.view(-1, self.window_size, self.window_size, C)
 shifted_x = window_reverse(attn_windows, self.window_size, Hp, Wp)  # B H' W' C
```

### partition window的代码

```
def window_partition(x, window_size):
    """
    一个是把window分片
    Args:
        x: (B, H, W, C)
        window_size (int): window size

    Returns:
        windows: (num_windows*B, window_size, window_size, C)
    """
    B, H, W, C = x.shape
    x = x.view(B, H // window_size, window_size, W // window_size, window_size, C)
    windows = x.permute(0, 1, 3, 2, 4, 5).contiguous().view(-1, window_size, window_size, C)
    return windows
```

### merge window的代码

```
def window_reverse(windows, window_size, H, W):
    """
    把window装回去
    Args:
        windows: (num_windows*B, window_size, window_size, C)
        window_size (int): Window size
        H (int): Height of image
        W (int): Width of image

    Returns:
        x: (B, H, W, C)
    """
    B = int(windows.shape[0] / (H * W / window_size / window_size))
    x = windows.view(B, H // window_size, W // window_size, window_size, window_size, -1)
    x = x.permute(0, 1, 3, 2, 4, 5).contiguous().view(B, H, W, -1)
    return x
```





```python
    def forward(self, x, mask_matrix):
        """ Forward function.

        Args:
            x: Input feature, tensor size (B, H*W, C).
            H, W: Spatial resolution of the input feature.
            mask_matrix: Attention mask for cyclic shift.
        """
        B, L, C = x.shape
        H, W = self.H, self.W
        assert L == H * W, "input feature has wrong size"

        shortcut = x
        x = self.norm1(x)
        x = x.view(B, H, W, C)

        # pad feature maps to multiples of window size
        pad_l = pad_t = 0
        pad_r = (self.window_size - W % self.window_size) % self.window_size
        pad_b = (self.window_size - H % self.window_size) % self.window_size
        x = F.pad(x, (0, 0, pad_l, pad_r, pad_t, pad_b))
        _, Hp, Wp, _ = x.shape

        # cyclic shift
        if self.shift_size > 0:
            shifted_x = torch.roll(x, shifts=(-self.shift_size, -self.shift_size), dims=(1, 2))
            attn_mask = mask_matrix
        else:
            shifted_x = x
            attn_mask = None

        # partition windows
        x_windows = window_partition(shifted_x, self.window_size)  # nW*B, window_size, window_size, C
        x_windows = x_windows.view(-1, self.window_size * self.window_size, C)  # nW*B, window_size*window_size, C

        # W-MSA/SW-MSA
        attn_windows = self.attn(x_windows, mask=attn_mask)  # nW*B, window_size*window_size, C

        # merge windows
        attn_windows = attn_windows.view(-1, self.window_size, self.window_size, C)
        shifted_x = window_reverse(attn_windows, self.window_size, Hp, Wp)  # B H' W' C

        # reverse cyclic shift
        if self.shift_size > 0:
            x = torch.roll(shifted_x, shifts=(self.shift_size, self.shift_size), dims=(1, 2))
        else:
            x = shifted_x 

        if pad_r > 0 or pad_b > 0:
            x = x[:, :H, :W, :].contiguous()

        x = x.view(B, H * W, C)

        # FFN
        x = shortcut + self.drop_path(x)
        x = x + self.drop_path(self.mlp(self.norm2(x)))

        return x
```



## Patch Merging

将窗口切成四份，cat起来以后再通过线性层减小

```
class PatchMerging(nn.Module):
    """ Patch Merging Layer

    Args:
        dim (int): Number of input channels.
        norm_layer (nn.Module, optional): Normalization layer.  Default: nn.LayerNorm
    """
    def __init__(self, dim, norm_layer=nn.LayerNorm):
        super().__init__()
        self.dim = dim
        self.reduction = nn.Linear(4 * dim, 2 * dim, bias=False)
        self.norm = norm_layer(4 * dim)

    def forward(self, x, H, W):
        """ Forward function.

        Args:
            x: Input feature, tensor size (B, H*W, C).
            H, W: Spatial resolution of the input feature.
        """
        B, L, C = x.shape
        assert L == H * W, "input feature has wrong size"

        x = x.view(B, H, W, C)

        # padding
        pad_input = (H % 2 == 1) or (W % 2 == 1)
        if pad_input:
            x = F.pad(x, (0, 0, 0, W % 2, 0, H % 2))

        x0 = x[:, 0::2, 0::2, :]  # B H/2 W/2 C
        x1 = x[:, 1::2, 0::2, :]  # B H/2 W/2 C
        x2 = x[:, 0::2, 1::2, :]  # B H/2 W/2 C
        x3 = x[:, 1::2, 1::2, :]  # B H/2 W/2 C
        x = torch.cat([x0, x1, x2, x3], -1)  # B H/2 W/2 4*C
        x = x.view(B, -1, 4 * C)  # B H/2*W/2 4*C

        x = self.norm(x)
        x = self.reduction(x)

        return x
```



win Transformer最重要的两点是hierarchical feature representation和SW-MSA（Shifted Window based Multi-head Self-attention）。

![img](https://pica.zhimg.com/50/v2-018744ccdff0523a9982ccc9c310783e_720w.jpg?source=1940ef5c)![img](https://pica.zhimg.com/80/v2-018744ccdff0523a9982ccc9c310783e_1440w.jpg?source=1940ef5c)Swin Transformer

### Hierarchical Feature Representation

Hierarchical feature representation的思路取自CNN结构，整个模型分为不同stage，每个stage对上一个stage输出的feature map进行降采样（H、W变小）；stage中每个block对局部进行建模而非全局。

![img](https://pic1.zhimg.com/50/v2-a86ab14a7a4cf8b471f3c969b118f9e7_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-a86ab14a7a4cf8b471f3c969b118f9e7_1440w.jpg?source=1940ef5c)Swin Transformer示意图

### 降采样

Swin Transformer中通过Patch Partition和Patch Merging来实现降采样，实际上二者是一个东西，也就是kernel size与stride相同的conv。Patch Partition的kernel size和stride为4，Patch Merging的kernel和stride为2，跟CNN中的降采样方法相同。

### 局部dependency

W-MSA（window based Multi-head Self-Attention）建模的是局部window的dependency，而不是MSA（Multi-head Self-Attention）中的全局，这有助于降低模型的计算复杂度，这个思路跟conv是一样的。

W-MSA跟conv类似，也有kernel size和stride两个参数，在同一个stage里，conv的kernel size一般大于stride，比如经典的kernel size为3，stride为1。但是W-MSA在同一个stage里的kernel size和stride相同，这导致了一个问题——feature map上相邻的window之间永远不会交互，也就是一个stage中有1个W-MSA和N个W-MSA的感受野是没区别的。

CNN中因为kernel size比stride大，随着conv的堆叠，感受野会逐步增加；MSA因为kernel size完全覆盖feature map，所以每个MSA都具备全局感受野。

为了缓解这个问题，作者给W-WSA打了个补丁，得到了SW-MSA（Shifted Window based Multi-head Self-attention）

### SW-MSA

![img](https://pic1.zhimg.com/50/v2-c55eedafdb686c377244a099954dcf6b_720w.jpg?source=1940ef5c)![img](https://pic1.zhimg.com/80/v2-c55eedafdb686c377244a099954dcf6b_1440w.jpg?source=1940ef5c)SW-MSA

Swin Transformer中连续的block会依次交替使用W-MSA和SW-MSA。SW-MSA相比W-MSA唯一不同的地方在于将window进行shift，这种思路跟[TSM](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1811.08383)异曲同工之妙。TSM想通过2D conv将时序信息encode进来，于是将各个frame的feature map在T维度进行shift；SW-MSA为了让相邻window之间产生交互，对window进行shift。

![img](https://pica.zhimg.com/50/v2-7f957f3ac8cf26f3972bfe6c1c31ca0b_720w.jpg?source=1940ef5c)![img](https://pica.zhimg.com/80/v2-7f957f3ac8cf26f3972bfe6c1c31ca0b_1440w.jpg?source=1940ef5c)TSM

### 一些感想

**关于Swin Transformer**

Swin Transformer使用CNN结构设计中的一些理念（降采样、局部dependency、TSM）来重新设计Transformer，和我之前在[如何看待Transformer在CV上的应用前景，未来有可能替代CNN吗？](https://www.zhihu.com/question/437495132/answer/1669853586)的回答不谋而合。

SW-MSA这种解决方法我感觉不够优雅，应该有其他替代方案，比如W-MSA中的kernel size大于stride，这样就类似于CNN，随着block的增加感受野可以逐渐变大。

**天下文章一大抄**

这里的抄不是抄袭的意思，而是说研究中的新思路大部分以前已经有了，面对一个问题，重新组合历史上已有的思路或者使用新的技术来实现历史上的思路，可能就会得到一个不错的结果。

一个领域的技术会不断发展，以计算机视觉为例，从局部特征子、CNN到Transformer，但是背后核心的思路是相对固定的。我们可以在CNN中看到局部特征子时代的印记，同样我们也会在Transformer应用在计算机视觉领域时，看到CNN的影子。

现在技术发展日新月异，只有抓住一个领域背后演进相对缓慢的思路，我们才可以以不变应万变，不然我们永远在追赶新技术，累得气喘吁吁。