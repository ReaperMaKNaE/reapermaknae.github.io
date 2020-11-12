---
layout : post
title : "Bluetooth check, New Probe"
date : 2020-11-11 +0900
description : USB Powermeter design, new probe.
img : 20201111-4.png
tag : [arduino]
---

### Bluetooth 확인

 

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201111-4.png)

 웬수같은 블루투스라고 할 수 있다. 모듈을 어떻게 4개를 샀는데 4개 전부다 펌웨어가 다를수가있지?

찾아보니 HC 05의 경우 펌웨어 업그레이드를 통해서 같은 통신을 할 수 있다는데, 일단 HC06과 통신할 수 있다는 포스트를 찾아서 나중에 시도해보려고한다.

 아래는 일단 샀던 블루투스 모듈의 펌웨어 버전.(새로 산게 2.0-20100601, 이전에 있던 것이 MLT-BT05, V4.1)

 좀 신기했던건, 새로 나온 HC05는 일반 통신모드와 AT Mode가 따로 있었다. 처음에 안되길래 뭔가 했더니, 버튼을 눌러주어야 동작하더라. 맨 위 그림을 보면 이름표가 붙지 않은 녀석은 우측 상단에 버튼이 하나있다. 전원을 넣기전에 이걸 누르고, 누르면서 전원을 넣으면 LED가 천천히 깜빡이면서 AT mode로 들어가고, 이 때에 통신이 된다.

 여튼 현재 상황은 아래와 같다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201111-1.png)

 위가 펌웨어 2.0-20100601 버전의 상태. 명령어를 잘못치면 ERROR: (0)이 뜬다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201111-2.png)

 위과 MLT-BT05, V4.1 버전의 상태



 둘 다 슬레이브/마스터를 바꿔서 어떻게 해보려고했는데 안됬다. 아래에 참조한 링크들이 있는데, 내일 USB Powermeter and HUB 제작하고 난 이후에 시도해볼 것 같다.



HC05, version = 2.0-20100601

[http://www.martyncurrey.com/hc-05-with-firmware-2-0-20100601/](http://www.martyncurrey.com/hc-05-with-firmware-2-0-20100601/)

HC05, version = MLT-BT05,V4.1

[https://blog.yavilevich.com/2017/03/mlt-bt05-ble-module-a-clone-of-a-clone/](https://blog.yavilevich.com/2017/03/mlt-bt05-ble-module-a-clone-of-a-clone/)

HC05 - HC06 연결

[http://www.martyncurrey.com/connecting-2-arduinos-by-bluetooth-using-a-hc-05-and-a-hc-06-pair-bind-and-link/](http://www.martyncurrey.com/connecting-2-arduinos-by-bluetooth-using-a-hc-05-and-a-hc-06-pair-bind-and-link/)



### 새로운 프로브

 이전에 쓰던 멀티미터 프로브가 박살나서 새로 샀다. 프로브 머리부분이 뽑혀서 선과 분리. 꽂아서 쓰면 어떻게 동작은 했는데 아무래도 새로 사는게 나은 것 같아서 하나 구입했다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201111-5.png)

 뚜껑을 따서 쓰는 방식. 뽑히지만 않는다면야.... 테스트해보니 잘 작동한다.



### 잡다한 이야기

 내일은 모터드라이버 PCB를 납땜하고, USB Powermeter and HUB를 제작할 예정이다. 오전에 드론 몸체를 뽑을 예정이긴 한데, 아마 드론 제작은 파워미터 다 만들고 난 후에나 할 듯. 모터도 오긴 왔는데 체크는 나중에 할 예정이다.