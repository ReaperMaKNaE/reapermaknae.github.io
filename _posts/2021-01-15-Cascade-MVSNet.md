---
layout : post
title : "Cascade-MVSNet"
date : 2021-01-15 +1344
description : Cascade Cost Volume for High-Resolution Multi-View Stereo and Stereo Matching 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Cascade Cost Volume for High-Resolution Multi-View Stereo and Stereo Matching, CVPR 2020



요약: feature pyramid를 응용하여 cascade cost volume을 만들어 성능 향상을 보여준 network.

 feature pyramid를 MVS에 적용한 사례는 CVP-MVSNet도 있는데, 다른 점이라면... MVSNet에서 Cost volume을 계산할 때 pyramid를 이용해서 계산 했으면, 얘는 cost volume에서 계산하면 시간도 오래 걸리니, 그냥 feature extraction에서 pyramid 식으로 뽑고, 각 pyramid에서 뽑은 feature로 cost volume을 따로 만들어서 coarse-to-fine하는데만 쓰자인 거고, CVP-MVSNet은 일단 2장의 image set에서만 feature를 뽑고, 여기서 작은 cost volume을 a, 큰 애를 A라고 하면, a에서 만들어진 결과물을 upsampling해서 A를 보완하고, A는 지속적으로 iterative하게 input image에 따라 값을 update하는 방식을 사용한다.

 프로세스만 보자면 CVP-MVSNet이 조금은 연산에 있어서 더 많은 parameter를 요구하는 것 같다.



### Abstract

 deep MVS와 stereo matching의 경우 일반적으로 depth와 disparity를 regularize하고 regress하기 위해 3D cost volume을 사용했다. 이러한 방법은 memory나 시간으로 인해서 high-resolution output을 내는 것에 문제가 있었다. 따라서, 이번 논문에서 우리는 memory, time efficient한 cost volume을 제안한다. 첫째로, 제안된 cost volume은 geometry와 context가 포함된 feature pyramid 위에 만들어진다. 그리고, 우리는 이전 단계에서의 예측으로 각 단계의 depth(혹은 disparity)를 narrow한다. 점점 커지는 cost volume의 resolution과 adaptive adjustment로 output은 coarse to fine 하게 복구된다.

 우린 이 cascade cost volume을 MVSNet에 적용해서 35.6%의 성능 향상을 보였고, GPU memory에서 거의 반정도를 절약했고, runtime도 절약했다. 추가로 Tanks and Temples benchmark에서는 first rank에 올랐다.



### Introduction

 이전의 coarse-to-fine learning-based stereo 접근 방법에 영감을 받아 우리는 novel한 cascade 형태의 3D cost volume을 propose한다. 우린 일반적인 MVS에서 사용되는 multi-scale feature에서 feature pyramid를 뽑아 만든다. coarse-to-fine 방법에서 cost volume은 sparsely sampled된 depth hypotheses에서 2D feature를 뽑아 scale segmentic 2D feature를 뽑아서 상대적으로 low resolution을 가지게 되는데, 이를 다음 stage에서 더 finer한 2D feature를 base로 한 feature들과 adjust하여 finer cost volume을 만든다. 이러한 방법은 memory resource와 연산ㅅ에 있어서 의미있는 영역이다. 이런 방법에서 우리의 cascade structure는 연산 시간과 memory 문제를 해결할 수 있게 되었다. 

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-24.PNG)

 추가로 우린 이러한 방법을 MVS와 stereo matching 두 가지 모두에 적용하여 validate했다. MVS의 경우는 DTU dataset을 이용하였고, MVSNet과 비교를 하였으며, Tanks and Temples의 경우에는 benchmark를 찍었다. stereo matching에 있어서 우리의 방법은 GwcNet의 end-point-error와 GPU memory를 각각 15.2%, 36.9%씩 줄였다.



### Related Work

__Stereo Matching__

__Multi-View Stereo__

__High-Resolution Output in Stereo and MVS__



### Methodology

 ![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-25.PNG)

__3.1 Cost Volume Formulation__

 3D cost volume은 input들이 얼마나 match가 되는지에 대해서 그 결과를 내 주는 것이다. 3D cost volume을 만드는 것에는 3가지 스텝이 존재한다.

- the discrete hypothesis depth가 결정되어야함
- 여기서 2D feature를 warp함.(각 view에서, 각 depth hypothesis plane에서)
- 그리고 얘내들을 합침.

 기존의 전통적인 방식에서 사용하던 pixelwise cost aggregation 방법은 textureless하거나 reflective  surface에 있어서는 별 효과적인 경우를 못 보였는데, 위와 같이 만들어진 3D CNN은 효과적인 결과를 확인할 수 있었음.

__3D Cost Volumes in Multi-View Stereo__

 MVSNet은 fronto-parallel plane을 서로 다른 depth, hypothesis plane에서 사용하는 것을 희망했고, depth range는 일반적으로 sparse reconstruction에 의해 결정되었다. 

 ![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-26.PNG)

 coordinate mapping은 위와 같은 homography에 의해 결정된다. 이렇게 모인 homography에서 feature map을 뽑고, 하나의 cost volume으로 만든다. 임의의 input feature에 적응하기 위해, variance-based cost metric이 적용되었다.

__3D Cost Volumes in Stereo Matching__

 PSMNet에서는 disparity level을 hypothesis plane과 range of disparity로 사용했다.

 ![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-27.PNG)

 여기서 해당 disparity만큼을 보상해주기 위해 다른 image를 warp하여 reference image로 project했다. 여기에 GCNet, PSMNet, DispNetC 등이 참가했다. 추가로 GwcNet은 이러한 feature를 group으로 나누어서 계산했다.

__3.2 Cascade Cost Volume__

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-28.PNG)

 위 그림은 standard cost volume을 나타내어 준다. depth hypothesis D를 늘릴수록 cost volume의 크기는 늘어나게 되고, 더 좋은 결과를 낼 수 있게 되나 memory와 run-time의 문제를 동시에 가져오게 된다. 따라서 R-MVSNet과 일반 MVSNet의 경우 한계가 좀 낮게 정해져있었다. 이러한 문제를 해결하기위해 우리는 cascade cost volume 형태를 제안한다.

__Hypothesis Range__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-29.PNG)

  Depth range가 첫번째 stage에 결정이 되었다고 하자.(R1으로). 우린 다음 stage에서, 이전 stage에서 예측한 결과를 바탕으로 이걸 얇게 할 수 있다. 결과적으로, R_(k+1)=R _k*w_k 가 된다. 여기서 R_k는 kth stage에서의 hypothesis range이고, w_k<1 은 hypothesis range를 줄이는 reducing factor이다.

__Hypothesis Plane Interval__

 depth interval을 I1이라고 하자. 그런데 초기에 이 I_1을 작게 만들어서 dense하게 만드는 것은 coarse depth estimation에는 조금 과분하다. 그래서 이러한 dense hypothesis plane은 나중에 output을 강화할 때 쓰이게 된다.
 위와 같이, I_(k+1) = I_k * p_k라 하고, I_k는 k stage의 hypothesis plane interval이고, p_k<1은 hypothesis plane interval의 reducing factor이다.

__Number of Hypothesis Planes__

 위 과정을 거치고 나면, hypothesis plane의 수는 R_k / I_k 이다. spatial volume이 fixed라면, 더 큰 크기의 D_k는 더 많은 hypothesis plane을 만들고 더 좋은 정확도를 만들지만 memory와 run-time에서 문제를 일으키게 된다. cascade 방식에서 우린 효과적으로 이 hypothesis plane의 수를 줄일 수 있다.

__Spatial Resolution__

 Feature pyramid network를 따라서, 우리는 cost volume의 resolution을 두배씩 올렸다. N을 cascade cost volume의 total stage number라고 하면, k번째 cost volume의 resolution은 다음과 같다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-30.PNG)

__Warping Operation__

 cascade cost volume 형태를 MVS에 적용하기 위해, 우리는 MVS에서 쓰던 homography warping function을 다음과 같이 썼다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-31.PNG)

 stereo matching과 유사하게, 식 (2)를 cascade cost volume에 맞게 변경했다. 

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-32.PNG)

__3.3 Feature Pyramid__

 high resolution depth map을 얻기 위해서 이전의 work들은 상대적으로 low-resolution depth map을 만들고 이것들을 upsample하여 refine하였다. standard cost volume의 경우에는 top level feature map을 가지고 있었지만, low-level finer representation을 하기에는 부족했다. 우리가 참고한 feature pyramid를 다른 network에 적용해서 성능향상을 볼 수 있었다. MVSNet에서 적용한 경우는, feature pyramid network의 feature map P1, P2, P3에서 3가지 cost volume을 뽑아내었고, 각각은 1/16, 1/4, 1의 spatial resolution을 가지고 있다.

__3.4 Loss function__

 N stage의 cascade cost volume은 N-1개의 output과 final prediction을 내뱉는다. 해당 loss는 아래와 같다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-33.PNG)



### Experiments

__4.1 Multi-view stereo__

__Datasets__

__Implementation__

__Benchmark Performance__

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-34.PNG)

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-35.PNG)

__4.2 Stereo Matching__

사실 여기는 큰 관심 없으므로 pass.

__Datasets__

__Implementation__

__Benchmark Performance__

__4.3 Ablation Study__

__Cascade Stage Number__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-36.PNG)

__Spatial Resolution__

__Feature Pyramid__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-37.PNG)

__Parameter Sharing in Cost Volume Regularization__

__4.4 Run-time and GPU Memory__



### Conclusion



### Acknowledgements



### References

Gu, Xiaodong, et al. "Cascade cost volume for high-resolution multi-view stereo and stereo matching." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.

