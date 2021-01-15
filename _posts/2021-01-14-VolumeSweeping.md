---
layout : post
title : "Shape Reconstruction Using Volume Sweeping and Learned Photoconsistency"
date : 2021-01-14 +2132
description : Shape Reconstruction Using Volume Sweeping and Learned Photoconsistency 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Shape Reconstruction Using Volume Sweeping and Learned Photoconsistency, ECCV 2018



SUMMARY: volume sampling으로 눈에 보이는 표면의 photoconsistency를 파악하고, 이로 인해 motion blur나 그 외의 SfM에서 발생하는 문제를 해결한 방법. 학습시간 같은 것들은 나와있지 않은데, 표면만 학습한다고 해도 다른 모델들에 비해서 학습 시간이 좋아보이진 않은 것 같다.



### Abstract

 가상현실, 증강현실의 상승에 따라 3D content를 포함하는 새로운 기술의 수요가 늘고 있다. 우리는 이번 문제에서 multi-view RGB image에서 3D shape recon을 하는 문제를 다루려고 한다. 조금 더 높은 정확성과 견고함을 가지는, reconstruction에 효과적인 learning based 전략을 조사한다. 우리가 타겟으로 하는 것은 현재 존재하는 것으로 복구하는 것이 힘든 복잡한 면들이 포함된 실제 생활에서 찍힌 것들이다. 가장 중요한 step은 depth information을 알기 위한, 서로 다른 view point에서의 matching point를 찾는 것이다. 우리는 viewing line에서 3D receptive field가 matching되는 것과  multi-view photoconsistency를 배우는 것을 목표로 한다. deep network에서 같은 surface point의 다양한 viewing line에 놓인 서로 다른 원점에 대해서도 대체적으로 local photometric configuration을 찾는 것은 직관적이다. 우리는 전통적인 2D feature based method에 의해 인지되지 않은 dynamic scene에서 surface detail을 회복하는 것에 도움을 주는 이 능력을 설명한다.(CNN이 어떻게 feature를 이용해서 recon을 할 수 있는지에 대해 말하는 듯 하다.) 그리고 각 SOTA를 찍은 방식들을 우리의 solution으로 비교하는 것을 확인할 수 있다.



### Introduction

 이번 논문에서, 우리는 real-life에서 찍힌 형태들의 multi-view shape recon의 문제를 말한다. 예를 들면 실제 옷, 움직임 등과 관련된 것들 말이다. 가상현실, 증강현실 등에 의해 현재 3D recon은 매우 인기있는 field이다.  하지만 여전히 이러한 관점에서 여전히, 필수적으로 개선되어야 할 측면은 정확도와 퀄리티이다. 

 MVS를 기반으로 하는 방법들은 feature extraction, matching, 3D shape inference등에서 좋은 수준의 퀄리티를 자랑하고 있다. 흥미롭게도, 꽤나 최근 일의 연구는 deep learning을 이용해서 연구중이다. 이러한 방법들은 기본적인 2D 방법들에 비해 2D에서도 data-driven prior를 잘 해내었고, 3D에서도 잘 해내었다. 이러한 novel MVS method들은 정적인 화면에서 촬영되고, data-aware feature상에서 좋은 성능을 제공하고 있다.

 우리의 main 목표는, 또다른 challenge인 live performance capture에서 조금 더 성능을 평가하는 것이다. 전통적인 도전들은 occlusion이나 self-occlusion과 같은, texture 정보의 부족함인 작은 영역에서도 조금 더 넓은 영역으로 시야를 넓히는 것이었다.(혹은 스포츠처럼 motion blur가 생기는 경우라던지) DTU dataset이나 ShapeNet dataset같은 경우에도 performance capture data는 아직 없다.

 이러한 형태의 data type을 일반화하는 것을 목표로, 우리는 다양한 곳에서 성공한 MVS algorithm을 적용하여 per view depth map extraction의 정확성을 유지하는 방법들의 이점을 가진 framework을 제안한다. 우리의 접근은 local volumetric unit의 추론과 함께 multi-view matching을 수행한다. 이전의 방법들과는 다르게, 우리의 volumetric unit은 주어진 view로 정의된다. occupancy를 채우는 대신, 우리는 학습을 쉽게하고 local shape pattern보다 photometric configuration 방법에 조금 더 집중하기 위해 disparity score를 infer한다. 우린 volumetric receptive field로 view를 sweep하고,(이 과정을 volume sweeping이라 한다.) geometric surface reconstruction에 의한 multi-view depth map extraction과 fusion pipeline의 알고리즘을 합친다. 이러한 전략으로, dynamic performance가 포함된 장면에서 classical MVS를 진행하는 것이 유효할 수 있다. 우린 complex sequence에서 높은 정확도를 얻을 수 있었고, CNN base와 classic한 방법들과 비교했을 때에도 좋은 성능을 보였다. 



### Related Work



### Method Overview

 Figure 2에 나와있는 것처럼, 우리는 calibrated image set을 input으로 받고, output으로 depth map을 융합함으로써 3D mesh를 내뱉는다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-1.PNG)

 pixel viewing rays를 따른 depth는 volume sweeping strategy를 이용해서 얻을 수 있다. 이 volume sweeping strategy는 ray에서의 multi-view photoconsistency를 sample하고 최대값을 확인한다. viewing ray에 놓여진 point는, 그 point 근처의 descretized된 3D volumetric patch를 사용해서 photoconsistency를 estimate한다. 3D patch의 경우, 각 지점에서, primary camera에서 온 color information은 다른 camera에서 온 incident ray와 짝이 된다. 우린 이러한 paried color volume을 수집한다. 그렇게 학습된 CNN은 3D patch에서 color sample pair로 photoconsistent configuration을 인식한다. 우리의 전략에서 key는 다음과 같다.

- 찍힌 자리에서 주는 photoconsistency를 sample한 per camera approach는 global approach와 비교했을 때 조금 더 local detail을 잘 살릴 수 있다.
- photoconsistency evaluation의 평가를 위한 3D receptive field.(몇 2D projection의 애매함을 해결하려는)
- CNN을 이용한 learning based 전략. dynamic captured scene에서 photoconsistency를 평가했을 때 전통적인 방법보다 훨씬 좋은 성능을 내는 전략.

 다음의 section들은 우리의 main contribution에 집중하고 있다. 3D volume sampling, photoconsistency evaluation의 learning base로 접근하는 방법이다. 마지막 step에서 우리는 TSDF를 사용했다.



### Depth Map Estimation by Volume Sweeping

 우리는 N개의 input과 그 input의 projection을 받고, 3D implicit form으로 융합될 수 있도록 input image의 depth map을 계산한다. 이번 section에서는 어떻게 depth map이 estimate되는 지에 대해서 설명한다. input image i에 대해서 pixel p가 주어졌을 때, 문제는 observed surface와 viewing ray의 교차점에 해당하는 지점의 depth d를 찾는 것이다. depth d에서 pixel p는 r_i(p,d)라고 notation을 정해보자. 우리의 접근 방식은 evaluation volume에 input color pair를 넣는 것이다.  전통적인 방법들이 고려되기도 하였으나, 우린 이러한 함수를 GT surface의 multiview dataset에서 배우기로 하였다. 이러한 목적은 우리가 주어진 reference camera i와 query point x에서 convolutional neural network를 만들었는데, 이는 x 주변의 color pair sample의 local volume이 scalar photoconsistency score인 \rho_i(x)로 map이 되도록 한다. photoconsistency score는 원래 자연 resolution(무한한 resolution을 말하는듯 하다)의 camera i 에서 color information과 다른 camera에서 찍혀진 color information 사이에서 만들어지는 volume color pair에 존재하는 근본적인 관계를 의미한다. 이것은 우리가 ray incidence를 구체적으로 적용하게 해준 중요한 특징이다. 의도적으로 불균형적인 자연은(아마도 위와 같은 photoconsistency 등을 의미하는듯) 자연스럽게 시각적인 결정을 할 수 있게 된다.(겹쳐진 부분이라던가, 다른 카메라는 촬영하지 못한 부분들) 이러한 것들로 인해서 대칭적인 함수가 만들어질 수는 없다.

 따라서 우리는 photoconsistency estimation을 binary classification problem으로 다루었다. 다음으로, 우리는 classification과 training에 필요한 CNN architecture 이전에 3D sampling에 대해서 자세하게 다룰 것이다. 그 이후 volume sweeping에 대해 설명한다.

__4.1 Volume Sampling__

 photoconsistency를 estimate하기 위해, 3D sampling region이 일반적인 거리에 있는 ray로 움직인다. 이러한 region에서, image에서 backproject된 color쌍이 sample된다. 

아래 Figure3 와 같이 서로 다른 camera로 본 image들 사이에서 k^3개의 sample이 만들어지고, 이는 volume처럼 형성이 된다. 이러한 3D sampling은 camera perception 특징인 resolution과 focal length에도 적용이 가능하다.

 여기서 center가 포함된(center는 아래 그림에서 frustum의 가운데 색칠된 녹색부분)면을 기준으로, 해당 점의 depth를 d라고 한다면, 여기서 부터 sample되는 depth는 d-k\lambda/2 ~ d+k\lambda/2가 된다.

 volume sampling은 reference camera에 대해서 항상 수행된다.

 volume size는 우리의 실험에서 항상 8^3이다.(k=8) 우리의 전략은 surface presence를 detect하기 위해 photoconsistent configuration pair를 찾는 것이다. 이러한 점은 voxel grid에서 진행했던 점들과는 다르다. surface detection 문제와, 조금 더 robust하고 consistent한 방법으로의 depth integrate을 고려해서 우리는 위와 같은 방법으로 간단하게 하는 방법을 선택했다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-2.PNG)

__4.2 Multi-view Neural Network__

 이전 section에서 설명했듯, N-1개의 color volume이 만들어진다. (즉, 다 합치면 (N-1)*k^3개의 color pair) 우린 2D patch대신에 3D volume으로 siamese encoder를 만들고 싶었다. 각 encoder는 pairwise volume에서 feature를 뽑아낸다. 이러한 feature는 마지막 layer로 넘어가게 된다. camera의 순서에 관계없도록 하기 위해 weight sharing과 averaging이 적용되었다.

 네트워크의 간략한 구조는 위 Figure 4에 나와있다. input은 k^3 * 6의 크기를 가진 N-1개의 colored volumes이다. (여기서 6개가 된 이유는 두 이미지의 RGB를 다 concatenate했기 때문임. Figure 3의 좌측 참조)

 Figure 4에 있는 방법을 조금씩 변형해서 다른 방식으로 진행을 해보았었으나,(예로들면 averaging pooling대신 max pooling을 이용한다거나) 성능이 좋진 않았다.

 이전에도 언급했지만, 우린 photoconsistency의 local property가 shape property보다는 적은 일관성을 필요로한다고 생각한다.

__4.3 Network Training__

 Tensorflow 사용했고, DTU Robot Image dataset 사용했음.

__4.4 Volume Sweeping__

 depth를 확인하는 방법으로, 우리는 기존에 사용하던 plane sweep algorithm을 채용했다. 모든 camera에 대해서, network score에 해당하도록 각 depth value를 test하고 가장 좋은 photoconsistent candidate을 선택했다. 일단 우리는 optical axe를 기준으로, 각 카메라가 이루는 각도에 cos을 취했을 때 0.5가 넘으면 사용했다. (즉, 가까운 카메라만 썼다 이 말) 그리고 estimate depth는 다음과 같이 구했다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-3.PNG)

 위 식에서 \rho는 network에 의해 estimate된 consistency 단위이다. d는 d_min, d_max 사이에서 값을 가지게 되고, 이 값들은 나중에 TSDF로 사용이 된다.



### Results

 __5.1 Surface Detection__

 ZNCC, CNN with planar receptive field, CNN with a volumetric receptive field를 비교했음.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-4.PNG)

__5.2 Quantitative Evaluation__

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-5.PNG)

__5.3 Qualitative Evaluation and Generalization__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-6.PNG)

 motion blur에 대한 영향이 거의 없음.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-7.PNG)

 위 그림에서, 해당 영역을 잘랐을 때, SurfaceNet은 우측 상단처럼 내부도 차 있지만, volume sweeping을 통한 photo consistency를 학습한 모델은 우측 아래처럼 내부가 비어있고 겉만 count하게 됨.



### Conclusion



### Acknowledgements



### References

Leroy, Vincent, Jean-Sébastien Franco, and Edmond Boyer. "Shape reconstruction using volume sweeping and learned photoconsistency." *Proceedings of the European Conference on Computer Vision (ECCV)*. 2018.
