---
layout : post
title : "[S-hero Rollivery] 3D 부품 조립 시작(2), 통신 확인"
date : 2020-10-30 +0900
description : 3D parts assemble
img : 20201030-1.png
tag : [S-hero Rollivery]
---

### 3D Parts Assemble

  어제는 조립한 후 복귀하고 IMU Sensor를 연구하다가 시간이 지나가버려서 이틀치를 한꺼번에 업데이트하게 되었다.

 일단은 조립상황이다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-1.png)

 좌측사진 : 하부구조, 우측 사진 : 상부구조.

 상부구조의 경우 연결이 되어있지 않은데, 추후 해머드릴을 이용해서 구멍을 뚫고 경첩을 달 예정이다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-2.png)

 구리를 잘라 무게추로 설치(좌측), 베어링과 베어링마운트, 홀더(우측)

 미스미에서 구입한 금속 절단기였는데, 사용할 때 악력이 최소 50kg은 필요하다. 두장을 자르니 숟가락 쥘 힘도 없어져서, 자른것만으로 일단 설치를 하였다.

 우측의 경우 베어링 마운트/홀더이다. 꽉 조여야 하기때문에 일부러 공차없이 설계했는데, 덕분에 두시간 동안 가공했다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-3.png)

 좌측 상단부터 시계방향으로, 베어링결합 -> 베어링마운트, 홀더, 기어 결합 -> 정면사진

 사진에서 좌측 기어의 기어를 잠깐 뺐는데, 가공__~~(사포질)~~__하다가 손이 기어에 긁혀서 잠시 뺐다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-4.png)

 베어링마운트/홀더가 모두 결합된 사진(좌측상단)과, 기어가 결합된 좌측하단, 그리고 내부구조를 하단부에 올려놓은 사진(우측)

 베어링과 마운트/홀더를 모두 끼워맞추고, 기어를 집어넣어서 조립하였다. 이후 우측 사진과 같이 하단부에 올려놓기만 하고 추가 촬영을 하였다. 하단부 좌/우측에 조립을 하기 위한 shaft가 꽂혀있는 것을 확인할 수 있다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-5.png)

 샤프트를 다 집어넣으면, 큰 기어와 고정이 되어 위 사진과 같이 결합이 된다. (베어링 마운트도 같이 결합이 된다! - 베어링 마운트도 위쪽 사진에서 보면 4개의 구멍이 있다.)

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-6.png)

 이후 완벽히 결합된 모습. 혹시나 샤프트가 빠질 것을 염려하여, 잘 보이지는 않지만 우측사진에서는 shaft에 와셔와 너트를 넣어서 고정해놓았다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-7.png)

 아래 무게추부분을 설치한 모습이다. 아래 모터가 회전하면 좌측과 같은 형태가 된다. 이를 다시 안정적인 상태로 돌려놓고 전/후면을 촬영한 사진(가운데, 우측사진)



 남은 부분은 상단부 결합, 입구 결합, 사이드암/헤드부 결합이다. 헤드부결합의 경우 라즈베리파이의 코드업데이트가 끝나면 바로 올릴 계획이다. 



### 라즈베리파이

 일전에 카메라의 IMU Sensor가 반응이 늦다고 했었는데, 사실이었다. flask를 이용한 video streaming을 제외하고 단순히 이미지 출력만 하는걸로 했는데, USB 3.0 port하나로 전송하는 데이터가 아무래도 연산속도가 느려서인지 IMU Sensor의 데이터를 제대로 못받아왔다. 

 이러한 문제가 발생하는 부분은 ioctl이였는데, 단순히 버퍼를 초기화시키고, 버퍼에 카메라 데이터를 쌓은 후, 이 데이터를 이미지로 조합해서 opencv로 넘기게 되는데 이 과정이 무려 IMU Sensor의 데이터를 4초나 delay시켰다. 아무래도 일정시간동안의 frame data를 미리 버퍼에 저장하고 전송하는 방식으로 인해 생긴 문제라고 생각되어, oCamS-1CGN-U에 내장되어있는 IMU Sensor인 BNO 005모델은 사용하지 않게 되었고, 원래 arduino 용으로 사용하려고 두었던 MPU9250을 꺼내서 연결하였다.



 mpu9250에서 data를 얻어오려면 i2c로 연결을 해야하는데, 라즈베리파이의 i2c가 이미 어디랑 연결이 되어있었다. 이는 확인해본 결과 라즈베리파이 배터리와 연결된 모듈이 사용하고 있는 것이었고, 이에 따라 hidden i2c pin인 i2c-0를 활성화 시키는 방법을 사용했다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-8.png)

 hidden i2c pin을 활성화 시키니 10과 11이 같이 떴다.(??) 

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-9.png)

 확인해보니 모르는 다른 기기가 한개 더 있는 것 같다. 이는 라즈베리파이4에서 추가가 된 것 같은데, 일단 작동은 되기에 큰 문제없다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-10.png)

 조금 이상한 점이 있는데, 라즈베리파이를 일절 건드리지 않고 i2cdetect -y 0을 입력하면 68번(IMU Sensor의 주소)가 비었다고 뜨다가, 다른 것들을 detect한번씩 해보고 다시 i2cdetect -y 0을 detect하면 갑자기 잡힌다. 이는 라즈베리파이를 부팅하자마자 하여도 해당이 되고, 부팅후 한참 뒤에 해도 다른 것을 detect하고 하여야 0번 i2c에 활성화되었다.



 다음은 평범한 mpu코드이다. 나는 라이브러리로 만든 후, 영상을 송출할 때에 한번씩만 사용할 계획이었기에, 아래와 같이 코드를 짰다.

```python
import smbus
import math
import time

class MPU9250:
    def __init__(self):
        ACCEL_CONFIG = 0x1C
        power_mgmt_0 = 0x6b
        power_mgmt_1 = 0x6c
        self.bus = smbus.SMBus(0) #i2c-0을 썼으므로 SMBus(0)을 지정.
        # IMU 깨우기
        self.bus.write_byte_data(self.address, power_mgmt_1, 0x00)
        time.sleep(0.1)
        # accelerometer 설정. +-2g
        self.bus.write_byte_data(self.address, ACCEL_CONFIG, 0x00)
        time.sleep(0.1)
	
    def read_byte(self.adr):
        return self.bus.read_byte_data(self.address, adr)
    
    def read_word(self.adr):
        # 각 데이터는 16비트(2바이트)로 저장되어있다.
        # high는 윗 8자리, low는 아래 8자리.
        # 아래와 같은 명령어를 통해 val에 16비트숫자를 저장한다.
        high = self.bus.read_byte_data(self.address, adr)
        low = self.bus.read_byte_data(self.address, adr+1)
        val = (high << 8) + low
        return val
    
    def read_word_2c(self, adr):
        val = self.read_word(adr)
        if (val >= 0x8000):
            return -((65535-val)+1)
        else:
            return val
        
    def dist(self, a, b):
        return math.sqrt((a*a)+b*b))
        
    def getRoll(self,x,y,z):
        radians = math.atan2(-y,z)
        return math.degrees(radians)
    
    def getPitch(self, x, y, z):
        radians = math.atan2(x, self.dist(y,z))
        return -math.degrees(radians)
    
    def GetRollPitch(self):
        gyrX = self.read_word_2c(0x43)/57.3
        gyrY = self.read_word_2c(0x45)/57.3
        gyrZ = self.read_word_2c(0x47)/57.3
        
        magX = self.read_word_2c(0x03)
        magY = self.read_word_2c(0x05)
        magZ = self.read_word_2c(0x07)
        
        #위에서 ACCEL_CONFIG에 0을 주었으므로, +-2g 보정, 16384를 나눠준다.
        accX = self.read_word_2c(0x3b)/16384
        accY = self.read_word_2c(0x3d)/16384
        accZ = self.read_word_2c(0x3f)/16384
        
        pitch = self.getPitch(accX, accY, accZ)
        roll = self.getRoll(accX, accY, accZ)
        
        return int(roll), int(pitch)
```

 위에서 보면 roll과 pitch만 return하는데, accelerometer로 yaw를 구할 수 없기 때문이다. 자기센서의 힘을 빌려서 사용을 하면 되지만, 일단 이걸로 통신이 잘 되는지 먼저 확인을 해 봐야되기 때문에 위와 같이 마무리를 짓고, flask를 이용한 영상송출에서는 GetRollPitch()만 불러내서 데이터 전송이 잘 되는지 확인해봤다.

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201030-11.png)

 실시간으로 잘 물어온다. 불러온 데이터가 좀 말이 안되는 것 같은데, 데이터를 애초에 전송해줄 때 1000을 더했다.(signed data를 flask로 송출할 수 없어서) 그래서 1000이 0도인것을 기준으로 위와 같이 각도가 잡힌다.

 그걸 감안해도 값이 이상하다면, 맞다. 위는 짐벌락이 걸린 상태다.

 짐벌락에 대해서는 다른 곳에서도 정말 말이 많고 한데, 정말 쉽게 설명하면,

__난 roll각만 돌렸는데 왜 yaw나 pitch가 같이 변하지??__ 하면 짐벌락이라고 보면 된다.



 다른곳에서 쿼터니언을 이용하여 각도를 받은 후 이를 다시 오일러로 바꿔서 짐벌락을 피하는 형태를 자주 보여주는데, 이를 응용해서 한번 직접 코드를 짠 후 완성이 되면 라즈베리파이를 장착한 로봇조립과 함께 포스팅할 계획이다. ~~*(다음주 월요일까지는 끝낼 수 있다는 희망을 품고...)*~~