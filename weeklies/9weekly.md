# 2020/11/13 周报

## [《Model-Based 3D Hand Pose Estimation from Monocular Video》](http://www.cs.toronto.edu/~fleet/research/Papers/handTrackerPAMI.pdf)
![](/picture/11.png)

**这是雨劲师兄推荐的一篇TPAMI 2011 的一篇文章，主要介绍了一种从视频进行 3D hand tracking 的方法，文章的亮点和值得注意的地方如下：**

* 文章认为 vision-based hand tracking 面临的问题主要是：1、high degree of freedom；2、availablity visual cues。尤其是第二点，作者认为轮廓和边缘提供的信息有限，提出了新的观点，认为对于手这样的物体 shading 是较为关键的视觉信息。

* 文章简单介绍了 discriminative 和 generative model 的概念，并提出了一种基于 genrative model 的 analysis-by-synthesis 的方法，将 shading 和 texture 的信息都容纳进来。

* 文章提出了一种 novel detailed derivartion of gradient in the vicinity of depth discontinuities，探究由于 occlusions 造成的，体现在 gradient 中的重要信息。

* 文章采用了 adpative albedo function 来 model texture（albedo variance）和其他可能的 hand appearance properties来；并且提到了通过 multiply reflectance 和 irradiance 可以得到 appearance of the point on surface。其中 illuminant model 可以用 4D vector 表示，irradiance 可以使用 ambient coefficient 加上 surface 法向量和 distant point direction 的标量积来表示。illuminant model 和 irradiance 的原文描述如下：
    >"The illuminant model includes ambient light and a distant point source, and is specified by a 4D vector denoted by L, comprising three elements for a directional component, and one for an ambient component."

    >"The irradiance at each vertex of the surface mesh is obtained by the sum of the ambient coefficient and the scalar product between the surface normal at the vertex and the light source direction. The irradiance across each face is then obtained through bilinear inter- polation."

* 在 texture model 上，首先文章采用了双线性插值保证了 facet 内部的连续性；但是由于手的拓扑结构不是圆盘，不能简单定义一个 surface 到 2D planar surface 的 bijective mapping，文章进一步采用了 patch-based texture mapping，每个 facet 被关联到一个 triangular region，由于在 edge 可以由相邻的两个 patch 分别得到，为了保证 consistency，文章又对于 texture map 增加了两个 linear constraints：第一个保证了 integer-coordinate 点 intensity 的一致性，第二个保证了在 diagonal edge 上进行双线性插值的的时候会和 horizontal or vertical edge 一样都对位置参数呈线性关系，从而保证了 non-integer coordinate 的一致性，这一步推导的过程直接令二阶导数为零，比较巧妙。尚存的问题是一个 facet 有三个 adjacent facet，那么按文章提出的方法，是否一个 facet 要在 texture map 中出现三次？

* 给定 mesh geometry，illuminant 和 texture map，可以通过 ray-tracing 确定 generative model 生成的 model image。

* 在 objective fuction 的设计上，文章提出了 discrepancy measure E：首先定义了一个 residual image R，再将 integral of the residual error 定义为需要 minimize 的 discrepancy measure E。文章指出可以通过转换积分域，将 hand region 内部的 integral of residual error 转换成 visible part of surface 上的连续积分，进而可以用 finite weighted sum over centers of visible faces来 表示；但是这种离散化的方法相当于是二值化 facet 的 visibility，所以会使  objective funtion 关于 ![](http://latex.codecogs.com/gif.latex?\theta) 不连续,对 gradient-based optimization 带来影响，所以文章在计算 functional gradient 的时候将 formulation 定义在连续的 image domain 上。

* 文章指出 parameter estimation 不能通过简单的 global optimization 得到，要采用 quasi-Newton，local optimization 的方法，但仍需要提供 discrepancy measure E 关于 pose parameters ![](http://latex.codecogs.com/gif.latex?\theta) 和 illuminant parameters L 的导数；其中后者的计算直接使用 chain rule 即可，但是关于 pose paramters 导数的估计由于 residual error 在边界上关于 pose 不连续，不可导，所以不能简单通过 chain rule 得到。

* 文章指出在 self-occlusion 和 occlusion boundaries 处 residual error 不连续，为了能推导出连续情况下 gradient of E with respect to ![](http://latex.codecogs.com/gif.latex?\theta) ,文章分别对 1D 和 2D 的情况做了说明，通过对 boundary 处做一些处理，虽然 residual error 不连续，但是积分得到的 objective function 依然是关于 ![](http://latex.codecogs.com/gif.latex?\theta) 可导的。

* 进一步，文章比较了在 numerical apxroximation to E 的基础上计算 gradient 几种方法，指出直接离散化 gradient of E 的结果与 gradient of approximatin to E 存在系统性差别；另一种方法直接计算 gradient of approximation to E 则由于采样时的 aliasing，会导致 obejective function E 关于 ![](http://latex.codecogs.com/gif.latex?\theta) 不连续。据此，文章提出了一种关于 E 的 differentiable discretization。

* 文章提出的关于 objective function E 的 differentiable discretization 受到了 discontinuity edge overdraw method 方法的启发，通过构造新的 residual function 解决了 aliasing 的问题；主要的思想是建立 anti-aliased region，这个区域内的 points 的 residual 使用 linear combination of the residuals on the occluded and occluding sides。

* 文章在 minimize  objective function E 方面使用了sequential quadratic method with BFGS Hessian approximation；大概的思想应该是通过对 Hessian 的估计得到一个 linearly constrained subspace，在这个 subspace 内沿着 gradient 的方向 update 可以保证 objective function decrease。 这一部分的内容如果有必要可以进一步学习。

* 在 texture update 的过程中，文章提出的方法需要 model texture for hidden area of hand，并且在 texture update 的过程中要避免被无关部分错误影响；所以提出的objective function 在 tracking 的基础上加入了 smoothness regularization term，来 diffuse the color to texels that don’t directly contribute to the image， 并加入了一些 tricks 规避了 boundary inaccurate estimation 对  texture 的影响，增强了robustness。

* 在实验部分，文章验证了提出的 occlusion forces 的效果，指出在 finger 弯曲的时候，由于提出的 residual error 使用了 occlusion forces 在 extremity of finger 部位可以更好的计算关于 pose parameter 的梯度，将其拉回到正确的位置。

* 文章在实验部分还指出尽管模型的 silhouette 与 ground truth 已经吻合的很好，但是 hand surface error 还是很明显，可能仍然存在一些 ambiguities 例如 depth ；同时值得一提的是，使用 shading 和 texture 确实能消除大部分这样的 ambiguities，相比于只利用 silhouette 的方法效果更好。

* 文章提到没有 model 中没有考虑 cast shadow，这是另一个尚存有疑问的地方，illuminant model 与 texture model 的综合效果不是已经足够对 cast shadow 进行表达了吗？

***小节：这篇应该可以算上是近期阅读 hand model 最难的一篇文章，给出了大量的 rigorous mathematical formulation，考虑的问题比较精细，涉及到边缘部分连续性的问题，全篇的精髓应该是提出的 occlusion forces，巧妙且效果显著；另一个比较大的收是 texture 和 shading，对 texture model 和 illuminant model 有了基本的了解，并且对其在 hand tracking 中的作用有了一定的体会。***
