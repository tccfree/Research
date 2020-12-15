
## [《Weakly-Supervised Mesh-Convolutional Hand Reconstruction in the Wild》]()
![](/picture)

**这是 CVPR 2020 的一篇文章，文章提出了一种 weakly-supervised mesh convolutions-based system，文章的亮点和值得注意的地方如下：**

* 文章指出现阶段的 SOTA 方法并不能 always generalize well to samples captured in non-laboratory environment，因此文章提出的方法主要针对 in the wild 数据，其中一个重要的 contribution 就是 a weakly-superviesed training method for 3D mesh reconstruction， 可以从 unlabeled images 中生成训练所用的 dataset。

* 文章指出在 hand pose estimation 方面，存在 variational approaches，与从 estimated 2D keypoint 恢复出 3D pose 不同；具体提到了 Gao et al 等人的工作，同时 Cai et al 利用 graph convolutions in the spectral domain 的工作也可以进一步阅读。

* 