---
layout : post
title : "Depth Completion"
date : 2021-05-29 +0900
description : Depth Completion에 대한 논문들을 정리해보았습니다.
tag : [PaperReview]
---

### Depth Completion (2021.05.28)



KITTI Depth Completion task에 등록된 것 중, 코드가 공개된 상위 10개 가량의 논문을 간단하게 정리한 글입니다. 작성일은 2021년 5월 28일로, 차후 확인 시 다를 수 있습니다.



### 1. Deep Architecture with Cross Guidance Between Single Image and Sparse LiDAR Data for Depth Completion



RGB image와 Sparse LiDAR 사이의 modality를 연결하기 위해 cross guidance를 이용하여 depth completion에서 성능을 뽑아낸 논문. encoding 과정 중에서 attention mechanism을 이용하였고, residual ASPP block을 이용하였다고 한다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-1.PNG)

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-2.PNG)

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-3.PNG)

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-4.PNG)

네트워크의 구성은 위와 같다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-5.PNG)

위는 결과.



### 2. Revisiting Sparsity Invariant Convolution: A Network for Image Guided Depth Completion



 Sparsity invariant한 convolution을 다시 분석하여, 3개의 aware operation을 제안함으로써 depth completion task를 수행했다고 한다.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-6.PNG)

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-7.PNG)

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-8.PNG)

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-9.PNG)

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-10.PNG)

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-11.PNG)

 위 사진은 Network의 구성을 보여준다.

아래는 결과.

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-12.PNG)



### 3. Depth Completion from Sparse LiDAR Data with Depth-Normal Constraints, ICCV 2019



 depth와 surface normal을 찾는 diffusion module과 noise를 줄이기 위한 confidence prediction을 이용하여 depth completion을 수행한 논문.

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-13.PNG)

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-14.PNG)

 네트워크의 구성.

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-15.PNG)

 위는 결과.



### 4. Sparse and noisy LiDAR completion with RGB guidance and uncertainty, MVA 2019



 simple한 fusion으로 depth completion을 수행한 논문.

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-16.PNG)

 위 구조는 네트워크. 매우 심플한 편이다. 대신 이렇게 저렇게 잘 섞었는데, Guidance Map, Global Depth prediction, Confidence, local depth prediction 등을 잘 섞은 것을 볼 수 있다.

 아래는 결과.

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-17.PNG)



### 5. Deformable Spatial Propagation Networks For Depth Completion



CSPN의 단점을 DSPN으로 극복하여 depth completion에 적용한 논문. confidence mask, coarse depth map을 만들고, 추가적인 DSPN network(Refinement network)를 이용하여 최종 output을 만들어 내는 work이다.

![img18](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-18.PNG)

![img19](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-19.PNG)

 결과는 위와 같다.



### 6. A Multi-Scale Guided Cascade Hourglass Network for Depth Completion



 multi-guided cascade hourglass network를 이용해서 depth completion을 수행한 논문이다. 아래 그림을 보면 팍 이해가 온다.

![img20](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-20.PNG)

 input을 어떻게 줄 지에 대해서 여러 실험을 해봤다고 한다. 결과적으로 선택된 것은 왼쪽에서 3번째, 우측에서 4번째에서 image와 sD(sparse Depth)의 위치가 뒤바뀐 sD(chn)I(e) 이다.

![img21](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-21.PNG)

아래는 결과.

![img22](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-22.PNG)



### 7. DeepLiDAR: Deep Surface Normal Guided Depth Prediction for Outdoor Scene From Sparse LiDAR Data and Single Color Image



 confidence mask를 예측하고, color image와 surface normal을 이용하여 depth completion을 수행하였다.

![img23](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-23.PNG)

![img24](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-24.PNG)

 아래는 결과.

![img25](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-25.PNG)



### 8. Learning Joint 2D-3D Representations for Depth Completion, ICCV 2019



 simple한 convolution은 여전히 강력하다는 것을 보여준 논문.

![img27](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-27.PNG)

여기서 2D-3D fuse block은 아래와 같다.

![img26](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-26.PNG)

 아래는 visualized 결과.

![img28](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-28.PNG)



### 9. Adaptive Context-Aware Multi-Modal Network for Depth Completion



 이제쯤 오면 어떤 창의적인 방식으로 두 모달리티를 잘 섞었느냐가 메인 컨텐츠인지 알 수 있다. 네트워크는 아래 참조.

![img29](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-29.PNG)

![img30](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-30.PNG)

![img31](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-31.PNG)

 아래는 visualization 결과.

![img32](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-32.PNG)



### 10. CSPN++  : Learning Context and Resource Aware Convolutional Spatial Propagation Networks for Depth Completion, AAAI 2020



![img33](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-33.PNG)

 구조는 위와 같다. kernel을 조금 더 유연하게 변경했다는데, 직관적으로 이해되지도 않고 지금은 한물간 network이니 자세한 내용은 skip.

![img34](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-34.PNG)

 visualization 결과는 아래와 같다.

![img35](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-35.PNG)



### 11.  Non-Local Spatial Propagation Network for Depth Completion, ECCV 2020



 말 그대로 non-local하게 spatial propagation하는 network.

![img36](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-36.PNG)

 non-local의 의미는 아래를 보면 바로 알 수 있다.

![img37](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-37.PNG)

 아래는 visualization.

![img38](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-38.PNG)



### 12. Learning Guided Convolutional Network for Depth Completion, TIP 2020



 memory consumption과 computation cost를 줄이면서 성능은 유지한 것에 focus를 맞춘 논문.

![img42](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-42.PNG)

![img43](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-43.PNG)

 visualization.

![img44](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-44.PNG)



### 13. FCFR-Net: Feature Fusion based Coarse-to-Fine Residual Learning for Monocular Depth Completion



 sparse-to-coarse, coarse-to-fine network을 이용하여 성능을 올렸다고 한다.

![img45](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-45.PNG)

 channel shuffle을 썼다고 한다.

![img46](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-46.PNG)

 아래는 visualization.

![img47](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-47.PNG)



### 14. PENet: Towards Precise and Efficient Image Guided Depth Completion, ICRA 2021



 color와 depth를 fusion한 방식. CD Depth는 Color와 Depth를 넣어서 나온 depth이고, DD Depth는 depth 2개를 input으로 넣어서 나온 depth이다. 마지막은 CSPN++를 이용해서 refine을 했다고 한다.

![img39](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-39.PNG)

![img40](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-40.PNG)

 아래는 visualization 결과.

![img41](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210529-41.PNG)



### Reference

Lee, Sihaeng, et al. "Deep architecture with cross guidance between single image and sparse lidar data for depth completion." *IEEE Access* 8 (2020): 79801-79810.

Yan, Lin, Kai Liu, and Evgeny Belyaev. "Revisiting Sparsity Invariant Convolution: A Network for Image Guided Depth Completion." *IEEE Access* 8 (2020): 126323-126332.

Xu, Yan, et al. "Depth completion from sparse lidar data with depth-normal constraints." *Proceedings of the IEEE/CVF International Conference on Computer Vision*. 2019.

Van Gansbeke, Wouter, et al. "Sparse and noisy lidar completion with rgb guidance and uncertainty." *2019 16th international conference on machine vision applications (MVA)*. IEEE, 2019.

Xu, Zheyuan, Hongche Yin, and Jian Yao. "Deformable Spatial Propagation Networks For Depth Completion." *2020 IEEE International Conference on Image Processing (ICIP)*. IEEE, 2020.

Li, Ang, et al. "A multi-scale guided cascade hourglass network for depth completion." *Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision*. 2020.

Qiu, Jiaxiong, et al. "Deeplidar: Deep surface normal guided depth prediction for outdoor scene from sparse lidar data and single color image." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2019.

Chen, Yun, et al. "Learning joint 2d-3d representations for depth completion." *Proceedings of the IEEE/CVF International Conference on Computer Vision*. 2019.

Zhao, Shanshan, et al. "Adaptive context-aware multi-modal network for depth completion." *arXiv preprint arXiv:2008.10833* (2020).

Cheng, Xinjing, et al. "Cspn++: Learning context and resource aware convolutional spatial propagation networks for depth completion." *Proceedings of the AAAI Conference on Artificial Intelligence*. Vol. 34. No. 07. 2020.

Park, Jinsun, et al. "Non-local spatial propagation network for depth completion." *arXiv preprint arXiv:2007.10042* 3.8 (2020).

Tang, Jie, et al. "Learning guided convolutional network for depth completion." *IEEE Transactions on Image Processing* 30 (2020): 1116-1129.

Liu, Lina, et al. "FCFR-Net: Feature Fusion based Coarse-to-Fine Residual Learning for Monocular Depth Completion." *arXiv preprint arXiv:2012.08270* (2020).

Hu, Mu, et al. "PENet: Towards Precise and Efficient Image Guided Depth Completion." *arXiv preprint arXiv:2103.00783* (2021).