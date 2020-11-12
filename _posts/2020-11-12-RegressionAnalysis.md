---
layout : post
title : "Regression Analysis를 이용한 야매 filter"
date : 2020-11-12 +0900
description : Filter using regression analysis
img : 20201111-4.png
tag : [math]
---

### Regression analysis

 갑자기 regression analysis를 포스팅하게 되었다.

 ~~*사실 USB Powermeter를 작성하려고 했는데, 아직 덜만들었다.*~~

 이전에 rollivery나 다른 작업들을 하면서 regression analysis를 이용해서 필터를 썼다고 했는데, 아무래도 기록으로 남겨두는 편이 좋을 것 같아 작성하게 되었다.

 regression analysis, 한글로 회귀분석이라고 불리는 이 방법은 정말 쉽게 말하면 '추세선' 이다. 수많은 점이 찍혀있다면, 추세선을 슥- 그려서 아, 대충 요런 분포구나. 하고 나타내주는 것이다.

 아마도 엑셀이나 데이터 관리 프로그램에서 많이 써 봤을 것이다. (한국인이라면 특히 엑셀에서.)

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201112-1.png)

출처 : 위키피디아 regression analysis

 이걸 AI에서 loss를 줄이는 방법을 가르쳐주는 것으로 상당히 많이 쓰는데, 아마도 공대생이 아닌 사람들은 대부분 이렇게 응용된 케이스를 먼저 봤을 것이다.

 하지만 실제로 공대생들은 수치해석 과목(numerical analysis)에서 배우게 된다.

 수치해석이란 과목에서는 보통 여러 수학적인 모델에서 컴퓨터가 계산할 수 있는 방식에 대해 배우게 된다.(물론 아마도 가르치는 교수님마다 다를 수도 있다).

 그럼 뭐라고 배우느냐?

 바로 위에 있는 위키피디아 그림의 빨간색 선을 파란색 점들로 부터 바로 그려낼 수 있는 '식'을 어떻게 draw할 수 있고, 그 식이 무엇인지 배운다.

 식을 증명하는 것은 여기저기 많으니, 생략하고 식만 보면 아래와 같이 된다.

 ![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201112-1.png)

 A와 B는 각각 y=Ax+B이다.

 그래서 그걸 어디다 써먹을 수 있는데요?

 라고 한다면, 아래와 같이 써먹을 수 있다.

[Regression analysis를 응용하여 만든 filter를 MPU6050에 적용한 사례(Roll각)](https://youtu.be/P6Vs73NxRJc)

 위 영상은 Rollivery에서 사용했던 데이터이다. 모두 똑같은 데이터로, regression analysis를 이용해 만든 filter(이하 regression filter)에서, 몇개의 데이터를 기준으로(이하 sample 개수) regression analysis를 적용하면 그 결과가 어떻게 되는지 보여준다.

 보면, No filter에선 뭔가 그래프에 꺼끌꺼끌한 느낌이 있다면, 5 data를 기준으로 regression을 정하는 경우 어느정도 깔끔해지고, sample 개수가 10개만 되어도 상당히 깔끔해진다.

 하지만 100개가 넘어가면 아두이노의 연산 속도상 다음 번 값을 예측하려면, 꽤나 시간이 걸리기 때문에 delay가 생긴다. 그래서 MCU의 사양에 따라 적절한 sample 개수를 설정해주어야 한다.



 이건 원래 옛날 배관로봇 관련 일을 할 때, C++에서 작업을 했던 것이었는데(encoder의 데이터 값에 노이즈가 너무 심해서 이걸 사용하니 정말 많이 괜찮아졌었다.), 당시에 요긴하게 써서 이후 데이터 fluctuation이 심한 미세먼지 센서 모듈, MPU6050 등등 다양한 곳에서 사용했었다. (발표할 때 교수님이 회귀분석을 쓴 학부생은 처음봤다고 했다. 덕분에 좋은 학점을 받은 것 같다.)



 rollivery에서는 아두이노뿐 아니라 라즈베리파이에서도 사용했고, 참고로 다음에 포스팅 될 USB powermeter and HUB에서도 사용할 계획이다.

 보통 Filter라 하면 kalman filter나 complementary filter가 주로 사용이 되는데, (사실 KF도 그냥 KF는 거의 못쓰고, EKF나 UKF가 사용된다) state equation이나 각종 error state를 기반으로 다음값을 예측하는 것이기 때문에, 시스템 해석이 우선이 되어야 사용할 수 있는 반면, regression filter는 그딴거 없이 continuous 한 system이면 모든 곳에서 다 사용이 가능하다.

 약간의 delay가 있긴 하지만, 이만하면 어떤가. 원래 훨씬 간단한 방법으로 노이즈를 업애는 방법인, 학부생 때 배우는 LPF나 HPF를 쓰긴 하는데, 써본 사람은 실제 상황에서 써먹기 힘든것을 알 것이다. 보통 센서가 data를 받아올 때 아두이노와 같은 경우라면 큰 문제가 없지만 상대적으로 큰 로봇을 할 때에는 I2C, SPI 통신, (거의 UART) 좀 케이스가 큰 경우라면 CAN을 쓰는데, 모든 경우에서 interrupt가 들어간다. 이 때 interrupt가 규칙적이면 상관없는데, 하드웨어를 조금 공부해봤으면 알겠지만 이 interrupt가 절대 규칙적이진 않다. 일정 bit를 받으면 interrupt가 일어나서 신호를 보내게 되는데, 이 때문에 불규칙적인 간격으로 데이터 수신을 하게 되다보니 LPF를 쓰면 데이터의 일부만 걸러지고, HPF만 쓰면 또 다른 데이터의 일부만 걸러지게 된다.

 뭐 어쩌다 운이 좋으면 다 잘 걸러지는건데, 정말 운이 좋은 케이스일 뿐이다.

 그거에 비하면 regression은 일단 이전의 data를 기반으로 하기 때문에 noise는 무조건 줄어들 수 밖에 없다. 여러번 사용해본 후기로는, 맥시멈은 30~40정도가 적당하다. 그 이후는 데이터 딜레이가 좀 많이 나기 때문. 개인적으로는 5~10 사이를 주로 쓰는 편이다.



```python
class RegressionAnalysis:
    def __init__(self):
        self.numData = 0
        self.dataArray = []
        self.avgX = 0
        self.avgXsquare = 0

    def GetNumberOfData(self, numberOfData):
        self.numData = numberOfData
        self.avgX = self.numData*(self.numData + 1)/(2*self.numData)
        self.avgXsquare = self.numData*(self.numData+1)*(2*self.numData+1)/(6*self.numData)

    def CalculateSlope(self, data1, data2):
        self.slope = (data1-data2) / 50
        if self.slope > 10:
            return 1
        else :
            return 0

    def SaveData(self, data):
        if len(self.dataArray) < self.numData:
            self.dataArray.append(data)
        else:
            self.dataArray.pop(0)
            self.dataArray.append(data)

    def CalculateFilteredData(self):
        if len(self.dataArray) < self.numData:
            return self.dataArray[len(self.dataArray)-1]
        else:
            avgY = 0
            avgXY = 0
            for cnt in range(0, self.numData):
                avgY += self.dataArray[cnt]/self.numData
                avgXY += (cnt+1)*self.dataArray[cnt]/self.numData

            mother = self.avgXsquare - self.avgX*self.avgX

            slope = (avgXY - self.avgX*avgY)/mother
            tip = (avgY*self.avgXsquare - self.avgX*avgXY)/mother
            #print('slope and tip for rf : ', slope, tip)
            return int(slope *(self.numData+1) + tip)
```

 그런데 이거 왜 예전에 올린 것 같은 기분이 나는지 모르겠다.

 아무튼 RegressionAnalysis()로 객체를 하나 부르고, GetNumberOfData(__*본인이 원하는 Sample data 개수*__ )로 sample data개수를 정한 후, SaveData(*__필터할 데이터 변수__*) 로 저장하고, filteredData = CalculateFilteredData()로 리턴을 받으면 된다.

 상황에 따라 본인의 입맛에 바꿔서 쓰면 될 것 같다.

 아두이노에 쓰는 C/C++은 추후 포스팅 예정. 라즈베리파이에서 MPU9250 데이터 노이즈를 제거한다고 썼던 건데, 그럭저럭 괜찮았다.





### Reference

Regression analysis 그래프 [https://en.wikipedia.org/wiki/Regression_analysis](https://en.wikipedia.org/wiki/Regression_analysis)

Regression analysis equation [https://owlcation.com/stem/How-to-Create-a-Simple-Linear-Regression-Equation](https://owlcation.com/stem/How-to-Create-a-Simple-Linear-Regression-Equation)

