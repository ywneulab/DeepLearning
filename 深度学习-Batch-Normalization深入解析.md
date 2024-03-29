---
title: Batch-Normalization深入解析
sitemap: true
categories: 深度学习
date: 2018-10-27 12:59:27
tags:
- 深度学习
---


InceptionV2 在 V1 的基础上引入了 BN 层, 减少了 Internal Covariate Shift(内部神经元的数据分布差异), 使得每一层的输出都可以规范化到一个标准高斯分布上.

<span id = "为什么要进行归一化">
# 为什么要进行归一化
在神经网络中, 数据分布对训练会产生影响. 当数据较大时, 有的激活函数比如 sigmoid 或者 tanh 的输出就会接近于 1, 此时的梯度就会接近于0, 处于激活函数的饱和阶段, 也就是说无论输入再怎么扩大, sigmoid 或者 tanh 激励函数输出值也还是接近1. 换句话说, 神经网络在初始阶段已经不对那些比较大的 x 特征范围敏感了. 因此我们需要用之前提到的对数据做 normalization 预处理, 使得输入的 x 变化范围不会太大, 让输入值经过激励函数的敏感部分. 需要注意的是, 这个不敏感问题不仅仅发生在神经网络的输入层, 而且在隐藏层中也经常会发生.
当没有进行normalizatin时,数据的分布是任意的,那么就会有大量的数据处在激活函数的敏感区域外, 对这样的数据分布进行激活后, 大部分的值都会变成1或-1, 造成激活后的数据分布不均衡, 而如果进行了Normallizatin,  那么相对来说数据的分布比较均衡.
**一句话总结就是: 通过Normalization让数据的分布始终处在激活函数敏感的区域**


<span id = "简述 BN 的原理">
# 简述 BN 的原理

在训练深度神经网络时, 每个隐藏层的参数变化会使得后一层的输入发生变化, 从而每一批训练数据的分布也随之改变, 导致网络在每次迭代中都需要拟合不同的数据分布, 增大了训练的复杂度以及过拟合的风险. 这使得我们需要非常谨慎的去设定学习率, 初始化权重等参数更新策略. 因此, 为了保证网络层中的每一批数据都处于相同的数据分布下, 我们需要在网络的每一层输入之前增加归一化处理, 具体来说, 就是对当前 batch 中的所有元素减去均值再除以标准差. 这样的归一化操作可以对原始数据添加额外的约束, 从而可以增强模型的泛化能力, 但同时, 由于简单归一化之后的数据分布被强制为 0 均值和 1 标准差, 因此可能会破坏原始数据本身的特征. 为了能够还原原始数据分布, BN 的第二个关键操作就是引入了用于变换重构的线性偏移参数 $\gamma$ 和 $\beta$, 它们分别对简单归一化后的数据执行 scale 和 shift 操作, 可以在一定程度还原数据本身的分布特别. 总体来说, BN 的作用可以简单概括为两步, 第一步是进行归一化, 用于统一不同网络层的数据分布; 第二步变换重构, 用于恢复原始数据的特征信息.

$$\mu = \frac{1}{m}\sum_{i=1}^{m}{x_i}$$

$$\sigma^2 = \frac{1}{m} \sum_{i=1}^{m}{(x_i - \mu)}$$

$$\hat x_i = \frac{x_i - \mu}{\sqrt{\sigma^2}+\varepsilon}$$

$$\hat y_i = \gamma \hat x_i + \beta$$

# BN 解决了什么问题

BN 主要解决的是深层网络中不同网络数据分布不断发生变化的问题, 也就是 Internal Covariate Shift. 该问题是指在深层网络训练的过程中，由于网络中参数变化而引起内部结点数据分布发生变化的这一过程被称作Internal Covariate Shift. ICS 带来了两个问题:
- 上层网络需要不停调整来适应输入数据分布的变化，导致网络学习速度的降低
- 网络的训练过程容易陷入梯度饱和区,减缓网络收敛速度

<span id = "使用 BN 有什么好处">
# 使用 BN 有什么好处

总的来说，BN通过将每一层网络的输入进行归一化, 保证输入分布的均值与方差固定在一定范围内, 减少了网络中的 Internal Covariate Shift问题, 加速了模型收敛; 并且 BN 可以使得网络对参数的设置如学习率, 初始权重等不那么敏感, 简化了调参过程, 使得网络学习更加稳定; 同时 BN 使得网络可以使用饱和性激活函数如 Sigmoid, tahh 等, 从而缓解了梯度消失问题; 最后 BN 在训练过程中由于使用的 mini-batch 的均值和方差每次都不同，因此引入了随机噪声，在一定程度上对模型起到了正则化的效果, 也就是说, BN 可以起到和 Dropout 类似的作用, 因此在使用 BN 时可以去掉 Dropout 层而不会降级模型精度.

原因如下: https://zhuanlan.zhihu.com/p/34879333
**(1) BN使得网络中每层输入数据的分布相对稳定，加速模型学习速度**
BN通过规范化与线性变换使得每一层网络的输入数据的均值与方差都在一定范围内，使得后一层网络不必不断去适应底层网络中输入的变化，从而实现了网络中层与层之间的解耦，更加有利于优化的过程,提高整个神经网络的学习速度。

**(2) BN使得模型对初始化方法和网络中的参数不那么敏感，简化调参过程，使得网络学习更加稳定**
在神经网络中，我们经常会谨慎地采用一些权重初始化方法（例如Xavier）或者合适的学习率来保证网络稳定训练。当学习率设置太高时，会使得参数更新步伐过大，容易出现震荡和不收敛...

**(3) BN允许网络使用饱和性激活函数（例如sigmoid，tanh等），缓解梯度消失问题**
在不使用BN层的时候，由于网络的深度与复杂性，很容易使得底层网络变化累积到上层网络中，导致模型的训练很容易进入到激活函数的梯度饱和区；通过normalize操作可以让激活函数的输入数据落在梯度非饱和区，缓解梯度消失的问题；另外通过自适应学习 $\gamma$ 与 $\beta$ 又让数据保留更多的原始信息。

**(4)BN具有一定的正则化效果**
在Batch Normalization中，由于我们使用mini-batch的均值与方差作为对整体训练样本均值与方差的估计，尽管每一个batch中的数据都是从总体样本中抽样得到，但不同mini-batch的均值与方差会有所不同，这就为网络的学习过程中增加了随机噪音，与Dropout通过关闭神经元给网络训练带来噪音类似，在一定程度上对模型起到了正则化的效果。原作者也证明了网络加入BN后，可以丢弃Dropout，模型也同样具有很好的泛化效果。

<span id = "BN 层通常处于网络中的什么位置">
# BN 层通常处于网络中的什么位置
BN 通常应用于网络中的非线性激活层之前, 将卷积层的输出归一化, 使得激活层的输入在 [0, 1] 之间, 避免梯度消失的问题.

<span id = "BN 中 batch 的大小对网络性能有什么影响">
# BN 中 batch 的大小对网络性能有什么影响
由于 BN 在计算均值和方差时是在当前的 batch 上进行计算的, 因此, 当 batch 较小时, 求出来的均值和方差就会有较大的随机性, 从而导致效果下降, 具体来说, 当 batch 的大小低于 16 时, 就不建议使用 BN, 当 batch 低于 8 时, 网络的性能就会有非常明显的下降.

<span id = "BN 中线性偏移的参数个数怎么计算的">
# BN 中线性偏移的参数个数怎么计算的

对于 BN 层来说, 如果它的输入 shape 均为为 $(N, C, H, W)$, 则其输出 shape 也为 $(N, C, H, W)$, **即保持输入输出 shape 不变.** BN 中的线性偏移参数 $\gamma$ 和 $beta$ 的个数 **与输入 shape 的通道数相同, 均为 $C$**. PyTorch 中 $\gamma$ 和 $\beta$ 参数分别对应着`weight`和`bias`, 下面是 BN 的声明.
```py
torch.nn.BatchNorm2d(num_features, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
```

<span id = "Inference 阶段使用的均值和方差是如何求得的">
# BN 中使用的均值和方差是如何求得的

在训练阶段, 就是利用当前 batch 中的均值和方差进行计算.
在测试阶段, 采用的是网络中维护的滑动平均值进行计算的, 滑动平均值的维护方式是用当前的滑动平均值乘以一个 $decay$ 系数, 然后再加上 $(1 - decay)$ 倍的当前 batch 的统计值. $decay$ 决定了数值的更新速度, 通常 $decay$ 会设成一个非常接近于 1 的数, 比如, 0.99 或 0.999.

$$shadow_variable = decay \times shadow_variable + (1 - decay) \times variable$$.

<span id = "在多卡训练使用 BN 时, 需要注意什么问题">
# 在多卡训练使用 BN 时, 需要注意什么问题
需要注意多卡之间的通信同步问题
目前大部分的框架, 包括 Caffe, PyTorch, TF 等, 对于 BatchNorm 的实现都只考虑了 single GPU, **也就是说 BN 使用的均值和标准差是单个 GPU 计算的, 这相当于缩小了 mini-batch size**. 至于为什么这样实现: (1) 因为没有 sync 的需求, 因为对于大多数 vision 问题, 单 GPU 上的 mini-batch 已经够大了, 完全不会影响结果. (2) 影响训练速度, BN layer 通常是在网络结构里面广泛使用的, 这样每次都同步一下 GPUs, 十分影响训练速度.

<span id = "BN 的具体实现及其反向传播公式是如何计算的">
# BN 的具体实现及其反向传播公式是如何计算的

具体实现: 两个 Scale 层.

反向

https://www.jianshu.com/p/4270f5acc066
https://zhuanlan.zhihu.com/p/27938792

在Caffe2实现中, BN层需要和Scale层配合使用, 其中, BN专门用于做归一化操作, 而后续的线性变换层, 会交给Scale层去做.


训练阶段:
在训练时利用当前batch的mean和variance来进行BN处理, 同时使用滑动平均的方式不断的更新global 的mean和variance, 并将其存储起来.

测试阶段:
在预测阶段, 直接使用模型存储好的均值和方差进行计算

<span id = "使用 BN 时, 前一层的卷积网络需不需要偏置项, 为什么">
# 使用 BN 时, 前一层的卷积网络需不需要偏置项, 为什么

使用 BN 时的前一层卷积网络可以不加偏置项(降低模型参数量)
当使用 BN 时, 无论加偏置还是不加偏置, 效果都是一样的, 公式证明如下:

当卷积层后跟batchnormalization层时为什么不要
https://blog.csdn.net/u010698086/article/details/78046671

CChttps://zhuanlan.zhihu.com/p/36222443

另外, 在 BN 中的 $\beta$ 参数也可以起到一定的偏置作用.

# 使用BN时应注意的问题

bn总结: https://www.cnblogs.com/makefile/p/batch-norm.html
