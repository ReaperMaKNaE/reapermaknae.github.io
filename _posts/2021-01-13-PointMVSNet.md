---
layout : post
title : "Point-MVSNet"
date : 2021-01-13 +2027
description : Point-Based Multi-View Stereo Network의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Point-Based Multi-View Stereo Network, ICCV 2019



 요약하면, 기존에 사용하던 방식은 3D Cost volume 방식의 network에서 벗어나 Point Cloud 형태를 바로 학습시키는 방법. 이런 형태의 network를 만들기 위해 Point Flow라는 새로운 것을 도입했음. 덕분에 네트워크가 많이 가벼워졌다고 함. 



### Abstract

 우리가 소개할 network는 point-based deep framework인 Point-MVSNet이다. 다른 network과는 달리, 우리 network은 target scene을 바로 point cloud로 잡는다. 조금 더 구체적으로는, coarse-to-fine 방법으로 depth를 예측한다. 일단 먼저 coarse depth map을 만든 후, 이를 point cloud로 만들고 GT와 depth를 iteration 돌려서 refine한다. 우리가 만든 network는 3D geometry와 2D texture information를 feature-augmented point cloud로 합쳐 이 둘을 같이, 효과적으로 저울질 할 수 있다. 그리고 각 point마다 3D flow를 estimate한다. 이러한 point-based architecture는 높은 정확도를 가지고 있고, 연산에 있어서 효율적이며, cost volume base로 이루어진 구조들보다 유연하다. 데이터셋은 DTU과 Tanks and Temples dataset을 사용했다.



### Introduction

 3D CNN을 이용한 recon이 발달되어 왔으나, 이러한 형태는 많은 memory를 필요로 하게 되고, 그에 따라 optimal performance를 얻기 힘들었다. 이러한 문제를 Octree structure로 해결하려는 방법도 있었으나, error가 누적되는 결과를 가지고 왔다.

 이번 논문에서 우리는 3D resolution이 높을 때 특히 효율적인 point cloud로 바로 output을 내는 point cloud multi-view stereo network를 소개한다. 우리의 framework은 two step으로 이루어져 있다.

- 전체적인 scene에서 object surface를 잘라내기 위해 초기 coarse depth map은 상대적으로 작은 3D cost volume으로 만들어진 후 바로 point cloud로 변환된다.
- 우리의 PointFlow model은 initial point cloud에서 정확도와 dense point cloud의 반복학습을 적용할 수 있다.

 ResNet과 유사하게, 현재의 iteration과 GT와의 차이를 예측하기 위해 PointFlow에 residual을 적용했다.(?? 이거 해석이 좀 이상한데, 아무튼 residual항을 추가했단 말 같음) 3D flow는 예측된 point cloud에서 오는 geometry prior와 2D image appearance 흔적에서 오는 것을 기반으로 만들어 진다.

 우린 이러한 point based MVS Network가 이전 3D volume들을 이용한 framework보다 정확도, 효율, 유연성등에서 이점이 있다는 것을 확인했다. 우리의 방법은 3D space에서 잠재적인 surface point를 샘플하는데, 이는 high precision recon에 필수적인 surface structure continuity를 유지하게 해 준다. 더욱이 전체 3D space가 아니라 object surface 근처에서의 정보만을 유효하게 치기 때문에 computation이 매우 효율적이다. 최근 adaptive refinement scheme은 우리가 target으로 하는 point cloud recon을 조금 더 dense하게 만들어주고 coarse resolution의 장면에서 first peek(?)을 하게 해주었다.(first peek이라는게 sneak peek과 유사한 표현인 것 같은데, 완성되기 전에 그것을 볼 기회가 있는 것을 의미한다고 한다. 여기에서는 아마도 coarse resolution임에도 불구하고, high resolution image의 형태를 확인할 수 있다는 의미로 쓰인 것 같다.) 이러한 점은 interaction-oriented robot vision과 같은 장면들에서, 전력을 아낄 수 있는 점 등 유연성을 가져다 줄 수 있음.

 우리의 방법은 DTU와 Tanks and Temples에서 SOTA를 찍음. 추가로 오목한 depth inference같은 것에서도 우리가 제안한 방법이 어느정도 먹힐 수 있음을 보여줄 거임.



### Related Work

__Multi-view Stereo Reconstruction__

 다양한 방법들이 적용되어, multi-view photometric consistency나 regularization optimization을 iterative하게 update하는 방식들도 사용되었다. 우리가 사용한 iterative 방법은 어찌보면 traditional한 방법과 유사하나, learning-based algorithm으로 robustness를 개선하고 지루한 parameter tuning 단계를 피하는 것을 이루었다.

__Learning-based MVS__

 image recognition task에서 deep learning이 최근 성공적인 것에 영감을 받아 연구자들이 이를 stereo recon에도 적용하기 시작했다. 초기에는 2D를 3D로 어떻게 만드는가에 대한 고민이 많았으나, 3D cost volume과 이를 이용한 방법들이 많이 등장하면서 많은 발전이 있었다. 3D cost volume의 key advantage는 3D geometry가 network에 잡혀 2D를 base로 한 결과보다 그 성능이 우수하다. voxel을 대신해서 이번 논문에서는, 3D CNN으로 인한 연산의 짐(load) 없이 point-based network를 이용한 방법을 제안한다.

__High-Resolution MVS__

 고해상도 MVS는 robot manipulation이나 증강현실 등에서 중요하게 적용이 된다. 전통적인 key-pint matching 방법은 그 시간을 오래 잡아먹고, noise나 view point의 변화에 매우 민감하다. 최근 연구는 이러한 문제를 해결하려는 접근을 하고 있다.(OctNet, O-CNN, Octree) 하지만 이러한 방법들은 fixed cost volume을 만들다 보니 flexibility가 떨어진다. 우린 point cloud를 사용함으로써 조금 더 유연하고 정확도 높은 접근 방법을 하게 된다.

__Point-based 3D Learning__

  최근 Point를 기반으로 한 새로운 형태의 deep learning network architecture들이 나오고 있다. voxel-based method와 비교했을 때, 이러한 형태의 architecture는 불필요한 연산을 없앤다. 또한 공간에서의 연속성이 학습 process간 유지가 된다. PointNet같은 경우는 object classification과 detection에서도 훌륭한 performance를 보여주었다. 이러한 architecture를 어떻게 MVS task에 적용시킬 수 있을지 연구가 되어왔다. 우리는, point hypotheses의 2D-3D feature를 base로 한 3D flow를 예측하는 PointFlow module을 제안한다.



### Method

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210113-18.PNG)

 여기에선 우린 architecture를 소개할 것이다. 우리가 소개할 architecture는 2 step으로 이루어져있다. __coarse depth prediction__ , __iterative depth refinement__ 이다. I_0를 reference image로 하고, I_i(i=1:N)은 neighbouring source image라 하자. 우린 처음 I_o에 대한 coarse depth map을 만든다. 이 때 만들어진 resolution은 매우 low하기 때문에, volumetric MVS method는 충분히 효과적이다. 두번째로 우리가 소개할 2D-3D feature lifting(section 3.2에서 소개 예정)는 3D geometry와 2D image 정보를 연관하는 것들을 소개한다. 이후 우리는 input depth map을 higher resolution depth map으로 iterative하게 refine하는 PointFlow를 소개한다.

__3.1 Coarse depth prediction__

 3D cost volume을 기반으로 한 연산은 상당히 많은 cost를 필요로 한다.(memory, time) 하지만 우리의 경우에서는 그 크기가 상당히 coarse한 case도 있기 때문에, 이 경우에는 이전에 MVSNet이 했던 방법을 그대로 사용한다.

 MVSNet에서는 camera frustum을 reference로 두고 3D cost volume을 만들었다. 여기에 multi-scale 3D CNN을 적용한 이후 soft argmin operation을 적용시켰다. (기타 MVSNet 내용) 하지만 우리가 쓰는 coarse depth estimation network에서는 cost volume이 reference image size의 1/8이고, 48 or 96 virtual depth plane을 사용하고 있다.(MVSNet에서는 256을 썼다.) 이런 방식은 MVSNet에서의 메모리 사용량의 1/20에 해당하는 결과를 가져왔다.

__3.2 2D-3D feature lifting__

__Image Feature Pyramid__

여러 scale에서 contextual information의 large receptive field를 얻기 위해 우리는 3-scale feature pyramid를 사용했다. 흔한 MVS 방식들과 마찬가지로, feature pyramid는 이러한 input 들을 share한다.

__Dynamic Feature Fetching__

 image appearance feauter로 3D map을 뽑았을 때, 해당 map의 한 점은 multi-view feature map에서 fetch해올 수 있다. (주어진 camera parameter를 이용한 differentiable unprojection으로. 여기서 unproject는 depth map에서 3D point cloud로 가는 과정을 말한다.) image resolution에 따라 cost volume이 서로 다르다고 하고 이 notation을 j로 표현했을 때, 우리가 사용하는 cost volume은 아래와 같이 기존 MVSNet에서 사용하는 것과 같다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210113-19.PNG)

 이렇게 서로 다른 image resolution에서 온 cost를 concatenation하는 방법은 다음과 같다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210113-20.PNG)

 이렇게 concat된 feautre augmented point C_p는 PointFlow의 input으로 사용된다.

 다음 문단에서 설명하겠지만 우린 depth residual을 반복적으로 예측할 것이기에, dynamic feature fetching이라고 이름 붙인 operation인 image feature pyramid에서 얻은 point feature C_p^k를 빼오고 iteration한 후 point position X_p를 update한다. 우리의 방법은 image의 서로 다른 영역에서 동적으로 feature를 가져오기에, 그들을 uniform하게 다루고 feature map에서 regions of interest에 concat을 할 수 있다.

__3.3 PointFlow__

section 3.1에서 생성된 depth map은 low resolution때문에 accuracy가 상당히 낮다. 그래서 우리는 반복적으로 depth map을 refinㄷ할 수 있는 PointFlow를 소개한다.

 만약 camera parameter들을 알고 있다면, 우린 depth map을 3D point cloud로 un-project할 수 있다. 각 point마다 주변 point들이 reference camera direction으로의 GT surface보다 얼마나 떨어져있는지(loss가 얼마나인지)를 estimate해서 이러한 point들의 flow를 target surface로 push하도록 한다.

__Point Hypotheses Generation__

 image feature map에서 각 point의 depth 차이를 regress하는 것은 결코 쉽지 않다. perspective transformation(아마 2D에서 3D로 바꾸는 camera geometry의 변경을 의미하는 듯하다) 때문에, 2D에서의 공간적인 맥락이 3D Euclidean space에 가깝게 반영될 수는 없다.(??)

 network의 modeling을 하기 위해, Figure 3에 나온 것과 같이 reference camera direction에서 얼마나 벗어나있는지를 나타내는 일련의 point hypothese를 만들어내는 것을 제안한다.

 t가 normalized reference camera direction이, s가 displacement step size, unproject point가 p라고 한다면, hypothesized point p의 집합은 아래와 같다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-1.PNG)

 이 point hypotheses는 image 주변의 필요한, 서로 다른 depth에서 image feature information을 위해, geometry relationship을 모으는 network에서 중요한 역할을 한다.(해석이 아주 이상하게 되어 있지만 대충 맥락은 비슷한 것 같다. m의 경우는 아래 Edge convolution에서 다룰 예정.)

 __Edge Convolution__

 전통적인 MVS 방법에 따르면 local neighborhood는 depth prediction을 robust하게 만들어주는 것에 좋은 역할을 했다. 유사하게, 우리는 이러한 이웃하는 점 간의 feature aggregation을 강화하기 위해 DGCNN의 전략을 취했다. 

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-2.PNG)

 위 그림과 같이, directed graph는 point의 feature propagation에 도움이 되는 local geometric structure information을 할 수 있도록 kNN을 이용해서 point set을 만든다.

 (여기서 m에 대해서 잠깐 살펴보면, unprojected point p 에 대해서, 각 방향(좌우, 혹은 상하)로 임의의 point를 만들어내는 데, 이 point를 각 방향으로 얼마나 만들지를 m으로 설정하는 듯 하다. 아마도 m이 커지면 연산량이 많아질거고, 적어지면 그만큼 spatial feature information을 얻어내기 힘들기에 마찬가지로 적절한 수를 찾는 leveraging이 필요할 것이라고 본다.)

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-3.PNG)

 위에서 C_p는 feature augmented point cloud를 의미한다. 그렇다면, edge convolution은 위 식과 같아진다. 위에서 h_\theta는 \theta로 parameter화 된 learnable non-linear function이고, 사각형은 channel-wise symmetric aggregation operation이다. symmetric operation을 위한 multiple option들이 있다.(max pooling, average pooling, weighted sum 등등) 우린 나중에 이러한 방법들을 비교할 것이다.

__Flow Prediction__

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-4.PNG)

 우리가 설계한 flow prediction의 network는 위와 같다. input은 feature augmented point cloud(위에서 계산한 C_p)이고, output은 depth residual map이다. 우린 서로 다른 scale의 point feature들을 aggregate하기 위해 3개의 EdgeConv layer를 썼다. shortcut connection 역시 사용되었다. 마지막으로, 각 unproject point의 hypothesized points를 softmax한 probability scalar를 output으로 내는 pointwise feature로 변형하는 것엔 MLP가 사용이 되었다. 

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-5.PNG)

 unprojected point의 displacement에 대한 값은 위와 같이 계산이 된다. 이러한 방식은 differentiable하여 back projection이 가능하다.

__Iterative Refinement with Upsampling__

우리의 point based network architecture의 유연성 때문에, flow prediction은 3D cost volume base network에서는 하기 힘든, iterative한 동작을 할 수 있다. (space partitioning이 고정되어 있기 때문에) coarse prediction이나 former residual prediction에서 온 depth map D^(i)에서 D^(i+1)을 얻기 위해 upsmaple, kNN등을 쓰고 flow prediction을 할 수 있다. 더욱이, 우리는 unprojected point와 hypothesized point 사이의 depth interval을 줄여서 조금 더 detail feature를 예측할 수 있다.(iteration이 coarse-to-fine 작업이라 점점 high resolution으로 변경되는데, 이 때 어떤 방법을 적용해서 이걸 가능하게 했는지에 대한 내용)

__3.4 Training loss__

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-6.PNG)

 대부분의 deep MVS network와 유사하게, 우리는 이러한 문제를 L1 loss로 다루었다. predicted depth와 GT의 차이를 더하고, 이를 iteration마다 누적시켰다. \lambda는 weight로 1로 뒀는데, s는 뭔지 안나와있다.



### Experiments

__4.1 DTU dataset__

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-7.PNG)

__4.2 Implementation details__

__Training__

__Evaluation__

 depth는 96, l은 3, N(view set)은 5를 썼음.

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-8.PNG)

__4.3 Benchmarking on DTU dataset__

__4.4 PointFlow iteration__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-9.PNG)

__4.5 Ablation study__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-10.PNG)

__Edge Convolution__

__Feature Pyramid__

__4.6 Reliance on initial depth maps__

__4.7 Comparison to point cloud upsampling__

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210114-11.PNG)

__4.8 Foveated detph inference__

__4.9 Generalizability of the PointFlow Module__



### Conclusion



### Acknowledgement



### References

Chen, Rui, et al. "Point-based multi-view stereo network." *Proceedings of the IEEE International Conference on Computer Vision*. 2019.
