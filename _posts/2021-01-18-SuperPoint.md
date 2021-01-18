---
layout : post
title : "SuperPoint"
date : 2021-01-18 +1236
description : SuperPoint, Self-Supervised Interest Point Detection and Description 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### SuperPoint: Self-Supervised Interest Point Detection and Description, CVPR 2018



SUMMARY: self-supervised로 point matching을 수행한 논문. MS-COCO dataset을 augmentation해서 self-supervised로 진행하였음. interest point가 찍힌 image를 pre-train시키고(MagicPoint 사용), 이들을 self labeling한 후(Homographic Adaptation을 진행), 해당 이미지를 warp해서 interest point와 descriptor에 대한 loss를 각각 파악하는 방법으로 self-supervised를 진행한다. 이렇게 해서 interest point와 descriptor를 찾아낸다.

 이렇게 학습된 superpoint는 interest point와 descriptor를 알아서 찾게 된다.



### Abstract

 Computer Vision에서 수많은 multiple view geometry를 해결해주기에 적합한 point detector와 descriptor를 학습시키는 self-supervised framework을 소개한다. patch-based neural network에 반해, 우리는 full-size인 이미지에 작동하고 pixel level의 point와 적절한 descriptor를 계산한다. 우린 homographic adaptation, multi-scale, multi-homography approach를 소개한다. 이러한 방법들은 point detection을 반복적으로 boosting하고, cross-domain adaptation을 수행한다.(예를 들자면, synthetic-to-real) 우리의 모델은 기존에 있던 그 어떤 모델보다도 interset points를 잘 잡아냈다.(MS-COCO dataset 기준, Homographic adaptation 적용) LIFT, SIFT, ORB와 비교했을 때 압도적인 성능을 냈다.



### Introduction

 SLAM, SfM, camera calbiration, image matching과 같은 computer vision의 가장 첫번째 step은 image에서 interest point를 추출하는 것이다. interest point는 다른 장면에서나 빛이 튀어도 안정적이고 반복적으로 나타나는 물체의 point이다. 이러한 interest point들을 다루는 , 수학과 computer vision의 subfield인 Multiple view geometry에서는 image에서 interest point들을 match하고 뽑아낼 수 있다는 가정 하에 세워졌다. 하지만, 대부분의 real-world에서 computer vision system은 point의 location을 알아차리지 못하고 있다.

 CNN이 hand-engineered representation보다, image가 input으로 오는 모든 task에서 우세인 것으로 보여지고 있다. 특히, key-point나 landmark와 같은 것들은 잘 연구되어있다. 그 예시로는 human pose estimation, object detection, room layout estimation 등이다. 이러한 기술의 중요한 점은 인간에 의해 label이 되어 GT를 만들어야 한다는 점이다.

 엄청난 규모로 학습된 machine들에겐 interest point detection에게는 그에 관련한 식을 세우는 것이 자연스러워 보였지만, 안타깝게도 human-body keypoint estimation과 같은 semantic task에서는 제대로 된 결과를 보이지 못했다. 따라서, strong supervision으로 학습하는 경우에 있어서 이러한 것들은 사소한 문제가 아니다.

 real image에서 interest point를 정의하는 것에 인간의 감독 대신, 우리는 self-supervised solution을 제공한다. 우리의 접근 방식에서, 우리는 거대한 dataset을 만들고, 그들 스스로 학습하게 한다.

 이 때 만들어진 거대한 dataset인 pseudo-ground truth interest points을 만들기 위해, 우리는 synthetic shapes라 불리는 synthetic dataset에서 example을 만든다. synthetic dataset은 간단한 geometric shape을 가지고 있다. 이러한 것들은 interest point들에 대한 애매모호함이 없다. 우리는 trained detector를 MagicPoint라고 부른다. 이것은 synthetic dataset에서 interest point를 상당히 잘 잡아낸다.(traditional한 interset point detector 보다도 더욱) MagicPoint는 domain adaptation과 같은 현실적인 문제에도 큰 어려움 없이 interest point를 잘 잡아냈다. 하지만 traditional한 pint detector들과 비교했을 때, magicpoint는 많은 interest point location을 잡아내지 못했다. 이러한 gap에 bridge를 놓기 위해, 우리는 multi-scale, multi-trainsform technique인 Homographic Adaptation을 만들게 되었다.

 Homographic Adaptation은 interest pint detector들의 self-supervised training이 가능하도록 만들어졌다. input image를 여러 번 warp한다. 이는 다른 viewpoint, 다른 scale에서도 많은 point detector를 잡을 수 있도록 도와준다. 우리는 이러한 Homographic adaptation을 pseudo-GT를 만들고 detector의 성능을 boosting하기 위해 magicpoint detector와 합쳤다. 이 모든 것들을 합쳐서, 우리는 SuperPoint라 부른다.

 interest point를 강건하고 반복적으로 detecting한 후 공통적인 단계는 고쳐진 dimensional descriptor vector를 각 point로 attach하는 것이다. 즉, image matching이다. 그래서 우리는 superpoint를 descriptor subnetwork와 합쳤다. superpoint architecture는 여러 scale에서 featur를 추출하는 CNN이 여러개 쌓여있기 때문에, interest point descriptor를 계산하는 추가적인 subnetwork와 함께 interest point network를 합치는 것은 어렵지 않다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-1.PNG)



### Related Work

 전통적인 interest point detector들이 지속적으로 평가되어 왔다. FAST, SIFT 등등이 그 예시이다.

 SuperPoint Architecture는 최근에 연구되어온 interest point detector와 descriptor learning에 영감을 받아서 만들어졌다. SIFT를 대체할 LIFT의 경우, interest point detection, orientation estimation, descriptor computation으로 이루어져있지만 classical한 SfM에서 오는 supervision이 필요했다.

 UCN, DeepDesc, SIFT 등과 우리 것의 차이는 아래와 같다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-2.PNG)

 이외에 Quad-Network라는 것도 있으나, 매우 작은 image patch를 input으로 받는다. TILDE의 경우도 우리와 같은 deep NN이나, 그 효과를 크게 받지는 못했다.

 우리의 방식은 또 다른 synthetic-to-real domain-adaptation 방식인 self-supervised method들과 비교될 수 있다. Homographic adaptation과 비슷한 것으로는 equivariant landmark transform의 이름인 것도 존재한다. Geometric Matching Networks나 Deep Image Homography Estimation의 경우 비슷한 self-supervision 전략을 사용하지만, 그 성능에는 약간 문제가 있다. SfMNet이나 DeMoN과 같은 model들도 있지만, interest point를 이용하진 않았다.



### SuperPoint Architecture

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-3.PNG)

 우리의 architecture는 full-size image에서 동작하고 single forward pass에서 fixed length descriptor에 의해 interest point detection을 만든다. 위 그림에서 처럼, 처음은 약한 encoder 과정을 거치고 난 이후, 2개의 head를 만든다. 각 head의 이름은 interest point decoder이고, 다른 head의 이름은 descriptor decoder이다.

__3.1 Shared Encoder__

 우리의 architecture는 VGG style이다. 여기에서 생기는 output은 cell의 집합이라 하자. 그럼 output은 input의 H, W에 비해 각각 1/8, 1/8로 줄어들게 된다. 

__3.2 Interest Point Decoder__

 interest point detection에서 각 output의 pixel들은 input에서의 pointness의 확률에 해당한다. 기본적인 network들은 encoder-decoder pair로 이루어져 있다.(대표적으로 SegNet) 불행하게도, upsampling은 많은 computation을 요구하고 원하지 않는 checkboard artifact를 만들어내게 된다.(아마 upsampling하면서 interpolation과정이 적합하지 않은 경우 생기게 되는... 해상도 저하와 비슷한 현상을 말하는 것 같다.) 그래서 우리는 interest point detection head를 explicit decoder로 설계했다. 이는 model의 연산을 줄여주게 된다.

 interest point detector는 H_c * W_c * 65의 input을 받고 H * W 의 size로 output을 내보낸다. 이 사이의 과정에서는 softmax와 reshape이 사용된다. softmax는 dustbin dimension을 날리는데, 이 dustbin은 interest point가 없는 지역을 말한다. reshape은 upsampling과 유사.

__3.3 Descriptor Decoder__

 여기에선 input을 H_c * W_c * D의 size로 받고, output으로 H * W * D 를 내보낸다. output dense map은 L2 norm을 통하게 된다. 우리는 UCN과 유사한 모델을 썼다. semi-densly learning descriptor는 보다 적은 memory와 run-time을 가지게 된다. L2 norm을 진행하기 전에는 Bi-Cubic interpolate을 진행한다.

__3.4 Loss Functions__

 마지막 Loss는 두 가지 loss를 더해서 만든다. 첫번째는 interest point detector loss(L_p) 이고, 다른 하나는 descriptor loss(L_d) 이다. 우리는 pseudo-GT interest point location과 랜덤하게 만들어진 homography에 상응하는 GT에서 warp한 image쌍을 사용한다. 이러한 것은 loss를 즉각적으로 optimize하도록 해 주었다. 마지막에 두 loss의 balance를 위해서 \lambda를 추가해주었다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-4.PNG)

 먼저 interest point detector loss는 아래와 같다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-5.PNG)

 위 식에서, x_hw는 cell을 의미하고, 이렇게 cell 마다 생기는 loss는 cross-entropy loss를 사용했다. Y는 GT에 상응하는 interest point의 set을 말한다.

 그리고 descriptor loss는 다음과 같다.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-6.PNG)

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-7.PNG)

 위 식에서 descriptor cell은 d_hw로 표시가 되고, first image의 경우는 d_hw, second image의 경우는 d'_h'w' 으로 표시한다. (즉 두 개는 모두 homography) 여기서 H _ p _ hw로 표시된 것에 hat을 씌운 형태가 있는데, homography를 의미한다. 그리고 여기서 1, 0으로 나타날 뿐 아니라 실제로는 lambda를 더하여, otherwise의 경우에는 0으로 만들어서 negative corresponde는 모두 다 날려버리고, 맞더라도 weight를 추가해서 최대한 오류를 줄이도록 한다. 이렇게 만들어진 식이 위 두 식이다.

 추가로 hinge loss를 사용해서, 각 positive margin과 negative margin을 주었다.



### Synthetic Pre-Training

우리는 여기서 기본적으로 사용되는 detector인 MagicPoint의 방법에 대해서 살펴본다.

__4.1 Synthetic Shapes__

아직까지 interest point label이 된 database는 큰 게 없다. 그래서 우리의 interest point detector를 위해 data augmentation(bootstrap)을 할 필요가 있다. 처음은 Synthetic Shapes라 불리는 large-scale synthetic dataset을 만든다. 우리는 이러한 dataset에서 Y-junctions, L-junctions, T-junctions를 이용하여 modeling 함으로써 label의 애매모호함을 없앨 수 있다.(여기서 Y, L, T junctions는 선들이 조합하는 방식인 junction의 일부를 말하는 것 같다. 저러한 junction을 찾도록 학습 시키는 듯 하다.)

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-8.PNG)

 synthetic image가 render 되면, 우리는 training example을 늘리기 위해 각 이미지에 homographic warp를 적용한다. 이렇게 만들어진 데이터는 2배가 된다.

__4.2 MagicPoint__

우리는 superpoint architecture에서 descriptor head 부분을 제외하고 detector pathway를 짜서, synthetic shape을 학습시켰다. 이것을 MagicPoint라고 하자. 우린 이러한 MagicPoint를 다른 alglorithm과 비교했다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-9.PNG)

 이러한 MagicPoint detector는 synthetic shape들에 대해 매우 잘 작동했다. 하지만 real image에 적용시킨다면? 이는 나중에 section 7.2에 나온다. 간단하게 말로 적자면, traditional한 방법들 보다 오히려 못한 결과가 나왔다. 어느정도는 되었지만, 우리가 만족할 수준은 아니다. corner-like structure인 table, chair, window와 같은 경우에는 잘 적용이 되었지만, 자연적인 image에서는 이러한 부분을 찾기 어려워서 인지 제대로 된 detection을 할 수 없었다. 이러한 점들은 우리가 self-supervised approach를 real-world image에 적용하는 것에 motivation이 되었고, 이렇게 만들어진 network가 Homographic Adaptation이다.

 

### Homographic Adaptation

 우리는 base interest point detector를 이용, target domain인 MS-COCO에서 unlabeled image들을 늘렸다. 각 target domain에 대해서 우리는 pseudo-GT를 만들고, 전통적인 supervised learning machinery를 사용했다. 우리의 방법에서 가장 중요한 부분은 random하게 warp하는 것이고, 이 warp한 image와 homography에서 matching하는 것이다.

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-14.PNG)

(첫번째 homography는 transformation이 없음.)

__5.1 Formulation__

 homography는 camera center, scene 근처에서 rotation만 추가된 형태를 사용했다. 더욱이, 대부분의 world는 planar하기에, 같은 3D point에 대해서 어떤 view point로 transform하더라도 homography는 good model이 된다. homography는 3D information을 필요로 하지 않기 때문에, 2D image에 쉽게 적용이 가능하다. 이러한 이유들로 인해 homography는 self-supervised approach에서 core에 해당한다.

 input image I에 대해서 우리가 적용하고자 하는 initial interest point function인 f_theta의 결과를 x라 하고, H를 random homography라고 한다면, notation은 다음과 같다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-10.PNG)

 만약 식 (9)에서 different homography가 나왔다면, 같은 x에 대해서도 result가 다르게 나온다. 따라서, 이를 해결하기 위해선 정말 많은 sample인 경우에는 empirical sum이 수행된다. 따라서, sample에 따라 aggregation된 결과는, 다음과 같다.

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-11.PNG)

__5.2 Choosing Homographies__

 3*3 matrices가 항상 homographic adaptation에 좋은 것은 아니다. good homographies를 sample하기 위해, 우리는 homography를 더 간단한 형태로 분해한다. 우리는 이전에 설정해놓은 translation, scale, in-plane rotation과 symmetric perspective distortion등으로 sample했다. 이러한 변형들은 artifact의 bordering을 avoid하는 것에 도움을 주도록 initial root center crop과 함께 했다.(??) 여하튼 그 과정은 아래와 같다.(대충 overfitting을 방지하기 위해서 이러한 것들을 했다... 정도 인 것 같다. 자세한 건 이 분야에 대해서 잘 몰라서 그런지 흠...)

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-12.PNG)

 우리의 실험에서 첫번째 homography는 adaptation을 적용하지 않았다. 이후 homography를 얼마나 적용하느냐에 따라서 성능 차이를 test해보았는데, 10개, 100개, 1000개 중에서 100개에서는 boosting이 21%, 1000개에서는 22%로, 100개는 쓰기로 했다. (10개를 썼을 경우에는 오히려 33% 떨어졌었음)

__5.3 Iterative Homographic Adaptation__

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-13.PNG)

 process가 iterative 할 수 있기에 이를 적용하였음. 그 결과는 위 사진과 같음.



### Experimental Details

 우리가 사용한 것은 MS-COO2014에 pseudo-GT label을 쓴 것임. 위에서 homography를 몇개 쓰느냐?라고 했는데, 우린 일단 100개를 썼음. D는 256을 썼고,(Descriptor Decoder에서 D를 말함) \lambda_d는 250, m_p는 1, m_n는 0.2, \lambda는 0.0001을 썼음. 



### Experiments

__7.1 System Runtime__

Titan X GPU를 기준으로 70FPS

__7.2 HPatches Repeatability__

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-15.PNG)

(근데 표를 보면 아직 성능이 traditional한 방법에 비해서 그렇게 좋아보이진 않는다. 겨우 이긴 것 같아 보이는데.)

__7.3 HPatches Homography Estimation__

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-16.PNG)

 여기서 homography estimation에 사용된 \epsilon이 뭔지 좀 궁금했는데, 아래와 같다.

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-17.PNG)

 보아하니 어떤 사람이 esitmation protocol을 만든 것 같은데, 그 과정을 따랐다고 한다.

 그 사람이 만든 estimation protocol은 해당 논문의 Appendix를 참조하거나, 아래 논문을 참조할 것.



### Conclusion

 우린 이 방법이 SLAM이나 SfM에 적용이 될 수 있을 거라고 생각함.



### Acknowledgements



### Appendix

__A. Evaluation Metrics__

__Corner Detection Average Precision.__

__Localization Error.__

__Repeatability__

__Nearest Neighbor mean Average Precision__

__Matching Score__

__Homography Estimation.__



__B. Additional Synthetic Shapes Experiments__

__Mean Average Precision and Mean Localization Error.__

__Effect of Noise Magnitude.__

__Effect of Noise Type.__

__Blob Detection.__



__C. Homographic Adaptation Experiment__

__Within-scale aggregation.__

__Across-scale aggregation.__



__D. Extra Qualitative Examples__



### References

DeTone, Daniel, Tomasz Malisiewicz, and Andrew Rabinovich. "Superpoint: Self-supervised interest point detection and description." *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition Workshops*. 2018.

 K. Mikolajczyk and C. Schmid. A performance evaluation of local descriptors. PAMI, 2005.

