---
layout : post
title : "RoutedFusion"
date : 2021-04-24 +0900
description : 새로운 Fusion Network로 더 높은 성능의 depth fusion결과를 보여준 Routed Fusion 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### RoutedFusion: Learning Real-time Depth Map Fusion, CVPR 2020



### Abstract

 Depth map의 효율적인 fusion은 3D recon method의 중요한 역할을 맡고 있다. 최근 이러한 것을 연구하고 있는 논문이 많이 나오는데, 보면 성능이 썩 그닥 좋지 않아 보인다. 그 이유로는 linear fusion을 사용하기 때문. 그래서 우리는 non-linear update를 이용한 더 나은 결과를 보여주는 neural network를 제안한다. 우리가 제안하는 network은 2D depth routing network와 3D depth fusion network로 이루어져있어, sensor noise나 outlier에 robust함을 보여준다. 이러한 것을 이용하여 우리는 SOTA를 찍었고 좋은 성능을 내게 되었다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-1.PNG)



### Introduction

 multiple camera viewpoint에서 3d fusion을 하는 것은 근래 3D recon의 작업에서 필수적인 process이다. depth fusion은 여러 방법이 있지만,  그 중에서 traditional한 것의 장단점이 무엇인지 일단 알아보자. 

 traditional tsdf fusion 방식의 장점은 일단 아래와 같다.

- update가 local로 이루어지고 일정한 시간에 일정한 만큼만 수행되기 때문에 memory efficient
- Online available
- Cost가 작고 parallelizable(?? 대충 여러개를 같이 돌릴 수 있단 말 같음. 여러 장의 사진을 한꺼번에 fusion)

 문제는 그 단점도 있어서, 살펴보면 아래와 같다.

- Zero-mean gaussian을 사용하는데, real-world에서 적용이 잘 안되는 거임.(에러가 많음)
- 두껍거나 얇은 model에 부적합함(linear update의 경우)
- 얇은 object일 수록 그 error가 더 심해짐
- linear fusion weight는 부적합함.(그냥 linear가 부적합하다는 것 같음)
- outlier가 커지는 것을 막을 수 없어서, pre-process로 좀 없애고 난 후 process 진행함
- parameter들의 trade-off 설정이 힘듬.

 등의 이야기를 다룬다.

 그래서 해당 논문의 contribution은,

- learning-based 방법으로 depth map fusion을 진행하는데, 구조가 간단하기 때문에 적은 data의 수로도 학습이 가능하고 overfitting의 경향이 없다고 함.
- scene-size에 무관하게 적용이 가능해서 real world여도 큰 문제가 없음
- multi-view setting등에서 오는 noise에 robust하고, surface thickening effect가 완화됨.



### Related Work

__Volumetric Depth map Fusion.__

__Surfel-based Fusion Methods.__

__Probabilistic Depth Map Fusion.__

__Learning-based Reconstruction Approaches.__

__Learned Scene Representations.__



### Method

 시작하기에 앞서, 간단하게 standard TSDF Fusion에 대해 이야기를 나누어보자.

__3.1. Review of Standard TSDF Fusion__

 Standard TSDF Fusion의 경우, 아래와 같은 식을 이용하여 이루어진다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-2.PNG)

 위 식에서 $V_t$ 는 discretized signed distance function을, $W_t$ 는 weight function을 말한다.(전체 scene에 대해서) $V_0$ 와 $W_0$ 의 경우는 zero-initialized 되어있으며, $v_t$ 와 $w_t$ 는 각각 updated 되는 signed distance와 weight이다.(다음 depth map인 $D_t$ 를 위한 value들)

 이러한 방법으로 fusion이 이루어졌으나, 그 computational cost가 너무 커서 다루기가 힘들다. 만약 truncation distance가 크다면, 작은 object들에 대한 thickening effect가 생기게 되고, 그 반대로 작다면 noise에 치명적인 결과를 낳게 된다. 따라서 저자는 이러한 factor들을 학습할 수 있는 hyper parameter로 둔 것이다.

__3.2. System Overview__

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-3.PNG)

 우리의 system은 2가지 network로 구성되어 있다. 첫번째는 depth routing network이고, 그 다음은 depth fusion network이다. 이 network들을 이용하여, 우리의 system은 위 그림처럼 4개의 단계를 거치게 된다.

 각 Step은 다음과 같다.

- Depth Routing: raw depth map을 받아서 refine하고, confidence map을 만든다.
- TSDF Extraction: routed depth value를 받은 다음 voxel grid와 weight를 뽑는다.(trilinear interpolation을 통해)
- Depth Fusion: 이전 processing step에서 구한 것들을 이용하여 local TSDF update인 $v_t^*$ 를 계산함.
- TSDF Update Integration: TSDF update를 이용하여 TSDF를 update함(식 1과 2를 통해)

 위 과정을 자세히 살펴보자. 일단 network의 구조를 보면 아래와 같다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-4.PNG)

__3.3 Depth Routing__

 구조에서 보듯 단순히 encoder-decoder 구조를 가지게 된다. 한가지 특이한 점으로는 normalization layer를 사용하지 않았다고 한다. 이를 사용하게 되면 depth쪽으로 bias가 잡히는 것 때문에, 제대로 된 학습이 안된다고 한다.

__3.4 TSDF Extraction__

 standard TSDF fusion에서는 독립적으로 view t의 ray에 대해 processing을 진행한 반면, 우리는 조금 더 넓은 neighborhood의 data를 기반으로 TSDF update를 계산했다. 이렇게 계산이 된 값들은 3D TSDF value와 2D input data로 나뉘게되는데, 이를 concat한 다음 network에 주게 된다. 

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-5.PNG)

 위 식을 이용해 다음 input으로 주게 되는데, confidence map의 경우 threshold보다 작으면 해당 값은 제외하는 것으로 진행했다.( $C_{thr}$ )

__3.5. Depth fusion__

 단순한 구조로 되어있다. 매우 간단하므로, Figure 3 참조.

__3.6. TSDF Update Integration__

 Depth fusion을 통해 update된 $v_t^*$ 를 받으면, 저장되어있던 같은 trilinear interpolation weight를 이용해서 이를 확장 시킨 다음 기존의 global coordinate frame인 $v_t$ 에 추가하게 된다.

__3.7. Loss function and Training Procedure__

 Network는 두 번 학습시키는데, 먼저 depth routing network를 학습시키고, 그 다음에 depth fusion network을 학습시켰다.

__Depth Routing Network__

 Depth routing network을 학습시킬 때에 사용했던 loss이다.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-6.PNG)

__Depth fusion network__

 Depth fusion network을 학습시킬 때 사용한 loss는 아래와 같다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-7.PNG)



### Experiments

 네트워크가 단순하기 때문에 적은 computational cost로도 학습이 가능했다.

__4.1. Implementation Details__

 RMSProp optimziation 사용 등등. ModelNet과 ShapeNet dataset, 3D Scene Data와 Street Sign Dataset, RGB-D Dataset 7 Scenes 등을 사용했다고 한다.

__Runtime__

W=320, H=240에서 depth routing network은 0.9ms, depth fusion network은 1.8ms이 걸렸다고 한다.

__4.2. Results__

 기존 TSDF, PSDF fusion, OccupancyNetworks, DeepSDF 와 비교했다고 한다.

__Evaluation Metrics.__

 MAD, MSE, Accuracy, IoU를 사용했다고 한다.

__4.3. Synthetic Data__

__ShapeNet__

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-8.PNG)

__ModelNet__

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-9.PNG)

 __4.4. Real-World Data__

__3D Scene Data__

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-10.PNG)

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-11.PNG)

__Street Sign Dataset__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-12.PNG)

__RGB-D Dataset 7-Scenes__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210424-13.PNG)

전반적으로 결과들을 보면 보다 더 깔끔한 것을 확인할 수 있다.(outlier를 제낀 덕인듯.)



### Conclusion



### Reference

Weder, Silvan, et al. "Routedfusion: Learning real-time depth map fusion." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.

