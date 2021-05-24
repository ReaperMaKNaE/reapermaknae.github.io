---
layout : post
title : "VisualTransformers"
date : 2021-05-23 +0900
description : ViT에 효율적인 backbone과 Tokens-to-Token 개념을 도입하여 transformer-based model의 무게를 줄이고 성능을 향상시킨 T2T-ViT을 소개한 논문의 리뷰입니다.
tag : [PaperReview]
---

### Visual Transformers: Token-based Image Representation and Processing for Computer Vision



### Abstract

최근 computer vision이 여러 task에서 놀라운 성능을 보여주었다. 하지만, convolution은 image의 pixel들에 대해서 중요도에 상관없이 모두 동등하게 취급하는 것이 문제였다. explicit하게 model의 모든 concept은 모든 image에 펼쳐져있어서, spatially-distant concept을 다루는 것이 힘들다. 이번 work에서, 우리는 모든 image의 정보를 uniform하게 다루는 문제인 (a)를 semantic visual task로 다루어야 하고, token의 relationship을 dense하게 model하는 transformer에 도전을 하게 된다. 중요한 것은, 우리의 visual transformer는 semantic token space에서 작동하고, 분별력있고 사려깊게 다른 image의 context에 기반해서 분별한다. 최근에 많이 발전한 training 기법들을 이용하여, 우리의 visual transformer는 convolutional network들에 비해 그 성능을 압도하였고 더 적은 FLOPs와 parameter로 성능을 끌어올렸다. LIP와 COCO-stuff semantic segmentation에서, VT-based feature pyramid network는 다른 것들에 비해 0.35% 높은 성능에도 불구하고 6.5배나 적은 FLOPs를 가진다.



### Introduction

 computer vision에서 visual information은 pixel로 우리어진 배열로 구성되어 있다. 이러한 pixel 배열들은 convolution으로 만들어지고, deep learning operator의 중요한 요소이다. 이러한 convolution들이 vision model에서 성공적인 결과를 얻었음에도, 몇몇 중요한 문제들이 있었다.

- Not all pixels are created equal

 Image classification model은 foreground object들에 우선순위가 있다. Segmentation model들의 경우 sky, road, 초목들에 비해 pedestrian에 우선순위가 있다. 하지만 그럼에도 불구하고 convolution의 경우에는 모든 image patch들을 중요도에 상관없이 uniform하게 다룬다. 이는 computation과 representation에 있어서 모두 비효율적인 것을 유발하게 된다.

- Not all images have all concepts

 corner나 edge와 같은 natural image에 존재하는 low-level feature는 CNN에서 적절하게 잘 적용이 된다. 하지만, high-level feature인 귀(ear)의 형태 등은 computationally inefficient하다. 예를 들자면, 개의 feature는 flow나 vehicle, 해양 생물들에 비해서 다른 feature를 가지게 된다. 이는 거의 사용하지 않는 filter들이 연산이 되고 있다는 뜻이다.(즉, 계산해봐야 쓸데없는 feature들을 계산하고 있으니 시간이 오래걸린다는 뜻)

- Convolutions struggle to relate spatially-distant concepts

 각 convolution filter는 작은 region에 대해서만 작용을 했고, semantic concept 사이의 long-range interaction은 힘들었다. spatially-distant concept와 연관해서, 이전의 접근들은 kernel size를 증가시키고, model depth를 증가시키거나 dilated conv, global pooling, non-local attention layer등을 적용했다. 하지만, pixel-convolution paradigm의 작동과 연관하여 이러한 접근들은 convolution network이 약해짐을 보상해주는 것에 대한,(아마 네트워크에 내에 convolution layer의 의존도를 약하게 만들면서) 문제를 완화시키는 가장 적합한 움직임이다.

(즉, 현재 접근하는 방식들이 convolution을 적당히 약하게 만들면서 어떻게 feature를 잘 잡아낼 수 있느냐로 발전하고 있다는 말인 것 같음)

 위와 같은 challenge들을 극복하기 위해 우리는 근본적인 문제인 pixel-convolution paradigm을 해결한, image의 high-level concept를 진행하고 표현하는 Visual Transformer를 소개한다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-1.PNG)

 우리의 직관은 몇몇 단어(혹은 visual token)가 image의 high-level feature를 설명하기에 충분한 것이다. 이러한 영감은 network의 pixel-array representation을 고정하는 것에서 왔다. 반면에, 우리는 feature map을 semantic token의 compact set으로 변환하는 spatial attention을 사용한다. 그리고 이후 우리는 이러한 token들을 transformer로 넘겨준다. 결과적으로 계산된 visual token들은 image level prediction task로 사용이 되거나 혹은 spatially re-projected 된다. convolution과는 다르게, 우리의 VT는 3가지 challenge를 더욱 잘 다룰 수 있다.

- 모든 pixel을 동일하게 다루기보다, 중요한 영역에 attending하는 것으로 적절한 computation을 가지는 것
- image의 모든 content를 다루는 것에 반해 visual token에 관련한 semantic concept를 encoding하는 것
- token-space에서 self-attention을 통해 spatially-distant concept사이를 다루는 것.

 우리는 VT의 효과와 key component의 유효성을 확인하기 위해, image classification에서 몇몇 실험을 진행했다. 결과적으로 우리는 적은 parameter로 더 좋은 성능을 냈다.



### Relationship to previous work

__Transformers in vision models__

__Graph convolutions in vision models__

__Attention in vision models__

__Efficient vision models__



### Visual Transformer

 우리의 visual transformer는 Figure 1에 나와있다. 첫번째로, 몇몇 convolution block에 대해서, input image의 process를 통과한 이후, output feature map을 VT로 먹여준다. 우리의 insight는 convolution과 VT의 강점을 모두 이용하는 것이다.

- network의 초기에는 convolution을 이용해서 densely-distributed된 low-level pattern을 배우고,
- network의 후기에는 VT를 이용하여 조금 더 sparsely-distributed 된 higher-oder semantic concept를 배우게 된다.

 결과적으로 image level prediction task의 visual token을 사용하고 pixel-level prediction task를 위한 augmented feature map을 사용한다.

 VT module은 3가지 step을 가지고 있다.

- pixel을 semantic concept로 group해서 visual token의 set을 만드는 것
- semantic concept 사이의 관계를 model하기 위해 transformer를 적용
- 이러한 visual token들을 pixel-space로 project해서 augmented feature map을 얻는다.

 이러한 paradigm은 다른 논문들에서도 사용하였지만, 중요한 차이점으로는 이전의 방법들은 수백가지의 semantic concept를 사용했지만, 우리의 경우에는 superior performance를 얻는 것에 있어 단 16개의 visual token들을 사용했다.

__3.1. Tokenizer__

 우리는 image가 몇몇의 visual token으로 요약이 될 수 있다는 직감이 있었다. 이는 수많은 filter를 사용하는 convolution이나 수많은 latent node들을 사용하는 graph convolution과는 반대되는 내용이다. 이러한 직감을 증명하기 위해, 우리는 feature map을 visual token의 set으로 변경할 수 있는 tokenizer module을 소개한다. 형식적으로, 우리는 input feature map을 $X$ 로, visual token을 $T$ 로 표시한다.

__3.1.1 Filter-based Tokenizer__

 A filter-based tokenizer는 visual token을 뽑아내기 위해 convolution을 이용했다. feature map X에 대해서 우리는 각 pixel을 point-wise convolution을 사용해서 L개의 semantic group으로 map한다. 그리고 각 group에서, 우리는 각 token T를 얻기 위해 spatially pool을 한다. 즉,

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-2.PNG)

 위 식에서 W는 semantic group을 말하고, softmax는 spatial attention으로의 acitvation을 말한다. 최종적으로, A는 X와 곱해지고 L개의 visual token을 만들기 위해 X의 pixel의 weighted 평균을 구한다.

 하지만 많은 high-level semantic concept가 sparse하고 적은 이미지에서만 그것이 드러나고 있다. 따라서 fixed set of learned weight W는 잠재적으로 몇몇 연산에서는 쓸모가 없게 된다. visual token을 뽑아내기 위해 convolution filter W를 사용하기 때문에 이를 filter-based tokenizer라고 한다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-3.PNG)

__3.1.2. Recurrent Tokenizer__

 filter-based tokenizer의 한계를 극복하기 위해, 우리는 이전 visual token에 dependent한 weight를 사용하는 recurrent tokenizer를 제안한다. 이전 layer의 token이 현재 layer에서 새로운 token을 extraction하는 것을 guide하게 한다. recurrent tokenizer는 current token에서 와서 이전의 것들에 의존하여 계산이 된다. 식으로 보면,

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-4.PNG)

 이러한 방식에서, VT는 visual token의 set을 refine할 수 있게 된다. 실제 환경에서 우리는 두번째 VT에서 recurrent tokenizer를 시작하게 된다.(이전의 VT에서 나온 token을 써야하므로.)

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-5.PNG)

__3.2. Transformer__

 tokenization 다음, 우리는 visual token의 관계에서 interaction을 설계할 필요가 있다. 이전의 work은 이를 묶기 위해 graph convolution을 사용했지만, 이러한 방식은 inference동안 fixed weight를 사용하게 되고, 이는 각 token이 특정한 concept에 묶여있게 된다는 것을 의미한다. 이로 인해 graph convolution은 모든 high-level concept를 modeling함으로써 computation을 낭비하게 된다. 이를 해결하기 위해 우리는 transformer를 이용했다. (transformer는 input-dependent weight를 씀) 이것 덕분에, transformer는 variable meaning을 가진 visual token을 가지고 있고, 적은 token에서도 다양한 concept을 커버할 수 있게 된다.

 우리는 standard transformer에서 약간의 변화를 주었다.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-6.PNG)

 graph convolution과 transformer의 다른 점으로는 token들 사이의 weight가 input-dependent하고 key-query product로 수행이 된다는 점이다.(transformer의 이야기) 이러한 방식은 16개의 visual token을 사용할 수 있다는 점이다. self-attention 이후, 우리는 non-linearity와 두 개의 point wise convolution을 사용한다.(Equation 4 참조) 위 식에서 F는 weight를, sigma는 ReLU function을 의미한다.

__3.3. Projector__

 많은 vision task에서 pixel-level detail을 요구하지만, 이러한 detail은 visual token에서는 유지가 되지 않는다. 그래서 우리는 transformer의 ouptut을 feature map과 융합한 다음 feature map의 pixel-array representation을 refine한다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-7.PNG)

 대충 query, key 이야기이다.



### Using Visual Transformers in vision models

 여기서는 대충 image classification에서는 어떤 모델을, semantic segmentation에서는 어떤 모델을 사용했는지를 알려준다.

__Image classification model:__

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-8.PNG)

__Semantic segmentation__

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-9.PNG)

 대충 네트워크 구조는 우측과 같다. 왼쪽은 vanilla FPN을 보여준다. 근데 tokenizer를 저렇게 하는 것이 저만큼이나 줄여줄 수 있는 거라면... depth에서도 잘 쓸 수 있지 않을까..?



### Experiments

__5.1. Visual Transformer for Classification__

__VT vs. ResNet with default training recipe:__

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-10.PNG)

__Tokenizer ablation studies:__

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-11.PNG)

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-12.PNG)

__Modeling token relationships:__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-13.PNG)

__Token efficiency ablation:__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-14.PNG)

__Projection ablation:__

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-15.PNG)

__5.2. Training with Advanced Recipe__

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-16.PNG)

__5.3. Visual Transformer for Semantic Segmentation__

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-17.PNG)

__5.4. Visualizing Visual Tokens__

![img18](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210523-18.PNG)



### Conclusion



### Reference

Wu, Bichen, et al. "Visual transformers: Token-based image representation and processing for computer vision." *arXiv preprint arXiv:2006.03677* (2020).

