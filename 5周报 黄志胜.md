#  2020/10/4 周报
paper学习，这周在看《Learning to Estimate 3D Hand Pose from Single RGB Images》，《Hand Keypoint Detection in Single Images using Multiview Bootstrapping》

代码学习，[yolo_v1_pytorch](https://github.com/motokimura/yolo_v1_pytorch)已经全部读完，感觉在loss函数这块收获比较大，并且在源代码基础上利用resnet作为backbone，网络结构做了一些调整，现在已经跑通，tensorboard返回的结果也是收敛的，但是原repo里的pytorch版本比较老，还可以进一步调整，同时数据集不全，需要进一步预处理。

docker学习，环境方面，由于四卡服务器不稳，八卡之前学长在打比赛，换服务器又要重新配置熟悉的terminal环境，同时公共的用户下，做一些配置不太方便，所以就去利用Ubuntu镜像用docker搭建了一个自己的机器学习环境，解决了docker使用gpu的问题，这里面碰到很多问题，也和大家有一些交流，也学到了很多,锻炼了动手能力，解决问题的能力,[镜像链接。](https://hub.docker.com/r/harrison523/hzs/tags)