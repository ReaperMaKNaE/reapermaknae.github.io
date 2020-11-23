---
layout : post
title : "[S-hero Rollivery] Arduino 통신 확인"
date : 2020-10-27 +0900
description : check arduino communication
img : 20201027-4.jpg
tag : [S-hero Rollivery, arduino]
---

### Arduino Update

라즈베리파이 통신 확인 이후 바로 아두이노도 확인했다.

아두이노는 단순히 통신이 되는지만 확인하면 되기에, 큰 어려움이 없었다. 먼저 아두이노에 업로드한 코드는 다음과 같다.

```c
#include <SoftwareSerial.h>

int BTx = 2;
int BRx = 3;
int value = 0;

SoftwareSerial bluetooth(2,3);

void setup() {
  Serial.begin(9600);
  bluetooth.begin(9600);
  // The pins which allow PWM is 5, 6, 9, 10, 11
  pinMode(5, OUTPUT);
  pinMode(6, OUTPUT);
  pinMode(9, OUTPUT);
  pinMode(10, OUTPUT);
  // Always 5V pin
  pinMode(8, OUTPUT);
  digitalWrite(8,HIGH);
}

void loop() {  
  if(bluetooth.available()){
    
    value = bluetooth.parseInt();
    analogWrite(5, 0);
    analogWrite(6, value);
    analogWrite(9, 0);
    analogWrite(10, value);
  }
}
```

Arduino Uno의 경우, 5, 6, 9, 10, 11 pin이 PWM이 가능하여 (물론 다른 pin도 몇개 더있는데 나는 4개만 필요하면 되기때문에 다른 것들은 일단 제외했다. Arduino 핀 옆에 어떤 핀이 PWM이 가능한지 적혀져있다.)



5, 6, 9, 10 핀을 DC Motor control용으로 만들었다.



대충 bluetooth를 통해서 어떤 값을 받으면, int로 받아서 그 값만큼 모터에 출력시키는 것이다. 

pinMode 8를 항상 HIGH로 설정해두었는데, 지금 회로에 5V와 GND를 부재시키는 바람에 회로를 다시 만들어야한다. 그래서 임시방편으로 5V Pin을 한 개 더 만들었다.(쉽게 말해 5V, GND pin만 늘리면 의미없는 핀이다)



 다음은 컴퓨터에서 컨트롤할 python코드이다.

```python
import serial

arduino = serial.Serial(
    port = 'COM4', baudrate=9600,
)

while True:
    op = int(input())
    if op == 100:
        arduino.write(bytes(str(0), "ascii"))
        break
    else:
        arduino.write(bytes(str(op), "ascii"))
```

상당히 simple하다. 

port의 경우, 아래와 같은 경로에서 확인이 가능하다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201027-5.jpg)

검색창에서 "bluetooth 및 기타 디바이스" 를 검색한 후, 아두이노 HC-06(기본인 경우 이름이 이렇다. AT명령어를 통해 변경한 경우 다른 이름으로 존재함.)을 연결한 후에 우측 관련설정 탭에서 '추가 Bluetooth 옵션' 을 클릭한 후, COM 포트 탭에서 송신에 할당된 포트를 찾으면 된다. 이 포트를 기준으로 write를 하면 된다.

위의 경우 op = int(input())으로 하였는데, variable 같은 것들을 넣어도 잘 동작한다.

(예를 들면 if i < 255: i+=1; else : i = 0    ㅡ ;는 엔터라고 친다. ㅡ 이후 i를 byets(str(i)로 보내는 것)) 

아래는 연결된 부품들의 사진을 찍어본 형태.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201027-4.jpg)



그리고 아래는 모터드라이버 회로도이다. 사진을 원노트에 저장해놔서 계속 불러와야하는데, 쉽게 참고하려고 블로그에도 기록을 남긴다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201027-3.jpg)



물론 여기서도 어김없이 문제가 발생했다. PWM 신호를 줄 때, 배터리로 아두이노에 파워를 주면 하다가 자꾸 모터가 멈췄는데, 아두이노의 파워를 노트북에 꽂아서 공급하니 아무 문제없이 잘 동작했다. 이건 배터리가 충전이 덜 된 상태라고 판단해서, 충전 제대로 한 후 내일 다시 확인해봐야겠다.

내일 할 것은 결국 5V, GND 핀 더 늘리고(납땜), 아두이노 통신 재확인, 라즈베리파이 및 아두이노 동시 확인을 해 볼 계획이다. 라즈베리파이에서 이미지에 생기게 되는 임의의 결과(내일 추가 보충할 계획)에 따라 변수를 다르게 설정하고, 그 값에 따라 모터를 돌게 해봐야겠다. 





### Reference

L298N Pin Diagram, [https://components101.com/ics/l298-pin-configuration-features-datasheet](https://components101.com/ics/l298-pin-configuration-features-datasheet)