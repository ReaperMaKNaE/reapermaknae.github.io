---
layout : post
title : "YoloV5와 Colab을 이용해서 Custom Dataset Object Detection 하기[성공]"
date : 2020-11-28 +0900
description : Object Detection
tag : [ComputerVision]
---

### Custom Dataset Object Detection with YoloV5 and Colab



 이전과 크게 바뀐게 없다. 그냥 데이터셋만 2배하고, 학습량에 충분한 시간을 줬다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201129-1.png)

 정확한 사진의 양은 108장.

 서로 섞인 사진 46장, 아두이노/라즈베리파이/우노 각각 20장 정도씩이다.



 마찬가지로 Labelimg를 이용해서 라벨링을 해주고, 학습을 시켜줬다.

 img size는 640, batch size는 3, epoch는 300번.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201129-2.png)



 아래는 학습이 다 끝나고 텐서보드.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201129-3.png)

 precision, mAP, recall 모두 아 이정도면 괜찮네. 하는 수준까지 올라갔고, loss도 저번처럼 엉성하게 있는게 아닌 만족할만큼 떨어졌다.



 학습 이후 샘플사진을 하나 넣어봤다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201129-4.png)

!!!!!

 이정도면 충분하겠지? 하고 바로 테스트영상을 넣어봤다.

 [약 20초 정도의 길이로, 이 문장을 누르면 영상을 볼 수 있다.](https://youtu.be/LnmDRXNdUCo)

 예전에 450프레임? 정도 짜리 영상을 ImageAI를 이용해서 내 노트북에서 할 때 거의 한시간 가까이 걸렸는데, 코랩 GPU를 쓰니까 10초컷이다. 

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201129-5.png)

 역시 코랩이 빠르다.



 사진이 100장 겨우 넘는데도 성능이 이 정도라서 좀 놀랐다.

 영상에서 박스를 열 때 라즈베리파이 제로 옆에 NANO가 있다고 우기긴 하는데, 사실 프로 미니다. 저건 학습을 안시켜줬으니 헷갈릴만도 하지. 애초에 비슷하게 생기기도 했다.



### YoloV5 이해하기



 이건 아마도 며칠동안 계속 좀 볼 것 같은데, 아무튼 코드를 보자.

(작성중)