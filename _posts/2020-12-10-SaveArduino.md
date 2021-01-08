---
layout : post
title : "역전압으로 죽은 아두이노 우노 살려보기"
date : 2020-12-10 +0900
description : 고장난 AMS1117 교체하여 Arduino Uno를 살린 포스팅입니다.
tag : [arduino]
---

### AMS1117 교체

 예전에 모터드라이버를 굴리다가 실수로 역전압을 넣어서 Uno에 연기가 났다.

 처음엔 뭐 잘 동작하네 하고 넘어갔는데, 어느순간부터 업로드가 안되더니 아예 불이 안들어오기 시작했다. 그래서 직접 고쳐보기로 결정.

 일단 자세히 보니, regulator부분이 고장난 것을 확인할 수 있었다.

 다른건 아니고, 우노를 자세히 살펴보니 불룩 튀어나와있었다.(아래 사진의 붉은색 상자 부분)

 그래서 열심히 페이스트발라서 뗐다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201210-1.png)

 페이스트 발라서 떼긴 뗐는데, 항상 조심하자. 힘으로 땡기다가 손에 핀셋을 박아넣는 바람에 후시딘을 사버렸다.

 original arduino uno의 경우, 위 레귤레이터의 이름은 MC33269D인데, 내가 지금 사용하고 있는게 중국산이라 그런지 AMS1117이라는 module을 사용하고 있었다.

(original aruduino uno datasheet : [https://www.arduino.cc/en/uploads/Main/arduino-uno-schematic.pdf](https://www.arduino.cc/en/uploads/Main/arduino-uno-schematic.pdf))

 그래서 AMS1117을 구입하고, 새걸로 교체했다. 구입은 아래 링크.

AMS1117, [https://www.devicemart.co.kr/goods/view?no=1321255](https://www.devicemart.co.kr/goods/view?no=1321255)

 하나에 150원인데, 혹시나해서 3개를 더 샀다.

 ![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201210-2.png)

 그리고 붙이면 위와 같이 된다.

 아래는 LED를 깜빡거리게 하는 예제를 넣고 한 테스트영상! 잘 작동한다.

<iframe width="100%" height="315" src="https://www.youtube.com/embed/-u7pCwiw4iQ" frameborder="0" allowfullscreen></iframe>