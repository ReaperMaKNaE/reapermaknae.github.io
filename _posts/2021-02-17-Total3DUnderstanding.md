---
layout : post
title : "Total 3D Understanding"
date : 2021-02-16 +1800
description : Total3DUnderstanding, Joint Layout, Object Pose and Mesh Reconstruction for Indoor Scenes from a Single Image의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Total3DUnderstanding: Joint Layout, Object Pose and Mesh Reconstruction for Indoor Scenes from a Single Image, CVPR 2020



__SUMMARY__

 총 4개의 network이 존재한다. 첫번째는 scene의 layout을 확인하는 network, 두 번째는 2D object detector(여기서는 Faster RCNN을 사용), 세 번째는 2D object detector에서 나온 결과를 3D object detector로 바꾸는 network(이후 scale, pose로 바꾸는 역할을 수행), 네 번째는 2D object detector에서 나온 결과를 mesh로 바꾸는 network(여기서는 AtlasNet에 조금 더해서 변경)이다. 이들을 조합해서 scene recon(이라고는 하는데, 완벽하진 않다), object recon을 하는 논문이다.



### Abstract

 indoor scene의 semantic reconstruction은 scene understanding과 object reconstruction 두 가지 모두 중요하다. 현재 존재하는 work은 이 문제중 하나를 해결하거나 independent objects에 초점을 맞추고 있다. 이번 논문에서 우리는 understanding과 reconstruction 사이의 gap에 다리를 놓아서 room layout, object bbox, mesh를 single image input을 이용한 end-to-end solution을 제안한다. scene understanding과 object recounstruction을 개별적으로 해결하는 것 대신에, 우리의 방법은 holistic scene context(전체적으로 구조에 대한 문맥)을 만들고, room layout with camera pose, 3D object bbox, object mesh를 이용해 coarse-to-fine hierarchy를 제안한다. 우리는 각 component끼리의 문맥을 이해하는 것이 scene을 이해하고 reconstruction하는 것에 도움이 될 수 있다고 생각한다. SUN RGB-D dataset과 Pix3D dataset에서 진행이 되었다.



### 1. Introduction

 요새 scene recon, understanding을 하면서 사용되는 방법들이 depth나 voxel을 대표하는 경우가 많은데, voxel로 만들면 그 shape의 형태에 한계가 있음.

 아무튼 우리는 scene understanding과 scene reconstruction을 합쳐서, room layout과 camera pose, 3D object bbox를 예측하고 object mesh를 reconstruction하는 것을 수행했다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-1.PNG)

 추가로 어떤 기법들을 더하긴 하였으나 자세한건 나중에 설명하기로 하고, 요약한다면 우리의 contribution은 아래와 같다.

- single image에서 room layout, object bbox, mesh를 reconstruct하는 해결책을 propose한다. 우리가 알기로, 이런 방식으로 접근한 것은 우리가 최초임. 
- 우리는 object mesh generation에서 novel한 density-aware topology modifier를 제안함. mesh topology를 점진적으로 수정하면서 target shape을 local density하게 approximate할 수 있다.
- 우리의 방법은 object 사이의 attention mechanism과 multilateral relation을 고려하는 것이다. 3D object detection에서, object pose는 주변의 물건들과의 특별한 관계를 가지고 있다. 우리의 전략은 조금 더 나은 object location, pose, 3D detection을 위해 이러한 feature를 뽑아내는 것이다.



### 2. Related Work

 여러 방법들이 있었음. Mesh RCNN의 경우에는 mesh를 뽑아내긴 했으나, 주변과의 context를 알지 못하기 때문에 scene에서 좀 isolate되어 있는 경향이 있음. 이와 달리 우리의 방법은 모든 것들을 연결하기에 훨씬 더 정밀한 3D scene understanding이 가능함.



### 3. Method

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-2.PNG)

 우리의 overview는 Figure 2a에 나와있다. box-in-the-box 방법을 사용하였고, 3가지의 module로 이루어져있다. 첫번째는 Layout Estimation Network(LEN), 두번째는 3D Object Detection Network(ODN), 세번째는 Mesh Generation Network(MGN)이다. single image에서, 우리는 2D object bbox를 Faster RCNN을 이용해서 뽑는다. 또한, 같은 single image에서 LEN을 통해 camera pose와 layout bbox를 제공한다. Faster RCNN으로 얻어진 2D bbox를 통해 ODN은 3D bbox를 잡아내고, MGN은 mesh를 만든다. 이렇게 만들어진 것들은 모두 합쳐지면서, MGN으로 만들어진 mesh는 ODN에 의해 만들어진 bbox로 scale/place되고, world system과 camera pose로 변형이 된다. 자세한 내용은 아래에 나와있다.

__3.1. 3D Object Detection and Layout Estimation__

 layout과 object의 bbox를 학습이 가능하도록 만들기 위해, 우리는 Figure 2b처럼 box를 parameterize했다. camera pose는 $\mathbf{R}(\beta, \gamma)$로 나타나며, $\beta, \gamma$는 각각 pitch, roll을 의미한다. world system에서 box는 3D center $C \in \mathbb{R}^3$ , spatial size $s \in \mathbb{R}^3$ , orientation angle $\theta \in [-\pi, \pi)$로 표현될 수 있다. indoor objects에서 3D center $C$는 camera center까지의 거리 $d \in \mathbb{R}$과 image plane의 2D projection $c \in \mathbb{R}^2$으로 표현될 수 있다. 따라서 camera intrinsic matrix $\mathbf{K} \in \mathbb{R}^3$ 이 주어졌을 때, box의 3D center는 아래와 같이 표현된다.
$$
C = R^{-1} (\beta, \gamma) \cdot d \cdot \frac{K^{-1}[c, 1]^T}{||K^{-1}[c,1]^T||_2} \qquad (1)
$$
 2D projection center $c$는 이후 $c^b + \delta$로 나뉘어질 수 있게 된다. $c^b$는 2D bbox center를 의미하고, $\delta \in \mathbb{R}^2$은 학습된 offset이다. 따라서, 2D detection $I$에서 3D bbox corner로 가게 되는 함수는 $F(I|\delta, d, \beta, \gamma, s, \theta) \in \mathbb{R}^{3\times8}$ 이 된다. ODN은 box property인 $(\delta, d, s, \theta)$를 estimate하고, LEN은 camera pose $R(\beta, \gamma)$ 와 layout box $(C, s^l, \theta^l)$을 결정한다.

__Object Detection Network(ODN).__

 indoor 환경에서, object pose는 인테리어 설계 원리의 set을 일반적으로 따르게 된다. 이전의 work들에서는 3D box를 object-wisely하거나 pair-wise한 관계만을 고려했다. 하지만 우리는 주변 물건들과의 관계를 나타내는 multi-lateral relation을 가정한다. 우리는 아래 논문에서 2D object detection의 attention mechanism에 영감을 받았다.(자세히 보진 않았는데, 대충 object detection할 때 막 하는게 아니라 context도 본다는 뜻 같다)

Han Hu, Jiayuan Gu, Zheng Zhang, Jifeng Dai, and Yichen Wei. Relation networks for object detection. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pages 3588–3597, 2018.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-3.PNG)

 ODN의 경우 network의 형태는 위와 같다. ResNet-34를 통해 feature map을 뽑아내고, 뽑아낸 feature에서 attention sum을 수행하게 된다. 이는 target과 다른 물체들 사이에서 오는 형태의 유사함과 geometry에 가중치를 두어 sum이 되는 형태이다. 이후 우리는 element-wise 하게 relational feature를 target에다 더하고 각 box parameter인 $(\delta, d, s, \theta)$를 regress하게 된다.(MLP를 이용해서) indoor reconstruction에서, object relation module은 physical world에서 중요한 것들을 가르쳐주게 된다. 이러한 자세한 내용은 ablation study에서 관찰할 수 있다.

__Layout Estimation Network(LEN).__

 LEN은 camera pose $\mathbf{R}(\beta, \gamma)$와 그 3D box인 $(C, s^l, \theta^l)$을 world system에서 예측한다. 이번 파트에서, 우리는 ODN에서 relational feature를 지운 것과 유사한 architecture를 사용했다. $(\beta, \gamma, C, s^l, \theta^l)$은 ResNet 이후 fc layer로 regress된다. 3D center $C$는 average layout center의 offset을 학습하면서 예측이 된다.

__3.2 Mesh Generation for Indoor Objects__

 우리의 mesh generation network은 최근 work인 Topology Modification Network(TMN)과 충돌하게 된다. TMN은 mesh topology를 deform, modify하여 object shape을 대략적으로 알아내게 된다. 하지만 이는 object mesh에서 threshold scale을 어떻게 잡느냐에 따라 그 결과가 좋지 않는 경우가 꽤나 있었다.(추후 나올 Figure 5의 (d), (e)를 참조하면 알 수 있음) 이러한 이유는, 서로 다른 category에 속해있다면 shape의 크기가 상당히 다양하게 존재하는 것이 그 원인이라고 보고 있다. 또 다른 이유로, 복잡한 background와 occlusion이 정확한 distance value를 추정하는데 실패하는 것이 원인이다.

__Density v.s. Distance.__

 topology modification을 쓰기 위해 엄격한 distance threshold를 사용한 TMN과는 달리, 우리는 face가 유지가 되던 말던 local geometry로 결정이 되어야 한다고 생각한다. 그래서 우리는 local density를 base로 한 mesh modify의 방법을 제안한다.

$p_i \in \mathbb{R}^3$를 reconstructed mesh의 point로 두고, $q_i \in \mathbb{R}^3$가 GT와의 nearest neighbor에 해당한다고 하자. 우리는 $p_i$가 GT mesh에 얼마나 가까운지 예측할 수 있는 binary classifier $f(*)$ 를 설계했다.
$$
f(p_i) = \begin{cases} False \qquad ||p_i - q_i||_2 > D(q_i) \\ True \qquad otherwise \end{cases}
$$

$$
D(q_i) = \underset{q_m, q_n \in N(q_i)}{max\;min}||q_m - q_n||_2, \; m \neq n \qquad(2)
$$

 에서 $N(q_i)$는 GT mesh에서 $q_i$의 neighbor를 말하고, $D(q_i)$는 local density를 말한다. 이러한 두 가지 분류는 다음과 같은 이유로 설계되었다. shape approximation에서, point는 GT에서 neighbor의 수인 $N(*)$ 에 속해있다면 point는 유지되어야 한다는 점이다. 우리는 이러한 것이 다른 mesh generator들 보다 훨씬 효과적인 것을 깨달았다.(GT와 nearest neighbor의 개수를 이용해서 쓸데없는 부분을 날릴지 말지를 결정하는 것을 의미하는 듯 하다.)

__Edges v.s. Faces.__

 face를 제거하는 것 대신에, 우리는 topology modification에서 mesh edge를 잘라내는 방식을 선택했다. 우리는 mesh edge를 랜덤하게 sample해서 classifier $f(*)$를 이용해 edge를 잘라내었다. 

__Mesh Generation Network.__

 우리의 mesh generation network은 아래와 같다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-4.PNG)

 이 network은 2D detection을 input으로 받고 ResNet-18을 이용해서 image feature를 만든다. 이렇게 만들어진 feature는 one-hot vector로 detect되고, image feature와 concat이 된다. 우리는 만들어진 것을 AtlasNet(scannet dataset을 뽑는 것이 아니라 shapenet dataset으로 쓴 atlasnet이 있음)을 통해 plausible shape을 만든다. edge classifier는 shape decoder와 같은 architecture로 구성이 되어있는데, 이 것과 boundary refinement를 통해 final mesh를 output으로 내뱉게 된다.

__3.3 Joint Learning for Total 3D Understanding__

 이번 section에서 우리는 어떻게 loss function을 잡았고, end-to-end training을 했는지에 대해 설명할 것이다.

__Individual losses.__

 ODN은 3D object box를 recover하기 위해 $(\delta, d, s ,\theta)$를 예측하고, LEN은 camera pose에 따른 world system의 3D object를 변형하기 위한 layout box인 $(\beta, \gamma, C, s^l, \theta^l)$을 만든다. angle과 length는 L2 loss로 regress된다. 우리는 classification과 regression loss를 사용해서 그들을 inline으로 유지할 수 있도록 한다.( $(\theta, \theta^l, \beta, \gamma, d, s, s^l)$을 최적화시키기 위한 loss: $\mathcal{L}^{cls,reg} = \mathcal{L}^{cls} + \lambda_r \mathcal{L}^{reg}$) 여기서 $C$와 $\delta$를 미리 계산된 center와의 offset으로 계산하기 위해, 우리는 그들을 L2 loss로 예측한다. MGN에서 우리는 Chamfer loss $\mathcal{L}_c$와 edge loss $\mathcal{L}_e$, boundary loss $\mathcal{L}_b$ 를 각각 10, 50, 30 으로 적용했다.(mesh modification에서 edge를 분류하기 위한 cross entropy loss $\mathcal{L}_{ce}$ 와 함께)

__Joint losses.__

 우리는 ODN, LEN, MGN 사이의 joint loss를 다음 두 가지와 같은 이유를 바탕으로 정의했다. 첫 번째로 camera pose estimation은 3D object detection을 개선시키고, 이는 반대도 성립한다. 두 번째로 현재 scene에서 부피를 차지하는 object mesh는 반드시 3D detection에 도움이 되고 그 반대도 성립한다. 첫 번째의 경우를 적용하기 위해 우리는 cooperative loss $\mathcal{L}_{co}$를 설정했고(GT와 예측된 coordinate 사이의 consistency를 위해), 두 번째의 경우를 적용하기 위해 우리는 scene에서 그들의 point cloud에 가까운 reconstructed mesh를 요구했다. 여기서 mesh coordinate과 GT를 align함으로써 global constraint가 존재하게 된다. 우리는 global loss를 chamfer distance의 일부인 아래로 정의했다.
$$
\mathcal{L}_g = \frac{1}{N}\sum_{i=1}^N \frac{1}{|\mathbb{S}_i|}\sum_{q\in \mathbb{S}_i}\underset{p\in\mathbb{M}_i}{min} ||p-q||_2^2, \qquad (3)
$$
 위 식에서 $p$와 $q$는 각각 world system에서 $i$-th object의 mesh $\mathbb{M}_i$와 GT surface $\mathbb{S}_i$를 의미한다. N은 object의 수를 의미하고, $|\mathbb{S}_i|$는 $\mathbb{S}_i$의 point number를 나타낸다. single object mesh들과는 달리, real-scene point cloud는 현재 coarse하고 부분적으로 커버가 되기에, 우리는 $\mathcal{L}_g$를 정의하는데에 있어서 Chamfer distance를 사용하지 않았다. 모든 loss function을 더하면, 식은 아래와 같이 된다.
$$
\mathcal{L} = \sum_{x\in\{\delta, d, s, \theta\}}\lambda_x\mathcal{L}_x + \sum_{y\in\{\beta, \gamma, C ,s^l, \theta^l\}}\lambda_y \mathcal{L}_y + \sum_{z\in{c,e,b,ce}}\lambda_z \mathcal{L}_z + \lambda_{co}\mathcal{L}_{co}+\lambda_g \mathcal{L}_g \qquad (4)
$$
 첫번째 3개의 term은 각각 ODN, LEN, MGN의 loss를 의미하고, 나머지 2개는 joint term에서의 loss를 의미한다.

$\{\lambda_*\}$는 그들의 중요도를 balance할 weight를 의미한다.



### 4. Results and Evaluation

__4.1. Experiment Setup__

__Datasets:__

 SUN RGB-D dataset과 Pix3D dataset을 이용하였다.

__Metrics:__

 IoU와 AP를 이용하였음.

__Implementation:__

 우리는 2D detector를 COCO dataset에서 학습시키고, SUN RGB-D에서 fine-tune 시켰다. MGN에서 template sphere는 2562개의 vertice를 가진다. 

__4.2. Qualitative Analysis and Comparison__

__Object Reconstruction:__

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-5.PNG)

__Scene Reconstruction:__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-6.PNG)

__4.3. Quantitative Analysis and Comparison__

 우리는 4가지 측면에서 성능을 비교했다.

- layout estimation
- camera pose prediction
- 3D object detection
- object and scene mesh reconstuction

__Layout Estimation:__

__Camera Pose Estimation:__

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-7.PNG)

__3D Object Detection:__

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-8.PNG)

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-9.PNG)

__Mesh Reconstruction:__

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-10.PNG)

__4.4 Ablation Analysis and Discussion__

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-11.PNG)

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-12.PNG)

 각 version에 따른 저자의 해석은 아래와 같다.

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210217-13.PNG)



### 5. Conclusion



### Acknowledgments

 

### Reference

Nie, Yinyu, et al. "Total3dunderstanding: Joint layout, object pose and mesh reconstruction for indoor scenes from a single image." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.

Han Hu, Jiayuan Gu, Zheng Zhang, Jifeng Dai, and Yichen Wei. Relation networks for object detection. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pages 3588–3597, 2018.
