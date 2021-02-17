---
layout : post
title : "MID-Fusion"
date : 2021-02-16 +1800
description : MID-Fusion, Octree-based Object-Level Multi-Instance Dynamic SLAM의 간단한 리뷰입니다.
tag : [PaperReview]
---

### MID-Fusion: Octree-based Object-Level Multi-Instance Dynamic SLAM, ICRA 2019



__SUMMARY__

 Octree를 base로하여 memory-friendly한 model을 만든 Fusion.

 Octree, Wikipedia, [https://en.wikipedia.org/wiki/Octree](https://en.wikipedia.org/wiki/Octree)

 dynamic한 motion이 포함되어있는 사람이나 몇몇 invalid한 object를 없애버리고, valid한 object를 기준으로 모두 좌표계를 세워서 얼마나 회전했는지 등을 기반으로 TSDF를 만들어 reconstruction을 진행하게 된다. 여기서 각 object들 마다 좌표계가 다 새롭기 때문에, 기준이 되는 reference coordinate에 대해서 왔다갔다를 엄청 해야하기에 이에 대한 규약에 대한 내용이 많이 포함되어있다. 결국 전체를 요약하면, segmentation을 때리고 난 이후, target하는 목표마다 ICP와 RGB loss를 더한 error를 최소화하는 network를 만든다.

 일단 처음은 static한 물체들을 기준으로 먼저 ICP, RGB loss의 합을 최소화 시켜서 map을 만들고, 이후 moving object들에 대한 ICP, RGB loss를 더해 움직이는 물체들(사람을 제외하고, 책이나 컵 같은 물체들)에 대한 loss를 계산한다. 마지막으로 다시 segmentation에 대한 loss를 추가하여 최종 loss로 더하게 된다.(마지막은 솔직히 왜 더했는지 모르겠는데, 어찌되었든 마지막에 더했으니 좋은 결과가 나와서 사용했을 것이다.)

 이렇게 하면 사람은 제외된 채 3D reconstruction이 이루어지게 된다.(Figure 1 참조)

 여튼 NN을 쓴 건 아니고, algorithm을 소개한 논문임.



### Abstract

 우리는 oblect-level octree-based volumetric representation을 이용한 새로운 multi-instance dynamic RGB-D SLAM system을 제안한다. 이는 dynamic한 환경에서도 robust한 camera tracking을 제공할 뿐 아니라, 동시에 scene에서 임의의 물체에 대해 geometric, semantic, motion properties 등을 estimate한다. 매 새롭게 들어오는 frame에 대해서 우리는 object를 detect하기 위해 instance segmentation을 수행하고, geometric과 motion information을 이용해서 mask boundary를 refine한다. 동시에, 우리는 object-oriented tracking 방법을 사용해서 각 움직이는 object의 pose를 예측하고, static한 물체에 대해서는 robust한 camera pose를 track한다. camera pose와 object pose들의 estimate을 기반으로 두고, 우리는 각 object model의 color, depth, semantic, foreground object probability등의 연관성을 융합한 segmented mask를 현재 model과 연관을 지었다. 현재 존재하는 접근들과는 대조적으로, 우리 system은 single RGB-D camera를 이용해서 object-level의 dynamic volumetric map을 만드는 최초의 system이다. 우리의 방법은 CPU에서 2~3Hz 정도이다.(instance segmentation part를 제외하고) 우리는 이 network을 synthetic, real-world sequence에서도 실험을 진행했다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-15.PNG)



### 1. Introduction

 SLAM에 관련해서 연구가 좀 오래되어 왔는데, 여전히 chicken-and-egg problem에 휩싸여있다.(map estimation이 먼저냐, sensor pose-estimation이 먼저냐) camera pose estimation에 있어서는 static한 물체를 기반으로 얼마나 움직였는지를 역으로 알아낼 수 있는 것이 기본적인 방법이지만, 매우 강한 방법이기도 하다. 이런 경우에는 dynamic한 물체들을 outlier로 두고 계산하게 된다.

 이런 경우에 실제 상황에서는 적용이 힘든데, 실제 길거리에는 dynamic한 요소가 정말 많기 때문이다.(움직이는 차량, 움직이는 사람 등등) 

 여튼 우리는 indoor scene에서 object-level dynamic volumetric map을 최초로 제안한다. octree-based structure로 memory efficiency를 향상시켰다.

 우리의 main contributions는 아래 4가지이다.

- volumetric representation을 실행한 최초의 RGB-D multi-instance dynamic SLAM
- 불확실한 측정에도 weight를 활용하고 object tracking을 새로 parameter화 하는, 견고한 tracking method
- geometric, photometric, semantic information을 이용한 integrated segmentation
- Octree based object models를 이용한 semantic distribution과 foreground object probability의 확률적인 fusion



### 2. Related Work

 StaticFusion, Co-Fusion, DynSLAM, Fusion++.

 Fusion++와의 차이로, Fusion++는 camera pose estimate에 geometric tracking만 사용한 반면, 우리는 camera와 object의 pose를 찾기 위해 photometric tracking과 geometric tracking을 사용했다. 더욱이, 우리는 fusion과 tracking에서 더 좋은 object mask boundary를 가지기 위해 geometric, motion, existing model information을 합쳐 Fusion++에서 사용한 direct mask와는 달리 mask boundary를 직접 refine하는 것을 사용하였다. map representation에서 Fusion++는 discrete voxel grid를 바탕으로 하여 scalability에 issue를 가지게 되는데, 우리의 model은 octree structure로 memory efficient한 구조를 가지고 있다.



### 3. Notations and preliminaries

 이번 논문에서 우리는 다음과 같은 notation을 사용할 것이다.

$\underset{\rightarrow}{\mathcal{F}}_A$: reference coordinate frame

$\underset{\rightarrow}{\mathcal{F}}_B$ to $\underset{\rightarrow}{\mathcal{F}}_A$ $:=T_{AB}$  with rotation matrix $C_{AB}$ and translation vector $_{A}\mathbb{r}_{AB}$

 각 image pair를 구분할 때에 live(L)과 reference (R) image로 구분

 예를 들면, live RGB-D image는 intensity image $I_L$ 과 depth image $D_L$ 을 가짐.(여기서 2D pixel position은 $u_L$ 로 선언되고 pixel lookup(bilinear interpolation을 포함해서)은 $[\cdot]$ 으로 표현)

 perspective projection: $\pi$

 perspective back-projection: $\pi^{-1}$

 우리 system에서, 우리는 detect된 모든 object를 각각의 object coordinate frame인 $\underset{\rightarrow}{\mathcal{F}}_{O_n}$ , $(n \in \{ 0,...,N \})$ 이고, 여기서 $N$은 total number of objects를 말하며, 0은 background를 의미한다. 우리는 canonical static volumetric model이 각 object coordinate frame에 저장이 되고, 이는 multi-instance SLAM system의 기본이 된다. 우리는 COCO dataset과 관련된 각 object를 사용했고, world coordinate $T_{WO_n}$ 에 대해 현재의 pose 확률을 계산한다. 각 object는 서로 분리된 octree structure로 표시되고, 모든 voxel은 SDF value, intensity, foreground probability와 이와 연관된 weight로 저장이된다.



### 4. Method

__A. System overview__

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-16.PNG)

 우리의 network은 4개의 part로 구분되어 있다. 각각 segmentation, tracking, fusion, 그리고 raycasting이다. 

 각 input인 RGB-D image는 Mask R-CNN에 의해 instance segmentation을 수행한다. tracking의 경우 우리는 사람이 mask를 한 area를 제외하고 모든 vertex에 대해 camera를 track한 이후, 이 frame에서 어떤 object가 관찰이 가능한지, 어떤 object가 이 pose에서 발견되는지 raycast를 사용한다. 우리는 각 motion이 있는지 없는지를 확인하기 위해 motion residual을 평가하고, motion이 있든 없든 움직이는 object를 track하고 static world에 대해서 camera pose를 refine한다. 이렇게 estimate된 camera와 object의 pose, depth, color inforamtion뿐 아니라 semantic과 foreground probability를 이용해서 object model을 fuse한다.

__RGB-D Camera tracking__

 이번 part에서는 live camera pose $T_{WC_L}$ 을 estimate한다. 이는 2단계로 구성이 되어 있다.

- 사람을 제외한 모든 model vertices를 detect한다.
- 모든 static scene part를 detect한다.

 두 step은 모두 dense point-to-plane ICP residual $e_g$ 와 photometric (RGB) residual $e_p$ 를 최소화하는 역할을 수행한다.(각각은 $w_g, w_p$로 weight되어 있다.)
$$
E_{track}(T_{WC_L}) = \frac{1}{2} \begin{pmatrix} \underset{u_L \in M_L}{\sum} w_g \rho (e_g) + \underset{u_R \in M_R}{\sum}w_p \rho(e_p) \end{pmatrix}, \qquad (1)
$$
 위 식에서 $\rho$는 cauchy loss function(what is this?)을 말하고, $M$은 invalid correspondence for ICP, occlusions(for RGB), 사람들을 제외하는 Mask이다.

 ICP residual에서 우리는 KinectFusion에서 제안된 방법으로 live depth map과 rendered depth map 사이의 point-plane depth error를 최소화했다.
$$
e_g(T_{WC_L})=_Wn^r[u_r]\cdot (T_{WC_LC_L}v[u_L]-_Wv^r[u_r]), \qquad (2)
$$
 위 식에서 $_{C_L}v$는 back projection으로 된 camera coordinate의 live vertex map을 말하고,  $_Wv^r, _Wn^r$ 은 world coordinate에서 rendered vertex map과 normal map을 말한다. live depth map에서 각각의 pixel $u_L$ 에 대해, 그들의 rendered depth map에서의 연관성 $u_r$은 projective data association에서 찾을 수 있다.
$$
u_r = \pi (T_{WC_R}^{-1} T_{WC_L}(\pi^{-1}(u_L,D_L[u_L]))), \qquad (3)
$$
 여기서 $T_{WC_R}$은 reference frame의 camera pose이다.

 robustness를 최대하하기 위해, 우리는 reference frame에서 온 model의 depth map을 render하고 photometric consistency를 align하는 depth map을 사용함으로써 ICP residual을 photometric과 합칠 수 있다.
$$
e_p(T_{WC_L}) = I_R[u_R]-I_L[\pi (T_{WC_L}^{-1}(T_{WC_R}\pi^{-1}(u_R,D_R^r[u_R])))], \qquad (4)
$$
 Co-fusion과는 달리, 우리는 model에서 depth map의 quality에 noise를 덜하게 하기 위해 live frame이나 reference frame의 raw depth map 대신 rendered reference depth map을 이용해서 photometric residual을 평가한다. 이러한 방법은 camera가 surface에 가까울 때 조금 더 개선이 된다.

 우리는 ICP와 RGB residual을 합치기 위해서 measurement uncertainty weight를 소개한다. RGB residual에서 measure uncertainty는 모든 pixel에서 상수로 가정한다. ICP residual에서는 input depth map의 quality가 depth sensor와 depth range의 구조에 연관이 되어있다. 우리는 inverse covariance definition을 depth measurement uncertainty에 적용했다.(다른 Dense RGB-D SLAM에서 이걸 사용해서 따라한듯) sensor parameter인 baseline b, disparity d, focal length f, x-y plane 사이의 uncertainty $\sigma_{xy}$ , disparity direction $\sigma_z$ , standard deviation $\sigma_D$ 라하면,
$$
\sigma_D = (\frac{D_L[u_L]}{f}\sigma_{xy}, \frac{D_L[u_L]}{f}\sigma_{xy}, \frac{D_L^2[u_L]}{fb}\sigma_z) \qquad (5)
$$
inverse covariance를 사용한 ICP residual의 weight는 아래와 같이 define된다.
$$
w_p = \frac{1}{(_Wn^r)^T \; _Wn^r \sigma_D^T \sigma_D} \qquad (6)
$$
 three-level coarse-to-fine scheme에서 Gauss-Newton approach를 사용하여 cost function을 최소화한다.  추가로 space constraint를 위해 Jacobian이 필요할 수 있다.

 초기 camera tracking을 수행한 후, 우리는 view에서 visible object를 찾기 위한 raycast를 한다. 어떤 object가 motion에 있는지를 알기 위해 live frame에서 다시 한번 $E_{track}(T_{WC_L})$로 평가한다. 이것이 끝나면, RGB residual이 다음과 같이 새로 formulate될 필요가 있다.
$$
e_p(T_{WC_L}) = I_L[u_L]-I_R[\pi (T_{WC_R}^{-1}(T_{WC_L}\pi^{-1}(u_L,D_L^r[u_L])))], \qquad (7)
$$
 우리는 motion inliers를 찾기 위해 residual $E_{track}(T_{WC_L})$ 을 합쳐 threshold를 적용할 수 있다. 만약 inlier ratio가 0.9보다 아래라면 우리는 해당 object가 움직이는 것으로 판단하고 pose를 regress한다.(자세한건 아래 section에서 설명) camera pose는 정지해있는 물체들에 대해서만 refine된다.

__C. Object pose estimation__

 이번 part에서 우리는 moving object의 pose를 어떻게 계산하는지 설명한다. virtual camera를 base로한 tracking에 반해, 우리는 새로운 object-centric 접근 방법을 통한 방법을 제안한다. 이러한 방법은 초기 pose 추정에서 더 좋은 성능을 낸다. 우리는 여전히 dense ICP와 RGB tracking, weight를 섞어서 사용한다. 식 (1)과 같지만, ICP와 RGB residual의 정의만 조금 달라진다.  reference object frame에 있는 vertex map과 live object frame에 있는 vertex map을 align시킴으로써 object와 camera의 relative pose인 $T_{C_LO_L}$ 을 구할 수 있다.
$$
e_g(T_{C_LO_L}) = C_{WO_R}^{-1}\;_Wn^r[u_r] \cdot ( T_{C_LO_L}^{-1}v[u_L] - T_{WO_R}^{-1} \; _W v^r[u_r]) \qquad (8)
$$
 이하의 몇번의 계산을 거치면, $C_{C_LO_L}^{-1}(_{C_L}v[u_L]- \; _{C_L}r_{C_LO_L})$ 이 매우 작다는 것을 알게 된다. 따라서, 우리는 RGB residual을 다음과 같이 re-formulate할 수 있다.
$$
e_p(T_{C_LO_L}) = I_R[u_R]-I_L[\pi T_{C_LO_L}T_{C_LO_L}^{-1}(\pi^{-1}(u_R,D_R^r[u_R])))], \qquad (9)
$$
 위와 같은 cost function은 Gauss-Newton을 이용해서 optimize된다.($T_{C_LO_L}$ 은 $T_{C_LO_R}$ 로 초기화 되어 있음)

__D. Combined semantic-geometric-motion segmentation__

 각 RGB-D frame에 대해서, 우리는 Mask R-CNN을 semantic instances에서 사용했다. 

 뭐 아무튼 mask에 대한 내용이 나온다. object model에 대해 연관된 segmentation mask를 씌운 이후, 한번 더 motion residual에 기반한 segmentation mask를 refine하게 된다. 이 때에는 photometric residual을 live frame에서 사용한다.
$$
e_p(T_{WC_L}) = I_L[u_L]-I_R[\pi (T_{C_RO_R}T_{C_RO_R}^{-1}(\pi^{-1}(u_L,D_L^r[u_L])))], \qquad (10)
$$
 모든 것을 합하기 전에, 우리는 local segmentation mask를 기반으로 한 foreground mask를 만든다. 

__E. Object-level fusion__

 각 frame마다 우리는 depth, color, semantics와 foreground probability information을 foreground, background mask를 이용해서 object model에 합성한다. Octree-based volumetric SLAM을 이용했던 방법을 따라 relative pose, depth, TSDF를 update한다. 

 이후 octree based structure에 대해 설명하는데, 여기서는 unused voxel은 따로 초기화를 시키지 않아 memory 측에서 봤을 때엔 system이 효율적이라고 말한다.

__F. Raycasting__

 우리는 Fusion++에서 사용한 방법과 유사한 방법으로 raycasting을 썼다. Figure 2에 따르면, 우리는 최소한 4번의 raycasting이 필요하다. (depth rendering in tracking, finding visible objects, IoU calculation, visualisation) 하지만 이를 모두 진행한다면, 그 연산은 매우 expensive하다. 이 속도를 올리기 위해, 우리는 보이는 object들에 대해서만 raycast를 실시하고 나머지는 raycast를 하지 않았다. Fusion++에서 했던 것과 마찬가지로, foreground probability가 0.5를 넘는 voxel만 raycast를 진행했다.



### 5. Experiments

__A. Robust camera pose estimation__

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-17.PNG)

 위 테이블에서 VO-SF는 visual odometry and scene flow, SF는 StaticFusion, CF는 Co-Fusion, MF는 MaskFusion, DS는 DynaSLAM을 말한다. ATE RMSE는 Absolute Trajectory Error의 Root Mean Square Error를 의미한다. f3s, f3w의 의미는 TUM RGB-D dataset에서 사용되는 것으로, fr3/sitting, fr3/walking을 의미한다.(respectively)

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-18.PNG)

__B. Object reconstruction evaluation for other components__

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-19.PNG)

__C. Real-world applications__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-20.PNG)

__D. Runtime analysis__

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210216-21.PNG)

MO는 moving object, VO는 visual object



### 6. Conclusions



### Acknowledgments



### Reference

Xu, Binbin, et al. "Mid-fusion: Octree-based object-level multi-instance dynamic slam." *2019 International Conference on Robotics and Automation (ICRA)*. IEEE, 2019.

Octree, Wikipedia, [https://en.wikipedia.org/wiki/Octree](https://en.wikipedia.org/wiki/Octree)