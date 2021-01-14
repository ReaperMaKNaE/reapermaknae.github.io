---
layout : post
title : "CVP-MVSNet"
date : 2021-01-14 +1817
description : Cost Volume Pyramid Based Depth Inference for Multi-View Stereo 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Cost Volume Pyramid Based Depth Inference for Multi-View Stereo, CVPR 2020



![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-32.PNG)

요약: Pyramid 형식의 cost volume을 만들었음. 몇몇 점에서 Point MVSNet과 유사함.

 차별되는 점으로는 low resolution depth map을 뽑기 위해 굳이 high resolution image를 쓸 필요 없고, low resolution image를 넣어도 depth map이 잘 나옴. high resolution에서는 더더욱 잘 나옴. P-MVSNet 수준만큼.

 구조 자체가 매우 가벼움. 그래서 연산속도는 매우 빠르다.



### Abstract

 우린 multi image에서 depth inference에 cost volume-based neural network를 제안한다. 우리는 coarse-to-fine 방법으로 cost volume pyramid를 사용했다. 이는 네트워크를 가볍게 만들었고 high resolution depth map을 얻는 것과 더 좋은 reconstruction result를 얻는 것에 도움을 주었다.

 이를 끝내기 위해 image의 coarse resolution에서 전체 depth range를 거쳐 fronto-parallel planes의 uniform sampling에 기반한 cost volume을 만들었다. 주어진 current depth estimate으로, depth map을 refine하기 위해 pixelwise depth residual을 위한 새로운 cost volume을 만들었다. depth를 iterative하게 예측하고 refine한다는 점에서 Point MVSNet과 유사하단 점이 있지만, 우리가 보여줄 cost volume pyramid는 이를 더욱 compact하게 만들고, 3D point에서 efficient한 network structure이다. 또한, cost volume pyramid를 compact하게 만드는 원리인 depth sampling과 image resolution 사이에서 자세한 분석을 제공한다. 우리가 사용하는 방법은 성능은 여태 나온 것들과 비슷하지만, 6x faster함.



### Introduction

 MVS는 3D model을 recon하는 것에 목표를 두고 있음. 이러한 것들은 이미 오랫동안 연구가 되어왔음.

 traditional한 방법도 있었고, deep learning에 이를 적용한 사례도 있었음. 대표적으로 MVSNet임. MVSNet에서 필수적인 것은 multi-scale 3D CNN을 regularization을 위한 plan sweep process위에서 cost volume을 만드는 과정임. depth inference accuracy에는 효과적이나, memory 소모가 매우 심함. high resolution image를 다루기 위해, recurrent를 쓰는 방식도 도입이 되었으나, 이는 memory는 적게 먹었으나 run time이 길어지게 되었음.

 이러한 computational한 네트워크를 해결하기 위해 Point MVSNet이 등장했음. 각 3D point에서 kNN의 edge convolution을 적용해서 depth residual을 예측했다. 이러한 방법은 효과적이었으나, iteration의 수에 비례하여 run time이 증가했다.

 이러한 work에서, 우리는 CVP-MVSNet을 제안한다. 초기에 우리는 각 image에 대한 image pyramid를 만든다. reference image의 coarest resolution에서, entire-depth range의 depth를 sampling하여 compact한 cost volume을 만든다. 이후, 다음 pyramid level은 regularization에서 multi-scale 3D CNN을 이용해서 partial cost volume을 만들고 현재의 estimate된 depth 근처에서 residual depth search를 수행한다. 우리가 작은 범위내에서 iterative하게 수행한 덕분에, 작고 compact한 network를 설계할 수 있었다. 결과적으로 우리의 network는 다른 network와 별 다른 성능 차이 없이 6배 빠른 속도를 보여주었다.

 우리가 coarse-to-fine 방법에서 depth map을 predicting하고 refine하는 것이 Point-MVSNet에서 한 것과 유사하지만, 다음 점들에서 차이를 보일 수 있다.

- Point-MVSNet은 3D point cloud에서 convolution을 진행했다. 대신에, 우리는 훨씬 빠른 시간에 recon을 성공했음. (cost volume base로)
- depth sampling과 image resolution 사이의 상관관계를 바탕으로 가벼운 cost volume pyramid를 만드는 원리를 제공함.
- 우리는 multi-scale 3D CNN regularization을 large receptive field와 local smoothness를 올리기 위해 사용했다.
- 우린 small resolution image에서 small resolution의 depth output을 얻을 수 있다.

 요약하면, 우리의 contribution은 다음과 같다.

- 우리의 목적은 cost volume base에, compact하고, 연산이 가벼운 MVS network.
- depth residual search range와 image resolution 사이에서의 상관관계를 가지고 만들어진, coarse-to-fine method 중 하나인 cost volume pyramid
- less memory requirement, 6배 빠른 속도(Point MVSNet을 상대로), 그럼에도 더 높은 정확도



### Related Work

__Traditional Multi-View Stereo__

 우리는 volumetric, depth-based MVS method에 대해서 discussion할 것.

 volumetric representation은 대부분의 object와 scene을 model할 수 있다. 이러한 방법은 작은 voxel로 나누고, photometric consistency를 이용해서 voxel이 surface인지 아닌지를 판별한다. 

하지만 이런 것들은 memory가 문제였음. 반면에 MVS method 방법의 depth-map은 3D geometry modeling에 있어서 유연성이 있었음.

__Deep learning-based MVS__

 Deep CNN은 vision 임무에서 여러가지 임무를 수행했음. image recognition, object detection, semantic segmantation 등등. 3D vision task에서, learning-based approach는 넓게 펼쳐졌다. 하지만 이런 방법들은 MVS problem을 쉽게 generalize할 수 없었다.

 몇몇 접근들이 직접적으로 MVS problem을 해결하려고 했다. 하지만 이런 여러 시도들 모두 memory가 expensive했음.

 large-scale scene recon은 MVSNet에 의해서 해결이 되었음. 여전히 이런 것들은 수많은 parameter를 필요로 했기에, GRU를 도입한 R-MVSNet등이 도입되기도 했음. 이건 parameter를 줄이는 것엔 성공했으나, run-time이 더 길어졌음. 우리랑 비슷한 것은 Point-MVSNet임.

 Point-MVSNet은 coarse-to-fine 방법을 바로 depth 예측에 사용한 framework임. 3D space에서 kNN의 정보를 aggregation했음. 그 외 몇몇 방법이 유사해보이지만, 몇몇 점들이 다름. 일단 우리는 cost volume base로, regular image grid에서 cost volume을 build했고, PWC Net에서 사용된 partial cost volume의 아이디어에 영감을 받아 partial cost volume을 depth residual을 예측하는 것에 사용했음. 차후 우리는 PointNet과 memory, computational efficiency를 비교할 것임. 이러한 것은 network을 좀 더 compact하게 만들어 주었음.

__Cost Volume__

 Cost Volume은 unrectified image에서 dense depth estimation을 하는 방법에서 사용되었다. 하지만 이를 deep learning에 넣은 거임. 그리고 최근 optical flow estimation을 partial cost volume의 개념으로 소개한 것이 최근에 나왔음. 간단하게 말하면, 현재의 optical flow를 예측한 것에서, partial cost volume은 warped source view image에서 지역적인 위치 안에 rectangle around 내 연관성을 찾으면서 만들어진다. 이러한 전략에 아이디어를 얻어 우리는 CVP를 소개한다.



### Method

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-33.PNG)

 위 그림에서 {I_i}_i=1^N은 N개의 neighboring source image임. K, R, t를 camera intrinsic, rotation, translation에 해당한다고 하자. 우리의 목표는 __I_0__ 의 depth map D를 찾는 것임. 우리의 key는 feed-forward deep network를 이용한 방법임. 

__3.1 Feature Pyramid__

raw image들은 다양한 빛의 변화에 노출되어 있기에, 우리는 dense feature를 추출하는 중요한 단계를 설명해온 learnable feature를 적용했다. 이러한 general practice는 high resolution image를 multi-scale image feature를 뽑는데 썼다.(그게 low resolution depth map을 내뱉더라도!) 그에 반해 우리는 low resolution image도 low resolution depth map을 내뱉는 것에 충분한 정보를 가지고 있다는 것을 보여준다.

 우리의 feature extraction pipeline은 2 step을 가지고 있다.(위 그림처럼)

 첫번째로, 우리는 (L+1) level image pyramid를 만든다.

 두번째로, feature extraction network라 불리는 CNN을 이용해서 l'th level feature representaion을 얻는다. 이러한 방법은 모든 image에서 feature를 추출한다. 

 구체적으로 이러한 feature extraction network는 9개의 convolutional layer로 이루어져있고, 각각은 ReLU를 이용한다. 같은 image, 같은 level에서는 같은 CNN을 쓴다. 이러한 방법은 같은 시간 내에 메모리를 훨씬 덜먹고, performance를 올려준 방법이다.

__3.2 Cost Volume Pyramid__

 주어진 extracted feature에서, 다음 단계는 reference view에서 depth inference의 cost volume을 만드는 것이다. large memory를 차지해서 high resolution image의 사용에 제한을 두는 fixed resolution에 해당하는 cost volume을 사용하지만, 우리는 high resolution depth inference를 이룬, iterative하게 depth map을 estimate하고 refine하는 process인 cost volume pyramid를 제안한다. 좀 더 정확하게, 우리는 image pyramid에서 coarsest resolution image를 기반으로 coarse depth map estimation cost volume을 만들고, fronto-parallel planes의 uniform sampling을 진행한다. 그러면 우린 coarse estimation과 depth residual hypotheses를 기반으로 한 partial cost volume을 만든다. 자세한 과정은 아래 2단계로 설명되어 있다.

__Cost Volume for Coarse Depth Map Inference__

 우린 Lth level에 있는, 가장 낮은 image resolution을 가진 것에서 cost volume을 만들기 시작한다. reference view의 depth에서 dmin과 dmax를 찾고, 전체적인 depth(dmin보단 크고 dmax보단 작은)에서 골고루 M fronto-parallel plane을 sampling해서 cost volume을 만든다.  

 MVSNet에서 했던 것처럼, Homography를 다음과 같이 정의한다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-34.PNG)

 각 homography H_i(d)에서, reference view의 pixel x와 source view i의 x_i사이에서 가능한 pixel correspondence를 찾는다.

 주어진 source view의 pixel x와 feature map f에 대해서, 우리는 differentiable bilinear interpolation을 쓴다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-35.PNG)

 따라서 depth d에서 모든 pixel에 대한 cost는 위와 같다. feature map위의 ~ 의 뜻은 reference image에서 왔다는 뜻이고, -(bar)의 뜻은 all view, each pixel에 대해서 feature volume의 평균을 의미한다. 이러한 방법은 각 depth에서 photometric consistency constraint에서의 연관성인, feature variance를 작게 가지게 한다. 우리는 각 depth hypothesis의 cost map을 계산하고 이 cost map을 하나의 cost volume으로 concatenate한다.

 가장 중요한 parameter는 바로 sampling resolution인 M이다.

__Cost Volume for Multi-scale Depth Residual Inference__

 우리의 목표는 reference image I_o의 depth인 D_o와 일치하는 D를 얻는 것이다. 주어진 (l+1)th level의 depth estimate인 D^(l+1)에서 시작해서, next level인 D^l의 refined depth map을 얻기 위해 반복학습을 한다.(bottom level에 도달할 때 까지) 좀 더 정확하게는, 우린 처음 D^(l+1)을 bicubic interpolation을 이용해서 다음 단계로 upsample하고, lth level에서 더 좋은 refined depth map을 얻기 위해 residual depth map으로 regress하기 위해 partial cost volume을 만든다.

 Point MVSNet과는 달리, cost volume base로 만들었음에도 compact하고, fast하고, 조금 더 높은 정확도를 가질 수 있도록 한다. 우리의 motivation은 이웃 픽셀들의 depth displacement가 상관관계가 있다는 것이다. 왜? regular multi-scale 3D convolution이 쓸모있는 contextual information을 depth residual estimation에 제공하도록. (흠...) 그러므로 우리는 regular 3D space에서 depth displacement hypotheses를 정리했고, cost volume을 다음과 같이 계산했다.

hypothesized 3D point의 연관의 projection은 view i에서 다음과 같다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-36.PNG)

 식을 보면 좀 골치아플 수도 있기에, 아래 그림을 먼저 살펴보자.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-37.PNG)

 대충 보면 이해가 좀 갈 것이다. 식 (3)의 맨 앞 \lambda는 view i에서 해당하는 pixel의 depth를 의미한다. 이걸 좀 엮으면, partial cost volume이 만들어진다.

__3.3 Depth Map Inference__

 여기에선 coarest image resolution과 cost volume을 만들기 위해 higher image resolution에서 local depth search의 discretisation의 depth sampling을 자세하게 설명할 것이다. 그러므로, 우리는 cost volume에서의 depth map estimator를 소개한다.

__Depth Sampling for Cost Volume Pyramid__

 우린 virtual depth plane에서의 depth sampling이 image resolution과 연관이 있다는 것을 관찰했다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-38.PNG)

  위 그림에서 보면, 굳이 dense하게 depth plane을 sample할 필요가 없다는 것을 알 수 있다. dense하게 만들게 된다면 너무 가까워서 depth inference에 extra information을 제공하는 것이 힘들다. 우리의 실험에서, virtual plane의 수를 결정하기 위해 우리는 mean depth sampling interval을 0.5 pixel distance로 결정했다.

 local search range를 결정하는 것에서 우리는 3D point를 source view로 project하고, epipolar line에 있는 2개의 점을 찾고, 이걸 다시 3D ray로 back project했다. 이 때 두 ray의 intersection이 depth refinement의 range를 탐색하는 것을 결정했다.

__Depth Map Estimator__

MVSNet과 유사하게, 우리는 context information을 aggregate하기 위해 cost volume pyramid를 만들고, output으로는 probability volume을 내뱉는다. 3D convolution의 자세한 내용은 Supplementary Material에 있음. P^L과 {P^l}_l=0^L-1 은 각각 absolute, residual depth에 의해 만들어진다. 그리고, 우리는 순서대로 refine하고 soft argmax를 적용함으로써 depth map을 얻는다.

 each pixel에 대해서 depth estimate은 다음과 같다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-39.PNG)

현재의 상태를 더욱 refine하기 위해, 우리는 residual depth를 estimate했다. r_P는 depth residual hypothesis를 의미한다고 하면, 다음 단계에서 update되는 depth는 다음과 같다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-40.PNG)

 pyramidal depth estimation 이후, depth map refinement를 하지 않는 것이 좋은 결과를 얻는 것에 필요했다...? 갑자기? 흠... 나중에 다시 검토

__3.4 Loss Function__

MVSNet과 유사하게 l1 norm을 loss로 사용했다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-41.PNG)



### Experiments

__4.1 Datasets__

__DTU Dataset__

__Tanks and Temples__

__4.2 Implementation__

__Training__

__Metrics__

__Evaluation__

__4.3 Results on DTU dataset__

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-42.PNG)

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-43.PNG)

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-45.PNG)

__4.4 Results on Tanks and Temples__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-44.PNG)

__without any fine-tuning__

__Evaluation pixel interval settings__




### Conclusions



### Acknowledgements



### References

Yang, Jiayu, et al. "Cost Volume Pyramid Based Depth Inference for Multi-View Stereo." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.
