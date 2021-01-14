---
layout : post
title : "Shape Reconstruction Using Volume Sweeping and Learned Photoconsistency"
date : 2021-01-14 +2132
description : Shape Reconstruction Using Volume Sweeping and Learned Photoconsistency 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Shape Reconstruction Using Volume Sweeping and Learned Photoconsistency, ECCV 2018



### Abstract

 가상현실, 증강현실의 상승에 따라 3D content를 포함하는 새로운 기술의 수요가 늘고 있다. 우리는 이번 문제에서 multi-view RGB image에서 3D shape recon을 하는 문제를 다루려고 한다. 조금 더 높은 정확성과 견고함을 가지는, reconstruction에 효과적인 learning based 전략을 조사한다. 우리가 타겟으로 하는 것은 현재 존재하는 것으로 복구하는 것이 힘든 복잡한 면들이 포함된 실제 생활에서 찍힌 것들이다. 가장 중요한 step은 depth information을 알기 위한, 서로 다른 view point에서의 matching point를 찾는 것이다. 우리는 viewing line에서 3D receptive field가 matching되는 것과  multi-view photoconsistency를 배우는 것을 목표로 한다. deep network에서 같은 surface point의 다양한 viewing line에 놓인 서로 다른 원점에 대해서도 대체적으로 local photometric configuration을 찾는 것은 직관적이다. 우리는 전통적인 2D feature based method에 의해 인지되지 않은 dynamic scene에서 surface detail을 회복하는 것에 도움을 주는 이 능력을 설명한다.(CNN이 어떻게 feature를 이용해서 recon을 할 수 있는지에 대해 말하는 듯 하다.) 그리고 각 SOTA를 찍은 방식들을 우리의 solution으로 비교하는 것을 확인할 수 있다.



### Introduction

 이번 논문에서, 우리는 real-life에서 찍힌 형태들의 multi-view shape recon의 문제를 말한다. 예를 들면 실제 옷, 움직임 등과 관련된 것들 말이다. 가상현실, 증강현실 등에 의해 현재 3D recon은 매우 인기있는 field이다.  하지만 여전히 이러한 관점에서 여전히, 필수적으로 개선되어야 할 측면은 정확도와 퀄리티이다. 

 MVS를 기반으로 하는 방법들은 feature extraction, matching, 3D shape inference등에서 좋은 수준의 퀄리티를 자랑하고 있다. 흥미롭게도, 꽤나 최근 일의 연구는 deep learning을 이용해서 연구중이다. 이러한 방법들은 기본적인 2D 방법들에 비해 2D에서도 data-driven prior를 잘 해내었고, 3D에서도 잘 해내었다. 이러한 novel MVS method들은 정적인 화면에서 촬영되고, data-aware feature상에서 좋은 성능을 제공하고 있다.

 우리의 main 목표는, 또다른 challenge인 live performance capture에서 조금 더 성능을 평가하는 것이다. 전통적인 도전들은 occlusion이나 self-occlusion과 같은, texture 정보의 부족함인 작은 영역에서도 조금 더 넓은 영역으로 시야를 넓히는 것이었다.(혹은 스포츠처럼 motion blur가 생기는 경우라던지) DTU dataset이나 ShapeNet dataset같은 경우에도 performance capture data는 아직 없다.

 이러한 형태의 data type을 일반화하는 것을 목표로, 우리는 다양한 곳에서 성공한 MVS algorithm을 적용하여 per view depth map extraction의 정확성을 유지하는 방법들의 이점을 가진 framework을 제안한다. 우리의 접근은 local volumetric unit의 추론과 함께 multi-view matching을 수행한다. 이전의 방법들과는 다르게, 우리의 volumetric unit은 주어진 view로 정의된다. occupancy를 채우는 대신, 우리는 학습을 쉽게하고 local shape pattern보다 photometric configuration 방법에 조금 더 집중하기 위해 disparity score를 infer한다. 우린 volumetric receptive field로 view를 sweep하고,(이 과정을 volume sweeping이라 한다.) geometric surface reconstruction에 의한 multi-view depth map extraction과 fusion pipeline의 알고리즘을 합친다. 이러한 전략으로, dynamic performance가 포함된 장면에서 classical MVS를 진행하는 것이 유효할 수 있다. 우린 complex sequence에서 높은 정확도를 얻을 수 있었고, CNN base와 classic한 방법들과 비교했을 때에도 좋은 성능을 보였다. 



### Related Work



### Method Overview

 Figure 2에 나와있는 것처럼, 우리는 calibrated image set을 input으로 받고, output으로 depth map을 융합함으로써 3D mesh를 내뱉는다. pixel viewing rays를 따른 depth는 volume sweeping strategy를 이용해서 얻을 수 있다. 이 volume sweeping strategy는 ray에서의 multi-view photoconsistency를 sample하고 최대값을 확인한다. viewing ray에 놓여진 point는, 그 point 근처의 descretized된 3D volumetric patch를 사용해서 photoconsistency를 estimate한다. 3D patch의 경우, 각 지점에서, primary camera에서 온 color information은 다른 camera에서 온 incident ray와 짝이 된다. 우린 이러한 paried color volume을 수집한다. 그렇게 학습된 CNN은 3D patch에서 color sample pair로 photoconsistent configuration을 인식한다. 우리의 전략에서 key는 다음과 같다.

- 찍힌 자리에서 주는 photoconsistency를 sample한 per camera approach는 global approach와 비교했을 때 조금 더 local detail을 잘 살릴 수 있다.
- photoconsistency evaluation의 평가를 위한 3D receptive field.(몇 2D projection의 애매함을 해결하려는)
- CNN을 이용한 learning based 전략. dynamic captured scene에서 photoconsistency를 평가했을 때 전통적인 방법보다 훨씬 좋은 성능을 내는 전략.

 다음의 section들은 우리의 main contribution에 집중하고 있다. 3D volume sampling, photoconsistency evaluation의 learning base로 접근하는 방법이다. 마지막 step에서 우리는 TSDF를 사용했다.



### Depth Map Estimation by Volume Sweeping

(작성중)




### Conclusions



### Acknowledgements



### References

Leroy, Vincent, Jean-Sébastien Franco, and Edmond Boyer. "Shape reconstruction using volume sweeping and learned photoconsistency." *Proceedings of the European Conference on Computer Vision (ECCV)*. 2018.
