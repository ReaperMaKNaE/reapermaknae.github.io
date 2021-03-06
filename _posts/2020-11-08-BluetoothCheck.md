---
layout : post
title : "Bluetooth Check"
date : 2020-11-08 +0900
description : communication
img : 20201108-1.png
tag : [arduino]
---

### 아두이노 무선통신 설정

 개인 프로젝트로 RC카와 드론을 할 예정인데, 이 조종을 무선으로 할 예정이다. 그래서 뭘로 할까 하다가, 나뒹굴어 다니는 블루투스 모듈이 많아서 이거랑 ESP8266 모듈을 쓸 예정이다. 아마도 블루투스로 RC카를 먼저 해볼 것 같다.

 일단 기록용으로 남기면,

 HC05는 기본 보드레이트가 38400이고, HC06은 기본 보드레이트가 9600이다.

 따라서 Error 103과 같은 것들이 뜨면, 일단 위와 같은 보드레이트로 변경해주자.

 그리고 아두이노 나노로 처음에 하려고했었는데, 얘는 시리얼통신 포트가 컴퓨터 USB에 꽂혔을 때 말고 없다. 그래서 우노와 같은 것들로 블루투스 모듈의 master/slave를 잡아주고 난 이후 이쪽으로 옮겨야 한다.

 아래는 일단 나뒹구는 블루투스 모듈 아무거나 하나 잡고 설정을 마친모습.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201108-1.png)

 HC05에 버전은 4.1이고, ROLE이 1(마스터란 소리), Baud가 6(38400이란 뜻)이고, 나중에 잡혔을 때 이름은 reapermk1이다. 비밀번호는 뭐 다를 거 있겠나. 1234다.

 데이터시트에서는 Baudrate 설정하려면 AT+UART 를 쓴다는데, 내껀 버전이 달라서인지 아니다.

 AT+ROLE0을 하면 slave, AT+ROLE1을 하면 마스터

 AT+BAUD0~8은 차례대로 1은 1200, 2는 2400, 3은 4800, 4는 9600, 5는 19200, 6은 38400, 7은 57600, 8은 115200이다. 기억해두자. 이거 모르면, 아두이노로 블루투스 연결 한 후 아두이노IDE에서 보드레이트 계속 바꿔가면서 AT 쳤을 때 "OK!" 뜨는 놈만 찾아야 한다.

 일단 한놈을 마스터에 위와 같이 설정해놨으니, 기록을 해줘야한다.

 참고로 아두이노 코드는, softwareserial(RXD, TXD)에서, RXD는 블루투스에 RXD 적힌 곳을 아두이노 핀에 연결하면 된다.(블루투스의 RXD핀이 8번이라 선언했으면, 우노의 8번핀에 블루투스 연결.)

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201108-2.png)

  기록은 반드시 해두자. 이거 까먹으면 나중에 정말 고생한다. (참고로)

 보드레이트를 찾는 다른 방법으로, putty를 이용하는 방법이 있다. serial로 설정해놓고, 1200, 2400, 4800, 9600 등등 한개씩 넣어보자. 결국 뭐 하나는 될 것이다.

 문제는 다른놈이었다. 이놈 찾느라 정말 힘들었다.

 일단 이 모델의 이름은 HC-06 B28090W bluetooth module이다. 중국에서 만들어진 모양.

 블루투스 모듈이 종류가 다양한데, 반드시 올바른 걸 제대로 알고 사자. 이거 몰라서 데이터시트 정말 많이 찾아봤다.

 ![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201108-3.png)

 both NL & CR로 하면 대개 통신이 잘 되는데, 얘는 안되더라. line ending 없음으로 바꿔야만 통신이 된다. 그리고 master, slave를 설정하는게 아예 없는 모양. 핸드폰으로 어떻게 찾아서 해 보니 slave만 가능한 녀석 같다.

 뭔가 해봤는데 명령어는 AT+NAME만 먹고, 대답은 OKsetname만 해대고, 이게 제대로 받는게 맞나? 라는 생각이들었을 때, 이렇게 저렇게 바꿔서 AT+VERSION을 쳐보고 linvorV1 이 뜬다면, HC-06 B28090W 모델인걸 감안하고, 찾아보길 바란다. 참고로 명령어가 진짜 별로없다.

 일단 두 블루투스 모듈의 보드레이트를 38400으로는 맞춰놓은 상태. 얘로 과연 되긴 할까? 흠..

 대충 회로 짜서 확인해보니 연결이 안되는 것 같다. 아무래도 하나가 문제가 좀 있는 듯. 나중에 Rollivery에 썼던 거를 들고와서 확인해 봐야겠다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/6XpvgjyXpHA" frameborder="0" allowfullscreen></iframe>

좀 찾아보니 기다리면 알아서 된다는데, 아무래도 버전도 서로 다르다보니 통신이 안되는 것 같다.

 물론 그전에 마킹을 해놓는 것은 잊지말자. 언젠간 쓰일수도 있으니. 그래도 수신은 되긴하니까 아마 나중에 어플같은거 만들어서 쓸 땐 쓸모가 있을 수도 있다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201108-4.png)

 다른 방식으로 ESP8266같은 것들을 이용하는 방법이 있는데, 이건 장소가 바뀔 때 마다 IP를 새로 등록해줘야 하고, 지금 가진  것들을 또 펌웨어 올려야한다. __~~*귀찮다*~~__

 블루투스가 잘 되면 굳이 할 필요 없으니 일단은 참아보자. 아니면 내일 가서 슥 떼와봐야겠다.

