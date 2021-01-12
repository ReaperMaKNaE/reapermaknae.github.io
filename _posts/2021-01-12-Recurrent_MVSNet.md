---
layout : post
title : "R-MVSNet"
date : 2021-01-12 +2122
description : Recurrent MVSNet for High-resolution Multi-view Stereo Depth Inference 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Recurrent MVSNet for High-resolution Multi-view Stereo Depth Inference, CVPR 2019



### Abstract

  Deep learning이 MVS(Multi View Stereo)에서 굉장한 성능을 보여준 것은 사실이나, High-resolution에서는 memory-consuming이 상당하다는 점이 문제가 되고 있다. 그래서 우리는 recurrent neural network를 base로 한 scalable MVS framework를 소개한다. 전체적인 3D cost volume을 regularizing하는 것이 아니라, 2D cost maps을 Gated Recurrent Unit(GRU)를 통해 depth direction으로 regularize하게 된다.



### Introduction

  MVS base 학습의 key advantage 중 하나는 cost volume regularization이다. 하지만, 이러한 것들은 상당히 무거운 결과를 수반한다.

 이러한 문제를 인식하고 3D 학습에 임한 논문들이 있다. OctNet, O-CNN, SurfaceNet, DeepMVS 등등이 다양한 시도를 했다.

 이러한 헛짓.... 이라고 하기엔 좀 그렇지만 아무튼 이러한 memory-consuming과의 전쟁을 끝내기 위해 우리는 scalable한 MVS framework을 propose한다. MVSNet architecture 베이스이지만, 3D CNN이 아닌 GRU를 이용해서 cost volume을 regularize한다. GRU를 이용한 일련의 과정에서, online memory는 quadratic model에서 cubic으로 줄이게 된다. 결과적으로 R-MVSNet은 제한이 없는 depth-wise resolution에서도 고해상도 3D reconstruction을 수행할 수 있다.

 dataset은 DTU, Tanks and Temples, ETH3D를 썼다.



### Related Work

__Learning-based MVS Reconstruction__

 최근 여러 논문들이 다양한 방법들을 이용해서 3D recon을 해오고 있음. SurfaceNet, DeepMVS, LSM, MVSNet 등등.

__Scalable MVS Reconstruction__

이전부터 3D Recon에서 지속된 문제 중 하나는 memory consuming problem임. 이러한 문제를 SGM등으로 해결하려는 전통적인 움직임도 있었고, 최근에는 OctNet이나 O-CNN등이 octree structure를 3D CNN에 접목해서 해결하려는 움직임도 있었음. 그러나 여전히 voxel 제한이 있다는 것이 문제임.

 반면에 전통적인 MVS algorithm들은 cost volume을 regularize하는 방법들이 많았음. plane sweep이라던지, winner take all을 이용한 2D spatial cost aggregation등. 우린 이러한 움직임을 따라, convolutional GRU를 이용한 cost volume regularize를 propose함. GRU는 RNN architecture를 응용해서 3D volume processing에 적용을 한 것임. 이렇게 적용된 convolutional GRU는 depth direction으로 context information을 모음. 이러한 과정은 3D CNN을 regularization하는 결과를 달성하게 됨.



### Network Architecture

 우리가 이번에 한 R-MVSNet은 기존 MVSNet의 확장판 수준임. 3.1에서 간단하게 MVSNet Review하고, 3.2에서 recurrent regularization 소개한 다음, loss formulation을 section 3.3에서 소개할 계획임.

__3.1 Review of MVSNet__

 reference image가 주어지고, 그 주변의 이미지들이 N-1개 주어지게 되면, feature map이 추출되어서 2D network로 모이고, 이렇게 모인 2D image feature들은 reference camera frustum에 differentiable homography로 warp되어 3D feature volume을 만드는데 사용이 됨. 이 때 N개의 임의의 view에서 온 image input을 다루기 위해서, variance based cost volume을 만듬. 여기서 이걸 regularize하는 데에 우리는 multi-scale 3D CNN을 사용했었고, regress할 때에는 soft argmin을 사용했었음. deep image feature는 feature extraction 중 scale이 점점 작아졌었음.

 기억이 나지 않을 수 있으니(사실 기억 안날테니), MVSNet 논문에서 그림을 따오면 아래와 같음.

 ![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210110-29.PNG)

비교를 하기 위해 바로 R-MVSNet architecture를 추가함.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-3.PNG)

__3.2 Recurrent Regularization__

__Sequential Processing__

 cost volume C를 regularize하는 대안으로는, depth direction을 따라서 volume을 짜는 것이다.(사실 왜 이게 되는지는 잘 모르겠다.) 가장 simple한 방법은 winner take all plane sweep이다. 이런 경우는 noise로부터 뭔가 고통을 받을 일이 없다.(아래 그림에서 (a))

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-2.PNG)

 이걸 좀 improve하기 위해, cost aggregation method가 matching cost를 서로 다른 depth에서 filter하기 시작했다.(위 그림에서 (b)) 우리가 사용할 방법은 (c)인데, uni-directional context information뿐 아니라 spatial한 정보도 모을 수 있다.

__Convolutional GRU__

 ![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-4.PNG)

 약속을 같이 한번 보자. C는 cost volume을 의미하고, t는 step을 말한다. 여기서 C_r에서 r은 regularized를 의미한다. U(t)는 update gate map인데, (t-1) step까지 regularize한 C_r(t-1)에다가 현재 step까지 update된 C_u(t)와의 비중을 어디에 더 둘 것인가를 의미하는 것인데, 이 때 현재까지 update된 cost volume은 다음과 같다.

 ![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-5.PNG)

 위에서 R(t)는 reset gate map으로 현재의 current update에 C_r(t-1)이 얼마나 영향을 끼치는 가를 의미한다. \sigma_c(dot)은 element-wise sigmoid function을 의미한다.

 위에서 U(t)와 R(t)는 다음과 같은 식을 가지게 된다.

 ![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-6.PNG)

 해당 식에서 W와 b는 learned parameter를 의미한다. 여기서 \sigma_g(dot)은 update를 위한 hyperbolic tangent를 의미한다.

 

 위 내용을 조금 정리하면... Reset Gate R(t)는 과거의 정보를 얼마나 reset 하느냐 를 의미하고, Update gate는 과거와 현재 정보 중에서 어떤 놈을 더 비중두느냐를 결정하게 된다. 자세한 내용은 조경현 교수님께서 작성하신 논문을 참조하자. LSTM에 대한 지식도 쌓아두고 가는 것이 좋다.

Cho, Kyunghyun, et al. "Learning phrase representations using RNN encoder-decoder for statistical machine translation." *arXiv preprint arXiv:1406.1078* (2014).



__Stacked GRU__

stacked GRU는 model을 훨씬 deep하게 만든다. 우리는 여기서 3-layer stacked GRU를 사용했다. 2D convolutional layer로 32 channel cost map C(t)를 만들고, 이걸 각각 16 channel, 4 channel, 1 channel을 가진 GRU를 통과시키고 이렇게 만들어진 결과물을 softmax하여 최종적으로 probability volume P를 만들게 된다.

__3.3 Training Loss__

 대부분의 deep stereo 혹은 MVS network들은 soft argmin을 사용했지만, 이 soft argmin의 경우엔 depth range가 unifrom distribution인 경우에만 유효하다. 우린 wide한 depth range를 효과적으로 handle하기 위해 inverse depth를 써서 softmax를 쓰게 되었다. 그래서 결과적으로 구하게 되는 Loss는 다음과 같다.

 ![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-7.PNG)

 p는 spatial image coordinate이고, P(i,p)는 probability volume P에서의 voxel을 의미한다. Q는 binary로 찬 GT volume이다.(one-hot encoding된 녀석임). 그래서 Q(i,p)는 P(i,p)에 해당하는 GT이다.

 여기서 좀 걱정이 있는데, section 4.2에서 조금 더 설명할 예정.(refinement algorithm)

 probability volume은 winner-take-all을 이용해서 regularized cost map에서 값을 받아온다(??)



### Reconstruction Pipeline

 이전 section에서 propose한 network은 depth map per-view를 만든다. 이번 pipeline은 learning하는 part가 아님.

__4.1 Preprocessing__

 R-MVSNet을 평가하기 위해서는 아래 3가지가 필요하다.

- reference image에 대한 source image들
- reference view에 대한 depth range
- inverse depth를 사용한 sampling depth value를 위한 depth sample number D

 source image 선택으로는, MVSNet에서 했던 sparse point cloud의 baseline angle을 이용한 piece-wise gaussian function을 사용한 image pair를 썼다. depth-range는 COLMAP을 썼고, D를 결정하는 것은 supplementary material에서 다룰 예정.(근데 없음?? 어디있음??)

__4.2 Variational Depth Map Refinement__

 section 3.3에서 설명한 것 처럼, depth map은 은 winner-take-all selection을 통해 regularized cost volume에서 값을 받아온다. soft argmin과 비교했을 때 argmax의 winner take all은 sub-pixel accuracy의 depth estimation을 할 수 없다. 

 ![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-8.PNG)

 위 그림에서 (g)와 (h)의 계단현상을 완화시키기 위해, multi-view photo-consistency를 enforcing하여 small depth range에도 depth map이 refine될 수 있도록 했다.

 reference image I_1과 i번째 image의 reprojection error는 다음과 같다.

 ![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-9.PNG)

 photo consistency C(dot)을 측정하기 위해 zero-mean normalized cross correlation(ZNCC)를 선택했고, p와 neighbors p' 간의 smoothness를 위해 bilateral squared depth difference S(dot)을 썼다.

 우린 위 과정을 각 Depth map에 대해서 reprojection error를 모든 pixel, 모든 source image에 대해 적용했다.

 ![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-10.PNG)

 이러한 방법은 DeepMVS에서 진행한 DenseCRF와 quadratic interpolation과 유사하다.

__4.3 Filtering and Fusion__

 MVS를 기반으로 한 다른 depth map과 마찬가지로, 우린 R-MVSNet의 depth map을 single 3D point cloud에 filer/fuse 했다. depth map filtering 과정에는 photo-metric / geometric consistency가 고려된다. Figure 3의 (f)에 probability Map이 나오는데, 우린 여기서 0.3보다 작은 pixel들을 filter했다. geometric consistency로는 MVSNet에서 했던 geometric criteria처럼 최소한 3개의 view에서는 관찰이 되어야 채택하는 것으로 결정했다.

 depth map fusion에 있어서는 depth map의 퀄리티와 3D point cloud를 생성하는 것을 강화하기 위해 mean average fusion(MVSNet 참조)뿐 아니라 visibility-based depth map fusion을 진행했다.

 __visibility-based depth map fusion:__ P. Merrell, A. Akbarzadeh, L. Wang, P. Mordohai, J.-M. Frahm, R. Yang, D. Nister, and M. Pollefeys. Real-time ´ visibility-based fusion of depth maps. International Conference on Computer Vision (ICCV), 2007.



### Experiments

__5.1 Implementation__

__Training__

__Testing__

__5.2 Benchmarks__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-11.PNG)

__DTU Dataset__

__Tanks and Temples Benchmark__

__ETH3D Benchmark__

__5.3 Scalability__

__Wide-range Depth Reconstructions__

__High-resolution Depth Reconstructions__

__5.4 Ablation Studies__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-12.PNG)

__5.4.1 Networks__

__2D CNNs + 3D CNNs__

__2D CNNs + GRU__

__2D CNNs + Spatial__

__2D CNNs + Winner-Take-All__

__ZNCC(??) + Winner-Take-All__

__5.4.2 Post-processing__

__Without Variational Refinement__

__Without Photo-metric Filtering__

__Without Geo-metric Filtering__

__Without Depth Map Fusion__

__5.5 Discussion__

__Running Time__

__Generalization__

__Limitation on Image Resolution__



### Conclusions



### Acknowledgement



### References

Yao, Yao, et al. "Recurrent mvsnet for high-resolution multi-view stereo depth inference." *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition*. 2019.

Cho, Kyunghyun, et al. "Learning phrase representations using RNN encoder-decoder for statistical machine translation." *arXiv preprint arXiv:1406.1078* (2014).

