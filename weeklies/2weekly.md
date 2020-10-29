#  2020/8/16 周报 
---
这周主要结束了线性代数部分书本的学习，看了两篇论文《Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift》和 VGG《VERY DEEP CONVOLUTIONAL NETWORKS FOR LARGE-SCALE IMAGE RECOGNITION》，Inception v1的《Going deeper with convoluNions》正在整理。
## 对于BN的收获：
* 介绍了什么是covariate shit、internal covariate shit，深层网络训练较难的原因，由此引出了batch normalization的处理方式，它希望可以获得固定分布的每层的input，这样网络的梯度传播更稳定，learning rate可以适度提高，同时每个样本的梯度计算都考虑到了batch整体的数据有regularization的作用。
* 理论上，理想的情况下我们希望input层的feature可以不相关，但是实际情况下由于batch size和 feature数目之间的悬殊一定会导致矩阵奇异，同时导数计算量的计算量也过于庞大，这里做的batch normalization实际上做了两个规定，首先对于每个feature分别进行normalize（尽管存在相关性，依然可以加速训练），其次方差和均值统计量只在batch内进行（所有变量都可以参与后向传播），同时为了让normalize后的层能与之前表达至少相同的信息，进一步对normalize后的数据进行恒等变化（只增加了一个位移量和尺度量 但好处多多）。
* 后向传播的过程，只是在做了BN的层有区别，梯度从恒等变化的y流向normalize的x_bar，再流向方差量，流向均值，最后流向batch中的每个x。
* 最后inference的过程和训练不同的地方在于此时BN层的方差和均值来源于训练中的统计数据对整个training set 的估计。
* 至于为什么是对Wu而不是u进行normalization，作者的解释是W u + b is more likely to have a symmetric, non-sparse distribution, that is “more Gaus- sian” (Hyva ̈rinen & Oja, 2000);可以进一步阅读。
* 提高learning rate原因是使用BN后每层的输入更稳定，参数的变化不会放大对后面的层产生灾难性的影响。同时BN层关于u的偏导数关于W的尺度不变性，以及在W过大时甚至会限制它的增长这个特性可以让参数训练更稳定，解决了梯度爆炸的问题。具有regularization特点的原因主要是均值和方差计算涉及到了整个batch，dropout的比例可以减少甚至直接取消。
* 在实验部分，为了测试BN，同时适应它的诸多特性，在baseline的基础上，提高了learning rate、移除了dropout、减少了L2规范化、加速了learning rate decay、LRN被发现没有作用也被相应移除，同时shuffle更彻底避免epoch间几张图片同时出现在同一batch内。
总的来说BN这篇文章可挖掘的细节很多，同时文章中提到了许多之前的研究和理论有一些是可以进一步阅读的，同时BN机制确实很强大，无论是理论上还是效果上都是很值得学习。
![f](/picture/BN.jpg )
## 对于VGG的收获：
* 采用小卷积核来提高网络的深度，一方面可以减少网络参数（有类似regularization的效果），另一方面感受野同样的情况下使用多层小卷积对比单层大卷积使用了更多的非线性激活函数，使网络表达效果更强。
* 训练数据处理上，一方面沿用了AlexNet的color shit方式提高网络对色彩和亮度的泛化能力，但和AlexNet简单降采样之后裁剪的数据增强方式不同，VGG的作者在降采样方面做了不同的处理，比较了固定降采样大小和规定降采样大小区间两种不同处理，由于裁剪后大小固定，两个操作实际是分别对应single scale和 multiple scale，后者输入的图片尺度随着降采样的大小会变化，最后的实验结果也说明在训练时采用这样的scale jittering的数据增强方式明显效果更好。
* 训练结束，对test image进行inference的时候，作者提到了使用dense convnet evaluation，就是将最后三层的FC全部变成卷积层，这样直接将降采样的test image输入，最终会得到一个类似“图像”的输出，每个像素其实近似就是普通的 mult-crop evaluation中的一个crop 图像的输出结果（实际在边界部分有差异），对其求平均，得到结果，这种方式主要的有点是一次计算免去了先crop再计算的过程，缺点是精度比mult-crop 略有损失，作者也比较了这两种不同方式单独使用和同时使用的效果，总之，这种FC layer变conv layer的操作还是挺有意思的。
总的来说VGG这篇继Alexnet之后对网络深度进行了探究，时间节点是2014年，文章中也提到了一些前人的工作和代表性理论，可以继续阅读，同时还有一些其他的一些小收获：先训浅网络，将浅层网络作为深层网络初始化，加速深层网络的构建的技巧，然后作者关于定位任务以及数据集迁移的一些讨论也是第一次接触。

