---
layout : post
title : "Typora Test for github"
date : 2020-10-24 08:26:00 +0900
description : typora test
img : 20201024-1.png
tag : [Typora, Introduction]
---

## Typora 설치

 아직은 대부분의 툴이 window에 있어서, 따로 git을 써야하는 상황이 아닌 한 linux를 들어갈 필요가 없다보니, pycharm과 visual studio, code block이 있는 window에서 블로그 내용을 주로 작성하기 위해 Typora를 설치했다.

 이전 두 개의 게시글은 vscode에서 markdown all in one을 받아 사용을 했었는데, 문법을 직접 써야 하다보니 조금 힘든감이 없잖아 있었다.  

 개행연습도 하고 했는데, 역시 Typora를 설치하는게 훨씬 더 편한 것 같다.



## 기타 내용

  blog를 운영하는 것은 처음인데, 아마 주 내용은 일기 비슷하지 않을까 한다.



 주로 사용하는 언어는 Python이며, 기타로 C, C++, MATLAB, Verilog, MIPS(이걸 쓸 데가 있을까?) 등이 있다. 원래는 로봇 하드웨어 분야로 진로를 희망했었으나, 이것 저것 만들다보니 관심이 오히려 바뀌면서 개발로 오게 되었다. (사실 처음에는 로봇도 아니었다.) 그래서 회로설계나 역학적 구조계산 등의 내용도 포스팅이 될 수 있다.



 아래는 현재 작업중인 자그마한 파이썬 코드.

```python
import cv2
import numpy as np
import urllib.request
import serial

# To start real time streaming in raspberryPi
# mjpg_streamer -i "input_raspicam.so -vf" -o "output_http.so -p 8090
#                   -w /usr/local/share/mjpg_streamer/www/"
video = urllib.request.urlopen("http://192.168.0.18:8080/video_feed")
total_bytes = b''

while(True):
    total_bytes += video.read(1024)
    b = total_bytes.find(b'\xff\xd9') # JPEG END
    if not b == -1:
        a = total_bytes.find(b'\xff\xd8') # JPEG start
        jpg = total_bytes[a:b+2] # actual image
        total_bytes = total_bytes[b+2:] # other informations

        #decode to colored image (another option is cv2.IMREAD_GRAYSCALE)
        img = cv2.imdecode(np.fromstring(jpg, dtype=np.uint8), 											cv2.IMREAD_COLOR)
        left = img[241:480, 1:640, :]
        right = img[241:480, 641:1280, :]
        
        cv2.imshow('Dual_image',img) # display image while receving data
        
        if cv2.waitKey(1)==27:
            break
            
cv2.destroyAllWindows()
```



## 관리하면서 알아낸 것.

tag를 안적으면 글이 안보인다. 일단 꾸역꾸역 써봐야겠다.

 ~~아마도 금방 테마를 바꿀 것 같다.~~  ~~그렇게 다른것들 기웃기웃거리다 다시 bef로 돌아왔다.~~

image는 optional인데 혹시나해서 아무거나 넣어본 상황.



