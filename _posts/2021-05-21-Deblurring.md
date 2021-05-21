---
layout : post
title : "Learning Event-Based Motion Deblurring"
date : 2021-05-21 +0900
description : Panoptic DeepLab을 확장해 depth prediction에서 성공적인 결과를 낸 ViP-DeepLab 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Learning Event-Based Motion Deblurring, CVPR 2020



### Abstract

 motion-blurred image에서 sharp video sequence를 복구하는 것은 blurring 때문에 뭉개진 motion information 때문에 매우 어려운 문제이다. 하지만, event-based camera의 fast motion은 높은 time rate로 event를 잡을 수 있어 효과적인 해결책으로 발견되고 있다. 이번 논문에서 우리는 event-based motion deblurring의 일련적인 formulation에서 시작하여 어떻게 end-to-end deep neural network architecture로 최적화 될 수 있는지에 대해 알려준다. 제안된 architecture는 principled manner로 global하고 local한 scale 모두를 포함하는 visual, temporal knowledge를 통합한 RNN이다. reconstruction을 향상시키고 효과적으로 stream of event에서 boundary prior를 추출하기 위해 우리는 differentiable directional event filtering module을 소개한다. 우리는 GoPro dataset과 DAVIS240C camera로 촬영된 새로운 dataset에서 실험했다. 제안한 방법은 reconstruction quality에서 SOTA를 찍었으며, real-world motion blur를 다루는 것을 더욱 잘 할 수 있게 되었다.



### Introduction

 camera의 exposure time동안 camera sensor들이 서로 다른 time stamp를 기록하고 이것이 축적되어 평균적인 signal을 내기에, exposure time이 필요한 최근 카메라들은 motion blur가 자주 일어난다. 이 반대 문제는 deblurring이라고 하는데, motion-blurred image를 바꾸고 새롭게 sharp하도록 recovery 시키는 과정이 computer vision에서 challenging한 task로 자리잡고 있다. 간단한 motion pattern이 잘 파악되고 있음에도 불구하고, 실제 세상에서 복잡한 motion pattern을 formulating하기란 훨씬 어려운 일이다.

 일반적인 motion blur를 다루기 위해, 최근 deep learning approach들은 blurred image를 다양한 sharp image나 blurred version을 이용하여 복원하는 것들을 제안했다. 몇몇 scenario에 대해서는 이러한 것들이 성공적이었으나, vehicle이나 drone-equipped, handheld 등의 camera에서 오는 심한 motion blur들에 대해서는 reconstruction을 지속적으로 실패하였다. 이러한 심한 손상을 받은 image의 경우 visual, temporal information의 loss로 인해 다루는 것이 거의 불가능하다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-1.PNG)

 computational architecture에 기대는 것에 반해, 이번 work은 data capture stage에서 이러한 문제를 완화시키기 위해 event-based camera를 적용했다. event camera들은 생물학에서 영감을 받은 센서로, pixel의 intensity변화 (event) 를 micro second 단위로 기록하면서 매우 적은 power consumption을 자랑하는 카메라이다. 이러한 sensor의 hybrid model은 image의 temporally calibrated된 event를 기록하게 된다. 결과적으로 이러한 data들은 자연적으로 motion deblurring을 할 수 있는 dense temporal information을 가지게 된다. Figure 1의 (a)와 (b)에서 image가 심각하게 blur가 된 사진과, temporally dense한 scene의 moving pattern인 event가 기록된 것을 확인할 수 있다.

 event-based motion deblurring의 높은 potential에도 불구하고, 중요한 문제는 





### Reference

Qiao, Siyuan, et al. "ViP-DeepLab: Learning Visual Perception with Depth-aware Video Panoptic Segmentation." *arXiv preprint arXiv:2012.05258* (2020).

