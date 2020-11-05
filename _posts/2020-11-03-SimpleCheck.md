---
layout : post
title : "[S-hero Rollivery] Data 전송 확인 및 연구실 환경 확인, WSL2"
date : 2020-11-03 +0900
description : S-hero Rollivery, WSL2
img : 20201103-4.png
tag : [S-hero Rollivery]
---

 ### Data 전송 확인 및 연구실 환경 확인



 어제 영상을 이용한 주행은 포기를 하고, 가속도 데이터와 각속도 데이터를 이용해서 데이터를 분석할 수 있도록 연구실에 둔 로봇에 확인을 하러 갔다.

 RPi에서 데이터를 보내줄 때, 음수를 방지하기위해 +10을 하고, 소수점 4째자리까지 살리기 위해서 만(10,000)을 곱한 후 int로 데이터를 보내주는 방식을 택했는데, 2byte로는 부족해서 3byte까지 올린 후 전송을 하니 성공을 하였다. 그런데...

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201103-1.png)

 연구실의 skku 와이파이를 이용해서 무선으로 송출한 이미지를 못불러왔다. __~~*(망했다)*~~__

 이거까지 못불러오면 도저히 뭘한건가 싶은 수준인데, 일단 데이터라도 좀 잘받아오는지 확인하기 위해 이것저것 코드를 좀 바꾸다가 왔다.

 아래는 정말 절망편.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201103-2.png)

 영상 통신이 가끔 잘 될 때도 있었는데, 그러다가도 딜레이가 7초이상 넘어가는 경우도 있었다. 아무래도 통신속도가 느린 것 같다는 생각이 들었다.(이럴 때에는 가속도, 각속도 데이터도 못받아왔다.)



 아무래도 내일은 가서 matplotlib을 이용해서 데이터 그려보고, 움직일 수 있다면 약간 움직여보는 걸로 하고 끝내야겠다.(다음주까지 끝낼 수 있을까?)



### WSL2(Windows Subsystem for Linux 2)

 WSL2를 한번 써봤다. 윈도우에서도 리눅스를 할 수 있도록 만들어주는 것. 원래 다른 사람들은 VM을 썼다는데, 사실 이런건 크게 관심없고, 파이참으로 코딩하던 중 어짜피 나중에 vscode로 넘어갈 것 같아서 vscode를 깔아서 쓰다가 터미널의 편리함을 잊을 수 없어, 리눅스 환경을 찾다가 wsl2를 선택하게 되었다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201103-3.png)

__~~*오오 이 영롱한 광채*~~__

 정말 어렵지 않았다. 중간에 windows 기능 켜기/끄기에서 windows ubsystem for linux가 체크되어있는데도 자꾸 안된다고 뜨길래, 어디서 보니 체크되어있으면 풀고 재부팅, 재부팅되면 다시 체크하고 재부팅하면 된다길래 해봤더니 ubuntu가 정상적으로 설치되면서 희망을 가질 수 있게 되었다.

 아나콘다 깔고 vscode에서 가상환경 하나 만든 다음, source activate *환경이름* , 그리고 pip3 install python-opencv 하는데 기분이 너무 좋았다. 아아... 그동안 성능이 좋지않은 노트북에 실험용으로 리눅스를 깔아서 썼는데, 다시는 안 돌아갈 것 같다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201103-4.png)

 아름다움 그 자체라고 할 수 있다. code를 치면 vscode가 나오고, 좌측 하단에는 WSL: Ubuntu-18.04가 적혀져있다. ctrl ` 를 누르면 나오는 터미널에도 리눅스에서 보던 환경이 나온다. 즐거운 코딩생활을 할 수 있을 것 같다.

