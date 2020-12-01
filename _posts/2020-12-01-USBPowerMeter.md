---
layout : post
title : "USB Powermeter and USB HUB"
date : 2020-12-01 +0900
description : Arduino Project - USB Powermeter and USB HUB
tag : [arduino]
---

### USB Powermeter and USB HUB



 회로는 예전에 만들었으나, 3D Printer를 사용하는 데에 기간이 자꾸 미뤄지다보니 이제야 완성하게 되었다.

만들면서 참고한 곳은 아래와 같다.

어떤 센서를 쓰면서 에러가 생겼는데, 그 에러를 해결한 곳 : [https://github.com/adafruit/Adafruit_LSM303_Accel/issues/3](https://github.com/adafruit/Adafruit_LSM303_Accel/issues/3)
아두이노 USB powermeter project 진행하신 LUFT - AQUILA님 블로그, [https://luftaquila.tistory.com/8?category=726570](https://luftaquila.tistory.com/8?category=726570)
전압 측정 센서, [https://m.blog.naver.com/kwy1052aa/221761758616](https://m.blog.naver.com/kwy1052aa/221761758616)



 USB Powermeter는 전력관리에서 중요한 역할을 해 주기도 하고, 소모전류 등을 파악하기 쉽기에 만들기로 결정하였고, 이왕 만들거 HUB를 만들기로 했다. 예전에 사놓은 HUB가 고장이 났기에.



 일단 사용할 품목은 아래와 같다.

 Arduino Nano 호환보드(4,000원), [https://www.devicemart.co.kr/goods/view?no=1342039](https://www.devicemart.co.kr/goods/view?no=1342039)

 전압 측정 센서 모듈(1200원), [https://www.devicemart.co.kr/goods/view?no=1327414](https://www.devicemart.co.kr/goods/view?no=1327414)

 INA219 아두이노 DC 전류 센서 모듈(3900원), [https://www.devicemart.co.kr/goods/view?no=1327562](https://www.devicemart.co.kr/goods/view?no=1327562)

 USB A/F 커넥터 5개(150*5 = 750원), [https://www.devicemart.co.kr/goods/view?no=1322109](https://www.devicemart.co.kr/goods/view?no=1322109)

 핀헤더(대충 많이), [https://www.devicemart.co.kr/goods/view?no=2825](https://www.devicemart.co.kr/goods/view?no=2825)

 핀헤더소켓(대충 많이), [https://www.devicemart.co.kr/goods/view?no=3587](https://www.devicemart.co.kr/goods/view?no=3587)

 USB - USB 케이블(900원), [https://www.devicemart.co.kr/goods/view?no=1112188](https://www.devicemart.co.kr/goods/view?no=1112188)

 만능기판 적당히 싼거(280원), [https://www.devicemart.co.kr/goods/view?no=1341710](https://www.devicemart.co.kr/goods/view?no=1341710)

 

 총 가격은 약 약 2만원 정도.(11030 + 핀헤더/핀헤더소켓 값 + 와이어 등 기본 장비들의 cost + TFT LCD 혹은 OLED)



 TFT LCD(혹은 OLED) 포함하면 2만원근처일텐데, 난 예전에 프로젝트하다가 남은 LCD가 하나 있어서 쓰기로 했다. 핀헤더, 핀헤더소켓은 DIY하는 사람이라면 정말 많을거라서 괜찮고, 추가하면 괜찮은 것은 microSD 메모리카드 리더기 정도면 괜찮을 것 같다.



![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-1.png)

 일단 주문한 부품이 오면 대충 각을 좀 잡는다. 음.. 이쯤 납땜하면 되겠구만! 하고 슥슥 해본다.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-2.png)

 좀 원시적이긴한데, 대충 이렇게 해놓으면 나중에 납땜하면서 안헷갈리고 잘 되었다.



 다음은 핀헤더소켓을 자르는건데, 나는 유튜브에서 보고 배운거라 정석은 아닐 수도 있다. 그래도 나름 잘 짤려서 이 방법 처럼 자르는 중. 예전에 핀헤더 자르듯이 가운데에 니퍼넣고 자르면 소켓이 자꾸 날아다녀서 한개씩 버리더라도 이게 훨씬 더 안전한 방법이라고 생각하고 있다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-3.png)

 일단 이렇게 한쪽을 끼운다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-4.png)

 그리고 개수에 맞게 롱노즈로 잡고, 이를 힘껏 당기면 뽑힌다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-5.png)

 그리고 뽑힌놈을 칼로 자르자. 우측 그림처럼 잘려서 울퉁불퉁한데, 사포로 갈아내는게 최고이지만, 없으면 없는대로 써도 된다.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-6.png)

 lcd까지 붙여준 이후, 위와 같이 대충 맞춰준 후, LCD에 코드를 업데이트했다.

 참고로 ST7735 1.8inch 를 썼는데, 이게 모델이 생산된 제조년도마다 조금씩 다른 것 같다. 어디서샀는지도 모르겠는데 옛날에 이걸로 작업하면서 작성해놓은 코드에 어떻게하면 된다라고 다 기록을 해놓아서 덕분에 사용이 가능했다. 아무튼 코드는 아래와 같다.

```c
#include <Adafruit_INA219.h>
#include <Adafruit_GFX.h>    // Core graphics library
#include <Adafruit_ST7735.h> // Hardware-specific library for ST7735
#include <Adafruit_BusIO_Register.h>
#include <Wire.h>
#include <ForWarning_1.0.h>

//Refer
//https://github.com/adafruit/Adafruit_LSM303_Accel/issues/3
//https://luftaquila.tistory.com/8?category=726570
//https://m.blog.naver.com/kwy1052aa/221761758616

#define SerialDebugging true

int TFT_SCLK = 12;  //CLK
int TFT_MOSI = 11;   //MOSI is same as SDA/SDI
int TFT_DC = 10;     //DC is same as SS(Display collect)
int TFT_RST = 9;
int TFT_CS = 8;

int voltageInput = A7;


float voltage = 0;
float filteredCurrentVal = 0;
float current = 0;
float power = 0;
float vin = 0;
float vout = 0;
// 전압 측정 센서에 달려있는 SMD resistor. 위 전압측정센서값을 정리한 블로그 참조
float R1 = 30000.0;
float R2 = 7500.0;

ForWarning filter;
Adafruit_ST7735 tft = Adafruit_ST7735(TFT_CS, TFT_DC, TFT_MOSI, TFT_SCLK, TFT_RST);
Adafruit_INA219 ina219;

void setup(void) {
  ina219.begin();
  Serial.begin(9600);
  
  // Use this initializer if using a 1.8" TFT screen:
  tft.initR(INITR_BLACKTAB);      // Init ST7735S chip, black tab
  tft.fillScreen(ST77XX_BLACK);
  //Rotate TFT LCD Screen
  tft.setRotation(1);

  // Filter reset
  filter.resetAll();
  filter.getNumberOfData(5);
}

void loop() {

  current = ina219.getCurrent_mA();
  if(current < 0){
    current = 0;
  }
  filter.SaveDataOfDust(current);
  filteredCurrentVal = filter.CalculateFilteredDataForDust();
  
  vout = (analogRead(voltageInput)*5.0)/1024.0;
  vin = vout / (R2 / (R1 + R2));
  voltage = vin - 0.2;
  if(voltage < 0){
    voltage = 0;
  }

  power = voltage * filteredCurrentVal;
  
  //TFT LCD output
  //tft.fillRect(화면의 왼쪽위부터 우측으로, 화면의 왼쪽위부터 아래로, 상자의 가로길이, 상자의 세로길이, 색깔);
  //tft.fillRoundRect(50,20,40,30,ST7735_BLACK);
  //위 방법 매우 별로. 지우는게 화면에보임
  tft.setCursor(0,0);
  //위 지우는 방법을 아래 textColor에서 해결.
  //setTextColor(쓸 글씨 색, overwrite할 글씨색)
  tft.setTextColor(ST77XX_WHITE,ST77XX_BLACK);
  tft.setTextSize(2);
  tft.println("   Voltage");
  tft.print("    ");    // 3 spaces
  tft.print(voltage);
  tft.println("V");
  tft.setTextSize(1);
  tft.println("");
  tft.setTextSize(2);
  tft.setTextColor(ST77XX_RED,ST77XX_BLACK);
  tft.println("   Current");
  tft.print("    ");    // 3 spaces
  tft.print(filteredCurrentVal);
  tft.println("mA");
  tft.print("    ");    // 3 spaces
  tft.setTextSize(1);
  tft.println("");
  tft.setTextSize(2);
  tft.setTextColor(ST77XX_CYAN,ST77XX_BLACK);
  tft.println("    Power");
  tft.print("    ");    // 3 spaces
  tft.print(power);
  tft.println("mW");
  tft.print("    ");    // 3 spaces
}
```



 그리고 자로 대충 USB를 넣을 칸 등의 길이를 재고 모델링을 한 후, 3D Printer로 뽑아낸다.(학교에서 예약하는데에만 3주는 걸린 것 같다)

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-8.png)

 아래는 납땜을 끝내고 케이스에 조립한 형태.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-7.png)

 그리고 굴려보면!

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-9.png)

 동작한다!



 여기서 D+, D-를 input으로 잘 설정해야한다. 

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201201-10.png)

 우측 사각형만 연결했는데, 저쪽으로 USB를 연결하면 단순 전원 공급 뿐 아니라 아두이노 코드 업로드, 마우스, 키보드 등도 사용이 가능하다. 다 연결해서 모든 곳을 다 쓰는 방법도 있는데, 그럴려면 설계를 다시 해야한다.

 아래는 간단한 시연 영상.

 전원을 연결할 때 마다 LCD가 reset이 되는데, 원인은 좀 알아봐야할 것 같다.



<iframe width=100% height=315 src="https://www.youtube.com/embed/OF3SSSf7ucY" frameborder="0" allowfullscreen></iframe>

 썩 마음에 들진 않아서 나중에 다시 만들 거지만, 일단 여기서 마무리.

