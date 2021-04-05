---
layout : post
title : "EfficientNet V2"
date : 2021-04-05 +0900
description : EfficientNet에 비해 더 효율적으로 compound scaling을 적용하고, progressive learning의 단점을 보완하여 image classification task에서 Top-1 Acc를 새로 달성한 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### EfficientNetV2: Smaller Models and Faster Training



### Abstract

 이번 논문에서는 EfficientNet을 개량해서 다시 한 번 SOTA를 달성한 EfficientNetV2를 다룬다. (저자는 EfficientNet을 쓴 저자와 똑같다.) 새로 제안한 방식에 따르면, training-aware neural architecture search와 scaling을 제안한다. 그리고 MB Conv block 뿐 아니라 새로운 Fused-MBConv block을 도입하였다. 그 결과로, 6.8배는 더 작으면서도 성능은 더 좋은 model을 만들었다.

 추가적으로 progressive learning을 도입하였는데, 이미지 사이즈를 점점 올리는 것이 학습 속도가 빨라지긴 하였으나, 어떤 상황에서는 accuracy drop을 유발하기도 하여서, 이러한 것을 극복하기 위해 adjust regularization을 도입했다.

 일반적인 Dataset인 ImageNet에서 학습을 시키면 성능은 NFNet-F4와 비슷함에도 6.8배 빠른 것이 한계이지만, large dataset인 ImageNet 21k를 이용해서 학습시킨 경우 Top-1 Acc는 87.3%로 SOTA를 달성하였다.



### Introduction

 최근 Image classification task에서 준수한 성능을 내는 여러 모델들이 있었다. NFNet이나 transformer를 사용한 ViT 등이 그 예이다. 하지만 이러한 것들은, 대개 parameter size가 매우 커지게 된다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-1.PNG)

 위 그림에 따르면 비슷한 성능의 model들에 비해 뒤지지않는 성능에도 parameter의 수는 확실히 더 적어진 것을 알 수 있다.

 저자는 EfficientNet을 실험하면서 다음과 같은 3가지를 파악했다고 한다.

- 큰 이미지 사이즈에서 실험을 진행하는 것은 너무 느리다.
- depthwise convolution이 초기 layer에 들어간 경우 속도가 느리다.
- 모든 size(depth, width, height)을 동일하게 scaling-up 하는 경우는 sub-optimal이다.

 이러한 관찰을 바탕으로, 저자는 Fused-MBConv block과 training-aware NAS and scaling method를 추가했다. 그 결과, training 속도는 4배나 더 빠르고, parameter size는 6.8배 적은 EfficientNetV2를 만들게 되었다.

 progressive learning을 하면서 점점 image scale은 커져가는데,(여기서 progressive learning의 경우는 아래 realblack0 님의 posting 참조)

realblack0님 작성, "딥러닝, 평생학습이란? (A Survey on Lifelong Learning)", [https://realblack0.github.io/2020/03/22/lifelong-learning.html](https://realblack0.github.io/2020/03/22/lifelong-learning.html)

 그렇게 image scale을 키워가면서 같은 regularization(dropout 등과 같은 설정들)을 적용하는 것이 좋지 않다고 생각했다.(결과적으로 정확도가 떨어지는 경우가 생겼기 때문에) 이를 약간의 실험을 통해서, 작은 사이즈의 image에는 weak regularization을, 큰 사이즈의 image에는 hard regularization을 적용하는 것이 overfitting을 방지하는데에 도움이 된다고 한다.

 ImageNet에서 학습을 하며, CIFAR-10, CIFAR-100, Cars, Flowers dataset에 transfer learning을 적용해보았고, ImageNet 21k에서 학습한 결과 Top-1 ACC를 87.3% 달성하며 SOTA를 달성하였다.

 해당 논문의 contribution을 따지면 다음 3가지와 같다.

- training-aware NAS and scaling
- Improved method of progressive learning
- 11배나 빠른 training speed(EfficientNet V1에 비해)와 6.8배 적은 Parameter



### Related Work

__Training and Parameter Efficiency__

__Progressive Training__

__Neural Architecture Search(NAS)__



### EfficientNetV2 Architecture Design

__3.1 Review of EfficientNet__

 EfficientNet에서는 parameter의 수와 FLOPs를 이용하여 efficient한 network을 찾는 시도를 진행했다. 하지만, 새로운 EfficientNet V2에서는 parameter efficiency는 유지한 채 training speed를 improve하는 것에 focus를 두었다. 아래는 이전의 work들에 대한 table이다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-2.PNG)

__3.2 Understanding Training Efficiency__

__Training with very large image sizes is slow:__

 이미지 사이즈를 키우는 것이 network의 성능을 올리는 건 맞으나, 너무 많은 training time, parameter size를 올린다고 한다. 이를 증명하는 것은 아래 table에서 확인할 수 있다. OOM은 out of memory로, 메모리 부족이라는 뜻.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-3.PNG)

__Depthwise convolutions are slow in early layers:__

 depth wise를 초반 layer에 배치시키는 것이 network의 속도를 느리게(training speed, inference time, parameter size가 커짐 등등과 동일한 말) 만드는 것을 발견하였다고 한다. 그래서 depthwise conv 3x3이 들어가지 않는, Fused-MBConv를 생각해보았다. 

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-4.PNG)

 MBConv block 몇개를 Fused-MBConv block으로 변경해보았는데, 그 결과로 accuracy가 오르기도하고 내리기도 하였다. 그래서 NAS를 다시 한번 적용을 했다고 한다. 그 결과는 아래 table에 나와있다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-5.PNG)

__Equally scaling up every stage is sub-optimal__

 EfficientNet V1에서 사용했던 compound scaling에 관한 내용이다. 다른 image size라면 다른 regularization이 적용되어야 한다는 점을 지적하고 있다. 그래서 새로운 non-uniform scaling strategy를 적용했다고 한다.

__3.3. Training-Aware NAS and Scaling__

__NAS Search:__

 조합을 많이 섞으면 섞을수록 NAS에서 그 시간은 엄청 오래 걸리게 된다. 따라서, 그들은 이를 적절하게 취할 것만 취하면서(search space를 줄이면서) NAS를 다시 돌렸다고 한다.

 일단 MnasNet에서 썼던 setting은 그대로 들고가되, EfficientNet 그 어디에도 없는 pooling skip operation과 같은 것들은 모두 없앴고, EfficientDet에서 쓰인 channel size와 똑같은 것을 사용했다고 한다. 그리고 MnasNet과 EfficientNet과는 다르게, reward는 $A \cdot S^w \cdot P^v $ 를 사용했는데, 여기서 A는 model accuracy, S는 normalized training step time, P는 parameter size를 의미한다. 위 식에서 w와 v는 각각 -0.07과 -0.05인데, 이는 실험적으로 발견했다고 한다.

__EfficientNetV2 Architecture:__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-6.PNG)

 그 결과로 만들어진 EfficientNet V2 Architecture는 위와 같다. Fused MBConv 가 초반에 배치되어있고, 이후 MBConv를 적용하는 것을 확인할 수 있다. 몇몇 특징을 살펴보면 다음과 같다.

- MBConv와 Fused MBConv 두 개를 모두 채택함
- expansion ratio가 적음(모델이 깊어지면서 점진적으로 커지는 것을 말함)
- 3x3 kernel size를 사용하는데, 그만큼 layer의 수가 더 늘어나게 되었음.

참고로, 아래는 EfficientNet V1-B0의 Architecture. SE는 모두 적용되어있다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-7.PNG)

__EfficientNetV2 Scaling__

 scaling up을 통해 가장 작은 model인 EfficientNetV2-S를 만들었고, 이후 M/L 까지 만들었다. 이렇게 만드는 것은 EfficientNet V1에서의 compound scaling과 유사하나, maximum inference image size를 480으로 두었다는 점, layer를 조금씩 더한 점 등이 EfficientNet V1과 다르다.

__Training speed comparison__

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-8.PNG)

 NFNet을 제외한 모든 model은 350 epoch로 실행되었으며, NFNet의 경우는 360 epoch로 실행이 되었다. 위 그림에서 확인할 수 있듯이 EfficientNetV2가 다른 model들에 비해 training time이 훨씬 줄어든 것을 확인할 수 있다.



### Progressive Learning

__4.1. Motivation__

 같은 regularization을 서로 다른 image size를 가진 network에 적용한 결과, input image size에 따른 regularization의 영향이 서로 다른 것을 알게 되었다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-9.PNG)

 위 Table에서 그 내용을 확인할 수 있다. input image size가 작을수록 regularization(여기서는 RandAug)가 약할수록 성능이 더 좋고, input image size가 클 수록 regularization은 커져야하는 것을 확인할 수 있다.

__4.2. Progressive Learning with adaptive Regularization__

 그래서 그냥 이미지가 커질 수록 regularization이 커지는 전략을 사용했다고 한다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-10.PNG)

 이를 visualization하면 위와 같다.

 그리고 해당 과정을 알고리즘으로 나타내면 아래와 같다.

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-11.PNG)

 위 식에서 M은 stage를 말하는데, image size가 몇단계를 통해 커지는지를 의미하는 것이다. 위 과정에서 사용한 regularization은, __Dropout, RandAugment, Mixup__ 3가지 이다.



### Main Results

__5.1. ImageNet ILSVRC2012__

__Setup__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-12.PNG)

 먼저 말하자면, M은 4로 되어있다. 그 외 각종 spec은 위 table에 나타나있다.

__Results__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-13.PNG)

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-14.PNG)

 위 Table에서 그 결과를 확인할 수 있다. ViT 등 다양한 model들이 나와있지만, 이번에 제안한 EfficientNet-V2가 얼마나 좋은 성능을 뽐내고 있는지 확인이 가능하다.

 정리한 그래프는 아래와 같다.

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-15.PNG)

__5.2. ImageNet21k__

__Setup__

 ImageNet21k에서 training을 했었다고 한다.(사실 위 figure에 다 나와있음. ~~뒷북~~ ) 고로 생략.

__Results__

 결과적으로는, data size를 올리는 것이 model size를 올리는 것보다 효율적이고, ImageNet 21k에서는 pretraining이 꽤나 중요하다고 한다.

__5.3 Transfer Learning datasets__

__Setup__

 Transfer learning dataset으로는 CIFAR-10, CIFAR-100, Flowers, Cars dataset을 썼다고 한다. 이 때, ImageNet 21k가 아닌 ImageNet 으로 학습한 EfficientNetV2를 사용했다고 한다.

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-17.PNG)

__Results__

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-16.PNG)



### Ablation Study

__6.1. Comparison to EfficientNet__

__Performance with the same training__

![img18](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-18.PNG)

 위 table에서 확인해보면, V2의 경우에는 L이 아닌 M을 사용했음을 확인할 수 있다.

__Scaling Down__

![img19](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-19.PNG)

 가장 작은 model인 EfficientNet V2 - S 에 simple compound scaling을 적용해서 더 작게 만들어 EfficientNet V1과 비교한 결과이다. progressive learning은 사용하지 않았다고 한다.

__6.2. Progressive Learning for Different Networks__

![img20](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-20.PNG)

 다른 모델들에 progressive learning을 적용해본 결과, Training time은 줄면서 Acc는 증가하는 것을 확인할 수 있다. 즉, 다른 곳에서도 적용이 가능한 좋은 방법이라는 것.

__6.3. Importance of Adaptive Regularization__

 adaptive regularization이 실제로도 좋은 결과를 내뱉는지에 대해 확인해봐야한다. 

![img21](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-21.PNG)

 그래서 확인해본 결과는 위와 같다. adaptive regularization을 사용한 경우, 사용하지 않은 경우에 비해 성능이 좋은 것을 확인할 수 있다.



### Conclusions



### Appendix

__A1. Source images__

![img22](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-22.PNG)

 Mixup을 보여주는 예시이다.

__A2. Design Choices for Progressive Learning__

![img23](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-23.PNG)

 맨 위에서 stage M을 선택하게 되는 이유는 위 table과 같다. training time과 accuracy의 ratio를 봤을 때 제일 효과적이기 때문.

__A3. Latency Measurements__

![img24](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210405-24.PNG)

 latency를 측정해본 결과, 같은 hardware에서 EfficientNet-B7은 OOM이 떠버렸고, 초당 image throughput은 EfficientNetV2-M이 가장 좋은 것을 확인할 수 있다.(비슷한 수준의 정확도 안에서 비교)



### Reference

Mingxing Tan, Quoc Le. EfficientNetV2: Smaller Models and Faster Training. https://arxiv.org/abs/2104.00298