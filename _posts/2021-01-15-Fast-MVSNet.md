---
layout : post
title : "Fast-MVSNet"
date : 2021-01-15 +1124
description : Fast-MVSNet, Sparse-to-Dense Multi-View Stereo With Learned Propagation and Gauss-Newton Refinement 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Fast-MVSNet: Sparse-to-Dense Multi-View Stereo With Learned Propagation and Gauss-Newton Refinement, CVPR 2020



SUMMARY: 자질구레한 방법들을 다 빼고 compact하게 만든 모델. sparse-to-dense에서 reference image를 이용해 weight를 구하고, 추가적으로 GN algorithm을 이용해서 depth를 refine한 것이 특징. weight를 구하는 이유도 보면 bilateral upsampling을 새롭게 생각한 개념.



### Abstract

 대부분의 MVS는 reconstruction quality에 focus를 하고 있다. 하지만 quality뿐 아니라 efficiency도 실제 상황에서는 중요하다. 이러한 것을 해결하기 위해 우리는 Fast-MVSNet을 소개한다. 이 방법은 MVS에서 정확하고 빠르게 depth를 얻기 위해, 새로운 방법의 sparse-to-dense, coarse-to-fine network이다. 구체적으로 우리의 Fast-MVSNet은 sparse cost volume을 만든다. 이후 sparse high resolution depth map을 dense하게 하기 위해 각 픽셀마다 작은 규모의 CNN을 하게 된다. 마지막으로 간단하면서도 효과적인 Gauss-Netwon layer를 추가하여 depth map을 훨씬 더 optimize한다. 한편으로 high-resolution depth map은 우리의 방법의 effectiveness를 보여준다. 그리고 우리의 방법은 무게도 가볍고, sparse depth representation에 의해 memory-friendly하다. Point MVSNet에 비해서는 5배, R-MVSNet에 비해서는 14배나 더 빠르다.



### Introduction

 MVS는 calibrate된 image에서 3D recon을 하는 것에 목적을 두고 있다. 하지만 오랫동안 연구가 되어 왔음에도, 이러한 것들은 근본적인 문제를 가지고 있다.

 중요한 점은 많은 feature가 image들 상에서 노출이 되어야했다. 전통적인 방법들은 photo-consistency 방법하에 해결이 되고 있었다. 하지만 몇몇 challenge들이 있었는데, regularization technique등도 있었다. 그 외 DeepCNN을 이용한 방법들이 많이 나왔다.

 그런데 이러한 multi-scale 3D CNN은 depth map이나 occupancy grid를 예측하는데에 메모리를 너무 많이 소모한다. 이러한 것ㄷ르은 high resolution MVS를 이루는 것에 방해가 된다. 그래서 R-MVSNet이나 Point-MVSNet 등이 최근에 발표가 되었는데, R-MVSNet은 memory를 적게 먹는 대신 시간이 오래 걸렸고, Point-MVSNet은 memory를 조금 더 먹는 대신 시간이 좀 덜 걸렸다. 이러한 방법들은 SOTA를 찍었으나, 만족스럽진 못했다.

 high resolution depth map의 경우에는 좋은 결과를 얻을 수 있으나 memory consumption이 심하고, low resolution depth map의 경우 memory는 적게 먹으나 detail한 결과를 얻기는 힘들었다. 이를 위한 절충안으로 우리는 sparse high resolution depth을 예측한다. 이렇게 만들어진 depth propagation은 image의 guidance로 쓰이게 된다. 그리고 추가적으로 간단하고 빠른 Gauss-Newton layer를 deep CNN feature를 잡아낼 수 있도록 사용했다. 이 layer는 coarse high resolution depth map을 input으로 받아서 dense high resolution depth map을 내뱉게 된다. 이러한 방법들은 모두 differentiable하여, end-to-end 학습이 가능하다. 그래서 우린 이러한 모델을 Fast-MVSNet이라 부른다.

 우리의 contribution은 아래와 같다.

- 새로운 sparse-to-dense, coarse-to-fine framework for MVS
- 작은 크기의 CNN을 local region에 써서 depth dependency를 배운 후 sparse depth map을 dense하게 만드는 것(이러한 방법은 bilateral upsampling에 영감을 받음)
- differentiable Gauss-Newton layer의 propose.(depth map optimize에 사용됨)
- 효과적이고 memory-friendly한데 SOTA



### Related Work

__2.1 Multi-view stereo reconstruction__

 우리 방법이 point cloud를 이용한 접근 방법과 유사함.

__2.2 Learning_based MVS__

__2.3 Depth map upsampling and propagation__

__2.4 learning-based optimization__



### Method

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-8.PNG)

 우리의 방법은 위와 같다. 첫번째로는 sparse-high-resolution dense map을 만들고, 다음으로 propagation module을 써서 dense depth map을 만든다. 이후 gauss-newton layer을 이용해 refined depth를 꺼집어낸다.

__3.1 Sparse high-resolution depth map prediction__

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-9.PNG)

 우리는 reference image에서 sparse high-resolution depth map을 estimate한다. 다른 모델과의 중요한 차이점은 위 그림에 나와있다. 이와 같은 모델은 low-resolution representation에 다음과 같은 이유로 더 적합하다고 생각한다.

- size를 유지하지 않는다면, GT를 downsampling해야함.(이런 경우에, bilinear interpolation등을 하게 되는 것은 결국 depth discontinuity를 유발하는 artifact임)
- low-resolution depth map에서 fine detail을 찾는 것은 어려움.

 우리는 추가로 다음과 같은 점에서 기존의 cost volume과의 차이점을 두고 있음.

- MVSNet보다 4배 작은 사이즈.
- MVSNet에서 256개의 depth plane을 썼지만, 우린 48 or 96을 사용했음.
- Point MVSNet이 2D CNN을 수행하는데 11개의 layer와 64 channel을 쓴 반면, 우리는 8 layer에 32 channel임.

또한, sparse representation때문에 3D CNN이 dilated convolution처럼 보이기도함.

__3.2 Depth map propagation__

 다음 단계는 high resolution이지만 sparse depth map D를 제공한다. sparse depth map에서 dense depth map을 얻는 simple한 방법은 NN을 사용하는 것이다. 그런데 original image를 전혀 고려하지 않는다는 문제점이 있다. 다른 방법으로는 bilateral upsampler를 쓰는 것이다. 식은 아래와 같다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-10.PNG)

 f는 spatial filter kernel이고, g는 range filter kernel이다. N(p)는 local k*k neighbor를 의미하고, z_p는 normalization term이다. 하지만 2개의 kernel parameter는 다양한 scene에서는 사용이 어렵고, 수동으로 고쳐줘야한다는 단점이 있었다.

 그래서 우리는 kernel이 들어간 term을 통째로 weight로 바꿨다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-11.PNG)

 해당 weight는 CNN의 output으로, data-driven 방법으로 학습한다.

__Implementation__

 sparse depth map이 dense depth map으로 propagate 한 후, 이와 평행하게 CNN은 reference image I_0를 받아서 k*k weight를 내놓는다. 그래서 위 식 (2)의 방법에 해당하는 weight는 여기서 구해질 수 있게 된다. 

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-12.PNG)

 따라서 해당 프로세스는 위와 같이 이루어진다. weight를 구하는 과정에서 우리는 MVSNet이 사용한 방법과 동일한 network architecture를 사용한다.

__3.3 Gauss-Newton refinement__

이전 까지의 단계만으로 결과를 냈을 때엔 depth map이 충분하지 않았다. 그래서 우린 depth map을 refine해야 했는데, 여러 방법 중에서 Gauss-Newton algorithm을 선택했다.

 수학적으로 봤을 때, 우리의 목표는 다음 error를 줄이는 것이다.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-13.PNG)

 F_i와 F_0는 각각 source image i와 reference image 0의 deep representation이다. 그리고 p'_i는 reprojected point p이고, F_i(p)는 해다 ㅇ점에서의 feature를 나타낸다. 여기서 p' _i는 아래와 같다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-14.PNG)

 여기서 K, R, t는 camera intrinsic, rotation, translation을 의미한다.

 여기서 E(p)를 줄이는 방법으로 Gauss-Newton algorithm을 적용했다. 구체적으로 initial depth D에서 시작해서, pixel p에 대한 residual을 계산한다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-15.PNG)

 이후의 과정은 다음과 같다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-16.PNG)

 위와 같은 방법으로 depth를 refine한다.(전형적인 Gauss-Newton refinement 방식)

 ![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-17.PNG)

 이러한 방법을 정리하면 위와 같다. (뭔가 볼때마다 GRU가 생각난다.) 이러한 방법은 gradient decent를 사용한 R-MVSNet과는 차이가 있다. 추가적으로 우리는 depth hypothese의 sample이 더이상 필요하지 않기에 훨씬 간단하다.

__3.4 Training Loss__

Loss는 다음과 같다.

 ![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-18.PNG)

 여기서 D hat은 GT를 말하고, p_valid는 valid GT depth, \lambda는 weight를 말한다. 우리는 \lambda값을 1로 두었다.

 

### Experiments

__4.1 The DTU dataset__

__4.2 Implementation Details__

__Training__

__Testing__

depth plane 96개 사용했음. 

__4.3 Results on the DTU dataset__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-19.PNG)

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-20.PNG)

__4.4 Ablation study__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-21.PNG)

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-22.PNG)

__Effectiveness of the sparse high-resolution depth map__

__Effectiveness of propagation module__

__Effectiveness of the Gauss-Newton Refinement__

__Efficiency of the Gauss-Newton Refinement__

__4.5 Generalization__

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210115-23.PNG)



### Conclusion



### Acknowledgements



### References

Yu, Zehao, and Shenghua Gao. "Fast-mvsnet: Sparse-to-dense multi-view stereo with learned propagation and gauss-newton refinement." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.
