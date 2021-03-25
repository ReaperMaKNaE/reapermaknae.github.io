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

 object class를 얻기 위해서, ROIAlign technique를 Mask R-CNN에서 들고 왔다. 하지만 거기서 사용하는 것은 굉장히 작은 수준이기 때문에, (내일 추가 작성 계획)





### Reference

Wang, Lijun, et al. "SDC-depth: Semantic divide-and-conquer network for monocular depth estimation." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.

