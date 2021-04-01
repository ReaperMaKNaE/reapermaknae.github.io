---
layout : post
title : "DPT"
date : 2021-04-02 +1200
description : Transformer를 depth estimation과 semantic segmentation에 응용한 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Vision Transformers for Dense Prediction



### Abstract

 Transformer가 vision에서 사용되는 방향을 연구해서, 해당 방향을 dense prediction으로 옮긴 것을 연구한 논문이다. 해당 논문에서는 dense prediction의 예로 depth estimation과 semantic segmentation을 예로 들었다. baseline model은 ViT를 참조하였으며, 해당 내용을 바탕으로 실험한 결과 여러 분야에서 SOTA를 찍었다고 한다.



### Introduction

 vision task에서 dense prediction의 경우, encoder decoder 구조를 이용하여 prediction하는 것을 얻어왔다. 그 아무리 잘 구성이 되어있는 구조라 하더라도, encoder를 통과하는 순간 information의 leak이 일어나게 된다.

 물론 이를 해결할 만한 여러 방법을 사람들이 제시해 왔다. 하지만 여전히 이들의 문제는 bottleneck에서 해결한다는 점이었고, 가장 중요한 점은 여전히 convolution을 사용한다는 것이었다.

 우리는 이러한 문제를 DPT(Dense Prediction Transformer)라는 network을 이용해서 해결하였다. 우리는 최근에 제안된 ViT를 backbone architecture로 사용하여 새로운 dense prediction을 위한 transformer based model을 제안한다.

 이렇게 제안된 model은 최근 SOTA를 찍은 model보다 depth estimation에서는 28% 좋은 성능을 보였다.



### Related Work



### Architecture

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-1.PNG)

 논문에서 제안하는 model은 위와 같다. 일단 간단하게 설명하자면, DPT-Base와 DPT-Large에서는 좌측 하단의 orange색 막대가 단순한 flattened representation이고,(ViT를 참조해야하는데, 그냥 image를 patch단위로 자른 다음 1D로 쭉 편 것이라고 보면 된다.) DPT-Hybrid 모델에서는 ResNet-50을 지나서 얻은 patch들의 feature라고 보면 된다. 먼저 말해서 좀 헷갈리겠지만, 여하튼 DPT는 총 3개의 model variation이 있다. 아무튼 이렇게 orange색 feature(혹은 flattened image)를 뽑고 나면, positional embedding과 patch-independent readout token(빨간색 block)을 추가해서 일련의 transformer 들을 거치게 된다. 여러 단계를 거치게 되는 transformer의 결과물들을 Re-assemble해서 조립을 하고, Fusion을 한 다음, 마지막에 Head를 거치면서 최종 output을 내게 된다.

 여기서 각각에 대해 자세히 들어가면 다음과 같다.

__Transformer encoder.__

 보통 convolution network로 구성된 encoder-decoder의 경우, feature가 결국 input 의 pixel보다 작아지기 때문에 information의 leak이 일어나게 되는데, transformer에서는 이를 막대한 양의 toekn으로 해결할 수 있다. 물론 이렇게 하면 막대한 양의 연산 등의 문제가 발생하기 때문에, 저자는 ViT의 개념을 따오면서 동시에 readout token이라는 것을 추가했다고 한다. 이를 제외한, 위에서 설명한 orange색 token을 서로 다른 3가지 방식으로 뽑았는데, __ViT-Base__ 의 경우에는 patch-based에 12개의 transformer layer를, __ViT-Large__ 의 경우에는 똑같은 patch-based에 24개의 transformer layer에 feature size를 조금 더 키웠고, __ViT-Hybrid__ 의 경우에는 12개의 transformer layer와 ResNet-50으로 feature를 뽑아서 token으로 썼다고 한다. feature size의 경우,(token의 channel을 말함) 기존은 768을, Large model에서는 1024를 의미한다.

 결국 이 정도로 channel이 커지게 되면, token이 input patch의 pixel보다 많은 정보를 담을 수 있기에, information을 유지하는 것에 큰 도움이 될 것이라고 한다. 아, 참고로 transformer의 decoder에서 BN은 안썼다고 한다. 다른 논문에서 그냥 안쓰는게 도움 되었다고 해서 그런 듯.

__Convolutional decoder__

 transformer를 거치고 나온 output은 convolution layer를 거치게 된다. 여기서 말하는 convolution layer는 위 그림에서 가운데 영역에 해당한다. 먹여진 token들을 __"Read"__ 하는 것은 transformer를 통과했다는 뜻이고, 여기서 __"Reassemble"__ 은 convolution decoder를 말하게 된다. 과정은 크게 복잡하지 않은데, transformer를 거친 token들이 concat이 된 이후 project/resample의 과정을 거쳐 마지막 단계인 Fusion으로 넘어가게 된다. 

 참고로, 여기서 Ablation study 거리를 추가하게 되는데, __"Read"__ 과정에서 

- readout token을 무시하느냐, 
- 추가하느냐,
-  cat을 한 다음 mlp를 거치느냐,

 로 총 3가지 이다. (근데 이게 왜 여기에 있는지 모르겠다. 논문에서는 이 section을 transformer encoder로 옮기는게 더 적합해보임. 아마 논문이 나온지 일주일도 안되어서 그런 것 같다. 나중에 리비전 될 듯)

 이후 concat, resample의 과정을 거치게 되는데, resample의 경우에는 s가 p보다 크냐 작냐에 따라 operation이 달라지긴 한다. 이 때 수 많은 transformer layer 중 어디에서 뽑아와 convolutional decoder를 통과 시키느냐를 생각해야하는데,

- ViT-Large: 5, 12, 18, 24
- ViT-Base: 3, 6, 9, 12
- ViT-Hybrid: 9, 12

 라고 한다. 참고로 ViT-Hybrid는 처음 2개는 그냥 ResNet-50 block의 첫번째와 두번째 block에서 뽑아왔다고 한다.

 이렇게 뽑아져서 만들어진 model들을 각각 __DPT-Base, DPT-Large, DPT-Hybrid__ 라 한다.

__Handling varying image sizes.__

 음... 이건 또 왜 여기에 있는지 모르겠는데 아무튼, input image size에 상관없이 행할 수 있다고 한다.



 그리고 지금 나오는 내용은 appendix에 있는 내용인데, 여기에 넣는게 아다리가 더 맞는 것 같아서 먼저 소개한다. 위 그림에서 나오는 걸 보면 뭔가 생략된 것이 하나있는데, 저자는 이걸 appendix에 넣어놨다. 아무래도 곧 수정할듯.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-2.PNG)

 맨 오른쪽 그림에서 residual convolutional unit과 head가 뭐냐? 할 수 있었을 것이다. Residual convolutional unit은 위와 같이 진행된다. Head의 경우 Task를 무엇으로 했느냐에 따라서 서로 다르게 본 것 같다. depth estimation의 경우 가운데 head를, semantic segmentation의 경우 우측 head를 사용했다.



### Experiments

__4.1. Monocular Depth Estimation__

__Experimental protocol.__

 dataset으로 MIX 5와 MIX 6를 썼다는데, 나는 처음 듣긴 한다. 아무튼 transformer에 맞게 막대한 양의 data가 있는 dataset인가 보다. MIX 6의 경우에는 1.4 million image를 가지고 있다고 한다.(저자가 알기론 가장 큰 mono-depth estimation 이라고 함)

 적당히 lr 쓰고 등등 해서 했다고 한다.

__Zero-shot cross-dataset transfer.__

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-3.PNG)

 위와 같은 Table이 결과라고 한다. 그런데 저자가 자꾸 다른 논문에서 자세한 내용을 파악하라고 한다. 뭐 아무튼 metric은 둘째치고 우리의 눈에 보이는 visualization으로 비교를 해보자.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-4.PNG)

 DPT-Hybrid 결과를 보고 할말을 잃었다. 나무의 detail을 어떻게 저만큼이나 살릴 수 있는지 신기할 따름이다.

__Fine-tuning on small datasets.__

 MIX 5, MIX 6보다 크기가 작은 dataset으로 실험을 하면 아래와 같다고 한다. 사용한 small dataset은 NYU v2 Depth와 KITTI. (근데 이거 두개가 작으면 도대체...)

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-5.PNG)

__4.2 Semantic Segmentation__

 semantic segmentation에서는, half resolution으로 뽑아내고 bilinear interpolation을 했다고 한다.

__Experimental protocol.__

 cross-entropy loss와 auxiliary loss를 사용했다고 한다. auxiliary loss는 fusion layer의 끝에서 2번째에서 나오는 output과 loss를 추가했다는데.... 무슨 말인지 모르겠다. 나중에 코드라도 잘 뜯어봐야할 듯. 아무튼 pixACC와 mIoU와 같은 결과를 냈다고 한다.

__ADE20K.__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-6.PNG)

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-7.PNG)

 ADE20K에서 실험한 결과라고 한다. 아래 table은 metric을 추가한 것.

__Fine-tuning on smaller datasets.__

 조금 더 작은 dataset에서 썼다고 한다. 처음 들어보긴 하는데, 아무튼 Pascal Context dataset이라고 한다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-8.PNG)

__4.3. Ablations__

 depth estimation task에 대해서만 ablation을 진행했다고 한다.

__Skip connections.__

 어떤 transformer의 layer에서 빼냈느냐를 말한다. 그 결과는 아래와 같음.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-9.PNG)

 위 table을 보면, 숫자가 작을수록 좋은것 같은데, 이게 무슨 metric을 썼는지를 안알려준다. 참고로 HRWSI, BlendedMVS 등은 dataset을 말한다.

__Readout token.__

 과연 readout token을 쓰는게 도움이 되는 것인가? 를 봤는데, 그 결과는 아래와 같다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-10.PNG)

 transformer에서 token을 무시하냐, 추가하냐, project하냐를 비교해봤더니, project가 가장 나았다고 한다. 참고로 project는 readout token을 각 patch에서 나온 token들에 concat을 하고 MLP를 때린 것이다.

__Backbones.__

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-11.PNG)

 어떤 backbone이 가장 좋은지를 골라낸다. 결국 최종 승자는 ViT-Large이긴 한데, 모델의 크기가 너무 커서 ViT-Hybrid가 trade-off가 가장 좋다고 한다.

__Inference resolution__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-12.PNG)

 Transformer의 encoder가 매우 큰 receptive field를 가지고 있어서(token의 채널이 image patch의 pixel보다 더 많기 때문에) inference time에 resolution이 끼치는 영향이 별 차이가 없다고 한다.

 근데 솔직히 이건 GPU빨 아닌가 싶기도 하다. 저정도로 큰데 과연 모델만 RAM을 얼마나 먹을지 궁금하다.

__Inference speed__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210402-13.PNG)

 inference speed 비교라고 한다. MiDaS가 어떤 model인지는 아직 정확하게 모르긴 하나, DPT-Base의 경우는 오히려 조금 더 좋은 inference time을 보여주고 있다. 해당 결과는 Intel Xeon Platinum 8280 CPU @ 2.70GHz와 RTX 2080을 기준으로, 400장의 demo에서 평균 시간을 낸 것이라고 한다.



### Conclusions



### Reference

Ranftl, René, Alexey Bochkovskiy, and Vladlen Koltun. "Vision Transformers for Dense Prediction." *arXiv preprint arXiv:2103.13413* (2021).

