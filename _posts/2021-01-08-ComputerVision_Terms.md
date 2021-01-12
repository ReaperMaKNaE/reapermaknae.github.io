---
layout : post
title : "Computer Vision with Machine Learning에서 쓰이는 용어들"
date : 2021-01-08 +1000
description : Outlier, Inlier, ICP, RANSAC, TSDF, BA, CRF와 같은 자주 사용되는 용어들에 대한 정리를 담은 포스팅입니다.
tag : [ComputerVision]
---

### Outlier, Inlier, RANSAC

 논문을 읽으면서 여러모로 와닿지 않았던 단어, outlier와 inlier이다.

하지만 어떤 블로그에서 정말 잘 설명해주셨다. ~~사실 나도 헷갈리면 다시 참고할 리스트를 정리하려고 만든 포스팅이다~~

[다크 프로그래머님이 포스팅하신 RANSAC의 이해와 영상처리 활용](https://darkpgmr.tistory.com/61)



그럼 여기서 조금 비슷한 형태로, 빗나간 걸 맞춰주는 ICP와 BA가 있다.

### ICP(Iterative Closest Point)

ICP가 다른 이름의 축약형도 존재하는 것 같은데, 일단 computer vision에서 사용하는 방식은 위와 같은 형태이다.

간단하게 말하자면 서로 떨어진 두 지점의 간격을 최소화 시켜주는 알고리즘이다.

2D에서나 3D에서나 모두 적용이 가능한데, 보통 카메라의 위치가 조금씩 다를 때 이 위치를 잡아주는 역할을 한다.

위키피디아를 보는 것이 가장 좋은 방법 같다.

[https://en.wikipedia.org/wiki/Iterative_closest_point](https://en.wikipedia.org/wiki/Iterative_closest_point)

3D Point에서 이를 증명하는 방법은 아래 논문을 참조.

Arun, K. Somani, Thomas S. Huang, and Steven D. Blostein. "Least-squares fitting of two 3-D point sets." *IEEE Transactions on pattern analysis and machine intelligence* 5 (1987): 698-700.



### BA(Bundle Adjustment)

 이걸 어디 논문에서 처음 봤는지 모르겠는데, 당시 "이게 뭐지?" 하면서 찾아봤던 건 역시나 위키피디아였다.

 간단하게 말하자면, 어떤 대상(물체)를 여러 카메라가 서로 다른 위치에서 찍는다고 할 때, 이론적으로라면 찍은 사진들을 바탕으로 물체의 입체도를 대충 그려낼 수 있다. 이렇게 같은 물체를 다른 위치에서 찍은 것을 Homography라고 하는데, 여기에 geometric calculation을 더하는 과정을 컴퓨터 계산넣으면 입체도가 나오겠지! 하는데 실제로는 안나온다. 왜냐하면 카메라로 찍은 사진마다 전부 noise가 존재하기 때문.

 따라서, 이를 해결해주기 위해 나온게 Bundle Adjustment다. 이러한 noise를 최소화시키는 알고리즘을 적용했다. 원본을 찾아가면 좋으나, 이게 각 논문마다 자기들 입맛에 맞게 다 조금씩 바꾸는 것 같다. 그래서 대충 위키피디아보고 "아, 대충 이런거구나"하고 넘어가면 되는 것 같다.

[https://en.wikipedia.org/wiki/Bundle_adjustment](https://en.wikipedia.org/wiki/Bundle_adjustment)



### CRF(Conditional Random Field)

 CRF는 정말 쉽게 말하면 classifier다. SVM같은 강력한 녀석들도 있지만, CRF만으로도 해결이 되면 이 친구를 사용하는 것 같다. 같이 경쟁하는 친구로 MEMM이 있는 것 같은데, 결국 따지고보면 CRF가 더 낫다는 것이 많이 보인다.

 해당 내용은 Lovit님 께서 작성하신 ["From Softmax Regression to Conditional Random Filed for Sequential labeling"](https://lovit.github.io/nlp/machine%20learning/2018/04/24/crf/) 의 내용을 참고하는 것이 좋다.

 내용이 조금 길기는 한데, 결국 어짜피 다 알아야하는 내용이고, 정말 유익하고 잘 정리되어 있다. CRF뿐 아니라 그 이전에 나오는 여러 function들이 AI 관련 논문을 읽을 때에 상당히 도움이 된다.



### TSDF(Truncated Signed Distance Function)

 TSDF는 Surface 관련한 내용, 혹은 3D reconstruction(3D recon이라고도 한다.) ~~군대 용어인 리컨과 헷갈려서 나도 처음에 뭔지 몰랐다~~  혹은 T를 빼고 SDF라고도 한다.(제한을 거는 경우 T를 붙여 TSDF로 표현하기도 함)

 어떤 공간이나 Image에서 depth data를 뽑는다고 가정할 때, 이 값들을 -1 ~ 1로 normalization하게 되면 0에 해당하는 곳은 같은 surface일 가능성이 높다는 것이다. (같은 surface라기보단, 서로 이어진 surface)

 이걸 어떻게 하느냐?

 논문을 읽는 편이 낫다. 좀 어렵다. ~~하여튼 세상 사람들 참 똑똑하다.~~

Curless, Brian, and Marc Levoy. "A volumetric method for building complex models from range images." *Proceedings of the 23rd annual conference on Computer graphics and interactive techniques*. 1996.

 난 이걸 처음 본 것이 Atlas: End to end 3D Scene Reconstruction from Posed Images 라는 network이다. 정말 감명깊게 읽은 논문이다. 참고로 AtlasNet과는 다르다. atlas에 대해서는 깃허브를 남기는 편이 좋은 것 같다.

[https://github.com/magicleap/Atlas](https://github.com/magicleap/Atlas)

 논문도 읽을만 하다. (추후 간단한 리뷰 예정)

Murez, Zak, et al. "Atlas: End-to-End 3D Scene Reconstruction from Posed Images." *arXiv preprint arXiv:2003.10432* (2020).



### Interpolation

 한국어로는 보간법이라고 하는데, 보통 사용하는 linear interpolation은 두 점을 잇는 선을 긋고, 그 선 위에 있는 점의 값이 얼마인가를 알아내는 방법.

 이를 확장해서 bilinear interpolation, trilinear interpolation등이 나오고 있음.

 아래 위키피디아 참조.

[https://en.wikipedia.org/wiki/Interpolation](https://en.wikipedia.org/wiki/Interpolation)



### Bilateral filter

 뜬금없이 얘만 쓰이는 것은 아니고, Guidance Filter와 같이 쓰일 수 있는데, 일단 bilateral filter는 gaussian value를 중심으로, pixel의 intensify를 주변 pixel과 대충 비슷하게 만들어주는 역할을 함.

 홍석쓰님 께서 작성하신 포스팅을 참조.

[https://redstarhong.tistory.com/57](https://redstarhong.tistory.com/57)



### SGM(Semi Global Matching)

 Hirschmuller가 발표한 내용으로, rectified stereo image pair(calibration 되어 있는 image pair를 의미)에서 dense disparity map을 estimation할 수 있는 computer vision algorithm이다.

 이는 GA Net을 참조하거나, 아래 위키피디아 링크를 참조.

[https://en.wikipedia.org/wiki/Semi-global_matching](https://en.wikipedia.org/wiki/Semi-global_matching)



### Dilated Convolution

 사진의 크기를 키우는 것에 있어서 여러 방법이 있는데, upsampling을 하면서 값을 1개만 그대로 따오고 나머지는 다 0으로 채우는 방법을 의미한다. 이걸 왜 쓰냐? 할 수 도있는데, 어쨌든 receptive field를 늘리고 대부분의 weight가 0이라서 연산능력이 좀 좋아진다는 뜻.

 sparse 하게 만든다고 해야하나? 그거와 비슷하다.



### PnP Algorithm

 Perspective - n - Point Algorithm.

 n개의 3D Point들을 여러 위치에 놓인 카메라로 찰칵 찰칵 하고 찍었을 때, 사진들을 비교해서 카메라의 위치(pose)를 estimation할 수 있는 알고리즘이다.

 보통 n은 3이면 되나, 이게 이론적인 수치고, 실제 카메라로 찍었을 때에는 intrinsic이라던지 extrinsic noise가 존재할 수 있기 때문에 아다리 맞게 잘 되진 않는다. 여기에 대한 error를 cost로 줘서 어떻게 줄였는가에 대해서는 많은 연구가 이루어졌었다.

 물론 자세한 것은 위키피디아 참조.

[https://en.wikipedia.org/wiki/Perspective-n-Point](https://en.wikipedia.org/wiki/Perspective-n-Point)



### Context-aware

http://yhs968.blogspot.com/2019/01/context-aware-1.html



### Uncertainty

Aleatoric(Homoscedastic, Heteroscedastic), Epistemic

CNN에 L2 norm penalty와 MC dropout을 적용하는 것이 Bayesian CNNs와 일치하는 것.

 이게 무슨 소릴까?

 일단 uncertainty라는 것은, 말 그대로 불확실성으로, machine learning에서는 확실하지 않은 것을 판단할 수 있는 것을 말한다.

 이 쪽이 연구가 된 이유로는, AI가 흑인 여성 둘을 고릴라로 판단하고, 자율주행차가 트레일러를 하늘로 착각하고 들이박은 사건 이후이다.

 간단하게만 짚고 넘어가면,

 Aleatoric Uncertainty(Data Uncertainty)는 데이터에 내재되어 있는 에러에 대한 불확실성

 Epistemic Uncertainty(Model Uncertainty)는 모델에 내재되어 있는 에러에 대한 불확실성.

 이 두 가지로 분류되는 uncertainty를 잘 설명하고, 잘 합쳐서 문제를 해결한 것은 아래 논문을 참조.

한글로 리뷰가 많이 나와있기에 그것을 참고해도 좋을 것 같다.

Kendall, Alex, and Yarin Gal. "What uncertainties do we need in bayesian deep learning for computer vision?." *Advances in neural information processing systems*. 2017.



### ORB, BRIEF descriptor

https://m.blog.naver.com/PostView.nhn?blogId=ghd3079&logNo=221496302601&proxyReferer=https:%2F%2Fwww.google.com%2F



### Beta distribution

[https://en.wikipedia.org/wiki/Beta_distribution](https://en.wikipedia.org/wiki/Beta_distribution)



### HDR Image

 Dynamic Range란 어두운 영역과 밝은 영역 사이의 비를 의미하는데, 우리가 눈으로 보는 세상은 이 비가 상당히 높지만, 카메라로 찍은 사진이나 컴퓨터의 디스플레이가 제공해주는 것은 자연이 제공하는 dynamic range에 비해 그 크기가 상당히 작을 수 밖에 없다.

 이를 보완해주기 위해 새로 생긴 것이 바로 'HDR image(High Dynamic Range image)' 이다. 각 pixel이 0~255의 8bit 이 아닌 32bit 의 floating point number로 나타내기 때문에, 16bit나 8bit로 불러내는 이미지 형식으로 HDR을 불러내게 되면 검정에서 흰색 사이에서만 표현을 하기 때문에 gray scale로 불러내게 된다.

 그럼 이런 image는 어떤 확장자로 주로 저장이 되는가?

 여기서 __.pfm file__ 이 등장한다.



### PFM(Portable Float Map) Format

.pfm 확장자라는 것이 있다. 윈도우에서 font로 사용하는 PFM(Printer Font Metrics)와는 다른 file 형식이다.(사실 꼴은 같으나, 같은 형태로 font로 불러내면 meta data같은 것들이 맞지 않아서 안 불러와진다.)

 이러한 확장자를 사용하는 이유는, 위에서 소개한 __HDR Image__ 에도 사용이 되지만, image의 depth를 나타내어주는 것에도 도움이 된다. 32bit floating point number이기에 depth를 결정하기에도 좋은 결과를 나타내어 준다.(보통 전용 뷰어로 따로 봐야 보인다.)

 이게 대충 감이 안 잡힌다면, 대충 아래와 같다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210112-1.PNG)

 좌측은 depth map이고 우측이 reference image인데, depth map의 확장자가 .png(png 확장자는 한 pixel이 8bit로 구성되어있다.)이기에, visible하게 보여준다곤 했지만 relative한 형태를 보여줄 수 없어 위와 같이 보이게 된다.(pfm의 경우 32bit로 구성)



### ML에서 multi-GPU가 hardware level에서 동작하는 방법

pytorch에서 multi-GPU 작동 방식: [https://www.telesens.co/2019/04/04/distributed-data-parallel-training-using-pytorch-on-aws/](https://www.telesens.co/2019/04/04/distributed-data-parallel-training-using-pytorch-on-aws/)

tensorflow에서 multi-GPU 작동 방식: [https://www.quora.com/Can-I-train-deep-learning-models-using-two-different-GPUs-in-one-build](https://www.quora.com/Can-I-train-deep-learning-models-using-two-different-GPUs-in-one-build)

전반적으로 distributed multi-GPU를 검색하면 각 framework마다 GPU를 쓰는 방법이 나오는 것 같다. pytorch의 경우는 distributedDataParallel을 쓰는 경우와 아닌 경우가 있는 것 같음. 해당 내용을 읽으면 될 듯 하다.



### LSTM과 GRU

 LSTM은 Forget Gate와 Input Gate로, GRU는 Reset Gate, Update Gate라는 서로 비슷해보이는 성분으로 구성되어 있다. 정말 간단히 소개하면, GRU가 더 가볍게 만들어졌으면서도 LSTM과의 성능에선 큰 차이를 보이지않는 모델이다. (R-MVSNet에서 사용이 되었다.)

Yjjo님의 LSTM 포스팅, https://yjjo.tistory.com/17

Yjjo님의 GRU 포스팅, [https://yjjo.tistory.com/18](https://yjjo.tistory.com/18)



### 이 외 참고하면 좋은 논문



__Batch Normalization__ :

Ioffe, Sergey, and Christian Szegedy. "Batch normalization: Accelerating deep network training by reducing internal covariate shift." *arXiv preprint arXiv:1502.03167* (2015).



__A taxonomy and evaluation of dense two frame stereo correspondence algorithm__

어떤 algorithm들이 stereo matching에서 사용되었는지에 대해 정리해준 논문.

Scharstein, Daniel, and Richard Szeliski. "A taxonomy and evaluation of dense two-frame stereo correspondence algorithms." *International journal of computer vision* 47.1-3 (2002): 7-42.



__Efficient deep learning for stereo matching__

stereo matching에서 어떤 방식이 효율적인가 를 연구한 논문. MRF(Markov Random field)라던지, bilateral filter등이 소개됨.

Luo, Wenjie, Alexander G. Schwing, and Raquel Urtasun. "Efficient deep learning for stereo matching." *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition*. 2016.



__A space sweep approach to true-multi-image matching__

DPSNet에서 사용된 기법의 근원은 이 논문임. 현재(2021년 01월) 기준으로, 비교적 최근에 나온 논문들은 space sweep에 대해서 상당히 좋은 평가를 하고 있음.

Collins, Robert T. "A space-sweep approach to true multi-image matching." *Proceedings CVPR IEEE Computer Society Conference on Computer Vision and Pattern Recognition*. IEEE, 1996.



__Introduction to camera pose estimation with deep learning__ 

 Pose Estimation을 machine learning으로 접근한 논문들을 분석해놓은 논문.

Shavit, Yoli, and Ron Ferens. "Introduction to camera pose estimation with deep learning." *arXiv preprint arXiv:1907.05272* (2019).



__Computing the stereo matching cost with a convolutional neural network__

Stereo image에서 어떻게 cost를 계산하는지에 대한 내용이 실려져있음. cost에 대한 개념을 잡으려면 해당 논문을 읽어보는 것도 좋음. CBCA가 여기에서 소개되기도 함. (근데 3d cost volume에 대한 이해를 하려면 코드를 돌려보는 것이 좋다. frustum에서 disparity에 대한 cost가 어떻게 잡히는지는 해당 논문을 봐도 알기 힘들다)

Zbontar, Jure, and Yann LeCun. "Computing the stereo matching cost with a convolutional neural network." *Proceedings of the IEEE conference on computer vision and pattern recognition*. 2015.



그런데 위 논문을 읽으려니 무슨 이해안되는 것이 나온다. CBCA??

__Cross-Based Local Stereo Matching Using Orthogonal Integral Images__

이 논문을 읽으면 된다. 사실 지금(2021년 01월)은 GCNet에서 cost volume이란게 나오고 나서 이러한 분야들이 잘 도입이 되고 있는 것 같지는 않지만, 그래도 혹시나해서 기록을 일단 해둔다.

Zhang, Ke, Jiangbo Lu, and Gauthier Lafruit. "Cross-based local stereo matching using orthogonal integral images." *IEEE transactions on circuits and systems for video technology* 19.7 (2009): 1073-1079.



 빠진 내용들이 좀 있을 수 있는데, 곧 Multiple View Geometry in Computer Vision이라는 책으로 뮌헨공대에서 강의를 해주신 Daniel Cremers 교수님의 강의자료를 공부한 내용을 포스팅하면서 짚고 넘어갈 예정이다.

 혹시나 참고할 사람을 위해 해당 수업의 첫번째 강의는 링크는 아래에 있다. 아래 자세히 보기에서(영상의 description) slidenote를 받아서 볼 수 있다. geometry에 관련한 내용들이 정말 잘 나와있어서, 3D에 관련해서 이론적인 내용을 공부하려면, 강의노트라도 한번은 보는 것이 필요하다.

[https://youtu.be/RDkwklFGMfo](https://youtu.be/RDkwklFGMfo)



### Reference

RANSAC, Outlier, Inlier, "RANSAC의 이해와 영상처리 활용", [https://darkpgmr.tistory.com/61](https://darkpgmr.tistory.com/61)

Iterative Closest Point, 위키피디아, [https://en.wikipedia.org/wiki/Iterative_closest_point](https://en.wikipedia.org/wiki/Iterative_closest_point)

Arun, K. Somani, Thomas S. Huang, and Steven D. Blostein. "Least-squares fitting of two 3-D point sets." *IEEE Transactions on pattern analysis and machine intelligence* 5 (1987): 698-700.

Bundle Adjustment, 위키피디아, [https://en.wikipedia.org/wiki/Bundle_adjustment](https://en.wikipedia.org/wiki/Bundle_adjustment)

Atlas, https://github.com/magicleap/Atlas

Murez, Zak, et al. "Atlas: End-to-End 3D Scene Reconstruction from Posed Images." *arXiv preprint arXiv:2003.10432* (2020).

Interpolation, 위키피디아, [https://en.wikipedia.org/wiki/Interpolation](https://en.wikipedia.org/wiki/Interpolation)

SGM, 위키피디아, [https://en.wikipedia.org/wiki/Semi-global_matching](https://en.wikipedia.org/wiki/Semi-global_matching)

PnP Algorithm, 위키피디아, [https://en.wikipedia.org/wiki/Perspective-n-Point](

Kendall, Alex, and Yarin Gal. "What uncertainties do we need in bayesian deep learning for computer vision?." *Advances in neural information processing systems*. 2017.

Yjjo님의 LSTM 포스팅, https://yjjo.tistory.com/17

Yjjo님의 GRU 포스팅, [https://yjjo.tistory.com/18](https://yjjo.tistory.com/18)