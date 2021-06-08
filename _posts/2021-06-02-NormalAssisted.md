---
layout : post
title : "Normal Assisted Stereo Depth Estimation"
date : 2021-06-02 +0900
description : Depth estimation에 surface normal을 응용한 논문인 Normal Assisted Stereo Depth Estimation의 리뷰입니다.
tag : [PaperReview]
---

### Normal Assisted Stereo Depth Estimation, CVPR 2020



### Abstract

 indoor와 outdoor에서 정확한 stereo depth estimation을 하는 것은 여러 3D task에서 중요한 요소이다. 최근, learning-based multi-view stereo 방법이 제한된 view에서 경쟁적인 성능을 보여주고 있다. 하지만, 도전적인 과제인 building cross-view와 같은 곳에서는 여전히 만족스러운 결과를 내고 있지 못하다. 이번 논문에서 우리는 어떻게 normal estimation을 이용해서 depth quality를 올릴 수 있는지에 대한 연구를 진행하였다. 우리는 multi-view normal estimation module과 multi-view depth estimation module을 묶었다. 추가적으로, 독립적인 consistency module을 훈련하기 위한 새로운 consistency loss를 제안한다. 우리는 joint learning이 normal과 depth prediction에서 좋은 성능을 보이고, accuracy와 smoothness에서 더 좋은 결과를 얻을 수 있음을 확인할 수 있었다. 실험은 MVS, SUN3D, RGBD와 Scenes11에서 이루어졌다.



### Introduction

 MVS는 수십년 동안 computer vistion에서 연구되어 온 가장 근본적인 문제 중 하나이다. 최근, learning-based MVS가 떠오르면서 다양한 traditional method의 counterpart가 되고 있다. 일반적으로, 이러한 방법들은 optimization problem에 부딪치고 있게된다. 하지만, geometric constraint의 결여로 인해 low texture에서는 별로 좋지 못한 성능을 내게 된다. 

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-1.PNG)

normal based geometric constraint를 monocular depth estimation에 적용하려는 움직임들이 있었다. 한가지 간단한 것은 depth와 normal 사이에 orthogonality를 강요하는 것이 있었다. 하지만, 학습 과정 중에서 loss function을 regularizing 하면서 우리는 naive한 consistency method가 주어진 normal에 대해서 정말 많은 optimal solution을 가지고 있기 때문에 이는 매우 약한 soft constraint라는 것을 발견했다.

_이 문장을 분석해보면, depth와 normal 사이에서 어떤 constraint를 주더라도 unique한 해가 존재하는 constraint가 따로 없어서 그 결과가 좋지 못하다는 것을 말하는 것 같다. 나도 normal과 depth를 이용해서 monocular depth estimation을 진행하려고 했었는데, normal이 이상하게 잡힘에도 불구하고 depth가 어느정도 뽑히는 기아한 현상을 경험한 적이 있다._

 depth와 normal을 optimizing하는 것을 post processing으로 한 적이 있다. 하지만, 이는 매우 expensive step으로 inference time에는 부적합 할 뿐 아니라, input image와는 거리가 멀어질 가능성이 존재한다.(lose grounding이 거리가 멀어진다 라고 해석하긴 했는데 맥락상 맞는 것 같기는 하다.) 그러므로, 우리는 새로운 depth-normal consistency를 제안한다. 우리의 consistency는 pixel coordinate에서 정의가 되고, 우리의 formulation이 간단한 consistency보다 훨씬 효과적인 것을 보여줄 것이다. 이러한 constraint는 multi-view formulation과 상관없고 single view setting에서도 depth와 normal의 pair에 상관없이 consistency를 강요하게 된다. 이를 끝내기 위해, 우리의 contribution은 다음과 같다.

- 우리는 새로운 cost-volume-based multi view surface normal prediction network(NNet)을 제안한다.

 plane sweeping과 projection을 통해 얻은 다양한 multi-view image information을 축적한 3D cost volume으로 부터 우리의 NNet은 정확한 depth에서 image information을 이용하여 정확한 normal을 뽑게 된다. multiple view에서 오는 image feature의 cost volume construction은 correspondences와 같은 추가적인 constraint를 강요하여 사용 가능한 feature의 정보를 가지고 있게 되어 각 point의 depth를 알 수 있게 된다. 우리는 cost volume이 surface normal에 놓여진 image feature위에서 더 잘 학습하는, 더 좋은 structural representation인 것을 보여줄 것이다. single image setting에서는 network가 texture와 color에 overfit하여 더 안 좋은 결과를 보여주게 되는 반면, 우리의 방법은 single view image에 조금 더 적합한 것임을 추가적으로 보여준다.

- 우리는 depth estimation과 normal estimation을 jointly하게 학습하는 model에 대해서 설명한다.

 traditional한 방법과 learning-based stereo method는 cost volume의 noisy nature를 겪게 된다. 가장 큰 문제는 textureless surface의 경우 충분한 증거가 없어서 matching이 힘들다는 점이다. 우리는 normal map을 이용하여 이러한 단점을 극복한 것을 보여준다. MVS, SUN3D, RGBD, Scenes11, Scene Flow dataset의 실험에서 우리의 방법이 SOTA임을 증명한다.



### Related Work



### Approach

 우리가 제안하는 network는 아래에 나와있다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-2.PNG)

 위 그림을 보면 파악할 수 있듯, 총 2개의 module로 구성이 되어 있다. 하나는 depth와 normal의 joint estimation을 포함해서 cost volume을 만드는 것이고, 그 다음 module은 refine을 진행하는 module이다. 이 때 refine을 진행하는 module의 경우에는 predicted depth와 normal map을 우리가 이번에 제안하는 consistency loss를 이용하여 줄이게 된다.

__3.1. Learning based Plane Sweep Stereo__

 첫번째로 우리는 depth prediction module을 설명한다. depth prediction task 관점에서, 현재 learning-based stereo method는 single object reconstruction, scene reconstruction으로 나뉘게 된다. 하지만 이러한 것들은 larger receptive field를 요구하게 된다. 우리의 work은 scene reconstruction에 focus를 하고 있기 때문에, DPSNet을 depth prediction module로 사용한다.

 network의 input은 reference image와 그 neighboring view images로 이루어져있다.(자세한 내용은 DPSNet 참조)

__3.2. Cost volume based Surface Normal Estimation__

 이번 section에서 우리는 surface normal estimation based의 cost volume을 사용하는 network architecture를 소개한다. probability volume은 각 pixel의 candidate plane 사이에서 depth distribution을 만든다. 수많은 후보의 plane이 존재하는데, 가장 정확한 plane을 뽑아내도록 probability를 이용한다. point가 surface 위에 있으면 1, 없으면 0을 반환하게 된다. 이러한 것은 우리의 cost volume이 surface normal map을 뽑아내는 것에 도움이 되도록 한다.

 cost volume이 주어졌을 때 우리는 모든 voxel의 world coordinate을 feature로 concatenate하게 된다. 우리는 2-strided convolution 3개를 사용하여 depth dimension을 줄인다. 그리고 이들을 잘라서 NNet에 보내게 된다. 최종적으로 우리는 모든 slice의 output을 더하고 normalize해서 estimate normal map을 만든다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-3.PNG)

 우리는 위 식에 대한 직관을 다음과 같이 설명한다. 각 slice는 주변 정보들과의 similarity를 가지게 되고 이는 주변 patch와의 정보를 가지는 것과 비슷하다. 추가적으로, strided 3D convolution때문에 잘려진 feature들은 주변 neighboring plane group의 feature에 대한 정보가 축적되어 있다. 이러한 positional information은 우리가 world coordinate을 concatenate할 때 explicit하게 encoding이 된다. 그래서 NNet은 현재 slice에서 각 pixel의 normal이 depth의 조건에 맞는지를 확인한다. 특정 pixel에 대해서 GT depth와 가깝게 slice가 되면 좋은 normal estimate을 내는 것이고, GT와 멀어질수록 zero estimate을 하게 된다. 우리는 첫번째 module을 GT depth map과 비교하였고, 이에 대한 loss는 아래와 같다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-4.PNG)

__3.3. Depth Normal Consistency__

 cost volume에서 depth와 normal을 estimate하는 것에 더해 우리는 depth와 normal map estimate을 위해 consistency를 강요하는 consistency loss를 제안한다. 우리는 이를 활용하기 위해 camera model을 이용하였다. pinhole model의 카메라는 다음과 같은 식을 만족한다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-5.PNG)

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-6.PNG)

 camera model에서, 우리는 위 그림을 통해 아래와 같은 식을 이끌어 낼 수 있다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-7.PNG)

- Estimate 1

 depth map의 spatial gradient는 sobel filter를 이용하여 다음을 만족한다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-8.PNG)

- Estimate 2

  smooth surface는 다음과 같은 조건을 만족한다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-9.PNG)

 따라서 estimate 2를 아래와 같은 depth spatial gradient로 예측할 수 있다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-10.PNG)

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-11.PNG)

 여기서 consistency loss는 다음과 같다.

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-12.PNG)

 estimate 2에서 나온 depth spatial gradient는 absolute depth에만 의존하지 주변 pixel의 depth에는 의존하지 않는다. 우리는 normal map에서 local surface information을 얻기 때문에, 더 쉽고 정확한 estimate을 할 수 있다. 우리의 consistency formulation은 relative depth뿐 아니라 absolute depth를 강요하는 constraint이다. 이전의 work들은 local surface tangent를 이용하여 constraint을 준 반면, 우리가 제안하는 것은 다른 것임을 알 수 있다.(이전의 work들은 normal과 depth사이의 직접적인 것들에 대한 loss를 주었다면, 이 친구는 gradient를 추가로 주면서 그 성능이 훨씬 좋아졌다고 하는 듯)

 우리의 pipeline에서 loss를 서로 다르게 줬다고 한다. UNet을 raw depth와 normal estimate으로 사용하고, 첫번째 module을 학습 시킨 이후 다음 module을 추가해서 학습시켰다고 한다. 더욱이, 우리의 loss function은 그 어떤 depth refinement/completion method로 쓰여도 좋은 것임을 알 수 있다.



### Experiments

__4.1. Datasets.__

SUN3D, RGBD, Scenes11을 사용했다.

__4.2. Comparison with state-of-the-art__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-13.PNG)

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-14.PNG)

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-15.PNG)

__4.3. Surface Normal Estimation__

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-16.PNG)

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-17.PNG)

![img18](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-18.PNG)

__4.4. Consistency Loss__

![img19](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-19.PNG)

__4.5. Visualizing the Cost Volume__

__Regularization__

![img20](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-20.PNG)

__Planar and Textureless Regions__

![img21](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-21.PNG)

![img22](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-22.PNG)

![img23](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210602-23.PNG)



### Conclusion



### Reference

Kusupati, Uday, et al. "Normal assisted stereo depth estimation." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.
