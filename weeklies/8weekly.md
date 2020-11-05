# 2020/11/1 周报

这周的主要工作是读了五篇paper，运行了SMPL的代码

## [《Learning to Reconstruct 3D Human Pose and Shape via Model-fitting in the Loop》](https://arxiv.org/pdf/1909.12828.pdf)
![](/picture/5.png)

**这是 ICCV 2019的一篇文章，提出了结合 regression 和 optimization 的一种学习方法 SPIN，文章的亮点和值得注意的细节如下：**

* optimization-based model 的精度更高但是容易受初值的影响，而 regression-based model 需要大量的训练数据；基于此，文章提出相结合的方法，利用 regression 得到的 estimate 使得 iterative optimization 更快速准确，同时 optimization 的结果反过来也可以作为 regression network 的 strong supervision。

* iterative fitting 只需要 2d keypoint 所以该方法在图片没有对应的 3d 标签的时候也可以训练。

* fitting model 通过 optimization 得到的 model parameters 可以给网络提供直接的 model-based supervision 而不需要通过计算其他的 2d 投影 loss 来进行监督学习。

* 虽然文章采用的 iterative fitting 是 SMPLify ，但由于使用了 regression network 来进行 initialization， 不需要利用 staged approach，只需要一个 optimization stage 通过较少的迭代就能很快收敛。

* 对于 ground truth 可能存在的 2d 标签人为错误问题，文章利用 openpose detection 提供的 2d 标签去给予置信度。

* 文章指出在 regression network 的训练过程中避免直接使用 2d reprojected loss，而使用 iterative fitting 的结果可以减少网络的学习负担。同时 SPIN 方法也有很好的 self-improving 特性。

* 文章提到在训练过程中一些 failure cases 可能会影像模型整体的训练效果，文章对于 optimization routine 中 joints reprojection error 大于阈值的，直接采用 2d reproduction loss 训练；同时对于 SMPLify 输出 shape 参数 与 SMPL prior 不符的，也不使用它进行训练，而是直接利用 L2 loss ，让 shape 参数更接近 mean pose。

* 为了加速和提升训练效果，对于单个训练数据，文章提出采用字典存储当前它的 best fit ，且字典的初始化可以采用 offline 的方式提前完成。

***小节：这篇文章提出的方法很巧妙的结合了 optimization-based 和 regression-based 方法的优点，具有启发性，同时训练中提到的一些 trick 也可以借鉴。***

## [《Keep it SMPL: Automatic Estimation of 3D Human Pose and Shape from a Single Image》](https://arxiv.org/pdf/1607.08128.pdf)
![](/picture/6.png)

**这是 ECCV 2016的一篇文章，提出了一种从单张图像恢复出 3d pose 和 shape 的方法 SMPLify，文章的亮点如下：**

* 相比之前的方法，既解决了 3d pose 也估计出了 3d shape，并且整个过程 fully automatic ，数据上也只需要单张影像即可。

* 文章提出了一种解决 interpenetration 的方法， 利用 capsules 去 approximate body shape，训练了一个 linear regressor 从 shape coefficient 得到 capsules‘ radii and axis length。

* 文章提出要 minimize 的 objective function 有五个 error terms：

    + ![](http://latex.codecogs.com/gif.latex?E_j) 减小 3d joints 投影后与 ground truth 2d joints 之间的差距。
        >"Our joint-based data term penalizes the weighted 2D distance between estimated joints, Jest, and corresponding projected SMPL joints"

    + ![](http://latex.codecogs.com/gif.latex?E_a) 对肘关节和膝关节的弯曲进行限定。
        >"We introduce a pose prior penalizing elbows and knees that bend unnaturally"

    + ![](http://latex.codecogs.com/gif.latex?E_\theta) 使用 mixture of Gaussian 来得到 pose 的 distribution, 计算熵函数。
        >"We then fit a mixture of Gaussians to approximately 1 million poses, spanning 100 subjects"

    + ![](http://latex.codecogs.com/gif.latex?E_{sp}) 用来解决 interpenetration 的问题，且仅用来对 pose 进行调整。
        >"We consider a 3D isotropic Gaussian with σ(β) = r(β)/3 for each sphere, and define the penalty as a scaled version of the integral of the product of Gaussians corresponding to “incompatible” parts" 

    + ![](http://latex.codecogs.com/gif.latex?E_\beta) 利用经过 PCA 变换的 SMPL shape 参数空间，计算马氏距离。

*  在训练过程中，文章假定 camera translation 和 body orientation 未知；训练前期固定shape 和 pose，完成camera translation 估计；之后采用 staged approach 来进行训练。

***小节： 比较早的一篇文章可以借鉴的地方不多，主要是一些基础概念和知识，以及实验等方面的学习，关于 camera translation 和 staged approach 可以进一步了解。***

## [《Embodied Hands: Modeling and Capturing Hands and Bodies Together》](https://www.is.mpg.de/uploads_file/attachment/attachment/392/Embodied_Hands_SiggraphAsia2017.pdf)
![](/picture/8.png)

**这是 Siggraph Asia 2017 的一篇文章，阅读这篇的主要目的是学习 MANO 模型，文章的亮点和值得注意的地方如下：**

* 文章提出了 hand Model with Articulated and Non-rigid defOrmations，即 MANO 方法，此种方法基于 SMPL，但是在 corrective pose blend shape 上做了 localize 的操作，并且也对 pose space 使用了 PCA 变化，降低了维数。

* 文章对于 hand capture、fitting、 model 的历史做了很多的介绍，之后如果有需要可以进一步了解。

* MANO 模型包含了 15 个 joints，但是很多 joints 的运动只有一个自由度，但模型中仍然将它们都视为 ball joints，所以事实上 hand pose 实际是 over-parameterized， 因此 pose-dependent blend shapes 需要 regularization，文章巧妙的为 P 建立了一个 cost matrix，其中的每一项都对应一个 cost for a pair of input joint and output vertex。

* 由于在实验数据集中 mean pose 并不是预先定义的 flat hand（zero pose），为了防止 Ti 在开始阶段 absorb the pose-dependent correctios between zero and mean pose，所以在 first iteration of Ti optimization 过程中，将与 flat hand 差距过大的 registraion 弱化，这里的处理比较细节，值得学习。 

* 文章还进行了 hand pose embedding，采用的方式是较为简单的 linear mapping obtained by PCA，在实验数据集上，大概需要6个 component。在 discussion 部分，文章也指出未来这一部分可以使用 deep learning 来学习非线性的 latent space，但是同时也会需要更多的训练数据。

* 在对 sequence 数据进行 mesh registration 的过程中，文章采取了 two stage approach，其中 first stage 主要是为了得到 subject-specific template，在完成 shape blend weight 的估计之后，在 second stage 中就只需要对 pose 参数进行优化，减少了 20 个参数的数量，可以加速之后的训练速度。在细节方面，文章还提出了使用 extra intial stage 主要是为了解决第一帧没有很好的初值的问题，同时考虑到视频连续的特点还引入了 temporal pose smotheness term 来 regularize 相邻帧之间 pose 的改变。

* 文章指出在 self-contact 和 hand-object contact 方面可以有进一步的工作，同时 hand motion 与 body motion 的 correlation 也可以进一步被探究。

***小节：这篇文章提出了 MANO 模型，阅读这篇主要也是学习 MANO 这样一个 hand model，但同时文章在实验和训练过程中做的考虑和一些细节的处理同样值得认真阅读和思考，可能可以在未来的工作中借鉴。***

## [《3D Hand Shape and Pose From Images in the Wild》](https://arxiv.org/pdf/1902.03451.pdf)
![](/picture/7.png)

**这是 CVPR 2019 的一篇文章，提出了端到端的从 in the wild 影像恢复出 3d hand pose 和 shape 的深度学习方法，文章的亮点和值得注意的地方如下：**

* 对 in the wild 单张影像进行 3d pose estimation 的主要困难在于：有合适 3d 标签的大数据集少，同时现有的数据集不能很好的 generalize 到 in the wild 的影像上。 to this end，文章提出了基于 MANO 模型的方法，从而减少了对 3d 标签的依赖，让问题转变为 learning-based model fitting，可以用 2d 或者 3d 标签进行训练。

* 文章提出的 model-based decoder 包含了 re-projection module 和 hand model，在合成数据集上经过了预训练，所以在整个端到端的结构中不需要训练，参数保持 fixed 。

* 文章利用了 weak perspective model 去解决 in the wild 影像没有 camera intrinsics 的问题。

* 在 training objective 方面，文章选用了 2d re-projection loss， 3d joint loss， hand mask loss 和 model paramter regularization loss；其中 2d loss 采用了 L1 loss，而对 3d loss 的 penalize 则采用了 L2 loss， 相比 2d L1 惩罚更多，主要是因为 2d 投影和标签都存在误差； hand mask loss 是文章提出的一种 novel loss， 主要惩罚 vertices 投影落在 hand 区域以外。

* 文章比较了只使用 2d 标签，通过投影 3d joints 来进行监督学习的 2d fitting 方法，发现在3d 的效果方面文章提出的方法要显著更好。

***小节：这篇提出的方法比较简单，但是模型的效果很好，L1 L2 loss 的使用可以借鉴。***

## [《Weakly Supervised 3D Hand Pose Estimation via Biomechanical Constraints》](https://arxiv.org/pdf/2003.09282.pdf)
![](/picture/9.png)

**这是 ECCV 2020 的一篇文章，文章提出了一系列 novel loss 来提升神经网络从 2d image 生成 feasible 3d hand pose 的能力，文章的亮点和值得注意的地方如下：**

* direct additional 2d supervision 对于减轻 3d depth 和 scale 方面的不确定性帮助很少，improvement 主要来自于生成的 3d pose 与 2d projection 吻合度提升，但是深度信息方面的不确定性依然没有得到正面的解决，因此仍然需要大量的 fully annotated training data，而文章提出的 BMC 方法可以减少对 3d 标签数据的依赖。文章认为 human hand 受制于 biomechanics，据此，文章：
    >While the bone length constraints have been used successfully [39,49], capturing other biomechanical aspects is more difficult. Instead of fitting a hand model to the predictions, we extract the quantities in question directly from the predictions to impose our constraints.

* 具体来说，文章提出的 soft constraints 通过几个 loss 来表达，可以加在任何输出 3d joints 的深度学习网络中，包含三个部分：valid bone lengths， valid palm structure， valid joint angles of thumb and fingers。

* 针对 Iqbal 等人提出的 2.5D representation 方法，文章指出 J2d 和 zr 的 small error 会对 Zroot 的估计产生很大的影像，从而降低最终 absolute pose 的精度，所以提出了一个 novel refinement network 计算 Zroot 的 residual term，进行优化。 

* 这篇文章对 hand pose estimation from monocular RGB 的两种方法 model-based（model-fitting） 和 learning-based（regression）做了较为详细的 review，可以参考。

* 文章提出的方法 extract quantities in question directly from predictions，并施加 biomechanical constrain，从而达到 predict anatomically plausible hand pose for weakly-supervised data 的目的，提高了模型的 generalizability。

* 文章考虑到了 finger angle 之间的 inter-dependency，因此近似了一个 convex hull，之后的 ablation study 也证实了 co-dependent angle limits 的重要性，相关内容可以进一步学习。

* 文章将 3d preiction error 通过 pinhole camera model 分解成 2D（J2D）和 depth component（Z），这样可以进一步对 3d prediction error 的分布和来源做分析，验证提出的模型是否对深度信息的估计有改善，这个方法可以借鉴。

* 通过实验，验证了 BMC 方法可以在相同模型精度要求的情况下减少训练所需要的 3d 标签数量。

***小节：总体来看文章提出的方法，抓住了 2d 标签相对 3d 标签提供的信息不足这个缺陷做了补充，效果很好，这个思路可以借鉴。其次文章实验部分的设计比较巧妙，尤其是探究精度改善是否来源于深度估计的改善，实验结果也充分说明了 HO-3D D+O 这些数据集的难度，以及 synthetic dataset 提供的 3d 标签对训练效果的影响。***

## SMPL 代码运行截图
![](/picture/SMPL.png)
