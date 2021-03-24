---
layout : post
title : "EfficientNet"
date : 2021-03-23 +1100
description : Compound Scaling을 최초로 제안한 EfficientNet 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks, ICML 2019



### Abstract

 같은 network에서 통상적으로 size를 의미하는 depth, width, resolution을 모두 고려한 효과적인 scaling인 __compound scaling__ 을 최초로 소개한 논문이다. 논문의 저자는 해당 내용을 MobileNet과 ResNet에 대해서 실험을 진행했다. 그 결과로 찾아낸 network는 최근 SOTA를 달성했던 GPipe와 거의 유사한 accuracy(GPipe: 84.3%, EfficientNet-B7: 84.4%)를 달성하면서도 __8.4배나 적은 FLOPs를 가지고, hardware에서 6.1배 빠른__ inference time을 가지게 되었다. 추가로, __transfer learning에서도 기존의 network들에 비해 robustness__ 를 보여주었다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-1.PNG)



### Introduction

 이전의 방식에서는 network의 성능을 올리기 위해 depth(layer의 개수)를 늘리거나, width(channel의 수)를 늘리는 방법을 선택했다. 하지만 해당 방법으로 필요한 network의 size를 맞추는 것은 굉장히 지루한 작업이기도 하고, 상당히 오랜 시간이 걸렸다.(NAS 이야기)

 이번 논문에서는 ConvNet(Convolutional network)에 대해 진지하게 생각하여, 어떻게 하면 더 잘 scaling을 할 수 있을까에 대해 생각하게 되었다. 그렇게 고민하다 생각하게 된 것이, 모든 것을 고려하자! 였고, 어떻게 고려할건데? 했을 때 novel한 방법으로 effective compound scaling method를 생각하게 되었다. 아래 그림에서 각 scaling에 대한 차이를 확인할 수 있다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-2.PNG)



### Compound Model Scaling

__3.1 Problem Formulation__

 일단 model을 scaling하기 위해, 저자는 network의 size에 대한 식을 만들어야 했다. 먼저 결과를 보면, 아래와 같다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-3.PNG)

 위 식에서 F는 layer를 의미하고, L은 해당 layer를 얼마나 반복하는지, H, W, C는 각각 height, width, channel을 의미하고, 이를 모두 연결하면 N(Network)이 된다고 표현을 하였다. 솔직히 이 식만 봐선 잘 이해가 안가기 때문에, 이해가 안 간다면 바로 이전 포스팅인 MnasNet에서 search space를 한번이라도 스캔해보자. 그림만 봐도 이해가 한번에 빡 될 것이다. 아무튼, 여기서 depth, width, resolution에 해당하는 항목들에 대해 coefficient를 multiplication해서 compound scaling에 대한 식을 만들었다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-4.PNG)

 위 식에서 w는 width, d는 depth, r은 resolution을 의미한다. 이제 여기서 __Network에서 Efficiency가 뭔데?__ 에 대한 정의를 해야하는데, 저자는 __target memory, target FLOPs를 기준으로 Maximum accuracy를 얻는 network가 efficient network__ 라고 했다. 즉, __efficiency의 기준을 network가 얼마나 가벼우면서 정확한가__ 로 둔 것이다.

__3.2 Scaling Dimensions__

__Depth(d):__ network의 layer를 키우는 것은 원래 그 성능이 좋은 것을 알 수 있다. 하지만, 너무 많이 커지면 vanishing gradient problem때문에 학습하는 것도 어렵다. 또한, depth가 커질 수록 학습 시간이 오래 걸리고, network size에 비해 accuracy gain이 떨어지는 것도 있었다. 그렇다고 너무 작은 depth를 가지게 되는 경우, higher level feature를 쉽게 찾아낼 수 없다.

__Resolution(r):__ depth와 비슷한 경향을 가지고 있다. depth와의 차이점은, layer의 개수는 network size에 선형이지만, resolution의 경우 그 크기가 height과 width 2 dimension으로 늘어나기 때문에 network의 사이즈가 square로 커지는 경향이 있다.

 저자는 width, depth, resolution을 개별적으로 점점 키워서 network의 성능을 확인해봤다. 그 결과는 아래 plot.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-5.PNG)

 위와 같은 결과를 바탕으로, 저자는 아래와 같은 관찰을 할 수 있었다.

 __Observation 1 - network의 width, depth, resolution 어느 것이라도 scaling만 하면 accuracy는 올라간다. 하지만, accuracy gain은 model이 커질 수록 줄어든다.__



__3.3 Compound Scaling__

 그럼 효율적으로 올리는게 어떻게 될까?를 실험해봤다. baseline model에서 depth만을 2배 올리거나, resolution만을 1.3배 하거나,  둘 다 적용하거나, 말거나로. 그 결과는 아래와 같다.(즉, 고정된 width에서 실험)

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-6.PNG)

 당연하게도 둘 다 올리는 것이 성능 향상에 더 좋았다. depth는 2배면서 resolution이 1.3배인 이유는, resolution의 경우 그 크기가 network에 끼치는 크기는 제곱배만큼이기 때문에, depth와의 balance를 위해 그만큼 작은 값을 고려한 것이다.(루트2가 1.414이긴 한데, 딱 안맞다고 뭐라하지 말자. 아무튼 저자는 고려했단다. ~~다시 실험하기 귀찮겠지~~ )

 위와 같은 결과를 통해, 아래와 같은 관찰을 추가적으로 할 수 있었다.

 __Observation 2- 더 좋은 accuracy와 efficiency를 위해, network의 width, depth, resolution을 잘 balancing하는 것은 중요하다.__

 __아니 그래서 어떻게 balance할건데?__

 라는 의문에서, 저자는 아래와 같은 방법을 제안한다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-7.PNG)

 depth, width, resolution의 coefficient를 위 그림과 같이 잡고, 각 coefficient의 곱에서 지수를 제외하며, width와 resolution은 제곱배 만큼 가중을 둬서 그 곱이 2에 가깝도록 target을 잡았다. 왜 이렇게 했냐면, 무작정 키울 수는 없기 때문에, 위와 같은 constraint를 잡아둔 것이다.

 여기서 STEP 1, STEP 2로 나뉘게 되는데, STEP 1의 경우는 __grid search라는 ~~노가다~~__ 방법을 통해 alpha, beta, gamma를 찾고, STEP 2는 찾은 alpha, beta, gamma는 fix하고, 단순히 $\phi$ 만큼 끌어올려 network의 전체적인 size를 점점 올리는 것이다.(아마 target으로 하는 각 FLOPs마다 network를 grid search로 찾기엔 google도 힘들었나보다.)



### EfficientNet Architecture

 위와 같은 방법을 통해 찾은 가장 작은 크기의 network architecture는 아래와 같다. network의 이름은 __EfficientNet-B0__

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-8.PNG)

 MBConv는 Inverted residual block으로, MobileNetV2에서 memory efficient로 사용하는 block이다. 뿐만 아니라 squeeze-and-excitation optimization이라는, SE block을 모두 적용했다고 한다. SE block은 정확도를 높이는 것에 focus가 된 block이다. 



### Experiments

__5.1 Scaling Up MobileNets and ResNets__

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-10.PNG)

 baseline model인 MobileNet과 ResNet의 크기를 조금씩 키워서 비교해본 것. 단순히 한 개의 dimension을 scaling up하는 것 보다, 골고루 올리는 것이 성능을 봤을 때 가장 좋은 것을 확인가능하다.

__5.2 ImageNet Results for EfficientNet__

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-9.PNG)

 위 Table에서 전반적으로 다른 network들과의 비교표를 보여주고 있다. network의 performance는 유사하나, FLOPs가 상당히 준 것을 확인할 수 있다. 해당 내용을 graph로 확인하면 아래와 같다.

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-11.PNG)

 뿐만 아니라 저자는 유사한 성능을 가진 network들끼리 아래 표와 같이 latency를 실제 CPU에서 측정해봤다고 한다. 결과는 같은 성능임에도 불구하고 약 6배나 빠른 속도를 보여준다.

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-12.PNG)

__5.3 Transfer Learning Results for Efficient__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-13.PNG)

 Transfer learning을 통해 위와 같은 transfer learning 결과를 얻을 수 있었다. 기존의 Network들 보다 그 성능/FLOPs 의 비가 좋아진 것을 확인할 수 있다.

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-14.PNG)

 transfer learning dataset은 위와 같다.



### Discussion

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-15.PNG)

 scale에 따른 성능 차이를 visualization 해 준 결과는 위와 같다. 위 뿐만 아니라, 실제로 어떻게 feature map level에서 더 image를 잘 파악하는지를 아래 그림과 같은 CAM(Class Activation Map)을 이용해서 확인할 수 있다.

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210324-16.PNG)

 그림의 윗부분에서는 object 부분만을 focus해서 잘 집어내는 것을 확인할 수 있고, 아래 부분에서도 maze를 잘 잡아내는 것을 확인할 수 있다.




### Conclusion



### Reference

Tan, Mingxing, et al. "Mnasnet: Platform-aware neural architecture search for mobile." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2019.

