---
layout : post
title : "Geometric loss functions for camera pose regression with deep learning"
date : 2021-01-10 +1530
description : PoseNet의 loss function에서 상수를 어떻게 선택하는 것이 더 좋은가에 대한 연구내용을 담고 있는 Geometric loss functions for camera pose regression with deep learning의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Geometric loss functions for camera pose regression with deep learning, CVPR 2017



 이 논문의 저자인 Alex Kendall은 2015년 ICCV에 PoseNet: A convolutional network for real-time 6-DOF camera relocalization이라는 논문을 냈었다. 해당 논문을 간단하게 요약하자면,  기존에 camera pose를 계산하는 것에서 SIFT algorithm이 광범위 하게 쓰였는데, 이 SIFT algorithm의 단점이 point-matching 방법이라서 조도같은 빛의 밝기에 영향을 많이 받고, motion blur, 서로 다른 camera에서 찍힌 경우 사진의 서로 다른 왜곡 정도에 영향을 굉장히 많이 받아, 이를 deep learning으로 접근해서 해결한 논문이다.

 물론 성능이 dramatic하게 좋은 것은 아니었다만, 이 때 당시 사용한 방법을 어떻게 더 개선할 수 있는가에 대해서 논문을 추가로 작성한 논문이 해당 논문이다. 즉, PoseNet을 업그레이드 시킨 것. 원래 PoseNet에서 사용하던 loss function의 상수값 beta가 있는데, 이 값을 어떻게 잘 선택할 수 있는가? 를 이 논문에서 다룬 것이라고 보면 된다.

 그럼 PoseNet부터 리뷰해야하는거 아니냐? 할 수 있는데, 해당 논문을 리뷰하면서 중간중간에 PoseNet이 어떻게 했는지 나오기때문에, PoseNet에 대한 리뷰는 제끼기로 결정했다.



 아무튼 요약하면, __어떻게 적절한 weight value를 찾았는가?__ 가 이 논문의 핵심.



들어가기 전, 아래 내용에서

__Camera Position(== Camera Pose)__ 은 3D 공간 상에서 Camera가 놓인 위치(x,y,z) 이고,

__Camera Orientation__은 카메라가 (x,y,z)에서 어떤 각도를 가지고 어디를 보고 있느냐를 의미한다.

해당 논문에서는 orientation 표현을 쿼터니언으로 표현하였음.



### Abstract

 우리가 만든 PoseNet에서 썼던 loss계산은 너무 비싼 튜닝을 요구했다.(학습시키기에 오래걸린다, loss가 뭐 마음에 안든다 대충 이런 뜻인듯 하다) 그래서 우리는 이번에 geometry와 scene reprojection error를 기반으로 한, 수많은 novel한 loss function을 보여줄 것이다. 이 방법을 우리가 예전에 했던 PoseNet에 적용하니, 실제로 성능도 좋아졌다.



### Introduction

 많은 알고리즘들이 SIFT나 ORB를 localise에 사용하고 있다. 이러한 것들은 real world scenarios에선 상당히 robust하지 않음. 이걸 해결하기 위해 우리가 PoseNet을 만들긴 했는데, 이게 accuracy가 그렇게 높지는 않음. (논문에서는 다른것들과 비교라고 하지만, 일단 자체적인 성능도 SIFT보다 떨어지긴 함) 아무튼 우리가 새로운 방법을 적용시켜서 PoseNet의 성능이 좀 더 좋아졌음.



### Related Work

localisation에 대한 것은 사실 크게 두 가지로 나뉨.

- Place Recognition
- Metric Localisation

 여튼 이러한 방법들이 있었는데, 전부 좋지는 않았고, 우리가 PoseNet을 낸 이후에도 여러 접근법들이 있었으나, 썩 좋지는 않았음. 근본적인 문제를 해결해서 performance를 improve할 수 있도록 하였음.



### Model for camera pose regression

__3.1 Architecture__

기본적으로 GoogLeNet의 구조를 따랐음. 그리고 image classification을 위해, PoseNet에 다음과 같은 것을 추가하였음.

- classification을 위해 사용되었던 network의 마지막부분인 linear regression과 softmax layer 삭제
- FC linear regression layer를 추가. 이러한 layer는 output으로 3개의 position과 4개의 quaternion을 내뱉음.
- quaternion을 unit length로 normalise하기 위한 normalisation layer 추가.

__3.2 Pose representation__

 pose를 나타내는 것은 Euclidean space에서 쉬움. 근데, orientation을 배우는 것은 좀 복잡함.

 그래서 자주 쓰이는 4가지를 비교할 것.

- Euler Angles
- Axis-angle
- SO(3) rotation matrices
- Quaternions

Euler angles랑 Axis angle은 알다시피 gimbal lock이라는 문제가 심각함.(3D 공간상에서 임의의 각도를 표시하는 방법이 unique하지 않음. 이러한 현상으로 인해 발생하게 되는 것이 gimbal lock). SO(3) rotation matrices의 경우, orthogonality constraint 때문에 back propagation이 힘듬.

 그래서 우리는 쿼터니언을 씀.

__3.3 Loss function__

  rotation이랑 translation을 같이 학습시킨다는 것은 꽤나 challenge한데, 아무튼 했음.

__3.3.1 Learning position and orientation__

injective regression loss로 camera position을 학습시킬 때 L_2 euclidean norm을 PoseNet에서 썼는데, 문제는 이걸 orientation에 어떻게 적용하느냐가 문제임.  우리는 아래와 같은 방법을 선택함.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210110-7.PNG)

 위 방식이 우리가 선택했던 방식임. (PoseNet에서도 쿼터니언을 썼나봄) 아무튼 쿼터니언의 경우 한개의 sphere에서 표현되는 방식이 2개이므로, 우리는 hemisphere를 썼음.

__3.3.2 Simultaenously learning position and orientation__

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210110-8.PNG)

 translation과 orientation의 경우 scale이 매우 다르기 떄문에, scale factor인 beta를 끼워넣음. 

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210110-9.PNG)

 위 그래프를 보면 beta를 대충 512로 두면 되겠구만! 인데, 실제로 적용할땐 그러면 안됨. 왜냐하면 indoor에서는 120~750이 적당했고, outdoor에서는 250~2000이 적당했기 때문임. 

(여기까지가 PoseNet이 해왔던 것)

 근데 우리가 이걸 training하는데 하루가 걸리는데, 종일 학습만 돌릴수가 없어서, optimal weight를 고려하게 되었음.

__3.3.3 Learning an optimal weighting__

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210110-10.PNG)

 우리가 쓴 다른 논문에서 다른 task로 사용된 loss를 어떻게 합쳤는가에 대해 설명이 되어있는데, 이걸 camera position과 orientation에 적용시킨 식이 바로 위에 있는 식임.

 sigma hat x, q, square는 homoscedastic uncertainty들로, scalar value에 영향을 받지 않고, model output도 아니고, input data에 영향을 받지도 않음. 얘들은 homoscedastic noise를 대표하는 애들임. 위 식으로 variance를 만들어서 어쩌구 저쩌구인데, 사실 이건 아래 논문을 봐야 이해가 됨.

Kendall, Alex, Yarin Gal, and Roberto Cipolla. "Multi-task learning using uncertainty to weigh losses for scene geometry and semantics." *Proceedings of the IEEE conference on computer vision and pattern recognition*. 2018.

 위 식은 두 가지 성분을 가지고 있음.

- The residual regressions
- The uncertainty regularization terms

먼저 __The residual regressions__의 경우, variance(uncertainty)가 커지게 된다면 더 작은 residual loss를 내게 됨.

두번째로 __The uncertainty regularization terms__의 경우 network가 무한한 uncertainty를 가지지 않도록 만들어 줌. quaternion과 position 사이에서의 unit은 그 scale이 심하게 차이나기 때문에, 이를 줄여주는 역할을 함. 즉, 이 파트는 beta와 비슷하다고 볼 수 있음.

 제대로 이해한 것인진 모르겠으나, 일단 생각을 조금 해보면 다음과 같음.

 quaternion의 경우, 그 scale이 position에 비해서는 그 값이 상당히 작을 수 밖에 없음. 그래서 normalize를 하는 성분이 필요한데, 이게 각 loss function 옆에 붙어있는 sigma의 지수인 -2를 의미함. 그리고 log의 경우, 계산을 해 보면 분산에다 log를 씌운 건데, 이는 단순히 loss function들로만 이루어진 식이 아니라 log 항을 추가하여 분산 자체의 크기에 대해서도 loss function을 이룰 수 있도록 만든 것이라고 볼 수 있다.

 ![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210110-11.PNG)

 s hat은 log sigma square hat으로 두면 위와 같음. 여기서 우리는 s_x hat을 0으로, s_q hat을 -3.0으로 initialization을 하였음.

 여하튼 위와 같은 과정을 가지게 되면, 우리가 원하는 적절한 beta값을 찾을 수 있음.

__3.3.4 Learning from geometric reprojection error__

 ![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210110-12.PNG)

 위 식에서 G는 해당 scene을 의미하고, G'은 해당 scene의 3D points를 의미함. x, q는 GT이고, x hat q hat은 predicted camera poses임. gamma의 경우는 어떤 norm의 형태를 띄는지를 말하는데, 우리는 1로 뒀음.(L1 norm) 왜냐하면 성능이 가장 좋았기 때문.

__3.3.5 Regression norm__



### Experiments

__4.1 Datasets__

Cambridge Landmarks, 7 Scenes, Dubrovnik 6K dataset 사용.

__4.2 Comparison of loss functions__

 ![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210110-13.PNG)

 위 표를 보면 알겠지만, 대충 beta가 500정도면 성능이 괜찮았으니 그 값을 때려넣었을 떄의 성능과, homoscedastic uncertainty를 이용해서 weight를 배우고 이를 적용했을 때의 결과가 위와 같다. 이걸로 pretraining을 해서 reprojection error를 따져보니 결과가 훨씬 좋아진 것을 확인할 수 있다.

 저자가 추천하는 2 step 방법도 아래와 같다.

- 먼저 equation (3)을 이용해서 position과 orientation 사이의 적절한 weight를 학습시키고,
- 이걸 실제 model에다가 때려넣어서 써라.

__4.3 Benchmarking localisation accuracy__

__4.4 Comparison to SIFT-feature approaches__



### Conclusions





### References

Kendall, Alex, and Roberto Cipolla. "Geometric loss functions for camera pose regression with deep learning." *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition*. 2017.

A. Kendall and Y. Gal. What uncertainties do we need in bayesian deep learning for computer vision? arXiv preprint arXiv:1703.04977, 2017.