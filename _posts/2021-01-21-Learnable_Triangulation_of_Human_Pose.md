---
layout : post
title : "Learnable Triangulation of Human Pose"
date : 2021-01-21 +0300
description : Learnable Triangulation of Human Pose 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Learnable Triangulation of Human Pose, ICCV 2019



SUMMARY: Multi-view 3D human pose의 정확도를 더 올리는 방법으로, algebraic triangulation과 volumetric triangulation을 적용했음. 그 결과 SOTA 찍음. input은 image set을 받게되고, output으로는 joint(관절)부분을 서로 연결하여 표현하게 됨.

 volumetric triangulation의 경우에는, 3D map에 뽑힌 feature(descriptor의 일종)을 때려넣는데, 이 때 pose를 알고 있으니 projection할 때 사용하고, 여기서 뽑힌 3D volume을 V2V network 통과시켜서 최종 모델을 얻게 됨.(posed RGB image set이 input)



### Abstract

 여러 2D view에서 3D information을 합친 learnable triangulation method를 base로 한 multi-view 3d human pose의 두 가지 새로운 해법을 제시함. 첫 번째 해법은 basic한, input image에서 얻은 confidence weight를 더한 differentiable algebraic triangulation 방법이다. 두 번째 방법은 2D backbone feature map에서 volumetric aggregation 의 새로운 방법을 기반으로 한다. aggregated volume은 final 3D joint heatmap을 생성하는 3D convolution으로 refine되며, human pose를 implicit modeling할 수 있도록 한다. 결정적으로, 두 방법 모두 end-to-end differentiable하여 바로 target metric을 optimize할 수 있다. 우리는 여러 dataset에서 유용함을 확인하였고, Human3.6M dataset에서 SOTA를 찍었다.



### Introduction

 Computer vision에서 사람의 pose를 3차원에서 estimation하는 것은 근본적인 문제 중 하나이다. 이는 행동 인식, 스포츠, 컴퓨터 보조의 생활 등 다양한 곳에서 적용이 가능하다. 대부분의 노력은 monocular 3D pose estimation으로 이루어지고 있다. 많은 progress에도 불구하고, monocular로 3D pose estmation을 하는 것은 아직 완전히 해결되기엔 멀었다. 그래서 우리는 심플하면서도 도전적인 multi-view 3D human pose estimation에 도전하게 되었다.

 multi-view human pose estimaion은 두 가지 흥미로운 이유가 있다.

- multi-view는 틀림없이 monocular 3d pose estimation의 gt를 얻는 것에 있어서 최고의 방법이다. marker-based motion captuer나 visual-inertial method 등의 방법들이 풍부한 pose를 잡는것에 한계가 있기 때문에 상당히 의미가 있다. 불리한 면으로는, 이전의 multi-view triangulation을 이용한 방법들이 3d GT를 거의 다 만들어놨다는 점이다. 이러한 점에서 새로운 dataset을 얻는 것은 상당히 도전적인 면이 있고, 덜 정확한 triangulation을 보정해주는 역할에 참여하게 되기도 한다.
- multi-view human pose estimation에서 두 번째 동기는, 몇몇 사람들의 동작은 real-time으로 얻어야 한다는 점이 있다. 이것은 multi-camera setup이 다양한 방면에서 적용이 가능하기 때문이다. 이러한 setup들은 아직 몇몇 view에 대해서만 행해지고 있다. 하지만 이미 monocular에서 어느정도 해결이 되고 있는데, 여기서 정확도를 더 올리는 방법으로 multi-view에서 pose estimation을 하는 것은 중요한 것이 된다.

 우리는 여기서 불균형적인 적은 attention을 받아 pose estimation을 하는 것을 이야기 한다. 우리는 두 가지의 multi-view human pose estimation에 관련된 방법들을 propose하고 investigate한다. 두 가지 모두 다 learnable triangulation의 idea에 놓여져있어, 정확한 3d pose를 예측하는 것에 필요한 view의 수를 줄여주는 역할을 했다. 학습 동안, 우리는 marker based GT와 meta GT를 썼다. 이를 이용한 방법은 다음과 같다.

- learnable camera-joint confidence weight와 algebraic triangulation에 base한 간단한 접근
- human pose를 model하는 것을 가능하게 하는 다른 view에서 오는 정보를 dense geometric aggregation하는 것에서 오는 복잡한 volumetric triangulation

 결정적으로, 두 방법 모두 differentiable하며 end-to-end training이 가능하다.

  그리고 아래에서 우리는 monocular, multi-view human pose estimation에 대해 이야기하고, 새로운 learnable triangulation method에 대한 자세한 내용을 discuss한다. experimental section에서 우리는 Human3.6M dataset과 CMU Panoptic dataset을 사용했다.



### Related Work

__SIngle view 3D pose estimation__

생략

__Multi-view 3D pose estimation__

 multi-view 3d human pose estimation은 monocular 3d human pose estimation의 gt를 위해 개발되어왔다. 몇몇 일은 2D coordinate의 joint를 모두 합쳐서, fc cnn을 보내 global 3D joint coordinate을 예측하는 것을 훈련하기도 하였다. 이러한 것은 다른 view에서 얻게 되는 정보를 효율적으로 훈련하였지만, 새로운 camera setup에 대해서는 새로운 학습을 필요로 했고, overfitting이 자주 일어났다.

 몇몇 work들이 multi-view setup에서 volumetric pose representation을 사용하기 시작했다. 특히, 2D keypoint probability heatmaps의 unprojection을, non-learnable aggregation의 결과로 volume을 만들어 활용하려는 work도 있었다. 우리의 work은 이 두 가지 모두와 다르다.

- 우리는 volume 안에서 learnable way로 정보를 제공한다.
- end-to-end training이 가능하기에, 2D heat map의 영상해석능력에 대한 필요를 완화할 수 있다. 

 이러한 방법은 2D detector에서 오는 pose hypotheses를 전송할 수 있다.(volumetric aggregation에)

 다른 방법 중에서 2D joint의 coordinate에서 3D pose를 이해하기 위해 multi stage로 3D pose를 찾아내려는 접근도 있었다. 첫 번째 stage에서는 모든 ivew의 image를 backbone convolutional neural network를 통과시켜서 2D joint heatmap을 얻고, heatmap에서 최대치의 position을 이용해 3D pose를 이해하려고 했다. 다음의 각각 stage에서는 3D pose가 모든 camera view로 reproject되고 이전의 layer에서 예측한 것과 융합된다.(residual 구조를 말하는 듯) 다음으로 3D pose는 다시 heatmap의 maximum value의 position에서 re-estimate되고, 이러한 process가 반복된다. 이러한 procedure는 human pose에서 전체적으로 직접적이지 않은 방법을 가능하게 한다. 이에 반대로, 우리는 3D prediction에서 2D heatmap에 gradient flow를 사용하지 않고, 3D coordinate에 대한 직접적인 신호도 없다. 최근, projection하지 않고 SOTA를 찍은 몇몇 work들이 제안 되었다. 

 

### Method

 일단 우리는 C camera에서 동기화 된 이미지를 받고, projection matrix를 알고 있다고 한다. 우리의 목적은 global position을 timestamp t에서 estimation하는 것이다.

 각 frame에서, 우리는 2D human detectors 혹은 GT에서 bbx를 얻어 image를 crop한다. 그리고 이 cropped image인 I _ c를 deep convolutional neural network backbone에 넣는다.

 이 backbone network는 ResNet-152 network이고, weight는 \theta로 구성이 되어 있으며, output은 g _ \theta로 되어있다. 다음의 2 section은 multi view에서 얻어진 aggregation information에서 joint 3D coordinate을 아는 서로 다른 방법을 설명한다.

__Algebraic triangulation approach.__

 algebraic triangulation baseline에서, 우리는 각 joint j를 독립적으로 진행했다. 이러한 접근은 j-joint's backbone heatmap에서 얻어진 2D position을 triangulation하는 것에 대한 접근이다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-1.PNG)

 여기서 얻어진 triangulation은 아래와 같이 표기한다 할 때, 이를 이용한 식은 그 아래와 같다.
$$
H_c,j =h_\theta (I_c ) _j
$$
![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-2.PNG)

 위 식에서 \alpha는 inverse temperature parameter이다. 이후 우리는 위 식을 이용해 heatmap의 center of mass를 joint position으로 설정하도록 계산한다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-3.PNG)

 heatmap에서 maximum의 index를 얻는 것보다 중요한 soft-argmax의 역할은 joint x에 대한 output 2D position에서 heatmaps H_c로 gradient flow back이 가능하다는 점이다. backbone이 다른 soft-argmax를 이용한 loss로 pretrain되어있기 때문에, 우리는 heatmap을 수정하기 위해 inverse temperature parameter인 alpha를 추가했다. 이로 인해 training 초기의 soft-argmax는 maximum position에 가까운 output을 내놓게 된다.

 2D estimate인 x _ c,j에서 joint의 3D position을 알기 위해, 우리는 linear algebraic triangulation 접근 법을 사용했다. 이 방법은 overdetermined system을 해결하기 위해 joint y _ j의 3D coordinate을 찾는 것을 줄여준다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-4.PNG)

 위 식에서 A는 full projection matrix의 성분과 x _ c,j로 이루어진 matrix이다.(즉, epipolar constraint에 대한 설명을 하고 있는 듯 하다.)

 전통적인 triangulation algorithm은 각 view에서 보는 joint coordinate은 모두 독립이 되어 있다고 하지만, 몇몇 view에서는 occlusion과 같은 것에 의해 평가가 될 수 없고 마지막 triangulation result에 불필요한 degradation을 유발한다. 이는 다른 view에서 보이는 algebraic reprojection error를 최적화 하는 방법을 약화시키는 경향이 있다. RANSAC과 Huber loss를 이용해서 outlier를 쳐내는 방법이 해결이 될 수도 있지만, 그들만의 단점이 또 있다. 예를 들어서, RANSAC은 추가로 포함되는 camera들의 gradient flow를 완전히 쳐내기도 한다.

 이러한 문제를 해결하기 위해, 우리는 learnable weight w _ c를 해당 matrix에 추가했다. 그리고 그에 관련한 내용은 아래와 같다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-5.PNG)

 이러한 식은 각 camera view가 nerual network branch에 들어가서 학습을 할 수 있도록 도와준다.

 식 (4)의 경우는 SVD(Singular Value Decomposition)에 의해 풀릴 수 있다. (위 식에서 각 변수가 의미하는 바로 w는 joint j에 대한 confidence vector, A는 2D joint coordinate와 camera parameter의 조합이 담긴 matrix, y ~는 joint j의 target 3D position을 의미한다.)

__Volumetric triangulation approach.__

 baseline algebraic triangulation의 주요한 단점으로는 서로 다른 camera에서 온 image I _ c는 서로 독립적으로 움직이고, sparse information만 조합하기에 3D human pose를 추가하는 것이 어려웠고 wrong projection matrix를 가진 camera를 쳐 낼 방법이 없다.

 이러한 단점을 해결하기 위해, 우리는 더욱 복잡하고 powerful한 triangulation procedure를 소개한다. 

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-6.PNG)

 우리는 2D backbone에서 나온 feature를 3D volume에 unproject한다.(위 그림처럼) 이 과정에서 projection ray를 따른 2D network의 output에 따라 3D cube를 채우게 된다. multiple view에서 얻게 된 cube는 서로 aggregate된 이후 진행된다. volumetric triangulation approach들 처럼, 2D output은 joint heatmaps처럼 interpretable하지 않기 때문에, H _ c를 쓰기보다는(triangulation을 나타내는 H _ c) o ^ \gamma kernel과 K output channel로 구성된 trainable network을 heatmap backbone의 input에 적용했다. 

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-7.PNG)

 volumetric grid를 적용하기 위해, 우리는 L * L * L size의 3D bbx를 적용했다. 

 우리는 64x64x64x3의 size로 구성된 coordinate volume을 각 voxel의 중심에 해당하는 global coordinate으로 채웠다.

 각 view에서, 우리는  coordinate volume 내에 있는 3D coordinate을 plane에다 project해서 camera c에 대한 project volume(2D coordinate으로 구성되어있기에 64 * 64 * 64 * 2)을 만들고, 이 project volume을 map인 M _ c,k에 맞게 bilinear sampling을 해서 각 feature output의 채널에 대한 view k를 만든다. 이를 정리하면 아래 식과 같다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-8.PNG)

 {}가 의미하는 것은 bilinear sampling이다. 우리는 이렇게 뽑힌 view들을 aggregation하는 방법으로 3가지 방법을 선택했다.

 첫번째는 그냥 쌩으로 더하기

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-9.PNG)

 두번째는 normalized된 confidence d 항을 곱해서 나온 voxel data를 더하기

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-10.PNG)

 세번째는 maximum을 계산할 때 사용하는 relax version을 사용하는 방법인데, 아래와 같다.

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-11.PNG)

 모든 camera에 대해서 각 individual voxel의 softmax를 계산하고, scalar d와 유사한 역할을 하는 volumetric coefficient를 사용한다.

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-12.PNG)

 그리고 이걸 곱해서 합한다. 이렇게 나온 input은 learnable volumetric convolutional neural network인 u 로 가서, voxel output을 내뱉게 된다.(V2V와 유사함) 이 voxel output을 softmax를 취해서 정리하면,

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-13.PNG)

 그리고 3D joint의 위치를 알리는 방법으로 무게중심을 구하게 되면, 아래와 같다.

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-14.PNG)

(했던걸 계속 또 적용하고 하는 것으로 보이긴 하나, 결과를 보자면 이 친구들의 목적은 joint를 구하는 것이고, 이는 descriptor가 표시한 것들의 중심이 된다. 이게 center of mass, 무게중심을 계산하는 식과 거의 유사하기에 같은 것이 반복된다. 그러다보니 관절의 연결같은 경우도 위 network를 통해 이루어지는 듯 하다.)

__Losses__

  algebraic triangulation loss와 volumetric triangulation loss를 각각 적용했고, algebraic triangulation loss는 MSE를,  volumetric triangulation에서는 L1 loss에 \beta 가 포함된 항을 추가했는데, 후자의 경우는 불충분한 dataset 때문인지, 불완성된 heatmap을 보강하기위해서 추가한 항이라고 한다. (이런다고 보강이 되는게 아직 감이 잘 안잡힌다.)



### Experiments

__Human3.6M dataset__

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-15.PNG)

__CMU Panoptic dataset__

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-16.PNG)

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-17.PNG)

![img18](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210121-18.PNG)



### Conclusion



### Acknowledgement.



### References

Iskakov, Karim, et al. "Learnable triangulation of human pose." *Proceedings of the IEEE International Conference on Computer Vision*. 2019.

