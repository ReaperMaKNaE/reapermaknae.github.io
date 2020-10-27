---
layout : post
title : "[S-hero Rollivery] RaspberryPI 통신 확인"
date : 2020-10-26 20:26:00 +0900
description : check raspberryPI communication
img : 20201027-1.jpg
tag : [raspberrypi, rollivery, oCamS-1CGN-U]
---

### RaspberryPI Update

 이전에 velocity를 이용해서 하겠다고 포스팅했었는데,  velocity는 따로 구하는 걸로 변경하는 바람에 다시 원래대로 돌아갔다. 이거말고 p나 h1, h2 같은 것들을 따로 뺀 후에 {{roll}}, {{velocity}}와 같은 방식으로 데이터를 받아오는 방법을 고려하였으나, 결국 데이터를 받아오기 위해선 새로운 format으로 변환을 해야했고, 여러번 읽어오는 과정에서 문제가 많이 생겼다.



 그래서 편법을 생각해냈다. __*이미지 뒤에 바이트 몇개를 추가해서 붙여 전송하면 안될까?*__

 결론부터 말하자면, __*잘 된다.*__

 근데 반응이 좀 느려서, 아두이노의 mpu6050과 같은 것들도 고려를 일단 해봐야겠다.

 아래는 html코드.



```html
<html>
	<head>
		<title>VideoStreamingExample</title>
	</head>
	<body>
		<img id="bg" src="{{ url_for('video_feed')}}">
	</body>
</html>
```

마찬가지로, 영상 송출 및 flask 명령어도 변경한다.



```python
import serial
import liboCams
import cv2
import time
import numpy as np
from flask import Flask, render_template, Response

PI = 3.141592

ser = serial.Serial("/dev/ttyACM0", 115200)
devpath = liboCams.FindCamera('video0')

if devpath is None:
  print ('oCam Device Not Found!')
  exit()

test = liboCams.oCams(devpath, verbose=0)

fmtlist = test.GetFormatList()
for fmt in fmtlist:
  print (fmt)

ctrlist = test.GetControlList()
for key in ctrlist:
  print (key, hex(ctrlist[key]))

test.SetControl(ctrlist[b'Gain'], 60)
test.SetControl(ctrlist[b'Exposure (Absolute)'],200)
test.Close()

roll = 0
pitch = 0
yaw = 0

def gen():
    test = liboCams.oCams(devpath, verbose=0)

    test.Set(fmtlist[2])
    ctrllist = test.GetControlList()
    name =  test.GetName()
    test.Start()
   
    while True:
        filteredData = ser.readline().decode()
        dataList = filteredData.split(",")
        roll = dataList[11]
        pitch = dataList[12]
        yaw = dataList[13].rstrip('\r\n')

        roll = int(int(roll)*180/(900*PI))+90
        pitch = int(int(pitch)*180/(900*PI))+180
        yaw = int(int(yaw)*180/(900*PI))

        frame = test.GetFrame(mode=1)
        rgb = cv2.cvtColor(frame, cv2.COLOR_BAYER_GB2BGR)
      
        rgb = cv2.resize(rgb, dsize=(1280,480), interpolation = cv2.INTER_AREA)
        ret, jpeg = cv2.imencode('.jpg', rgb)
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + jpeg.tobytes() + b'\x42\x4D\xF6\x04\x00\x00' + roll.to_bytes(2,"big") + pitch.to_bytes(2, "big") + yaw.to_bytes(2,"big") + b'\r\n\r\n')
        
        char = cv2.waitKey(1)

app = Flask(__name__)

@app.route('/')
def index():
    #rendering webpage
    return render_template('index.html')


@app.route('/video_feed')
def video_feed():
    return Response(gen(),mimetype='multipart/x-mixed-replace;boundary=frame')

if __name__ == '__main__':
    app.run(host='(raspberryPI ip)', port='8080', debug=True)
```

라즈베리파이가 송출하는 비디오의 IP:8080/video_feed에 송출하려면 byte로 data를 주어야한다. 무슨 오류가 떴었는데, 아래와 같이 byte를 구성했다.(물론 위 코드에도 있다.)



yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + jpeg.tobytes() + b'\x42\x4D\xF6\x04\x00\x00' + roll.to_bytes(2,"big") + pitch.to_bytes(2, "big") + yaw.to_bytes(2,"big") + b'\r\n\r\n')



 왜 다른 방식이 아니라 image뒤에 덧 붙였나면, 다른 방식으로 시도를 했었는데, 글자를 덮어 씌우는 방식이 아닌 계속 출력이 되는 바람에 송출 page에서 데이터를 읽어올 수가 없었다.(일단 내가 쓰는 python에서 byte를 읽어오는 방식은 .find()를 사용하는 방법인데, find_all이 아니기도 하고 해봐야 또 알고리즘을 추가해야 하는 것 때문에 덮어씌우는 방식이 필요했다. -.find()는 처음에 발견된 수치만 제공한다. 예를들어, 12315151512123123123 과 같은 곳에서 .find(123)이라고 하면, 맨 처음 123만 추출하고 끝낸다.)



 그럼 왜 하필 byte를 저렇게 길게 붙였냐는 문제가 또 생긴다. 너무 쓸데없이 긴거 아닌가?

jpeg를 byte로 변환할 때, 약속이 있다. jpeg signature format이라고도 하는데, 시작은 FF D8 FF로 시작을 하고, 끝낼 때에는 FF D9를 쓴다. 이러한 정해진 규약이 있기 때문에, 다른 영상들과의 충돌을 피할 수 있고 각 확장자에 따른 코덱이 필요하다.

 자 그럼, 이 jpg와 내가 보낸 데이터가 같은 파일이 아님을 증명해야한다. 단순히 숫자만을 보내는 것이기 때문에, 만에 하나 약속된 형식이 아니라면 jpg와 충돌을 일으키게 될 것이고, 통신간에 문제가 발생할 수 있다고 생각했다. 그럼 전혀 데이터 형식이 겹치지 않는 방법을 찾아야하는데, 이는 다른 signature format을 이용하는 방법을 생각했다.

 바로 비트맵이다. bitmap signature format의 경우, b'\x42\x4D\xF6\x04\x00\x00' 로 시작한다. 따라서 이걸 넣게되면, 실제로 비트맵은 아니지만 jpg와의 혹시모르는 데이터 중복을 피하기 위해 위와같은 fake byte를 넣게 되었다. 이렇게 해서 구분이 된 video는 템플릿에서 보면 다음과 같다.

(roll과 pitch에서 각각 90과 180을 더했는데, 이는 camera에서 echo "@LMGEUL" > /dev/ttyACM0를 실행하고 얻게 되는 euler angle이 음수가 나오는 경우가 있었다. roll의 경우 -90에서 90도, pitch의 경우 -180에서 180도여서 해당값을 더해 unsigned로 바꾸었다. signed number로 보내면 이래저래 귀찮아지므로 단순히 더해주어서 해결했다.)



![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201027-1.jpg)



 다음은 여기에 송출되고 있는 byte를 불러오는 작업이다.



```python
import cv2
import numpy as np
import urllib.request

video = urllib.request.urlopen("http://(raspberrypi_IP):8080/video_feed")
total_bytes = b''

totalAngleBytes = b''

while(True):
    total_bytes += video.read(1024)
    b = total_bytes.find(b'\xff\xd9') # JPEG END

    if not b == -1:
        a = total_bytes.find(b'\xff\xd8') # JPEG start
        jpg = total_bytes[a:b+2] # actual image

        #decode to colored image (another option is cv2.IMREAD_GRAYSCALE)
        img = cv2.imdecode(np.fromstring(jpg, dtype=np.uint8), cv2.IMREAD_COLOR)
        left = img[241:480, 1:640, :]
        right = img[241:480, 641:1280, :]

        cv2.imshow('Dual_image',img) # display image while receving data

        angle_start = total_bytes.find(b'\x42\x4D\xF6\x04\x00\x00')
        roll = int.from_bytes(total_bytes[angle_start+6:angle_start+8], "big")
        pitch = int.from_bytes(total_bytes[angle_start+8:angle_start+10], "big")
        yaw = int.from_bytes(total_bytes[angle_start+10:angle_start+12], "big")

        AngleInBytes = [roll, pitch, yaw]
        print('AngleInBytes : ', AngleInBytes)

        if cv2.waitKey(1)==27:
            break

cv2.destroyAllWindows()
```

 urllib 을 이용해서 byte로 출력된 데이터에 접근하고, 1byte씩 읽는 다는 의미에서 while문 처음에 total_bytes += video.read(1024)를 더한다.

 여기서 jpeg의 시작과 끝은 각각 \xff\xd8, \xff\xd9이므로 (시작에서 xff를 한번 더 안쓴건 없어도 잘되었기에.) 이를 찾아서 해당하는 바이트만큼 jpg라는 변수에 저장하고, imencode를 이용해서 이미지로 바꾼다. 이 때 받은 이미지는 640x480의 이미지 2개가 붙어서 출력된 1280x480의 이미지이므로, left와 right에 각각 따로 저장을 한다.

 angle_start에 순서를 집어넣고, 각각 roll, pitch, yaw를 불러온다. ("big"의 의미는 아래 refer의 python byte변환 참조.) 그리고 각 데이터를 일단 만들어서 잘 불러오는지 본다.

결과는 아래와 같다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201027-2.jpg)



문제가 있다면, 데이터를 받아오는 속도가 느리다. __*~~아니 영상은 실시간으로 잘 받아오면서 IMU는 왜??~~*__

이러한 현상이 계속 지속된다면, 아무래도 그냥 아두이노에 IMU sensor를 따로 쓰는게 낫겠다는 생각이 들었다. 어짜피 feedback만 잘 줄거면 아두이노로 컨트롤해도 문제없지 않을까....?

(원래 이 부분은 camera calibration으로 해결을 도와주려고 시작했던 파트인데, 잘 안된다. 일단은 아두이노 통신 점검이라는 할 것이 남아서 이부분을 먼저 해결하고 다시 돌아와서 *~~과연 돌아올까?~~* 해결해보려고 한다.)



video를 flask를 이용해서 얻어오는 방법은 옛날에 봤었고 notion에 업데이트를 해놨었지만, 혹시나 필요한 사람이 있을까 하여 레퍼에 추가한다.





### Reference

flask를 이용한 video 송출, [https://medium.com/datadriveninvestor/video-streaming-using-flask-and-opencv-c464bf8473d6](https://medium.com/datadriveninvestor/video-streaming-using-flask-and-opencv-c464bf8473d6)

utterances, [https://github.com/utterance/utterances](https://github.com/utterance/utterances)

jpeg signature format, [https://www.file-recovery.com/jpg-signature-format.htm](https://www.file-recovery.com/jpg-signature-format.htm)

bitmap signature format, [https://www.file-recovery.com/bmp-signature-format.htm](https://www.file-recovery.com/bmp-signature-format.htm)

python byte 변환 : [https://docs.python.org/3/library/stdtypes.html#bytes.hex](https://docs.python.org/3/library/stdtypes.html#bytes.hex)