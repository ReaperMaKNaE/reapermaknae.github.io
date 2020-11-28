---
layout : post
title : "[S-hero Rollivery] Rollivery 마지막, 개인 프로젝트 시작."
date : 2020-11-05 +0900
description : S-hero Rollivery
img : 20201105-1.png
tag : [S-hero Rollivery]
---

### 마지막

 S-Hero 제출을 위해 팀원 한명을 제외하고(현재 다른 캠퍼스에서 복수전공으로 인해 참여불가) 모여서 마지막 영상을 촬영한 후, 제출용 영상을 제작했다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Z6FvZ3U1rx0" frameborder="0" allowfullscreen></iframe>



### 개인 프로젝트 시작

 몇가지 개인 프로젝트를 생각중이다.

 USB powermeter, BMS(Battery management system) - PCB제작도 겸할 예정, 소형 컴퓨터(말이 컴퓨터지 nand와 같은 것들로 기본적인 디지털시스템 이해용 회로제작 예정), RC카(slam, 매핑, RaspberryPI로 제작 예정), 드론(slam, 매핑, STM32로 제작 예정)

 리스트는 이런데 3월 입학전에 이걸 다 끝낼 수 있을지 모르겠다. 코딩공부도 해야하는데...

 추가로 atmega328p-pu를 2개 사서 부트로더 구워서 쓰려고했는데, 정말 멍청하게도 22pF을 22muF로 잘못봐서 구워지질 않는다. 위에것들 만들면서 또 좀 사야겠다.



### 코딩 공부 시작

조금밖에 못풀었지만 1005, 1006 풀고 릿코드로 넘어가는 방법을 생각해야겠다.



### STM32

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201105-2.jpg)

STM32F103. 디바이스마트에서 구입했다. Cube로 짜서 어떻게 굴려볼 계획이다.

 추가로 이를 연결해줄 ST-link. 얘가 없으면 연결이안되니 연결하고 해봐야지.

 내일 납땜하고 바로 테스트해봐야겠다. 아래는 구입한 모델들의 사진. 재밌겠다!

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201105-3.jpg)



### PCB

 이전에 회로를 짤 때 진짜 고생을 너무 해서, PCB를 한번 짜보자 하고 PCB를 만들어봤다.

 만드는 것은 정말 쉽기로 유명한 EasyEDA.

 ![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201105-4.png)

 생각보다 되게 간단했다. 각 핀에 헤더핀을 달 건 아닌데, 일단 임시로 달아놓은 케이스. 실제론 BR-5001을 몇군데에 달 것이다. hole을 몇군데 뚫어뒀는데, L298N위의 hole 2개는 방열판용이고, LM317 우측도 방열판이다. 나머지 2개는 나사박을때 쓸려고 표시해둔것. 아, 나사의 차이는 좌우로 3mm, 위아래로 46mm, 3.2mm로 팠다.(3mm로 하면 아마 드릴로 박다가 PCB판 깨질듯..)

 아래는 3D형태.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201105-5.png)

 최소 5장부터 주문이 가능하고, 가격은 4딸러!

 이럴거면 만능기판에 쓰잘데기없는 핀 여러개 살바에야 그냥 이거 사는게 훨씬 싸다.(진짜로)

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201105-6.png)

 다행히 패널티로 배송비가 붙는다. 

 근데 2달러 할인이 붙었다.(???!)

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201105-7.png)

 배송비 포함해도 만능기판이랑 이것저것 합하면 더 싼것같다.

 이거말고 아마 BMS 할 것도 기판을 짜서 내일 주문 넣어봐야겠다. 아무래도 배송비 한번은 더 아낄 수 있을테니. 근데 진짜 싸네...

 

### Reference

STM32F103, [https://www.devicemart.co.kr/goods/view?no=1329653](https://www.devicemart.co.kr/goods/view?no=1329653)