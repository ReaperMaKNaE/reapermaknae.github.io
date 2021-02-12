---
layout : post
title : "NeuralFusion"
date : 2021-02-08 +1400
description : NeuralFusion, Online Depth Fusion in Latent Space 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### NeuralFusion: Online Depth Fusion in Latent Space, CVPR 2020



### Abstract

 우리는 feature space에서 depth map aggregation을 학습하는 접근을 이용한 새로운 online depth map fusion을 제시한다. 이전의 fusion 방법들은 SDF와 같이 겉으로 보이는 장면들을 합성한 것이나, 우리는 fusion을 위해서 feature를 학습한다. key idea는 추가적인 translator network를 이용한, fusion을 위해 사용되는 scene representation과 output scene representation 사이의 분리이다. 우리의 neural network architecture는 2 가지의 main part로 나뉘어져있다. depth와 feature fusion sub network이다. 여기서 후자는 마지막 visualization, 혹은 다른 task를 위해 TSDF와 같은 final surface representation을 만드는 translator sub-network를 따른다. 우리의 접근은 real time으로 사용이 가능하며, 높은 noise handle, 특히 outlier를 걸러내는 것에 능하다. 실험은 real과 synthetic 둘 모두 진행했으며, 많은 양의 noise와 outlier가 존재하는 scenario에서도 비교를 하였다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-1.PNG)



### introduction

 3D reconstruction에 대한 여러 것들이 나오고 있는데, 요새는 특히 TSDF에 대한 것들이 많이 나온다. 하지만 이런 TSDF는 그 자체적으로 outlier를 다루기가 힘들고, 얇은 object들에 대한 처리가 어렵다.

 이러한 근본적인 한계를 다루기 위해, outlier를 제거하는 작업을 pre 혹은 post process로 진행해온 것들이 있다. 이러한 filtering 기술들은 완성도와 정확도 사이에서의 trade-off를 balancing하는 작업을 수반하게 된다. 특히 online fusion system에서, 이러한 balance를 맞추는 것은 상당히 challenge하다. 왜냐하면 첫번째 관측과 outlier 사이의 구별은 어렵기 때문이다.(그냥 outlier를 걸러내는 것이 힘들다 라는 것 정도를 말하고 있는 듯 하다.) 결과적으로, surface reconstruction을 완성하기 위해, 가장 많이 사용되는 방법으로는 TSDF에서 volume이 아닌 부분을 쳐내는 post processing이다. 이는 그 이전의 depth 기반으로 다루어진 방법들과는 다른면이 있다. 하지만 이에 반해, 우리가 수행하는 fusion step은 내부적으로 features like confidence information 혹은 local scene descriptors를 학습하고, 뿐만 아니라 super-resolution compression과 복잡한 형태의 정보 역시 학습이 가능하다. 마지막 translation step은 이러한 학습된 scene representation을 final output으로 내놓게 된다.

 요약하면, 우리가 제공하는 contribution은 다음과 같다.

- 우리는 2개의 다른 module에서 depth fusion과 final output을 내뱉는, novel end-to-end trainable network architecture를 제안한다.
- 우리가 제안한 방법은 resource 수요와 정확도 사이를 balance할 수 있는, large feature dimension의 보다 정확한 complete fusion을 얻는다.(feature 기반으로해서 더 정확하게 했다 이 말 같음)
- 우리의 network architecture는 outlier handling을 증진시키는 것에 효과적인, end-to-end trainable outlier filtering을 수행한다.
- fully trainable함에도 불구하고, 우리의 접근은 여전히 모든 접근에서 real-time을 유지할 수 있는 global map으로의 localized update를 수행할 수 있다.(이건 뜻을 잘 모르겠다. 두 차이가 큰 상관관계가 있나?)



### Related Work

__Representation Learning for 3D Reconstruction__

 DISN, pixel2mesh, DIST, Differentiable Volumetric Rendering에 대한 설명

__Classic Online Depth Fusion Approaches.__

 TSDF Fusion이 요새 인기를 끌고 있음. 이것 말고도 surfel-based representation이나 sparse point cloud를 이용한 방법들이 존재함.

__Classic Global Depth Fusion Approaches.__

__Learned Global Depth Fusion Approaches.__

 OctNet, OctNetFusion, RayNet, SurfaceNet, 3DMV

__Learned Online Depth Fusion Approaches.__

 CodeSLAM, SceneCode, DeepFactors, DeepTAM, DeFuSR, ATLAS.

 atlas를 까는데, 단순히 weighted averaging을 사용했다고 함. 그리고 large ResNet50 backbone은 real-time capability에 한계가 있다고도 함.

 __Sequence to Vector Learning.__

 

### Method

__Overview.__

 각 time step마다, camera calbiration을 알고 있다고 하고 input depth map을 받을 때, 우리는 모든 surface information을 noise와 outlier를 없애고 잠재적으로 완벽한 observation을 하면서 global하게 scene을 융합하는 것을 목표로 했다. 우리의 마지막 output은 TSDF형태로, iso-surface(등-평면, 같은 평면 위에 존재한다는 뜻) extraction method 뿐 아니라 map의 어떤 부분이 공간을 차지하는지(occupancy)를 알 수 있는 mesh로 process될 수 있도록 하였다. 우리가 제안하는 방법의 overview는 아래 그림에 나타나있다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-2.PNG)

 key idea는 output scene representation에서 geometry fusion을 위해 scene represenatation을 분리하는 것이다.(??) 이러한 방법은 online fusion method에서 outlier를 다루는 방법의 어려움에 영감을 받았다. 그러므로, 우리는 그 어떤 pre-outlier filtering을 거치지 않고도 geometric information을 latent feature space로 fuse할 수 있다. 그리고 일련의 translator network는 latent feature space를 output scene representation으로 deocde하게 된다. 이러한 접근은 outlier를 더 좋은 성능으로 다룰 수 있고, handcrafted post혹은 pre filtering을 피할 수 있다. 더욱이, 학습된 latent representation은 조금 더 복잡하고 높은 해상도의 정보를 잡아낼 수 있게 된다. 이러한 것은 조금 더 정확한 reconstruction result를 가져올 수 있게 된다.

 우리의 feature fusion은 4개의 key stages를 가지고 있다. 첫번째 stage는 global feature volume을 local, 즉 주어진 camera parameter를 통한 affine mapping을 사용해서 view-aligned feature volume을 추출한다. 추출된 local feature volume은 새로운 depth measurement와 ray direction과 함께 feature fusion network를 통과한다. 이러한 feature fusion network는 주어진 새로운 measurement와 old state를 이용해서 local feature volume을 위한 optimal update를 예측한다. 이러한 update는 global feature volume으로 합쳐지고,(inverse affine mapping을 이용해서) 첫번째 stage로 설정이 된다. 위 3가지 stage가 fusion pipeline에서 핵심이 되고, iterative하게 depth map stream으로 실행한다. 부가적인 4번째 단계는 TSDF volume과 같은 specific scene representation으로 feature volume을 준다. 이들에 대한 자세한 사항은 아래에 있다.

__Feature Extraction.__

 iteratively fusing depth measurement의 목적은,

- 이전에 몰랐던 geometry에 대한 정보
- 이미 fuse된 gemoetry에 대한 confidence 증가
- 현재 장면에서 잘못된 부분을 고치기 위함

 이다. 이러한 목적들을 달성하기 위해, fusion process는 이전의 scene state인 $g^{t-1}$ 를 update하기 위해 새로운 measurement를 가진다. depth integration을 빠르게 하기 위해, 우리는 grid position에서 nn search를 이용해 measured depth 중에서 center depth마다의 ray에서 local view-aligned feature subvolume $v^{t-1}$ 을 추출한다.(global feature grid에서 대충 뽑아낸것(만약에 있다면. 없으면 depth map 대충 뽑아낸것만 사용) + depth map 대충 뽑아낸거를 합쳐서 $v^{t-1}$ 을 만든다는 것 같음) 이렇게 만들어진 feature volume은 fusion network로 넘어간다.

__Feature Fusion.__

 fusion network는 new depth measurement인 $D^t$ 를 local feature representation인 $v^{t-1}$ 에 fuse한다. 그러므로, 우리는 feature volume을 4개의 convolutional block을 통해 통과시킨다.(larger receptive field에서 오는 neighborhood information을 encode하기 위해) network를 설계하면서 우리는 layer normalization이 training convergence에 매우 중요하다는 것을 찾았다. 각 block의 output은 concat이 되어 나중에 larger feature volume이 되고, 이는 receptive field가 커지는 것을 의미한다. decoder 역시 4개의 block으로 되어 있다.마지막 output은 1개의 linear layer를 통과한다. 마지막으로, predicted feature update는 normalized되고 $v^t$ 로 feature integration을 통과하게 된다.

__Feature Integration.__

 update된 feature state는 extraction mapping의 inverse global-local grid correspondences를 이용하여 back integrate된다. extraction과 유사하게, 우리는 mapped feature들을 nn grid location으로 쓴다. 이러한 mapping은 unique하지 않기 때문에, average pooling operation을 이용해서 collding update를 aggregate했다.(이건 코드를 봐야 이해가 될 듯...?) 마지막으로, pooled feature들은 per-voxel running average operation을 이용해 이전의 것들과 합쳐진다.(여기서 update count는 weight로 사용한다.) 이러한 residual update operation은 직접적인 global feature를 prediction하는 것보다 안정적인 훈련과 homogenoeus latent space를 보장한다. 여기서 extraction과 integration step은 RoutedFusion에서 영감을 받았다. 하지만 그들은 nn sampling이 아닌 tri-linear interpolation을 사용했다는 것에서 우리와 차이를 보인다. SDF value 대신 feature를 extracing하고 integrating할 때, 우리는 nn interpolation이 훨씬 더 안정적이고 성능이 좋은 것을 확인할 수 있었다.

__Feature Translation.__

 마지막으로 우리는 만들어진 latent scene representation $g^t$를 visualization하도록 translate한다. 이러한 network architecture는 IM-Net에 영감을 받았다. 효율적이고 완성된 translation을 위해, 우리는 regular grid of world coordinate을 sample했다. 이후 각 sample된 point $p_i$ 에 대해서, translator는 local neighborhood의 feature에 저장된 information을 aggregate하고, TSDF $s(p_i)$ 뿐 아니라 occupancy $o(p_i)$ 도 예측한다. 일련의 network를 통과한 후 합쳐진 feature는 query point feature인 $g^t(p_i)$와 concat이 되고, 남은 translation network을 통과한다. 여기서 각 layer를 통과할 때 마다, query point feature인 $g^t(p_i)$ 를 output에 계속 concat을 한다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-3.PNG)

__Training Procedure and Loss Function.__

 shuffle을 사용했음. 사용한 loss function은 아래와 같다.
$$
\mathcal{L}=\frac{1}{n}\sum_i \lambda_1 \mathcal{L}_1(s_i, \hat{s}_i) + \lambda_2 \mathcal{L}_2(s_i,\hat{s_i})+\lambda_o \mathcal{L}_o(o_i,\hat{o_i})+\lambda_g\overline{\sigma^2_{ch}(g)}
$$
 위 식에서 $\mathcal{L}_1$ , $\mathcal{L}_2$는 각각 $L_1, L_2$의 norms를, $\mathcal{L}_o$는 예측된 occupancy에서의 binary cross-entropy이다. 여기서 $\mathcal{L}_2$는 outlier를 줄이는 데 도움이 되고, $\mathcal{L}_1$은 reconstruction의 detail을 향상시키는 것에 도움이 된다. 각 step에서 $n$ 은 모든 update된 grid location의 수를 의미한다. outlier contaminated data를 학습할 때, 우리는 n을 모든 feature grid location의 수와 같게 하는 것이 최고의 결과를 얻는 다는 것을 확인할 수 있었다. 그러므로, n은 training pipeline에서 중요한 hyperparameter이다. 더욱이, $\hat{s_i}$와 $\hat{o_i}$는 각각 GT TSDF와 occupancy value를 의미한다. latent space에서 single feature의 large deviation을 피하기 위해, 우리는 feature grid $g$를 channel-wise variance로 penalizing함으로써 regularize를 할 수 있다.



### Experiments

__Implementation Details.__

__4.1. Results on Synthetic Data__

__Datasets.__

 synthetic ShapeNet과 ModelNet의 data를 사용하였음.

__Comparision to Existing Methods.__

 DeepSDF, OccupancyNetworks, IF-Net, TSDF Fusion, Routed Fusion 등과 비교하였음.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-4.PNG)

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-5.PNG)

__Higher Input Noise Levels.__

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-6.PNG)

__Outlier Handling.__

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-7.PNG)

__4.2. Ablation Study__

__Iterative Fusion__

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-8.PNG)

__Frame Order Permutation__

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-9.PNG)

__Feature Dimension.__

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-10.PNG)

__Latent Space Visualization.__

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-11.PNG)

__Scene3D Dataset.__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-12.PNG)

__Tanks and Temples Dataset.__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-13.PNG)

__Limitations.__

 우리의 pipeline이 좋은 결과를 내고 있는 건 맞는데, 만약 비슷한 장면을 여러개 찍은 경우 그 곳으로 학습이 bias되는 경향이 있다. 이는 다양한 dataset을 이용해서 학습시키게 된다면 많이 해결될 것으로 본다.



### Conclusion



__A. Evaluation Metrics__

__Mean Squared Error(MSE) and Mean Absolute Distance(MAD)__

__Accuracy (Acc), F1 Score, Intersection-over-Union(IoU)__

__Mesh Completeness(M.C.) and Accuracy(M.A.)__

 위 방법을 섞어서 총 6가지의 실험을 했다고 함.



__B. Reproducibility__

 사실 아래 부분들은 코드를 직접 보는 것이 훨씬 좋은 것 같지만, 일단 기록.

__B.1. Details on Pipeline Architecture__

__(i) Extraction Layer.__

__(ii) Feature Fusion Network.__

- Feature Encoder
- Feature Decoder
- Feature Normalization

__(iii) Integration Layer.__

__(iv) Feature Translation Network.__

- Neighborhood Interpolator
- Translation MLP
- Network Output Heads

__B.2. Details on Hyperparameters__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210208-14.PNG)



__C. Qualtitative Results__

__C.1. Synthetic Data__

__C.2. Real-World Data__



__D. Further Evaluation__

__D.1. Generalization from a Single Object__



### Reference

Weder, Silvan, et al. "NeuralFusion: Online Depth Fusion in Latent Space." *arXiv preprint arXiv:2011.14791* (2020).