---
layout : post
title : "SDC-Depth"
date : 2021-03-25 +2000
description : Segmentation 결과로 depth estimation을 진행한 SDC-Depth 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### SDC-Depth: Semantic Divide-and-Conquer Network for Monocular Depth Estimation, CVPR 2020



### Abstract

 Divide-and-Conquer 접근을 통해 image의 global context에 대해 확인하고 scale과 shift를 예측하여 local depth segment를 합치는 방법을 찾아내었다.



### Introduction

divide and conquer는 찾아보면 "문제를 나눌 수 없을 때 까지 나눈 후 다시 합치는 것" 인데, 이를 통해 segment를 모두 나눈 후 다시 합치면서 local depth의 연관성을 찾는다는 내용이 들어있다.

 시연 결과는 아래와 같다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-26.PNG)



### Related Work

__Single image depth prediction__

__Augmenting Depth with Semantic Segmentation__



### Semantic Divide-and-Conquer Network for Monocular Depth Estimation

 SDC-Depth net은 총 4개의 module로 구성이 되어있는데, backbone network, segmentation module, depth prediction module, depth aggregation module이다. 

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-27.PNG)

 위 그림에서 파악을 할 수 있듯, segment result를 다 쪼갠 다음, global context를 통해 scale과 shift를 파악해서 depth를 파악하게 된다.

__3.1 Per-Segment Depth Estimation__

 우리는 semantic category segment와 instance segment를 각각 구분하기 위해 2개의 depth prediction stream을 사용한다. category segment를 구하고 난 이후, 각 instance의 성능을 올리기 위한 방법으로 instance-wise depth stream을 수행하게 된다.

__Category-wise Depth-Estimation__

category wise depth estimation에서는 canonical space(하늘이나 땅 같은 것들을 표현하기 위한 space)로 output depth를 normalize한다. global depth는 단순히 weight, bias를 통해 구한다.

__Instance-wise Depth Estimation__

 object class를 얻기 위해서, ROIAlign technique를 Mask R-CNN에서 들고 왔다. 하지만 거기서 사용하는 것은 굉장히 작은 수준이기 때문에, 개념만 따오고 아래와 같은 새로운 network을 만들었다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-28.PNG)

 위 entwork에서 위쪽은 category를, 아래쪽은 instance를 담당하게 된다. 

__3.2 Segmentation Guided Depth Aggregation__

 final depth prediction을 위해, 우리는 semantic segmentation과 instance segmentation 결과를 바탕으로 per-segment depth map regression을 실행한다. 해당 알고리즘은 아래와 같다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-29.PNG)

 위 알고리즘에서 p는 probability, S는 segmentation mask, F는 Instance depth map을 의미한다.

__3.3 Network Training__

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-30.PNG)

 위 식과 같은 Loss를 바탕으로 network를 훈련한다. 우측 아래 notation에서 I는 instance를, S는 segmentation을, d는 depth prediction을 의미한다.



### Experiments

__4.1 Implementation__

몇몇 기법들이 사용되었는데, Error metric에서는 RMSE와 WHDR(Weighted human disagreement rate)가 추가되었다.

__4.2 Cityscapes Results__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-31.PNG)

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-32.PNG)

__4.3 DIW Results__

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-33.PNG)

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-34.PNG)

__4.4 NYU-Depth V2 Results__

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-35.PNG)

__4.5 Ablation study__

__Effects of semantic divide-and-conquer__

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-36.PNG)

 model의 variant는 Category, Instance, depth estimation을 선택했느냐 안했느냐로 결정하게 된다. 해당 표를 plot하면 아래와 같다.

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-37.PNG)

 그리고 추가적으로, averaged depth map과 random instance depth map을 뽑아봤는데, 그 결관느 아래와 같다.

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-38.PNG)

 위 그림에서 붉은색 테두리는 averaged depth map을 말하고, 파란색은 random instance depth map을 의미한다.

__Benefits of segmentation annotation.__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210325-39.PNG)

 학습을 시키면 시킬수록 WHDR이 줄어드는 것을 확인할 수 있어, model이 점점 segmentation annotation label에 접근할 수 있다는 사실을 확인할 수 있다.



### Conclusion



### Acknowledgement



### Reference

Wang, Lijun, et al. "SDC-depth: Semantic divide-and-conquer network for monocular depth estimation." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.

