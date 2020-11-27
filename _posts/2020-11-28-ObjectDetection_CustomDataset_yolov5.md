---
layout : post
title : "YoloV5와 Colab을 이용해서 Custom Dataset Object Detection 하기"
date : 2020-11-28 +0900
description : Object Detection
tag : [ComputerVision]
---

### 들어가기 전!

mAP란? mean Average Precision의 약자.

AP는 Average Precision.

Recall value값들에 대응하는 Precision 값들의 Average를 말함.

Precision은 정확도. (맞춘거)/(맞춘거+틀린거)

Recall은 또다른 의미의 정확도. (맞춘거)/(진짜 맞는거)



 즉 100개의 데이터 중에서 70개를 맞추고 30개를 틀렸으면, precision은 0.7임

 근데 알고보니 쓸데없는 것도 맞다고 해서 틀린게 30개였던거임

 그래서 제대로 된 70개는 잘 구분했음. 이러면 recall은 1임.

 

똑같은 상황에서, 100개 중에 70개 맞추고 30개 틀렸는데, 이번엔 맞는걸 틀렸다고 해서 precision이 0.7임.

판별기한테 100개가 전부 true인 데이터를 줬는데, 임마가 30개가 틀렸다고 잘못 알고있는거임

그럼뭐다? recall이 0.7이다.



 AP는 각 recall 값들에 대한 precision값의 평균임. 

 이게 뭔소리냐하면, 데이터가 N개가 있다고 치자.

판별기한테 전부 틀린걸 줬다고 했을 때의 precision[0], 한개만 맞는걸 줬다고 했을 때의 precision[1]..... 쭉쭉쭉.... 전부다 맞는걸 줬다고 했을때(precision[N]) 의 precision 값 평균임. 즉 avg(precision)



 그럼 mAP는 뭔데요?

 object가 한개면 AP=mAP임. 근데 2개면? 3개면? 각 물체마다 AP가 다를것 아니오?

 그래서 모든 물체들의 평균까지 싸악 다 구한 것이 mAP.



Ground Truth는 학습하고자 하는 데이터



### Custom Dataset Object Detection with YoloV5 and Colab



자, Custom Dataset을 한번 만들어보자. 난 이번에 Arduino Nano, Uno, RaspberryPI Zero를 구분해보려고 한다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-5.png)

 사진은 대충 요정도. 샘플 개수가 많진않다. 뭐 얘내들이 옷 갈아입고 다니는 것도 아니고 괜찮지 않을까?



 일단 요놈들을 라벨링해주기 위해, 나는 Labelimg를 썼다. 굉장히 편하고 좋은 것 같다. 아래는 링크.

[https://github.com/tzutalin/labelImg#labelimg](https://github.com/tzutalin/labelImg#labelimg)

 설치는 쉽다. 

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-11.png)

 README.md에 위와 같이 필요한 걸 다 적어주셨다. 난 Windows에 Anaconda를 쓰니까 위 명령어를 그대로 입력. 그러면 켜진다! (맨 마지막 줄은 따로 할 필요가없다. python labelimg.py를 치는 순간 프로그램이 실행된다.)

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-1.png)

 실행된 프로그램에서 먼저 좌측 메뉴에서 Open Dir를 눌러 사진 폴더를 불러오고, Change Save Dir를 눌러서 데이터를 저장할 곳을 변경한다. 우린 어짜피 코랩에다가 다 때려박을 거기 때문에, 대충 아무폴더 하나씩 만들어서 쑤셔넣어주자.

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-12.png)

 좀 주의할 점은, 좌측에 보면 PascalVOC 라고 되어있다. 저걸 눌러서 Yolo로 바꿔줘야한다.  사실 이거 모르고 난 PascalVOC로 먼저 다 만들어버리는 바람에, 나중에 torchvision으로 강제로 한번 실습을 진행하게 되어버렸다. 아 뭐 해보면 되지.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-2.png)

 단축키인 w를 누르면 사각형을 선택할 수 있는 영역이 나온다. 드래그를 해주면 기록이 된다. 그리고 d를 눌러서 다음으로 이동.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-3.png)

 __*STAY*__

 저따구로 라벨링하는데 뭔가 쓰잘데기없는것들이 많다면 일단 끄고 다시 켜서 default어쩌고를 지워야한다. 난 이걸 모르고 했다가 추후에 코드를 짜서 조금 고치게 되었다.

__*?? 님 저 이미 다 했는데요?*__

 라고 하시면, 조금 아래에서 다시 뵙겠다. 어쨌든 쉽게 해결이 된다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-4.png)

 하다보니 재밌다. 처음에 pascalVOC로 저장을 하고, yolo로 바꾼 다음 다시 save, a누르고 save 꾹꾹 눌러서 하나만 하면 나머지 데이터셋도 금방 끝내는 것 같다. 이왕 할꺼면 json도 구하게 createML도 할걸 그랬는데, 새로운 저장소 선택이었나? 누르니까 다 날라가버렸다. ~~*젠장*~~

 ![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-6.png)

 여튼 데이터 형식은 대충 이따구로 저장이 된다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-7.png)

 라벨링 된 데이터는 이렇게 terminal에도 다 남게 된다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-8.png)

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-9.png)

 절망적인 두 짤이다. 이거 때문에 코드를 추가했다. 이전에도 할 때 말했지만, 결국 코드 어딘가에서 label을 불러오는 값에 우리가 원하는 값을 빼는 방법은 결코 좋지 않다. 모든 label을 불러오는 값을 우리가 알 수 없기 때문인데, 그래서 txt파일 하나하나를 다 바꿔주는 코드를 짰다.

 일단 그 코드를 보기 전에, 다음과 같이 세팅하자.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-10.png)

 예전처럼 다 해놓고, 저장된 labels를 dataset 밖으로 뺀다. 그리고 export 내부에 새로운 빈 폴더인 labels를 만든다.

 그리고 아래 코드를 시작.



```python
wrongClassLabelList = glob('/content/labels/*.txt')
print(wrongClassLabelList)
for k in range(len(wrongClassLabelList)):
  print('i : ', i)
  with open('/content/'+wrongClassLabelList[k][9:], 'r+') as f:
    data = f.readlines()
    new_data = []
    for i in data:
      datum = i.split()
      for j in datum:
        if float(j) >= 15: #여기는 자기가 실수한 데이터 개수만큼 넣기
          idx = datum.index(j)
          datum[idx] = int(datum[idx]) - 15 #여기는 자기가 실수한 데이터 개수만큼 넣기
          for i in range(4):
            datum[i+1] = float(datum[i+1])
          new_data.append(list(datum))
  print(wrongClassLabelList[k][9:], 'is completed')
  with open('/content/dataset/export/'+wrongClassLabelList[k][9:], 'w') as f:
    for i in range(len(new_data)):
      f.write(' '.join(map(str, new_data[i]))+'\n')
```

 나의 경우 기존 데이터에 클래스가 15개 있는 상태에서 시작해서, 나노가 15, 우노가 16, 라즈베리파이가 17로 저장이 되었다. 그래서 이만큼 다 빼주고 새로 저장을 하는 코드다.



 그리고 돌리면 학습이 잘 된다!



![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-13.png)

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-14.png)

 텐서보드에서 결과를 보는데, 아무래도 데이터 개수가 많질 않아서인지 로스도 잘 안떨어진다.

 img사이즈는 크게 키운다고 무조건 좋아지는 것 같지는 않다.

 파란색은 저번과 똑같은 img 416에 batch size만 2인 경우

 자주색(핑크색?)은 img 600에 batch size가 1인 경우

 제일 성능이 좋은 하늘색은 img 600에 batch size 1인 경우

 제일 성능이 구린 붉은색은 img 416에 batch size 23인 경우이다.

11번째가 제일 잘되었으니 한번 확인해보자.



![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-15.png)

??

 이상하네. 한 개 더 해보자.

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-16.png)

??????

 데이터셋이 적긴 적었나보다. 아무래도 conf(정확도)를 0.5가 아니라 0.3으로 한번 낮춰보고 다시 해보자.

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-17.png)

 아 이제서야 좀 잡히네. 나노는 잡히지도 않았고, 멀쩡한 우노를 라즈베리파이로 찍어버렸다.

![img18](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-18.png)

 흠... loss가 제일 적은게 너였는데...

 여튼 며번 더 굴려봤는데, 결과가 참담한건 매나 마찬가지였다.

 영상으로 딱 찍어서 내 toolbox안의 나노/우노/파이제로를 잡아주는 걸 기대했는데, 아무래도 좀 공부를 더 해야겠다. yolo에 대한 근본적인 공부부터 한번 해보자.

![img19](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-19.png)

 마지막 학습. 에라몰라 100번학습해 하고 시켰는데, recall이 은근 높게나왔다. precision이 저따구로 올라가는걸 보면 한 300번하면 좀 나아질 수도 있을 것 같다. 아닌가? 그냥 한번 천번은 해봐야겠다.

여하튼 추후 다시 해보자.

 일단 샘플 돌린 결과.

![img20](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201128-20.png)



### Reference

mAP, Precision/Recall, [https://better-today.tistory.com/1](https://better-today.tistory.com/1)

Ground Truth, [https://eair.tistory.com/16](https://eair.tistory.com/16)
