#  2020/10/18 周报
这周的工作主要是读了三篇hand pose、shape的paper，下面分别总结一下这三篇的收获：

## [《Weakly-supervised 3D Hand Pose Estimation from Monocular RGB Images》](https://openaccess.thecvf.com/content_ECCV_2018/papers/Yujun_Cai_Weakly-supervised_3D_Hand_ECCV_2018_paper.pdf)
![f](/picture/1.png)

**这是ECCV2018的一篇文章，文章提出了一种3d pose估计方法，在合成数据集上完成训练并通过深度图弱监督过渡到真实世界数据集，文章亮点体现在：**

* 对于在真实数据集获取带标签的训练数据困难的问题，文章提出了使用容易获得的深度图对训练过程进行弱监督的方式进行训练。

* 文章提出的深度图监督的方式对于弱监督和全监督的效果都有贡献。

* 文章提出的全监督方式，在2d到3d的pose估计过程中除了利用了heat-map 还利用了中间过程中提取到的图像特征。

文章可能的不足：

* 只利用了3d pose回归深度图，只考虑了pose，没有考虑shape，虽然文章没有涉及shape，但是是否可以利用图像中的形状信息对深度估计做shape补充值得考虑。

* 深度图对于模型在监督和非监督上都有提升，但是是否应该考虑真实世界所获取的深度信息在边界上噪声比较明显，是否可以减少这些噪声在训练中的影响。

## [《3D Hand Shape and Pose Estimation from a Single RGB Image》](https://arxiv.org/pdf/1903.00812.pdf)

![f](/picture/2.png)

**这是CVPR2019的一篇文章，文章提出一种从单个RGB图像恢复出手形状的方法，在此之前的工作只关注从RGB中恢复出手的关键点坐标。文章的亮点体现在：**

* 为了解决3d mesh的高维性，文章中使用图结构来表示mesh，并利用GCNN将backbone提取到的图像特征转换成3d mesh。

* 为了解决从人工合成的数据集上训练的模型在真实世界数据集上效果不好的问题，文章提出了一种利用深度信息进行弱监督的方法，即利用ground truth对得到的3d mesh经过render后的深度图进行监督。

* 模型表现上，在mesh方面由于之前的工作没有涉及到直接从单张RGB图像中直接恢复出mesh，文章进行了和LBS以及MANO方法的简单对比，效果优于LBS，LBS优于MANO；在3d pose估计方面，和其他方法的比较上取得了明显的优势，尤其是与上一篇Cai提出的方法的比较，这两种方法都是采用了深度图作为弱监督的一部分。

文章可能的不足：

* MANO方法基于SMPL方法提出，SMPL是针对LBS的不足针对性的进行改进，但是3d mesh的表现上从作者给出的数据来看MANO方法要比LBS方法差很多，可能是采用文章所用的loss以及训练方法后，MANO的潜力无法得到全部体现。

* 在弱监督的环节，文章也指出只是用heat map加depth map这两个ground truth进行监督学习的模型效果很不好，因此引入了pesudo-ground truth mesh loss，就是用预先训练好的模型得到一个mesh，用它来指导训练，在这里loss的设计上可能可以进一步优化。

* 模型表现上，此种方法相对于其他方法的优势可能来源于预先设计好的mesh图结构上，但是hand和body一样有个体差异同时不同的pose也会带来变形，是否可以更直接的引入shape和pose对于图结构的影响值得进一步探究。

## [《Pushing the Envelope for RGB-based Dense 3D Hand Pose Estimation via Neural Rendering》](https://openaccess.thecvf.com/content_CVPR_2019/papers/Baek_Pushing_the_Envelope_for_RGB-Based_Dense_3D_Hand_Pose_Estimation_CVPR_2019_paper.pdf)

![f](/picture/3.png)

**这是CVPR2019的一篇文章，文章的亮点体现在：**

* 可进行梯度传播的 neural render生成出分割和3d joint的结果，用于训练mesh。

* 在test过程中，可以通过减少render输出和中间结果的差异，利用梯度进行迭代提高mesh精度。

* 自监督数据增强，通过改变得到mesh的形状和视角合成出相应的训练数据用于数据增强。

* 文章提出的方法实际上整合了多篇paper的研究成果，同时借鉴了body pose、shape领域的经验，模型精度得到保证。

* 在联合训练的过程中文章提出了对articulation loss和shape loss层次化的训练方式，训练初期不考虑shape，当skeleton loss减小到阈值以内再考虑shape loss，这种方法确实更加合理，有助于模型的收敛，可以借鉴。

文章可能存在的不足：

* 本文通过联合估计3d pose和shape方法，在3d skeleton和2d hand segmentation上都取得了很好的效果，那么如果将训练数据中的2d hand mask替换成depth mask是否可以进一步提高shape估计的效果，同时也可以不用考虑2d hand mask 中 shape loss在训练初期对整体的影响。