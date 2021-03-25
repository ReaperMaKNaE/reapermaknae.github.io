---
layout : post
title : "Deformable DETR"
date : 2021-03-25 +1100
description : DETR의 small object detection에 대한 낮은 accuracy를, FPN이 아닌 새로운 구조를 제안하여 성능을 올린 Deformable DETR 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Deformable DETR: Deformable Transformers for End-to-End Object Detection, ICLR 2021



### Abstract

 DETR의 느린 속도와 제한된 feature spatial resolution을 해결하기 위해 고안한 새로운 방식이라고 한다. 그 성능으로는 10배나 적은 training epoch로도 준수한 성능을 냈으며, small object에 대한 정확도를 올려 전체적인 성능이 좋아졌다고 한다.



### Introduction

 DETR은 정말 대단한 것이지만, small-object에 대한 detect 능력이 썩 좋지 못했다. 또한, attention의 연산이 엄청 오래걸리기 때문에 그 시간 역시 오래 걸렸다. 이를 해결하기 위한 것으로, 우리는 deformable attention module을 제안한다. 해당 module은 FPN 없이 multi-scale feature map을 뽑아낼 수 있다.

 추가적으로, 우리는 single stage deformable DETR뿐 아니라 two-stage deformable DETR을 고안하게 되었다. 그 결과로 우리는 deformable DETR의 성능을 더욱 끌어올릴 수 있었다.

 아래는 제안한 deformable DETR의 구조.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-1.PNG)



### Related Work

__Efficient Attention Mechanism.__

 self-attention 과 cross-attention 등을 사용한 Transformer 구조에 대해 말하고 있다. 하지만 Transformer는 그 자체적으로 엄청 큰 모델로, 이러한 문제를 해결하기 위한 새로운 attention에 대한 연구들이 있었다. 그 종류는 다음과 같다.

- Pre-defined sparse attention patterns on keys
- learn data-dependent sparse attention
- low-rank property in self-attention

 이 친구들이 행한 것은 2번째에 가깝다고 한다. sampling point의 고정된 적은 수의 set에 집중해서 어떤 image인지를 predict한다고 한다.

__Multi-scale Feature Representation for Object Detection__

 내용 생략.



### Revisiting Transformers and DETR

__Multi-head attention in Transformers__

__DETR__

 위 두 가지 모두 multi-head attention과 DETR의 complexity에 대해 조사하는 내용이 담겨져있다. 이전에, 왜 transformer로 학습을 하면 시간이 오래걸리는 지에 대해서도 나와있다. 그래서 아마 사전 지식이 많이 부족하다면 해당 내용을 읽는 편이 좋을 것같다. 의외로 다른 어디보다도 여기서 transformer의 time complexity에 대해 정말 잘 설명한 것 같다. 



### Method

__4.1 Deformable Transformers for End-to-end object detection__

__Deformable Attention Module__

 논문에서 제안하는 deformable attention module은 reference point에서 정해진 조금의 key sampling point만을 찾는 경향이 있다. 이를 통해 좋은 결과를 얻을 수 있었다고 한다. 

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-2.PNG)

 그 구조는 위와 같다. 그럼 뭐가 다른지 한번 알아보자.

 deformable attention에서 attention의 연산은 다음과 같다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-4.PNG)

 대충 알아볼 수 있다고 생각한다. 위 식에서 q는 index, z는 feature map, p는 2d reference point이다. $\triangle p$ 의 경우 sampling offset을, A는 attention weight를 말한다. 그럼 이를 deformable multi-head attention으로 확장하기 전, 기존에 사용되던 attention의 계산 식을 먼저 보자.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-3.PNG)

 위 식이 기존에 사용되던 attention의 계산 식이고, 아래가 deformable multi-head attention에서 사용하는 식이다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-5.PNG)

 그냥 normalize가 더 들어가고 한 개념이다. 자세한 사항은 논문 참조. M과 k에 대해 헷갈릴 수 있는데, 위쪽의 deformable DETR의 overivew에서 Linear 이후 나오는 sampling offset에서 각 head가 가지는 개수가 sampled key의 개수인 K이고, 각 head들이 모두 모여 aggregated sampled value를 이루는 곳에서 저장되는 attention의 개수는 M이다.

__Deformable Transformer encoder__

 encoder에서 특이사항으로는 어떤 feature level인지를 알려주기 위해 scale-level embedding을 했다고 한다. 해당 notation은 $e_I$ 

__Deformable Transformer decoder__

 두 decoder의 query element는 모두 object query이다. 여기서 이 친구들은 convolutional feature map에서만 deformable attention module이 되도록 설계했기 때문에, decoder에서 self-attention module은 그대로 두고 cross-attention module만 multi-scale deformable attention module로 변경했다고 한다. 

__4.2 Additional improvements and variants for deformable DETR__

__Iterative bounding box refinement__ : 이전 layer에서 predict된 bbox를 다음 layer로 보내면서 iterative.

__Two-stage deformable DETR__ : deformable DETR을 2개로 나누어서, 처음 DETR을 region proposal로 썼다고 한다. 이렇게 만들어진 region proposal들은 두 번째 stage의 deformable DETR의 object query로 decoder에 들어간다고 한다.

 여기서 region proposal의 경우, decoder는 없고 encoder만 존재하는 deformable DETR이다. 저자의 말에 따르면 decoder의 경우 time complexity가 엄청나져서 바꿨다고 한다.



### Experiment

__Dataset__

 COCO 2017 dataset 사용했다고 한다.

__Implementation Details.__

 M=8, K=4, object query의 수는 100에서 300으로 증가 시키고, 기타 몇가지 설정들을 했다. 자세한 내용은 논문 참조.

__5.1 Comparison with DETR__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-6.PNG)

 적은 epoch에도 좋은 성능을 보이는 것을 확인할 수 있다.

 위에서 DETR-DC5$^+$ 는 DETR-DC5에서 focal loss를 없앤 대신 object query의 개수를 100개에서 300개로 늘린 결과라고 한다. 

__5.2 Ablation study on deformable attention__

 MS attention을 채용하느냐 안하느냐, MS input을 채용하느냐 안하느냐, K의 개수를 늘리는 행위 등이 성능 향상에 좋았다고 한다. FPNs에 w/o(without)이 쓰여져 있는 이유는, 애초에 구조에 없기 때문이다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-7.PNG)

__5.3 Comparison with State-of-the-art methods__

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-8.PNG)

 다른 그 어떤 SOTA model들과 비교해도 그 성능이 뒤지지 않으면서, bells and whistles 없이 준수한 성능을 뽐내는 것을 확인할 수 있다.




### Conclusion



### Appendix

__A.1. Complexity for deformable attention__

 complexity에 대한 내용이 담겨져있다. 쉽지 않으니 해당 논문 참고하면서 공부. Transformer를 쓸 거면 한번은 봐야할 것 같다.

__A.2. Constructing Multi-scale feature maps for deformable DETR__

 어떻게 FPN없이 성공적으로 작은 feature를 잡아낼 수 있었는가에 대한 내용이 나온다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-9.PNG)

 위 그림과 같이, FPN은 아니고 각 layer에서 뽑아낸 feature들을 이용해 알아낸 것. 그럼 왜 FPN의 구조가 필요 없이(깊은 차원의 feature map에서 상대적으로 low feature map으로의 mapping이 진행되는 구조) 잘 되냐면, attention끼리 알아서 정보를 공유하기 때문이라고 한다. 그래서 굳이 수동적으로 이어줄 필요가 없다는 의미.

__A.3. Bounding box prediction in deformable DETR__

 reference point에 대한 bbox regression은 다음과 같다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-10.PNG)

 위 식에서 $\sigma$ 는 sigmoid function을 의미한다.

__A.4. More implementation details__

__Iterative Bounding box refinement__

 처음 bbox refinement의 경우엔,

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-11.PNG)

 위 식과 같이 진행이 된다.

__Two-Stage deformable DETR__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-12.PNG)

 여기서 Two-stage로 넘어가게 된다면, 위와 같은 형태로 bbox refinement 식이 변하게 된다.

__Initialization for multi-scale deformable at tention.__

 initial을 어떻게 시키느냐에서, random하게 잡아낸 k에서 sampling offset을 정하게 된다.

__A.5. What deformable DETR Looks at?__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-13.PNG)

 deformable DETR이 어떤 것을 catch하는지에 대해 나와있다. DETR의 decoder 부분을 확인한 것과 유사. x, y, w, h에 따라서 필요한 것을 잘 잡아내는 것을 볼 수 있다.

__A.6. Visualization of multi-scale deformable attention__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-14.PNG)

 위 그림은 encoder의 output을 확인한 것이다. 녹색 십자가는 target을, 점들이 각각 sampling point로 나타나있고 그들의 weight가 색깔별로 표시되어 있다.

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-15.PNG)

 위 그림은 decoder의 output을 확인한 결과.



### Reference

Zhu, Xizhou, et al. "Deformable DETR: Deformable Transformers for End-to-End Object Detection." *arXiv preprint arXiv:2010.04159* (2020).

