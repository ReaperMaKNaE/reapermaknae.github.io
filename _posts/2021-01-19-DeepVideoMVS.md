---
layout : post
title : "DeepVideoMVS"
date : 2021-01-18 +1631
description : DeepVideoMVS, Multi-View Stereo on Video with Recurrent Spatio-Temporal Fusion 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### DeepVideoMVS: Multi-View Stereo on Video with Recurrent Spatio-Temporal Fusion, CVPR 2020



SUMMARY: Video Frame을 Input으로 받아서 Depth regression과 refinement를 진행하는 network. 이 때 pose는 알고 있다고 가정한다. 이를 응용하여 이전의 frame에서 뽑힌 feature와 현재의 frame에서 뽑힌 feature를 합쳐 plane sweep warp를 진행하고, 이들로 cost volume을 만든 다음, 해당 cost volume을 encode한다. 이후 ConvLSTM Cell을 지나면서, output을 decoder로 보내고, 얻어진 depth를 다음 ConvLSTM Cell의 input에 warp해서 추가하는 방식으로 얻어낸다.



### Abstract

 우리는 posed video stream들에서 online multi-view depth prediction을 접근하는 것을 propose한다. 우리의 backbone은 이미지쌍에서 cost volume을 베이스로 한, real-time의 lightweight encoder-decoder이다. 우리는 과거의 정보를 압축해놓은 임의의 양을 가진 bottleneck layer에 ConvLSTM을 확장시켰다. time step 사이에서 viewpoint가 변화하는 것을 계산하는 것으로 cell의 hidden state로 propagate하는 것에 novelty가 있다. 주어진 time step에 대해서 우리는 이전의 depth prediction을 이용하여 현재의 camera plane에 이전의 hidden state를 warp한다. 우리가 추가한 것들은 실제 성능 향상에 비해서 computation time이나 memory consumption은 조금만 증가했다. 결과적으로 우리는 SOTA를 찍었다.



### Introduction

 환경에 대해서 dense한 3D information을 얻어내는 것은 중요하다(예를 들면 autonomous vehicle, mixed reality, 3D modeling, industrial control 등) LiDAR와 같은 active depth sensing과 비교했을 때, 다른 센서들이 energy와 cost efficient, size와 작동 환경 등에서 얻게되는 이점이 많다.

 여튼 이러한 점들을 mobile에 적용을 하는데, 이런 것들이 여러 어려움에 직면해있는 상황임. 우리가 받는 것들은 calibrated camera와 pose를 알고 있다고한 다음, dense depth recovery에 focus를 할 것이다. pose information과 같은 경우는 visual inertial odometry technique이나 mixed reality headset 같은 것에서 얻을 수 있다. single image depth에 반해서, camera pose를 알고 있다는 것은 triangulation based metric의 reconstruction에서 연산을 가능하게 한다. 그리고 최근에는 on-device로 real-time이 가능한 구조를 연구하고 있다. 우리는 video의 이점을 활용하는 것을 목표로 했다.

 우리는 이번 work에서 현재 존재하는 다양한 MVS method의 framework를 소개할거다. 우리는 일단 ConvLSTM(Convolutional Long Short-Term Memory)와 hidden state propagation을 쓰는데, 이는 latent space에서 information flow와 같은 것을 달성하기 위해 사용된다. 우리의 접근은 Gaussian Process로 약하게 묶인 cost volume encoding과, consistent depth map을 일시적으로 달성하기 위해 ConvLSTM으로 묶인 latent encoding of image feature와 sparse depth map cue에 영향을 받았다. 하지만 우리는 geometry와 explicitly account에 대해서 저울질을 해야했다.(when propagating the latent encoding -> 둘 중에서 어디에 weight를 더 주어야 하는가의 문제를 의미)

 우리의 key contribution은 다음과 같다.

- 우리는 real-time과 memory efficient를 위한 2D conv에 기반을 둔 작고 cost volume base의 stereo depth estimation를 propose한다. image feature에서 뽑아낸 것들을 cost volume으로 합치면서 network가 더 robust해지게 되었다.
- 우리는 우리의 모델을 ConvLSTM과 spatio-temporal information flow를 가능하도록 하는 학습/추론 전략을 추가했다.
- ScanNet, 7-Scenes, TUM RGB-D, ICL-NUIM에서 SOTA찍음.



### Related Work

 3D 공간상에 나타내는 voxel discretization, point-based representation, TSDF에 비해서 depth map은 다재다능하고 3D recon뿐 아니라 다른 task도 수행할 수 있다. 더욱이, 이러한 simple 2D representation은 real-time information을 전달하고 memory를 효과적으로 사용할 수 있다.

__Depth Map Prediction with Learned MVS.__

 대부분의 MVS method에서는 traditional plane-sweeping이 사용되고 있음. 대표적인 예가 MVSNet과 DPSNet. 근데 이런 방법 말고 MVDepthNet등과 같은 경우는 image의 특징이나 RGB value를 이용해서 얻어낸 cost들로 3D recon을 함. 이러한 접근은 속도가 빨라서 real-time에 적합함. 하지만 이는 color나 information을 decimate하는 경향이 있음. real-time의 성능과 기존의 원리를 바탕으로, 우리는 lightweight network를 소개함. DELTAS라고 dense depth map을 뽑아낼 때 interest point를 기반으로 하는 network도 있는데, 조금 다르긴 함.

__Depth Estimation from Video__

 Video에서 일련의 image를 받는 것은 camera pose에 대해 unknown이더라도 modeling을 가능하게 했다. 몇몇 방법들이 ConvLSTM cell을 이용한 방법이 효과적임을 보여주고 있다.

 몇몇 논문들이 ConvLSTM을 이용해서 temporal consistency와 visual pleasant result를 달성했음에도, geometric correctness는 제대로 얻지 못했다.

 최근, video를 input으로 주는 경우도 있다. GP-MVS가 그 예이다. 또 다른 예로는 Neural RGBD가 있다.



### Method

 online multi-view stereo system에서, supervised learning은 아래와 같다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-1.PNG)

 여기에서 task는 depth map D hat _ t를 예측하는 learnable parameter \theta의 집합인 predictor f_\theta 를 예측하는 것이다. 

__3.1 Pair Network__

 이번 section에서는 MVDepthNet에서 사용했던 방법을 수정한 pair netowkr를 소개한다. model function인 f_theta는 \delta를 1로 assign하여 얻어지게 된다.(Equation 1 참조) network는 다섯 개의 main part로 구성되어 있다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-2.PNG)

 __Feature Extraction with FPN.__ 

 real-time operation goal에 맞게 low-latency, lightweightness 때문에 선택된 MnasNet을 기반으로 feature extraction이 이루어진다. object detection task에서 훌륭한 성능을 보여준 FPN은 receptive field와 spatial resolution을 recover하는 것에 의미가 있다. 우린 feature map을 절반 사이즈로 줄이고, skip connection도 사용했다.

__Cost Volume Construction.__

 ![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-3.PNG)

 많이 사용하는 plane-sweep stereo를 이용해서 cost volume을 만들고, inverse depth map을 사용했다.

__Cost Volume Encoder-Decoder.__

 encoder decoder architecture의 중요한 목적은 U-Net style architecture와 함께 cost volume을 regularize하는 것이다. encoder는 high-level, global scene의 정보를 종합하는 역할을 하고, decoder는 upsample을 하며 fine resolution으로 복구하는 것이다.(물론 이 때 inverse depth map을 사용하고 skip connection을 사용한다 - guidance 개념)

__Depth Regression and Refinement.__

  ![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-4.PNG)

 각 decoder를 거치고 나온 output을 Y라 하면, 이를 3*3 convolution filter w와 sigmoid activation \sigma를 지나도록 만들었다. 그리고 이를 통해 depth regress는 위와 같은 과정을 지나게 된다.

__Loss function.__

 inverse depth map에 대해서 average L1 error를 축적했음.

__3.2 Spatio-temporal Fusion__

 H와 C를 각각 hidden state와 ceel state라고 하고, X를 bottleneck에서의 output이라고 한다면, 우리의 cell logic은 다음과 같다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-5.PNG)

 '*' 은 convolution을 말하고, dot은 Hadamard product, \sigma는 sigmoid activation, w는 learned convolution filter를 말한다. 우린 tanh 보다는 ELU activation이 더 좋은 것을 확인했다. 또한 learnable parameter 없이 Layer Normalization은 통제불가능하게 자라는 value를 막아주고 cell state에서 channel마다 unit variance와 zero mean을 강요해서 model을 안정화시키게 해준다. 여기서 H와 X는 dimension이 같아서, 그 어떤 modification도 필요하지 않는다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-8.PNG)

__Naive Fusion.__

 S는 encoder에서 decoder로의 skip connection을 의미한다고 하면, fusion의 simple model은 다음과 같이 적혀진다.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-6.PNG)

 equation 5를 보면, X _ t와 H _ t-1은 서로 상호적인 관계에 있는데, equation 6를 보면 이들이 가진 정보를 합치게 되는 것을 볼 수 있다. 이렇게 만들어진 정보는 pose-induced image motion을 가지게 된다.(이 때 pose induced image motion은 occlusion이나 rapid rotation과 같은 큰 large disparity에도 도전적인 문제이다.)

__Proposed Fusion.__

 우리는 과거와 현재의 camera pose에 접근하고 current recon을 proxy로 사용하기 위해, hidden representation H _ t-1을 현재의 viewpoint에 warp했다. 이는 H hat _ t-1을 얻기 위한 것이다. 이러한 warping은 forward or inverse mapping이 모두 가능하다. forward mapping의 경우는 processing time과 non-trivial visibility handling과 differentiable rendering을 필요로한다. bilinear interpolation을 사용하면, 후자의 경우는 fully differentiable하고 lightweight operation이며, 전체적인 runtime을 줄일 수 있게 된다. 우리는 sampling location을 계산하기 위해, forward pass를 시작하기 전에, current time step의 depth map을 estimate한다. 그것이 끝나고 나면, 우리는 현재 camera plane에 작은 point cloud를 project한다.(이는 previous depth prediction을 estimate하기 위함인데, occlusion을 다루기도 한다.) 그리고 우리는 hidden representation을 sample한다. (여기서 물결표시는 warp의 표시인듯 하다) 이후 encoding을 하고, ConvLSTM cell에 알맞게 encoding을 한다. 이러한 것들을 적용한 모델은 다음과 같다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-7.PNG)

 

### Experiments

__4.1 Dataset__

__4.2 Frame Selection__

 reference image를 설정하는 것은 중요한데, 우리는 다른 논문에서 나온 방법을 그대로 썼음.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-9.PNG)

 위 식에서 T _ rel은 two camera의 relative pose임. indoor dataset의 경우 maximum pose distance는 0.35 +- 0.05이고, translation은 10cm +- 5cm임을 확인했음. 우리는 online system에서 평가를 하기 위해 buffer를 주기적으로 update해야 했는데, 이 때 각 T _ rel의 penalty는 아래와 같음.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-10.PNG)

__4.3 Comparison with Existing Methods__

__Quantitative Evaluation.__

__Qualitative Evaluation.__

__Runtime and Memory.__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-12.PNG)

__4.4 Ablation Studies__

__Propagating the Hidden State by Warping.__

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-11.PNG)

__Number of Measurement Frames.__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-13.PNG)

__Frame Selection.__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-14.PNG)



### Conclusion



### Supplementary Material for DeepVideoMVS: Multi-View Stereo on Video with Recurrent Spatio-Temporal Fusion

__A. Training Details__

__Pair Network Training__

__Fusion Network Training__

__Data Augmentation.__

__B. Evaluation Metrics__

__C. Inference Time and Memory Consumption__

__D. Evaluation Set Details__

__E. Additional Ablation Studies__

 이는 해당 논문 참조. ConvLSTM cell에서 어떻게 조금씩 바꿨는지를 나타내주고있음.

__Activation and Normalization in the ConvLSTM Cell.__

__Skip Connections from Feature Pyramid to Encoder.__

__Finetuning the Cell.__

__F. Additional Qualitative Results and Supplementary Video__



### References

Düzçeker, Arda, et al. "DeepVideoMVS: Multi-View Stereo on Video with Recurrent Spatio-Temporal Fusion." *arXiv preprint arXiv:2012.02177* (2020).

