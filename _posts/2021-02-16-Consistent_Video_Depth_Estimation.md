---
layout : post
title : "Consistent Video Depth Estimation"
date : 2021-02-16 +1500
description : Consistent Video Depth Estimation의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Consistent Video Depth Estimation, ACM 2020



__SUMMARY__

 single image depth estimation으로 depth 대충 따내고(다른 network를 써서), COLMAP으로 camera pose 대충 따고, 사람들은 depth에서 걸리적거리니 Mask R-CNN으로 걸러내고, video에서 각 frame마다 depth를 뽑되 이 depth가 유연하게 잘 이어지도록 만들어서 오히려 좋은 depth를 얻을 수 있다고 말하는 논문.

 video를 frame으로 다 짤라서 depth를 뽑은 후에, 이를 다시 video로 재생시키게 되면 flicker가 일어나는데,(아마도 depth가 커졌다 작아졌다 하는 부분이 있을테니) 이를 보완하기 위해 flow와 back project를 이용한 disparity, spatial loss를 이용해 network를 학습시키는 방법을 도입한 경우.



### Abstract

 우리는 monocular video에서 모든 pixel의 dense하고 consistent depth를 뽑아내는 알고리즘을 소개한다. 우리는 전통적인 SfM reconstruction을 video의 pixel에서 geometric constraint로 이용했다. 이전의 classical한 reconstruction 방법들과는 달리, 우리는 learning-based 이다. 즉, single-image depth estimation을 위한 CNN이다. test time 동안 우리는 특정한 input video에 대해서 geometric constraint를 만족시킬 수 있도록 fine-tune을 했다. 우리는 이전의 monocular reconstruction method에 비해 높은 정확도와 높은 geometric consistency를 가지고 있다는 것을 보여준다. visually, 우리의 결과는 조금 더 안정적이다. 그래서 dynamic motion이나 hand-held captured input-video에 대해서는 강한 면을 보여준다.(흔들림이 심한 경우에도 잘 잡는다는 뜻 같음) 이렇게 개선된 quality는 scene reconstruction이나 video-based visual effect에 영향을 미칠 수 있다.



### 1. Introduction

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-1.PNG)

 가장 쉽게 3D recon을 할 수 있는 방법으로는, 우리가 가진 핸드폰과 같은 것으로 영상을 찍은 후 3D recon을 하는 것이다. 왜냐하면 우리 모두가 가지고 있고, 엄청 많이 사용되는 device이기 때문이다. 하지만, 이 작업은 꽤나 어렵다.(위에서 쉽다는 뜻이, 많은 양의 data가 존재해서 쉽다고 말하는 듯 하다. absolute하게 easy하다는 것은 아닌 것을 의미하는 것 같다.)

 traditional한 방법이나 neural network를 이용하여 recon을 한 경우에도 그 결과가 썩 만족스럽지는 않았다.

 

### 2. Related Work

__Supervised monocular depth estimation.__

__Self-supervised monocular depth estimation.__

__Multi-view reconstruction.__

__Depth from video.__

__Temporal consistency.__

__Depth-aware visual effects.__

__Test-time training.__

 위와 같은 목적들이 많았음. 그런데 우리가 하는 방법과 이전의 방법은 중요한 기술이 다르다. 아래 논문의 방법은 binary object-level segmentation을 진행하고 object마다 rigid transformation을 예측했는데, 이는 자동차와 같은 static한 형태로 존재하는 것들에게는 효과적이었지만 사람과 같은 형태에는 좋은 결과를 얻지 못했다.

 Vincent Casser, Soeren Pirk, Reza Mahjourian, and Anelia Angelova. 2019. Depth prediction without the sensors: Leveraging structure for unsupervised learning from monocular videos. In AAAI Conference on Artificial Intelligence (AAAI).

 다른 논문인 아래 paper에서는 우리와 비슷한 geometric loss를 사용했지만, frame pair와 relative pose로 계산했다. 우리의 경우는 absolute pose와 long-term temporal connection을 이용하여 좋은 결과를 얻을 수 있었다. 

Yuhua Chen, Cordelia Schmid, and Cristian Sminchisescu. 2019b. Self-supervised learning with geometric constraints in monocular video: Connecting flow, depth, and camera. In International Conference on Computer Vision



### 3. Overview

 우리의 방법은 monocular video를 input으로 받아서 camera pose뿐 아니라 각 frame마다 dense, geometrically consistent depth map을 뽑아내는 것이다.(scale은 애매모호하지만) geometric consistency는 빛의 flicker현상과 같은 것 처럼 depth map이 반짝거리지 않게 만들 뿐 아니라 모든 depth map이 상호보완적인 관계를 가지게 된다. 이것은 우리가 frame에 대해서 depth와 camera pose를 정확하게 project하게 되는 것을 말한다. 예를 들어, 모든 static point의 observation은 world coordinate에 한 개의 3D 점으로 mapping된다.(static은 멈춰있기 때문에 따로 안움직인다 이 뜻 같음)

 depth recon에 대해서 captured input video는 다양한 도전적인 특징을 가지고 있다. 그들이 handheld, uncalibrated camera, blur나 shutter deform등과 같은 것들에 영향을 받기 때문이다. 빛의 상황이 좋지 않다면(예를 들자면 밤이라던가...) noise level이나 blur의 단계가 더욱 올라가게 되어, 판단하기 힘들어지게 된다.

 이전 section에서 설명한 것과 같이, scene에서 문제가 되는 부분은, 전통적인 방법들이 'hole'을 만든다는 것이다.(아마 위 Figure1의 COLMAP과 같은 경우를 말하는 것 같다.)

 최근 learning based method는 상호 보완적인 특징을 가지고 있다. 이러한 network들은 아주 challenging한 조건에서도 동작을 했지만, 각 video의 frame에 씌우는 순간 플리커 현상과 같은 일들이 일어났다.(왜냐하면 각 frame이 서로 연결되어있다고 생각하지 못하고, 따로 따로 계산을 지속적으로 하기에 constrain이 충돌할 수 밖에 없음)

 우리의 idea는 두 type의 방법의 장점을 섞는 것이다. 우리는 현재 존재하는 single-image depth estimation network들을 이용하였다.(일반적인 color 사진의 depth를 종합하기 위해 훈련되도록) 그리고 traditional한 reconstruction methods를 이용하여 video에서 geometric constraint를 뽑아냈다.

 우리의 방법은 2 stages로 이루어진다.

__Pre-processing(Section 4):__

 Video frame에 대해서 geometric constraint를 추출하여, 우리는 처음으로 traditional한 SfM 방식을 이용한다.(COLMAP을 이용) dynamic motion이 섞인 video의 pose estimation을 개선하기 위해 우리는 Mask R-CNN을 이용하여 사람들의 segmentation을 뽑고 이러한 영역을 지워 조금 더 keypoint extraction과 matching이 잘 이루어지도록 했다.(사람을 지운 이유는 video에서 dynamic한 motion의 대부분을 차지하기 때문임) 이러한 단계는 정확한 intrinsic, extrinsic camera parameter뿐 아니라 sparse point cloud reconstruction을 제공한다. 우리는 optical flow를 이용하여 frame 쌍 사이의 dense correspondence를 estimate한다. camera calibration과 dense correspondence는 아래에 설명하는 대로 geometric losses를 계산하게 된다.

 SfM recon의 두번째 역할은 우리에게 scene의 scale을 제공한다. 우리의 방법이 monocular input으로 동작하기 때문에, scale에 있어서는 조금 애매모호한 감이 있다. learning-based depth estimation network들도 scale-invarient하다. 결과적으로, network가 바뀌게 되는 양에 제한을 걸어 우리는 SfM recon의 scale을 조정하고 robust average scene으로 learning-based method를 match한다.

__Test-time Training(Section 5):__

 이번 stage에서, 우리는 pre-trained depth estimation을 fine-tune하여 particular input video의 geometrically consistent depth를 더 만들게 된다. 각 iteration에서 우리는 현재 network parameters를 사용하여 pair of frames를 sample하고 depth map을 estimate한다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-2.PNG)

 내용이 좀 있는데, 요약하면, pre-trained depth estimation network을 이용해서 무작위로 추출된 frame의 depth를 뽑아내고, spatial loss와 disparity loss를 이용해서 network에 back-propagation을 하게 된다.



### 4. Pre-processing

 __Camera registration.__

 우리는 COLMAP을 이용해서, N video frame의 각 frame i 마다 intrinsic camera parameter $K_i$, extrinsic camera parameter $(R_i, t_i)$ 뿐 아니라 semi-dense depth map $D_i^{MVS}$ 를 얻는다. 그리고 depth가 define되지 않는 부분의 pixel의 값은 0로 설정했다.(hole이 생기는 부분을 어떻게 처리했는지 설명하는 듯)

 dynamic object의 경우는 recon에서 자주 에러를 일으키기 때문에, Mask R-CNN을 이용해서 모든 frame에서 people을 segment-out 시키고(일반적인 환경에서 사람이 제일 dynamic한 object이기 때문) 이 영역을 feature extraction으로 넣게 된다.(COLMAP이 이걸 지원한다고 한다.) 우리가 들고다니는 스마트폰의 경우 distorted가 되어있진 않기 때문에(근데 얘내들이 찍은 카메라는 fisheye camera여서, rectification을 통해 distort를 없앴다고 한다. 이건 근데 모든 카메라에 대해 해당하지 않는 경우 아닌가..?), 우리는 SIPMLE_PINHOLE camera model을 사용해서 모든 frame마다 camera intrinsic을 share하고 결과적으로 faster/more robust recon을 하게 된다. 우리는 exhaustive matcher와 guided matching을 사용했다.

 __Scale calibration.__

 SfM에서의 scale과 learning-based recon으로 만들어진 model의 scale은 서로 match가 되지 않는다. 이는 두 모델이 모두 scale-invariant하기 때문인데, 우리는 여기서 geometric losses를 계산할 적합한 scale을 선택하기 위해서 SfM scale을 조정하는 것으로 선택했다. 단순히 camera translation을 곱해주는 것으로 해결이 되기 때문이다.

 구체적으로, $D_i^{NN}$이 learning-based depth estimation 방법에서 만들어진 initial depth map이라고 하자. 우리는 image i에 대해서 relative scale을 아래와 같이 계산할 수 있다.
$$
s_i = \underset{x}{median}\{D_i^{NN}(x) / D_i^{MVS}(x)|D_i^{MVS}(x) \neq 0\} \qquad (1)
$$
 위 식에서 $D(x)$는 pixel $x$ 에서의 depth를 말한다.

 우리는 여기서 scale adjustment factor $s$를 아래와 같이 설정할 수 있다.
$$
s = \underset{i}{mean}\{s_i\} \qquad (2)
$$
  그리고 모든 camera translation을 다음과 같이 update하게 된다.
$$
\tilde{t}_i = s\cdot t_i \qquad (3)
$$
__Frame sampling.__

 next step에서 우리는 특정 frame 쌍의 dense optical flow를 계산할 수 있다. 이러한 step은 $O(N^2)$의 계산을 하기 때문에 엄청 expensive하다. 그래서 우리는 simple hierarchical scheme을 통해 $O(N)$으로 줄일 수 있다.

 hierarchy의 첫번째 level은 모든 consecutive frame pairs를 가진다.
$$
S_0 = \{(i,j) | \; |i-j| = 1\} \qquad (4)
$$
 조금 더 높은 level은 frame의 sparser sampling을 포함한다.
$$
S_l = \{(i,j) | \; |i-j| = 2^l, i \; mod \; 2^{l-1} = 0\} \qquad (5)
$$
  그래서 마지막 set은 아래와 같다.
$$
S = \underset{0 \leq l \leq \lfloor log_2(N-1) \rfloor}{\bigcup}S_l. \qquad(6)
$$
__Optical flow estimation.__

 S안의 모든 frame 쌍 $(i,j)$에 대해서 우리는 dense optical flow field $F_{i \rightarrow j}$ 를 계산할 필요가 있다. flow estimation은 이미지 pair가 align이 되어 있을수록 성능이 더 좋아지기 때문에, 우리는 처음으로 frame을 homography warp로 align을 하게 된다.(RANSAC based fitting method 이용) 이후 FlowNet2를 aligned frame 사이의 optical flow 예측에 사용하게 된다. moving object와 occlusion/dis-occlusion을 세기 위해, 우리는 forward-backward consistency check를 적용하고 forward-backward error가 1을 넘는 pixel은 제거를 하게 되었다.(이 때 binary Map $M_{i \rightarrow j}$ 를 만듬) 더욱이, 우리는 flow estimation 결과들이 각 frame pair마다 약간의 overlap을 두고 믿을만 한 결과를 주지 않는다는 것을 발견했다. 그래서 우리는 $|M_{i \rightarrow j}|$가 20% 이하인 경우는 제외하였다.



### 5. Test-time Training on input video

이제 우리는 test-time training procedure를 설명할 준비가 되었다. 즉, 어떻게 우리가 particular input video에 대해서 fine-tuning된 geometric consistency를 통한 depth network가 좀 더 정확한 consistent depth를 만들 수 있는지를 설명한다. 일단 처음으로 geometric loss에 대해 설명하고, 전체적인 optimization procedure를 설명한다.

__Geometric loss.__

$(i,j) \in S$ 에서 주어진 given frame에 대해서, optical flow $F_{i \rightarrow j}$ 는 어떤 pixel pair가 같은 scene point를 가리키는지를 설명한다. 우리는 우리의 현재 depth estimate의 geometric consistency를 test하기 위해서 flow를 사용할 수 있다. 만약 flow가 올바르고 flow-displaced point $f_{i \rightarrow j} (x)$가 depth-reprojected point $p_{i \rightarrow j}(x)$와 동일하다면, depth는 당연히 consistent하다.

 우리 model의 idea는 이러한 optical flow와 depth-reprojected point의 matching을 geometric loss로 바꾸고, back-propagate을 시켜서 이전보다 정확한 depth map을 뽑는 것이다. $\mathcal{L}_{i \rightarrow j}$ 은 $\mathcal{L}_{i \rightarrow j}^{spatial}$ $\mathcal{L}_{i \rightarrow j}^{disparity}$로 구성되어있다. 이들을 정의하기 위해, 우리는 처음으로 notation에 대해 discuss해야 한다.

 $x$를 frame $i$의 2D pixel coordinate이라고 하자. 그러면, flow-displaced point는 아래와 같다.
$$
f_{i \rightarrow j } (x) = x + F_{i \rightarrow j } (x). \qquad (7)
$$
 depth-reprojected point $p_{i \rightarrow j } (x)$를 계산하기 위해, 우리는 2D coordinate을 $i$ 번째 camera coordinate system의 frame에 해당하는 3D point $c_i(x)$로 옮겨야 한다. 이 때에는 current depth estimate $D_i$ 뿐 아니라 camera intrinsic인 $K_i$를 사용한다.
$$
c_i(x) = D_i (x) K_i ^{-1} \tilde{x} \qquad (8)
$$
 여기서 $\tilde{x}$는 $x$의 homogeneous augmentation이다. 그리고 우리는 이러한 point를 $j$ 번째 camera coordinate system으로 옮기게 되는데, 그 방법은 아래와 같다.
$$
c_{i \rightarrow j} (x) = R_j^T(R_i c_i (x) + \tilde{t}_i - \tilde{t}_j), \qquad(9)
$$
 그리고 마지막으로 frame $j$ 의 pixel coordinate으로 변경하면 아래와 같다.
$$
p_{i \rightarrow j}(x) = \pi(K_j c_{i \rightarrow j}(x)), \qquad (10)
$$
 이고, $\pi ([x, y, z]^T) = [\frac{x}{z}, \frac{y}{z}]^T$ 이다.

 이러한 notation과 함께, image-space loss는 아래와 같이 쉽게 계산이 가능하다.
$$
\mathcal{L}_{i \rightarrow j}^{spatial}(x) = ||p_{i \rightarrow j}(x) - f_{i \rightarrow j } (x) ||_2 \qquad (11)
$$
 비슷하게, disparity loss는 아래와 같이 계산이 된다.
$$
\mathcal{L}_{i \rightarrow j}^{disparity}(x) = u_i | z_{i \rightarrow j}^{-1}(x) - z_j^{-1}(f_{i \rightarrow j} (x))|, \qquad (12)
$$
 $u_i$는 $i$ frame의 focal length이고, $z_i$와 $z_{i \rightarrow j}$는 각각 $c_i$와 $c_{i \rightarrow j}$의 scalar z-component이다.

 따라서 total loss는 아래와 같다.
$$
\mathcal{L}_{i \rightarrow j} = \frac{1}{|M_{i \rightarrow j }|} \sum_{x\in M_{i \rightarrow j}} \mathcal{L}_{i\rightarrow j}^{spatial}(x) + \lambda \mathcal{L}_{i \rightarrow j} ^{disparity} (x) , \qquad (13)
$$
 이고, $\lambda$는 balancing coefficient로 0.1의 값을 가지게 된다.

__Discussion__

식 (11)의 2번째 term(flow mapping)은 dynamic motion을 다룰 수 있고, 첫번째 term(depth reprojection)은 static scene을 다룰 수 있게 된다. 어떻게 이것이 높은 depth estimation을 지닐 수 있을까? 여기엔 두 가지 종류가 있다.

- Consistent motion(예를 들면 움직이는 자동차) 의 경우에는 wrong depth를 예측할 가능성이 생긴다.
- Consistent motion(손인사와 같이 움직이는 동작) 같은 경우 epipolar-align이 되어있지 않고 inconsistent motion이므로 constraint에서 conflict가 일어난다.

 하지만 실험적으로, 우리의 test-time training은 이러한 conflicting constraint에 내성이 좀 있고, 꽤나 정확한 결과를 내뱉개 되었다.

__Optimization__

 $\mathcal{L}_{i\rightarrow j}$  에서의 loss를 backpropagation해서 network의 weight를 fine-tune 했다. 

__Implementation details.__



### 6. Results and evaluation

__6.1 Experimental Setup__

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-3.PNG)

__Dataset.__

__Evaluation metrics.__

- Photometric error $E_p$
- Instability $E_s$ 

우리의 경우 camera parameter와 depth를 통해서 KLT track을 2D에서 3D로 바꿀 수 있게 된다. 그래서 이를 이용해 얼마나 video에서 depth가 unstable한지를 확인할 수 있다.

- Drift $E_d$ 

 만약 3D track들이 어쩌다 consecutive frame에서 안정적이게 된다면, error는 계속 커지게 된다. 이를 drift라 하자. 움직이는 물체에 대해서는 drift factor가 별 쓸모가 없어서, dynamic sequence에 대해서는 drift error를 계산하지 않음.

 (좀 정리하면, photometric은 다 알거고, instability는 다음 frame과의 depth map이 얼마나 유연한가, drift는 다른애들은 다 depth에 맞게 움직이는데 몇몇 애들이 안움직이는 바람에 error가 누적이 되는 경우가 생겨서 이를 얼마나 count하는가 를 말하게 된다.)

__6.2 Comparative Evaluation__

- Traditional multi-view stereo system: COLAMP
- Single-image depth estimation: Mannequin Challenge and MiDaS-v2
- Video-based depth estimation: WSVD, NeuralRGBD

__Quantitative comparison.__

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-4.PNG)

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-5.PNG)

__Visual comparison.__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-6.PNG)

__6.3 Ablation Study__

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-7.PNG)

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-8.PNG)

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-9.PNG)

__6.4 Quantitative Comparisons on Public Benchmarks__

__TUM-RGBD dataset.__

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-11.PNG)

__ScanNet dataset.__

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-10.PNG)

__KITTI dataset.__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-12.PNG)

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-13.PNG)

__6.5 Video-based Visual Effects__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-14.PNG)

 __6.6 Limitations__

- __Poses__ 우린 COLAMP을 썼는데, 이거 사실 성능 별로임.
- __Dynamic motion__ object motion이 심해지면 video가 박살남.(성능이 되게 안좋아진다는 뜻)
- __Flow__ flow 측정에 FlowNet2를 썼는데, 얘도 좀 별로임. 좀 다른 방법을 섞으려해봤으나 실패함.(sparse flow를 추가하는 방법)
- __Speed__ online processing이 안됨. 244 frame에서 708개의 sample을 뽑는 경우, NVIDIA Tesla M40 4대로 40분이 걸림.



### 7. Conclusions



### Reference

Luo, Xuan, et al. "Consistent video depth estimation." *ACM Transactions on Graphics (TOG)* 39.4 (2020): 71-1.

Vincent Casser, Soeren Pirk, Reza Mahjourian, and Anelia Angelova. 2019. Depth prediction without the sensors: Leveraging structure for unsupervised learning from monocular videos. In AAAI Conference on Artificial Intelligence (AAAI).

Yuhua Chen, Cordelia Schmid, and Cristian Sminchisescu. 2019b. Self-supervised learning with geometric constraints in monocular video: Connecting flow, depth, and camera. In International Conference on Computer Vision