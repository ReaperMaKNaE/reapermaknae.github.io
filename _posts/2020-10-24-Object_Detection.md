---
layout : post
title : "[S-hero Rollivery]opencv 물체 판별 확인을 위한 준비와 과정"
date : 2020-10-24 20:26:00 +0900
description : object detection using opencv
img : 20201024-5.jpg
tag : [Opencv, ObjectDetection, S-hero Rollivery]
---

## Algorithm

장애물의 좌표와 로봇으로부터의 거리 [cx, cy, d]를 저장하는 obstacle 변수가 존재

cx와 cy는 로봇으로부터 상대적인 x, y좌표, d는 로봇으로부터 상대적인 거리를 의미

여기서 d를 따로 구하는 이유는 지속적인 update를 해주기 위해서 따로 계산(따로 계산하면 연산량이 조금 더 줄어들기에 위와 같이 판단함. sqrt와 같은 것을 최대한 배제한 형태로 접근)~~*근데 써버렸다*~~



로봇의 움직이는 속도는 현재 사용중인 oCamS-1CGN-U에 내장되어있는 IMU Sensor를 이용하여 계산. (Accelerator value와 Gyro value를 이용하여 구한 Euler angle과 accelerator value를 적당히 섞어서 계산할 수 있도록 설계.) 이러한 계산은 raspberrypi에서 다 끝마치고, 움직이는 속도와 euler angle만 넘겨줄 수 있도록 html pamphlet을 수정할 계획.



```html
<!-- Pamphlet 수정계획안. (아직 안함) -->
<html>
	<head>
		<title>VideoStreamingExample</title>
	</head>
	<body>
		<img id="bg" src="{{ url_for('video_feed')}}">
        <h>
            \{{velocity}}
        </h>
        <p>
            \{{eulerAngle}}
        </p>
	</body>
</html>
```



위와 같은 코드를 넘겨줄 때에는 flask에서 다음과 같은 명령어가 필요.

```python
# route 하기 전 velocity, eulerAngle 계산 끝냈다고 가정(둘 다 정수)

@app.route('/')
def index():
    # Rendering webpage and Send data
    return render_template('index.html', velocity, eulerAngle)
    
#Send Video
@app.route('/video_feed')
def video_feed():
    return Response(gen(),mimetype='multipart/x-mixed-replace;boundary=frame')
```



이전에 포스팅하였던 code에 아래와 같은 내용을 추가하여 data parsing.

```python
import requests
from bs4 import BeautifulSoup

res = requests.get('http://(RaspberryPI 주소:8080)')
soup = BeautifulSoup(res.content, 'html.parser')
velocity = int(soup.find.text('h'))
eulerAngle = int(soup.find.text('p'))
```

 이렇게 되면 굳이 같은 html에서 두 번 읽어와서 연산속도에 큰 문제가 생기지 않을까? 하고 있다. 그런데 두 개가 불러오는 형태가 달라서(특히 flask로 불러온 걸 파이썬에서 제대로 읽어서 쓰려면 좀 고생했던 것으로 기억한다) 결국 int로 받으려면 byte로 된걸 또 imdecode해서 읽어야 하는데, 이럴바에야 bs4 써서 쓰는게 낫겠다 싶어서 일단은 이렇게 생각하고 있다. 물론 두 방법 중에 무엇이 더 빠른지는 비교를 해봐야한다. 이는 추후 시간이 남는다면 생각할 문제.



 위와 같은 방식으로 data를 얻어왔으면, 이제 Algorithm으로 넘어간다. 이제 우리가 필요한 움직이는 속도, 로봇 내부 부품의 기울기, 이미지를 모두 받아왔다. 다음은 장애물을 회피해서 올바른 경로를 주행할 수 있도록 하는 능력을 길러주는 것. 하지만 이는 후에 하기로 하고, 제대로 된 velocity를 구할 수 있는지 확인해보아야 한다. 기본적으로 사용할 전/후진의 velocity control은 notion에 기록이 되어 있다. 추후(아마도 이번주 내로) 좌/우회전에 따른 회전각 control을 다룰 계획이다. (일단 머리속에 있는 방법으로, 7월 prototype을 제작했을 때 좌/우로 흔들림이 심했는데, 처음엔 회전축이 이상해서 그런줄 알았는데 자세히 생각해보면 무게중심의 이동때문에 좌/우 균형이 비틀어져 생긴 문제라고 생각하고 있다. 이러한 문제 때문에 이번에는 정말 느린속도로 주행을 할 수 있도록 설계를 할 계획이다.)



 이전에도 유사한 코드를 완성한 적이 있었으나, velocity와 independent한 코드여서, 새로 코드를 짜게 되었다.



 기존의 알고리즘은 아래와 같다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201024-4.jpg)

 countour에서 obstacle 찾고 위치, 거리 파악의 경우, 이전에는 obstacle의 위치를 기반으로만 설계를 하였던 반면, 새로 만들 알고리즘은 위치 뿐 아니라 로봇의 회전각도 및 속도등을 기반으로 하게 되어, 이전 코드의 문제였던 contour 실패 시(obstacle을 놓친 경우)에도 기존에 기억된 경로를 바탕으로 계산하여 위치를 추정할 수 있다는 것이 장점이다. 



 속도와 angle을 제외한 모든 부분에 대해서 계산이 끝났다! obstacle을 detection만 할 수 있다면 로봇의 위치를 어느정도 파악할 수 있다. obstacle detection이 되지 않았다면, 속도와 angle만으로 로봇의 위치를 컨트롤 할 수 있도록 설계하는 것을 추가한다.



속도를 제외하곤 어떻게 계산했는지에 대한 코드는 아래 링크에서 확인할 수 있다.

[https://github.com/ReaperMaKNaE/Streaming/blob/master/DrawDepthMap_Ver2.py](https://github.com/ReaperMaKNaE/Streaming/blob/master/DrawDepthMap_Ver2.py)





Reference : 

flask 데이터 관련, [https://ychae-leah.tistory.com/22](https://ychae-leah.tistory.com/22)