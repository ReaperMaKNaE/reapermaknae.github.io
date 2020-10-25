---
layout : post
title : "[S-hero Rollivery] oCamS-1CGN-U - IMU Sensor값 받아오기, 로봇 속도 결정"
date : 2020-10-25 +0900
description : Find velocity from IMU
tag : [Opencv, S-hero Rollivery, IMU, Kalman filter]
---

### oCamS-1CGN-U, IMU Sensor값 받아오기



일단 IMU Sensor data를 불러와야 한다.

notion에도 업로드 되어있지만, 찾으려면 다시 notion을 들어가기보다 로컬에 위치한 마크다운 문서를 보는게 더 빨라서 추가업로드를 진행하게 되었다.

일단 노션에 있는 내용을 그대로 들고와서 보면,



__*우리가 사용하는 oCamS-1CGN-U stereo camera에서 다행히도 IMU sensor의 값을 제공해준다. 이 값을 지속적으로 받는다. 기존 값은 LMGQUA로 등록이 되어 있기 때문에, 라즈베리파이 ssh 연결 이후 아래와 같은 명령을 입력하여 카메라에서 LMGEUL로 euler각을 얻을 수 있도록 한다.*__

__*echo "@LMGEUL" > /dev/ttyACM0*__

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201025-3.jpg)

자료출처 : withrobot 홈페이지 oCamS-1CGN-U 제품 브로셔



이 때까지만 해도 putty를 이용해서 ssh접속을 했었는데, 컴퓨터가 알 수 없는 이유로 디렉토리와의 연결이 전부 끊겨서(무슨 data관리 프로그램을 삭제한 것 때문인 것 같다. 아무리 그래도 윈도우 하나만 깔고 코드 에디터/캐드같은 것들만 깔았는데 230기가를 차지하는건 좀 문제가 있었다. 지금은 필요한 것만 깔았는데도 200기가를 넘어갔다.) 다시 putty를 깔려다가, window openssh를 이용하면 cmd에서도 ssh접속이 가능하다는 것을 깨닫고 접속을 하게 되었다.



아래는 그 접속화면.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201025-1.jpg)

이후 ls와 cd를 이용해 적당히 경로를 찾아가서, echo 명령어로 받는다.

다음은 예시로 roll, pitch, yaw data를 들고오는 방법.

``` python
import serial
PI = 3.141592
ser = serial.Serial("dev/ttyACM0", 115200)

while True:
    filteredData = ser.readline().decode()
    dataList = filteredData.split(",")
    roll = dataList[11]
    pitch = dataList[12]
    yaw = dataList[13].rstrip('\r\n')
    
    roll = int(roll)*180/(900*PI)
    pitch = int(pitch)*180/(900*PI)
    yaw = int(yaw)*180/(900*PI)
    
    print('roll : ', int(roll), 'pitch : ', int(pitch), 'yaw : ', int(yaw))
```

위 코드를 실행하면,

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201025-2.jpg)

이렇게 된다.

확실히 라즈베리파이라서 연산이 빠르다.





### Velocity



이제 다음은 velocity를 어떻게 주는가- 에 대해서 조금 고민을 해야하는데, 이 부분에 대해서는 control부분에서 다루게 될 내용이기에 여기에서는 생략할 계획이다. 원래는 IMU Sensor의 값으로부터(Acceleration, Magnetometer, Gyro values) 계산을 다 하려고 했는데, 일단 라즈베리파이가 image data 전송에만 더 힘을 쓰게 만들기 위해 이러한 계산은 다 빼버리는게 낫다는 판단하에 제외시켰다. 어짜피 input을 주는 것은 노트북으로 줄 것이고, 속도는 천천히 맞추어 나갈 계획이기 때문에 전혀 무리하게 맞출 생각이 없다. 물론 이 속도를 맞추는 건 아마도 조금 고생을 할 것 같은데(~~*노가다*~~), 일단 대충 계산한 내용은 아래와 같다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201025-4.jpg)

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201025-5.jpg)

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201025-6.jpg)

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201025-7.jpg)



결론만 말하자면(어짜피 에너지를 중심으로 푼 식이기 때문에 가운데가 어쨌단건 필요가 없다. ~~*사실 정리하면 풀리긴 하는데 지금 MATLAB이 없고, 있다해도 그냥 노가다 뛰는게 속이 편하다*~~ ), 두 개의 각속도가 같아야지만 등속운동을 한다.(물론 pendulum은 아래를 향하는 기준)



 여기서 등속운동 조건을 맞추기 위해서, 전진을 하고 있을 때에 roll각을 어느정도 유지하도록 feedback을 주어야 하는데, PID 계수 맞추듯 아마 노가다를 해야할 것이다. 이걸로 velocity를 맞출수만 있다면, 위 처럼 IMU Sensor에서 필터까지 써가며 속도데이터를 얻는 과정이 생략되므로 데이터처리나 시스템의 response가 더 확보된다고 생각한다.

 velocity를 구하는 건 기어비/rpm만 알면 쉽게 구하기 때문에, 주문한 3d printer 부품이 도착하고 조립한 후에 끼워넣고 돌려보면 된다.



 위 내용과 비슷한 방식으로 분석한 [논문](https://ieeexplore.ieee.org/document/7219709)이 있다. 계산 결과는 조금 다른데, 조건이 다르기 때문.



 이제 남은 것은 좌/우 회전을 할 때 control이 안정적으로 되는지 확인하는 방법이 남았다. 이것도 엄청나게 계산을 해서 때려넣어야 하는데, 마찬가지로 연산에 부담을 덜 주기 위해 실험적으로 안정적인 값을 찾을 계획이다. 통신속도에 따른 data 처리 방법이 다 달라지기 때문에, 결국 이게 제일 현실적일 것 같다.(~~*결국 노가다*~~)





Reference :

oCamS-1CGN-U - 위드로봇, [http://withrobot.com/camera/ocams-1cgn-u/](http://withrobot.com/camera/ocams-1cgn-u/)

Lagrangian, [https://en.wikipedia.org/wiki/Lagrangian_mechanics](https://en.wikipedia.org/wiki/Lagrangian_mechanics)

K. Landa and A. K. Pilat, "Design and start-up of spherical robot with internal pendulum," 2015 10th International Workshop on Robot Motion and Control (RoMoCo), Poznan, 2015, pp. 27-32, doi: 10.1109/RoMoCo.2015.7219709.

