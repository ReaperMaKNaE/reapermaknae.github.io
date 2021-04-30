---
layout : post
title : "ViPDeepLab"
date : 2021-05-04 +0900
description : Panoptic DeepLab을 확장해 depth prediction에서 성공적인 결과를 낸 ViP-DeepLab 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### ViP-DeepLab: Learning Visual Perception with Depth-aware Video panoptic Segmentation



### Abstract

 이번 논문에서는 ViP-DeepLab이라는, vision에서 도전적인 inverse projection에 도전할 통합된 모델을 제안한다. 우리의 모델은 perspective image sequence가 instance-level semantic interpretation을 줄 때 까지 저장을 한다. 이를 해결하기 위해서는 spatial location, semnatic class, 그리고 consistent instance label등이 필요하다.

 ViP-DeepLab은 monocular depth estimation과 video panoptic segmentation을 수행하기 한다. 새로운 evaluation metric로 제안한다. 위와 같은 task들을 통해 우리는 SOTA를 찍었다.



### Introduction

 inverse projection의 문제점은, 해당하는 지점이 애매한 경우이다. 사람의 경우에는 쉽게 물체를 보고 어디있는지, 크기는 얼마인지 등을 파악할 수 있으나, machine에게 이러한 것을 부여하는 것은 꽤나 어려운 일이다.(inverse projection)

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-1.PNG)

 위 그림은 우리가 이번에 연구할 inverse projection problem의 예시에 대해 보여준다. 이러한 문제는 DVPS(Depth-aware Video panoptic Segmentation) 으로 되는데, 두 가지 task를 가지고 있다.

- Monocular depth estimation
- Video panoptic segmentation

  새로운 DVPS task를 위해, 우리는 두가지 데이터셋과 새로운 evaluation metric을 소개한다. DVPS dataset은 구하기가 힘들어서, Cityscapes-DVPS와 SemKITTI-DVPS 두 가지 데이터셋을 만들게 되었다. Cityscapes-DVPS는 Cityscapes에 depth annotation을 더한 Cityscapes-VPS 에 의해 구할 수 있었고, SemKITTI-DVPS는 SemanticKITTI에서 backproject를 통해 얻은 dataset이다. 추가적으로, 우리가 제안하는 metric인 DVPQ는 depth estimation과 video panoptic segmentation을 포함하게 된다. 이 모든 것을 포함해서 우리는 ViP-DeepLab을 제안한다. 우리는 차례로 ViP-DeepLab이 어떻게 두 task를 수행하는지 설명할 것이다.

 DVPS의 첫번째 task는 video panoptic segmentation이다. Panoptic segmentation은 semantic segmentation과 instance segmentation을 합친 것이다. 최근 VPSNet이 이에 대해서 좋은 성과를 내고 있었는데, 우리가 이를 이겼고, 추가적으로 MOTS(Multi-Object Tracking and Segmentation) 에서도 기존에 좋은 성능을 뽐내던 PointTrack을 압도했다. (MOTS는 panoptic segmentation이랑 비슷한데, 차랑 사람만 구분 하는 것이다.)

 DVPS의 2번째 task는 monocular depth estimation이다. 우리는 이를 Panoptic-Deeplab에 추가적인 head를 다는 것으로 성공적인 결과를 이끌어냈다. 우리는 이를 이용해 KITTI benchmark에서 우수한 성능을 냈고, DORN과 MPSD를 모두 이겼다.

 요약하자면, 우리의 contribution은 아래와 같다.

- 새로운 DVPS task를 제안하고, 이로 인해 video panoptic segmentation과 monocular depth estimation을 통합한다.
- 새로운 2가지의 DVPS dataset을 만들고, 새로운 metric인 DVPQ를 제안한다.
- ViP-DeepLab을 제안한다. 우리는 이를 이용해 Cityscapes-VPS, KITTI-MOTS pedestrian, KITTI monocular depth estimation에서 SOTA를 달성하였다.



### Related Work

__Panoptic Segmentation__

__Object Tracking__

__Monocular Depth Estimation__



### ViP-DeepLab

__3.1. Video Panoptic Segmentation__

__Rethinking Image and Video Panoptic Segmentation__

 Vidoe panoptic segmentation task에서, 각 instance는 image plane과 time axis에 의해 표현이 된다. (frame이 쌓이면서) time window k에 대해 주어진 clip이 $I^{t:t+k}$ 라고 한다면, true positive(TP) 는 IoU가 0.5 이상인 경우를 의미하고, 각 frame에서 잡힌 instance를 predicted인 경우엔 $\hat{U}$ , GT인 경우에는 $U$ 라고 하자. FP(False positive)와 FN(False negative)의 경우도 각각 정의가 된다. 각 class와 window size k에 따라서 TP, FP, FN을 모두 구하게 되면, Video Panoptic Quality인 VPQ는 아래와 같이 정의 된다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-2.PNG)

 만약 k가 1이라면, VPQ는 PQ와 같다. 우리의 방법은 PQ와 VPQ를 연결하는 것이다. image sequence에서, $P_t$ 는 panoptic prediction을, $Q_t$ 는 GT panoptic segmentation을 말한다. window size k에 대해서 VPQ는 $P_t, Q_t$ 에 의해 PQ와 다음과 같은 관계가 있다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-3.PNG)

 위 식에서 $||_{i=t}^{t+k-1}P_i$ 는 $P_i$ 를 t부터 t+k-1 까지 horizontal concat한 것을 말하고, $[P_t, Q_t]_{t=1}^T$  는 $P_t, Q_t$  쌍의 function input을 의미한다.

 위 식은, video panoptic segmentation이 image panoptic segmentation에서 image의 concat과 똑같은 형태라는 것을 볼 수 있다. 이러한 발견은 image panoptic segmentation을 video panoptic segmentation model로 변경하게 되는 모티브가 되었다.

__From Image to Video Panoptic Segmentation__

 Panoptic DeepLab은 아래와 같은 sub task를 해결하여 image panoptic segmentation을 이루었다.

- 'Thing'과 'Stuff'에 대한 semantic prediction
- 'thing'의 각 class에 대한 center prediction
- 각 object들에 대한 center regression

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-4.PNG)

 위 그림에서 좌측에 해당하는 부분이 Image Panoptic Segmentation을 의미한다. 여기서 'stuff' prediction과 'thing' prediction을 합치는 것 덕분에 panoptic deeplab은 최종적인 panoptic prediction을 할 수 있었다.

 우리의 방법은 panoptic DeepLab을 확장한 것이다. 이는 위 그림에서 우측에 보이는 부분인데, Panoptic-DeepLab과 마찬가지로 3개의 sub task를 가지게 된다. 바로 semantic segmentation, center prediction, center regression이다. inference동안 우리의 방법은 time t 와 time t+1 의 image를 horizontal concat을 해서 input으로 받은 다음, image t에 대해서만 center를 predict한다. 그리고 image t에 대해서 t와 t+1 image 모두 center regression을 하게 되는데, 이를 하면서 첫번째 frame과 두번째frame 사이의 모든 pixel이 대응이 되는 것을 찾는다. 즉, 두번째 frame(t+1 때의 frame)에만 존재하는 object는 여기서 무시되고, 다음 step인 t+1과 t+2의 frame 비교일 때 찾게 된다. 우리의 방법은 image panoptic segmentation을 concat한 것이기에, VPQ에 매우 강한 모습을 보여준다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-5.PNG)

 위 그림이 ViP-DeepLab의 architecutre이다. 회색 영역은 Panoptic-DeepLab을 의미한다. next-frame instance branch를 추가한 것이 끝인데, 이 branch가 합쳐지기 전에 backbone feature에서 t와 t+1을 concat을 해버린다. backbone feature는 concat 전에는 분리되어있어야 하기 때문에, next-frame instance branch는 large receptive field를 받아야했고, 이를 해결하기 위해 4개의 ASPP module을 사용했다. 우리는 이렇게 연결된 ASPP module을 Cascade-ASPP라고 한다. 이에 대한 자세한 구조는 Appendix를 참조.

__Stitching Video Panoptic predictions__

 우리의 방법은 두 consecutive frame의 일시적인 consistent를 이용해 panoptic prediction을 내뱉는다. 하지만 전체 sequence와의 prediction을 matching하기 위해, 우리는 stitch를 해야했다.(대충 서로 연결한다는 뜻. stitch의 사전 뜻은 _바느질하다_  이다.)

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-6.PNG)

 위 그림이 stitching method에 대한 에시이다. concat이 된 input을 가운데로 몰아넣는다 치고, $P_t$ 를 왼쪽 그림에서 온 left prediction, $R_t$ 를 우측 그림에서 온 right prediction이라고 하자. 우리의 목적은 $R_t$ 에서 온 propagate ID를 $P_{t+1}$  로 연결하는 것이다. 이러한 것은 mask IoU를 기반으로 한다. 만약 같은 class를 가지고 있다면 IoU를 비교해서 propagate하도록 했다. 어떤 ID도 matching이 되지 않은 경우에는 새로운 instance라고 판단한다.

__3.2. Monocular Depth Estimation__

 network overview 그림에서 확인할 수 있듯이, 우리는 depth 역시 파악하도록 했다. 이 때 식은 아래와 같다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-7.PNG)

 참고로 MaxDepth는 말 그대로 maximum depth로, 0m~88m까지 있는 KITTI를 기준으로 88로 설정이 된다.

 몇몇 error들이 있는데, 우리는 이들을 combine하려고 했다. d와 $\hat{d}$ 는 각각 GT, predicted depth를 의미한다. 따라서, depth prediction에 우리의 loss는 아래와 같다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-8.PNG)

__3.3. Depth-aware Video Panoptic Segmentation__

 inverse projection problem을 해결하면서, 우리는 DVPS라는 도전적인 task를 소개한다. 등등... 대충 DVPS 소개..

 이러한 DVPS에서의 error metric은 DVPQ로, Video Panoptic Quality에서 depth prediction을 추가한 형태이다. 구체적으로, P와 Q를 prediction과 GT라고 하자. $P_i^c, P_i^{id}, P_i^d$ 를 각각 class, instance ID, depth에 대한 prediction을 나타낸다고 하고(Prediction에 대해서), GT에서도 이에 해당한다고 하였을 때, DVPQ는 아래와 같다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-9.PNG)

 위 식에서 hat은 absolute relative를 의미한다. 위 식을 이용하면, VPQ와 depth inlier metric은 DVPQ의 special한 case로 볼 수 있게 된다. 우리는 dataset에 따라 위 식에서 $\lambda$ (depth threshold) 를 다르게 하여 비교해보았다. (참고로 NYU 같은 대중적으로 쓰이는 accuracy metric과는 $\delta = \lambda +1$ 의 관계가 있다.)  이렇게 된 이유는, 사실 1.25로 두는 것들에서 대부분 99%를 넘기기 때문에 새로 만들었다고 한다.(이건 그럴듯 하다.)



### Datasets

 새로운 task에서 평가하기 위해, 우리는 Cityscapes-DVPS와 SemKITTI-DVPS를 만들었다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-10.PNG)

__4.1. Cityscapes-DVPS__

__4.2. SemKITTI-DVPS__ 

(둘 다 dataset 설명)

 그런데 KITTI의 경우 아래와 같은 문제점이 있다. camera에는 보이지않는데 label이 되어있는 것들이 있었던 것. 이러한 것들을 없앴다.(우측 상단 그림에서 빨간색으로 표시된 부분) 그리고 또 다른 점은 뒷 배경에 의해 겹쳐지는 부분이다. 아래 그림에서 왼쪽 아래의 경우를 보면, 배경인 분홍색 background가 카메라인지 뭔지 아무튼 노란색으로 표시된 것의 영역을 약간씩 차지하고 있는 것을 보여준다. 이러한 것들을 모두 없앴다고 한다.

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-11.PNG)



### Experiments

__5.1. Depth-aware Video Panoptic Segmentation__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-12.PNG)

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-13.PNG)

__5.2. Video Panoptic Segmentation__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-14.PNG)

__5.3. Monocular Depth Estimation__

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-15.PNG)

__5.4. Multi-Object Tracking and Segmentation__

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-16.PNG)



### Conclusions



### Appendix

몇몇 내용들이 담겨져있는데, 여기서는 Cascade-ASPP만 다룬다.

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210504-17.PNG)



### Reference

Qiao, Siyuan, et al. "ViP-DeepLab: Learning Visual Perception with Depth-aware Video Panoptic Segmentation." *arXiv preprint arXiv:2012.05258* (2020).

