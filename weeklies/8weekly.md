#2020/11/1 周报
这周的主要工作是读了五篇paper，运行了SMPL的代码

## 《Learning to Reconstruct 3D Human Pose and Shape via Model-fitting in the Loop》
这是 ICCV 2019的一篇文章，提出了结合 regression 和 optimization 的一种学习方法 SPIN，文章的亮点和值得注意的细节如下：
* optimization-based model 的精度更高但是容易受初值的影响，而 regression-based model 需要大量的训练数据；基于此，文章提出相结合的方法，利用 regression 得到的 estimate 使得 iterative optimization 更快速准确，同时 optimization 的结果反过来也可以作为 regression network 的 strong supervision。
* iterative fitting 只需要 2d keypoint 所以该方法在图片没有对应的 3d 标签的时候也可以训练。
* fitting model 通过 optimization 得到的 model parameters 可以给网络提供直接的 model-based supervision 而不需要通过计算其他的 2d 投影 loss 来进行监督学习。
* 虽然文章采用的 iterative fitting 是 SMPLify ，但由于使用了 regression network 来进行 initialization， 不需要利用 staged approach，只需要一个 optimization stage 通过较少的迭代就能很快收敛。
* 对于 ground truth 可能存在的 2d 标签人为错误问题，文章利用 openpose detection 提供的 2d 标签去给予置信度。
* 文章指出在 regression network 的训练过程中避免直接使用 2d reprojected loss，而使用 iterative fitting 的结果可以减少网络的学习负担。同时 SPIN 方法也有很好的 self-improving 特性。
* 文章提到在训练过程中一些 failure cases 可能会影像模型整体的训练效果，文章对于 optimization routine 中 joints reproduction error 大于阈值的，直接采用 2d reproduction loss 训练；同时对于 SMPLify 输出 shape 参数 与 SMPL prior 不符的，也不使用它进行训练，而是直接利用 L2 loss ，让 shape 参数更接近 mean pose。
* 为了加速和提升训练效果，对于单个训练数据，文章提出采用字典存储当前它的 best fit ，且字典的初始化可以采用 offline 的方式提前完成。

小节：这篇文章提出的方法很巧妙的结合了 optimization-based 和 regression-based 方法的优点，具有启发性，同时训练中提到的一些 trick 也可以借鉴。

## 《Keep it SMPL: Automatic Estimation of 3D Human Pose and Shape from a Single Image<br>ECCV 2016》
这是 ECCV 2016的一篇文章，提出了一种从单张图像恢复出 3d pose 和 shape 的方法 SMPLify，文章的亮点如下：
* 相比之前的方法，既解决了 3d pose 也估计出了 3d shape，并且整个过程 fully automatic ，数据上也只需要单张影像即可。
* 文章提出了一种解决 interpenetration 的方法， 利用 capsules 去 approximate body shape，训练了一个 linear regressor 从 shape coefficient 得到 capsules‘ radii and axis length。
* 文章提出要 minimize 的 objective function 有五个 error terms：
    1. ![](http://latex.codecogs.com/gif.latex?E_j) 减小 3d joints 投影后与 ground truth 2d joints 之间的差距。
        >"Our joint-based data term penalizes the weighted 2D distance between estimated joints, Jest, and corresponding projected SMPL joints"
    2. ![](http://latex.codecogs.com/gif.latex?E_a) 对肘关节和膝关节的弯曲进行限定。
        >"We introduce a pose prior penalizing elbows and knees that bend unnaturally"
    3. ![](http://latex.codecogs.com/gif.latex?E_\theta) 使用 mixture of Gaussian 来得到 pose 的 distribution, 计算熵函数。
        >"We then fit a mixture of Gaussians to approximately 1 million poses, spanning 100 subjects"
    4. ![](http://latex.codecogs.com/gif.latex?E_{sp}) 用来解决 interpenetration 的问题，且仅用来对 pose 进行调整。
        >"We consider a 3D isotropic Gaussian with σ(β) = r(β)/3 for each sphere, and define the penalty as a scaled version of the integral of the product of Gaussians corresponding to “incompatible” parts" 
    5. ![](http://latex.codecogs.com/gif.latex?E_\beta) 利用经过 PCA 变换的 SMPL pose 参数空间，计算马氏距离。
*  在训练过程中，文章假定 camera translation 和 body orientation 未知；训练前期固定shape 和 pose，完成camera translation 估计；之后采用 staged approach 来进行训练。

小节： 比较早的一篇文章可以借鉴的地方不多，主要是一些基础概念和知识，以及实验等方面的学习，关于 camera translation 和 staged approach 可以进一步了解。

## 