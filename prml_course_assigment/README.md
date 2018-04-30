# **调研报告**

#### 小组成员：

##### 成员一：姓名：尹志雨    学号：2017Z8009061116

##### 成员二：姓名：张超        学号：2017Z8009061120

本作业及相关代码托管在GitHub上，如需查阅相关代码，可通过网址https://github.com/Blssel/TF-learing/tree/master/prml_course_assigment 访问

----------------------

## 前言

沿着CNN的发展历史，将几种经典的卷积神经网络做一个梳理，揣摩每一种网络设计之初的构想以及相对应采用的解决方案。然后结合CNN在目标检测上的应用做一个总结。
## 12年以后的发展脉络
CNN网络的发展中最有代表性的几种网络结构分别是AlexNet，VGGNet，GoogLeNet，ResNet，这几种网络的整体对比如下表，本文主要就更有涉及感，贡献更大，效果更好的后两中网络做详细介绍。
![](https://note.youdao.com/yws/public/resource/3f007aef5f79a9fa8a01b51a43ab1108/xmlnote/60980BACC33C4196986F8DFEB7C25984/22465)
### 各种网络的性能参数量对比

![](https://static.leiphone.com/uploads/new/article/740_740/201708/59969bc24ab27.png?imageMogr2/format/jpg/quality/90)


## ResNet
ResNet网络出现于2015年ImageNet图像分类大赛上，由微软Kaiming He等人提出，在原有网路的基础之上增加了若干跨层的短连接，在不增加计算消耗的情况下使得网络可以做的更深，并且性能有了大幅提升。对应论文发表于CVPR2015.
### Motivation
在ResNet出现之前，卷积神经网络的发展有这么一个趋势，那就是研究者倾向于将网络越做越深，因为从理论上来讲，更多层数的堆叠会使得网络具备更强的非线性拟合能力。但实验现象并不符合我们的预期，首先，更深的网络会引来梯度消失和梯度弥散的问题;其次，实验现象印证，随着层数的增加，网络的分类精度越发趋向于饱和，然后开始快速下降，如图1所示。

![](https://note.youdao.com/yws/public/resource/3f007aef5f79a9fa8a01b51a43ab1108/xmlnote/19760CFE6F1F4653BFB0F0A5CEB26477/22455)

关于梯度消失和弥散问题，研究者已经找到了许多相当好的解决方案，比如normalized initialization和intermediate normalization layers，使得SGD可以让网络很好的收敛。ResNet主要解决性能退化(degradation)问题，导致性能退化的主要原因是更深的网络使得训练变得非常困难，He等人做了一个有意思的实验，他们在已有的浅层网络基础上增加了若干恒等映射(identity mapping)来使得网络变得更深，所谓恒等映射，按笔者理解，应该是由num_channels个1*1*num_channels的卷积核构成的，其中每个卷积核都是只有某一位为1其它位置全为0的稀疏向量，比如(1,0,0,0,...,0)。从理论上说，该网络和浅层网络是等价的，因此性能至少不会比对应的浅层网络差，但事实证明其训练误差要比浅层的大，这就印证了一个规律：太深的网络会增加优化难度，而且这种难度并不是由过拟合造成的，因为其训练误差比浅层的还大。

### 解决方案
为解决过深网络存在的degradation问题，He等人设计了一个残差网络(Residual Network)。设计思想是：在网络中的某一层或隔层之间额外增加一个恒等映射，并且寄希望于让该层或几层去学习残差，而非直接学习直接的映射关系。
形式化的表达如下：假设要拟合一个映射$$H(x)$$，我们不直接学习，而是去拟合另一个映射$F(x)=H(x)-x$，然后再加上$x$，这样就达到了同样的效果，即$$F(x)+x$$等价于$$H(x)$$。这么做是基于这样一个假设，即学习残差比直接学习映射要容易。一个极端情况是，如果需要学习一个恒等映射，学习一个0残差比直接学习映射要简单许多。总之，构建的模块形式化表达为为下式：
$y=F(x,{W_i})+x$

实现在网络中则采用如下方式，即在层之间增加一个短连接(shortcut connections)，如图2所示。短连接的作用就是加了一个跨层的恒等映射，如此一来，短连接之间的映射就变成了一个残差映射。图2所示是一个跨2层的短连接，残差映射部分的数学表达为$$F=W_2 o(W_1 x)$$，最后$$F$$与$$x$$做一个元素级别的相加。值得注意的是，$$F$$和$$x$$的维度有可能不相同，在这种情况下，我们采用$$y=F(x,{W_i})+W_s x$$的方式，即给$$x$$做一个映射。如果是全连接网络，直接匹配输入层维度即可；但如果是卷积网络，则要么采用zero padding的方式强制保持维度一直，要么采用1*1，stride=2的卷积核，使得维度保持一致（注：此处的维度不一致专指通道数，因为采用3*3的卷积核加padding，feature map的长宽尺寸保持不变）

![](https://note.youdao.com/yws/public/resource/3f007aef5f79a9fa8a01b51a43ab1108/xmlnote/8C9D2ADC17854796B16FB9DABC39EE2C/22457)

### 效果
实验证明，ResNet所采用的思想不仅使得优化变得容易，而且在不增加参数量的情况下提升了预测精度。更值得注意的是，ResNet支持将网络做的非常深，甚至可以达到1000层。下表所示为网络参数。

![](https://note.youdao.com/yws/public/resource/3f007aef5f79a9fa8a01b51a43ab1108/xmlnote/86321014246E42F9A9041F49A0751F85/22463)

### Tensorflow实现ResNet代码

本作业包括完整代码已上传至本人[GitHub](github.com/blssel)上，代码说明如下，代码地址为：https://github.com/Blssel/TF-learing/blob/master/prml_course_assigment/resnet_model.py

```python
"""根据Kaiming He等人的Kaiming He, Xiangyu Zhang, Shaoqing Ren, Jian Sun 
Deep Residual Learning for Image Recognition. arXiv:1512.03385所搭建的
ResNet50 为v1版，源代码来自GitHub tensorflow开源项目，由谷歌官方搭建，
源码地址为：https://github.com/tensorflow/models/blob/master/official/resnet/resnet_model.py
由Zhiyu Yin在学习过程中注释为中文，且仅作为学习使用。
"""
```

### TF-slim版代码

该版本为Tf-slim版，与原生tensorflow实现方式不同，所以学习价值也比较大，代码地址为：https://github.com/Blssel/TF-learing/tree/master/prml_course_assigment/resnet_model_slim


## Google net
### Inception V1
#### Motivation
提升网络性能最直接的办法就是增加网络深度和宽度。但在实际操作中这并非一帆风顺，因为这会导致巨量的参数，同时过拟合的风险也会大大增加（我们通过ResNet的解读还知道网络过深还会导致优化难度的增加）。Google net想解决的问题是：1.通过使用较为密集的子矩阵来模拟CNN的稀疏连接，从而提升计算性能（因为大量的文献表明可以将稀疏矩阵聚类为较为密集的子矩阵来提高计算性能），这也就是inception模块所做的贡献之一。2.inception模块还可以帮助增加模型宽度，从而更好地兼顾不同大小的感受野和尺度。3.inception模块通过加入1×1的卷积核还能大幅减少参数量。
#### Inception 模块
Inception 结构的主要思路是怎样用多个密集成分来近似最优的局部稀疏结构，如图3所示。

- 采用不同大小的卷积核意味着不同大小的感受野，最后拼接意味着不同尺度特征的融合

- 之所以卷积核大小采用1、3和5，主要是为了方便对齐。设定卷积步长stride=1之后，只要分别设定pad=0、1、2，那么卷积之后便可以得到相同维度的特征，然后这些特征就可以直接拼接在一起了

- 文章说很多地方都表明pooling挺有效，所以Inception里面也嵌入了

- 网络越到后面，特征越抽象，而且每个特征所涉及的感受野也更大了，因此随着层数的增加，3x3和5x5卷积的比例也要增加

  ![](https://img-blog.csdn.net/20160904155917864)

当然，这并未inception模块的最终形态，因为使用5x5的卷积核仍然会带来巨大的计算量。 为此，文章借鉴NIN2，采用1x1卷积核来进行降维，如图4所示。

例如：上一层的输出为100x100x128，经过具有256个输出的5x5卷积层之后(stride=1，pad=2)，输出数据为100x100x256。其中，卷积层的参数为128x5x5x256。假如上一层输出先经过具有32个输出的1x1卷积层，再经过具有256个输出的5x5卷积层，那么最终的输出数据仍为为100x100x256，但卷积参数量已经减少为128x1x1x32 + 32x5x5x256，大约减少了4倍。总之，图4结构的inception模块在图3的基础上进一步减少了参数。

![](https://img-blog.csdn.net/20160904160721902)

#### Inception v1网络结构
网络结构如图5所示。

![](https://img-blog.csdn.net/20160904161917654)

对图5做如下说明： 
- 显然GoogLeNet采用了模块化的结构，方便增添和修改
- 网络最后采用了average pooling来代替全连接层，想法来自NIN,事实证明可以将TOP1 accuracy提高0.6%。但是，实际在最后还是加了一个全连接层，主要是为了方便以后大家finetune。
- 虽然移除了全连接，但是网络中依然使用了Dropout
- 为了避免梯度消失，网络额外增加了2个辅助的softmax用于向前传导梯度。文章中说这两个辅助的分类器的loss应该加一个衰减系数，但看caffe中的model也没有加任何衰减。此外，实际测试的时候，这两个额外的softmax会被去掉。

### Inception V2
#### Motivation
本文可以说其实并没有理论上的创新，仅仅是想在v1的基础上进一步减少参数量，提供了扩展inception模块大小的可能性。本文，谷歌给出了许多实验上的经验与结论：
- 信息流前向传播过程中不能使用高度压缩的层，即表达瓶颈。从input到output，feature map的宽和高基本都会逐渐变小，但是不能一下子就变得很小。比如你上来就来个kernel = 7, stride = 5 ,这样显然不合适。 另外输出的维度channel，一般来说会逐渐增多(每层的num_output)，否则网络会很难训练。（特征维度并不代表信息的多少，只是作为一种估计的手段）
- 高维特征更易处理。 高维特征更易区分，会加快训练
- 在进行小尺度卷积之前，可以对输入先进行降维而不会产生严重的后果。假设信息可以被简单压缩，那么训练就会加快
- 平衡网络的宽度与深度是比较合理的设计
#### Solution
首先，尝试是否能用两个3×3的卷积核代替5×5的卷积核，可以减少参数量，而且实验表明这样不会造成表达缺失，作者也做了对比试验，表明在3×3卷积核之后添加非线性激活会提高性能。
进一步的，文章考虑进一步缩减参数的方法。任意nxn的卷积都可以通过1xn卷积后接nx1卷积来替代，如图6所示。实际上，作者发现在网络的前期使用这种分解效果并不好，而在中度大小的feature map上使用效果才会更好。（对于mxm大小的feature map,建议m在12到20之间）

![](https://note.youdao.com/yws/public/resource/3f007aef5f79a9fa8a01b51a43ab1108/xmlnote/4A467EBD16014733BC410725E56F0C9D/22461)

图7，图8，图9循序渐进地展现了文章的设计思路。

![](https://note.youdao.com/yws/public/resource/3f007aef5f79a9fa8a01b51a43ab1108/xmlnote/76D6A940D7C942CFBD59A196EC10088C/22459)

其中，
- 图7是GoogLeNet V1中使用的Inception结构
- 图8是用3x3卷积序列来代替大卷积核
- 图9是用nx1卷积来代替大卷积核，这里设定n=7来应对17x17大小的feature map。该结构被正式用在GoogLeNet V2中

### Inception V3
v3一个最重要的改进是分解（Factorization），将7x7分解成两个一维的卷积（1x7,7x1），3x3也是一样（1x3,3x1），这样的好处，既可以加速计算（多余的计算能力可以用来加深网络），又可以将1个conv拆成2个conv，使得网络深度进一步增加，增加了网络的非线性，还有值得注意的地方是网络输入从224x224变为了299x299，更加精细设计了35x35/17x17/8x8的模块。
### Inception V4
v4研究了Inception模块结合Residual Connection能不能有改进？发现ResNet的结构可以极大地加速训练，同时性能也有提升，得到一个Inception-ResNet v2网络，同时还设计了一个更深更优化的Inception v4模型，能达到与Inception-ResNet v2相媲美的性能。


## Dropout技术
### Motivation and intuitive analysis
过拟合是Deep Neural Networks(DNN)网络存在的问题之一。过拟合的特点是模型对训练数据的拟合非常好，但对测试数据的拟合却非常差，具体表现为loss和在训练集上的错误率非常低，而在验证集或测试集上却都要高很多。针对解决过拟合问题设计出来的方法很多，dropout就是其中一种最简单，也是最有效的方法。

如何使用Dropout？在训练DNN网络的过程中，对于每一个神经元，以p的概率被随机的drop out，也就是将其值置零。这样，在该轮前传和反传的过程中，该神经元将失去作用，相当于不存在，如图3所示。

一种intuitive的理解方式是，当某一部分神经元被置零后，整个网络看起来就像一个新的网络，对该网络做单独的训练。下一个阶段又是另一个“新的”网络被训练，最后，这些网络结合起来就相当于一种集成学习，每个网络既相互提升，又相互制约，使得最终的结果也有所提升。从更细的方向考虑，在训练网络的过程中，神经元通过梯度指引来调整自身取值，使得整体loss下降，这时候某些神经元很有可能会“迁就其它神经元的错误”，换句话说，这些神经元可能已经接近或达到“正确取值”，也就是它们已经无需再优化了，错误是由其它神经元引起的，但这时该神经元很可能会继续调整自身取值，来优化其它神经元所导致的错误，这种现象称作co-adaptations，即相互适应。这时候网络的训练就出现了偏颇，因为尽管最终loss降下去了，但得到的却不是“标准答案”，而仅仅是在训练数据上适用，并不具备泛化性能。我们有理由假设，随机的使某些神经元置零，可以阻止这种相互适应的发生。比如A神经元已经接近“标准取值”，而B神经元还需要优化，这时很有可能A会迁就B，继续调整自身取值，从而导致过拟合；而此时如果将B神经元隐去，A就会认为网络已经优化的很不错了，就会保持自身的值不变，这样一来就抑制了相互适应现象的出现。

![](https://pgaleone.eu/images/dropout/dropout.jpeg)

## Batch Normalization

![](https://img-blog.csdn.net/20170421111120014?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGhhbmNoYW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)