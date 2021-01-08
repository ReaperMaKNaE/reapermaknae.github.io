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

 TSDF는 Surface 관련한 내용, 혹은 3D reconstruction(3D recon이라고도 한다.) ~~군대 용어인 리컨과 헷갈려서 나도 처음에 뭔지 몰랐다~~ 

 어떤 공간이나 Image에서 depth data를 뽑는다고 가정할 때, 이 값들을 -1 ~ 1로 normalization하게 되면 0에 해당하는 곳은 같은 surface일 가능성이 높다는 것이다. (같은 surface라기보단, 서로 이어진 surface)

 이걸 어떻게 하느냐?

 논문을 읽는 편이 낫다. 좀 어렵다. ~~하여튼 세상 사람들 참 똑똑하다.~~

Curless, Brian, and Marc Levoy. "A volumetric method for building complex models from range images." *Proceedings of the 23rd annual conference on Computer graphics and interactive techniques*. 1996.

 난 이걸 처음 본 것이 Atlas: End to end 3D Scene Reconstruction from Posed Images 라는 network이다. 정말 감명깊게 읽은 논문이다. 참고로 AtlasNet과는 다르다. atlas에 대해서는 깃허브를 남기는 편이 좋은 것 같다.

[https://github.com/magicleap/Atlas](https://github.com/magicleap/Atlas)

 논문도 읽을만 하다. (추후 간단한 리뷰 예정)

Murez, Zak, et al. "Atlas: End-to-End 3D Scene Reconstruction from Posed Images." *arXiv preprint arXiv:2003.10432* (2020).



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