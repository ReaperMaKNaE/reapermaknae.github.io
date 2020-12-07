---
layout : post
title : "논문 읽기 - Segment2Regress: Monocular 3D Vehicle Localization in Two Stages"
date : 2020-12-04 +0900
description : Review
tag : [PaperReview]
---

### Segment2Regress: Monocular 3D Vehicle Localization in Two Stages



 일단 Computer Vision 입문 논문으로 정하게 되었다. 이유는 비밀.



 일단 전반적으로 요약하면,

 차량의 Segments와 Regress를 이용해서, 한장의 RGB 사진으로 Car의 3D 위치를 파악하는 것이다.

 내가 이전에 localization을 진행했을 때에는 Stereo Camera를 이용해서 진행했었는데, 위 논문을 읽고나니 '아, 이런 방법으로도 되는구나'를 알았다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201204-1.png)

 왜 굳이 컬러이미지 인지는 사실 잘 모르겠다. 논문대로라면 흑백도 잘 될 것 같다. 아무튼 2D bounding box는 아무 2D object detector를 쓰면 된다고 한다.

 위 그림에서 X1, X2, X3, X4의 위치를 딱 잡아내는 것.

 각 X는 x,y,z 좌표로 나타내게 되므로, 최종 output은 12개의 array가 나오게 된다.



 어떻게 한장의 RGB 사진만으로 localization이 되느냐 하면, 3개의 준비물이 필요하다.

 하나는 __*Image Depth Map*__ 이고,

 다른 하나는 __*Any 2D Object Detector*__ (논문에서는 Yolov3를 이용),

 또 다른 하나는 __*stacked hourglass network를 이용해 차량의 자세를 파악하는 것*__ 이다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201204-2.png)

 일단 input은 Any 2D object detector로 잡아낸 bounding box와 color image를 넣고, 그 다음 4개의 hourgalss network를 이용해서 vehicle의 segment를 잡아낸다. 4개인 이유는 아무래도 자동차의 left line과 front line, right line, back line을 잡아내기 위한 것 같다. 어짜피 pose를 잡아내는 데 한개만 쓰면 성능이 많이 떨어지는지도 궁금하다. 아무래도 4개가 더 좋겠지. 그리고 이후 plane depth를 집어넣은 후, 원하는 output을 끌어내도록 layer를 설계한다. 근데 plane parameters가 무엇인가?



![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201204-3.png)

 Image Depth Map의 경우 몽땅 segmentation을 때려서 computation을 복잡하게 만들지 말고, 간단하게 땅이랑 하늘. 두 개로만 나눠도 충분하니, 불필요한 computation은 줄이고 plane depth만 가지고 계산해서 대충 땅의 depth를 구하자. 여기서 plane depth가 plane parameters다. 

 Fig4는 전반적인 요약인데, batch normalization과 instance normalization을 각각 적용한 모습을 보여준다.

 batch normalization은 mini batch들을 normalization해주고, instance normalization은 train가능한 parameter들과 독립적으로 각 특징들(느낌상 segment들)을 normalize해준다는데, 이건 아직 감이 잘 안잡힌다.

 Fig 5의 경우는 loss를 어떻게 설정해서 이쁘게 자세를 잘 잡을 수 있는지 보여주는데, coupling loss가 없으면 지멋대로 뒤틀린 곳을 만들게 되고, minimal-parametrization을 사용하면(output이 x,y,z로 이루어진 4개로 이루어진 것이 아닌 center좌표, width, length, 면이랑 휘어진 각도로 이루어진 것) 원하는 것이 아닌 다른 방향으로 error가 발생하게 되므로, 위 두 case를 모두 고려한 loss가 바로 coupling loss이다.



![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201204-4.png)

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201204-5.png)

 coupling loss의 경우 위 3가지의 합으로 구성이 되는데,(여기서 j는 1,2,3,4로, 자동차 아래면을 구성하는 4개의 좌표를 의미) 다른건 다 이해가 되더라도 head는 조금 의문이 갔다. 제대로 하려면 짝수 혹은 홀수만 적용시켜야 하지 않나? 라는 생각이 들었다. 1번, 2번, 3번, 4번 중에서 e1(x axe in the camera's referential)과 각을 이루는 것과 e3(z axe in the camera's referential)과 각을 이루는 것의 차이를 보게 되면, j가 1일 떄와 j가 2일 때는 e1과 e3가 각각 바뀌어야 하지 않나? 라는 생각이 들었다. 일단 이것도 나중에 물어보는 것으로 해결하기로 하고 다음으로 넘어가면, 성능 측정 방법이다.



![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201204-6.png)

 이 때 성능을 측정하는 방법으로는 dataset은 KITTI의 BEV(Bird's Eye View) dataset과 ablation study를 선택하였다. ablation study는 논문에서 적용한 방식을 하나씩 빼 가면서, 적용한 각 방식들이 빠진 것과 빠지지 않은 것을 비교해 성능을 비교하는 방식인데, 좀 쉽게 말하면, '논문에서 선택한 방식을 모두 적용한게 성능이 제일 좋다' 라는 것을 보여주는 방식 중 하나로 이해했다.(Coupling loss의 경우 planarity를 뺀 것이 Moderate 난이도에서는 성능이 더 좋았는데, 전반적인 것들을 모두 고려했을 때는 결국 다 고려하는게 맞다 가 나왔다.)



논문의 결론은 아래와 같다.

 We have presented a novel approach for 3D vehicle localization from a single RGB image and plane parameters. To fully exploit the road environment assumption (vehicles lie on the road surface), we formulate the 3D vehicle localization as two sub-tasks (two stages): 1) Segment the vehicle region in the image domain (segment network) and 2) regress the vehicle points in the 3D domain (regression network), where we newly introduce a coupling loss to enforce the structure and heading of the vehicles. In addition, we estimate the 3D vehicle localization in metric units through a fusionby-normalization approach with the plane depth, which can be computed from simple plane parameters without heavy computation. We successfully validated our method on the bird’s eye view KITTI dataset and by an ablation study. The proposed approach can be considered as an independent 3D localization module applicable to any 2D object detector.



 Code를 보고싶었으나, 왠지 모르겠는데 현재 github에서 볼 수 없는 상황.



### Reference

Choe, Jaesung, et al. "Segment2Regress: Monocular 3D Vehicle Localization in Two Stages." *Robotics: Science and Systems*. 2019.