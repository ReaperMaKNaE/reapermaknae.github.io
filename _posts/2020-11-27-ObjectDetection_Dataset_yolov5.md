---
layout : post
title : "YoloV5와 Colab을 이용해서 Object Detection 하기"
date : 2020-11-27 +0900
description : Object Detection
tag : [ComputerVision]
---

### Object Detection with YoloV5 and Colab



 이 게시글은 유튜버 '빵형의 개발도상국' 님의 [https://www.youtube.com/watch?v=T0DO1C8uYP8](https://www.youtube.com/watch?v=T0DO1C8uYP8) 영상을 참고하여 작성되었습니다.



 위 유튜버님의 영상을 따라하면 큰 어려움 없이 원하는 dataset을 받을 수 있다. 그런데 저런 케이스 말고 다른 것들은 안될까? 예를 들자면, kaggle과 같은 대형 dataset을 취급하는 곳에서 데이터를 받아서 써도 좋을 것 같다는 생각이 들었다.

 그래서 한번 응용을 조금 해봤다.



 일단 나는 kaggle에서 Gun Detection Dataset with yolo v2 and v3 labels 데이터셋을 받았다.

[https://www.kaggle.com/atulyakumar98/gundetection](https://www.kaggle.com/atulyakumar98/gundetection)

 __*아니 YoloV5라면서, v2랑 v3를 왜 받아요?*__

 왜냐면 __*조금만 코드를 바꾸면 되기 때문*__이다. 일단 yolo의 data 형태는 다음과 같다.

__*class x y width height*__

 각 요소는 띄워쓰기만으로 구분되며, x, y, width, height는 사진의 전체 크기를 1x1이라 가정했을 때 object가 해당하는 box를 0~1 사이의 숫자를 이용해 좌표형식으로 나타내게 된다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201127-1.png)

 대충 이런 식으로.

 여기서 yolo v2, v3와 v5의 차이는, class가 1부터 시작하냐 0부터 시작하냐의 차이다. V5는 0부터 시작하게 된다. 난 v2, v3 형태를 받았기 때문에 1부터 시작하고 있다.



 일단 kaggle dataset을 먼저 받자.

 dataset을 받는 방법은 아래 링크를 참조했다.

[https://www.kaggle.com/general/74235](https://www.kaggle.com/general/74235)

 해당 내용을 따라가서 json 파일을 넣는다.

 kaggle에서 데이터를 받으면 아마도 zip 파일로 되어있을 텐데, 

__*!unzip datasets.zip -d export*__

같은 방식으로 풀어주자.



 그리고 yoloV5가 어떻게 train을 시키는 지 좀 봐야하는데, images 폴더 안에 있는 .jpg들과 labels 폴더 안에 들어있는 .txt파일을 이용해서 알아서 학습하도록 만들어져있다. 그런데 보통 kaggle은 이런식으로 데이터를 분류하지 않은 케이스가 많다.

 예로들어 내가 받은 dataset의 경우 그냥 zip을 풀기만 하면 .jpg와 .txt가 같이 묶여져있다.

 따라서 이를 새로 만들어주기 위해, images라는 폴더와 labels라는 폴더를 만들고 파일들을 모두 옮겨주어야 한다.

 ![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201127-2.png)

 이렇게 export 폴더에 우클릭을 하게 되면 새폴더 만들기가 있다. 나는 미리 만들어서 있지만, 아마 export 폴더 내에 1.jpg 1.txt 10.jpg 10.txt 이렇게 쭈루루룩 있을 것이다.



 그 다음, jpg파일을 몽땅 긁어서 옮겨야하는데, 나의 경우만 해도 3천개나 되기에 절대 쉽지 않다. 아래 명령어 두줄을 써서 .jpg 파일들은 images로, .txt파일 들은 labels로 옮겨준다.

__*mv /content/dataset/export/***.jpg /content/dataset/export/images/*__

__*mv /content/dataset/export/***.txt /content/dataset/export/labels/*__

 몽땅 정리가 되었다!



 그럼 다음으로 넘어가서, 나머지는 다 따라하겠다고 쳐도 우리에겐 .yaml파일이 없다.(json파일과 거의 똑같음)

 그래서 이것도 직접 만들어줘야한다.

 하지만 어렵지 않으니 그냥 만들면 된다.

dataset 폴더 내에 파일을 하나 만들고, 이름을 나는 datasets.yaml이라고 저장했다. 그리고 더블클릭을해서 다음과 같이 수정 후, Ctrl+S를 눌러 저장.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201127-3.png)

```yaml
'train': '/content/dataset/train.txt'
'val': '/content/dataset/val.txt'

'nc': 1
'names': ['GUN']
```



 그다음 애초에 Yolov5 dataset을 받았으면 상관없겠지만, 나의 경우 yolo v2 v3 dataset이다. 제대로 되길 빌었지만, 아쉽게도 이상태로 돌리면 아래와 같은 에러가 뜨면서 안된다.

__*Label class 1 exceeds nc=1 in datasets.yaml. Possible class labels are 0-0*__

따라서 코드를 조금 수정해주어야 한다.

 위 에러는 무슨 말이냐면, __*난 dataset이 0부터 시작해서 labels 폴더 안의 txt 파일들에 class가 0이어야하는데 넌 1이 있음. 고로 안돌아감 ㅅㄱ*__ 

란 뜻이다. 아 그럼 1을 그냥 0으로 바꿔주면 되잖아.

 근데 파일이 3천개다. 어느세월에 그걸 다 바꿔줄까... 싶다가 그냥 train.py를 직접 열어서, 1이라고 들고오는 label을 0으로 바꿔주면 되겠다 싶어서 찾아봤다. 

일단 __*/content/yolov5/train.py*__ 를 더블클릭해서 열어준다.

 그리고 186번째와 198번째 줄로 가게 되면, 아래 사진과 같은 코드가 있는데, 여기에 사진처럼 뒤쪽에 -1을 붙여준다. 나는 class가 전부 1로 되어있으니, 이를 0으로 바꿔주기만 하면 된다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201127-4.png)

 그리고 영상을 따라한다면...!

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201127-5.png)

 학습이 된다!

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201127-6.png)

 영상에서 가르쳐주신대로 따라하면, 확인도 가능하다.



 이후 __*/content/yolov5/runs/train/본인이 설정한 폴더이름/weights/best.pt*__를 받는다. (가장 score가 높은 weight)

 그리고 yolov5를 받은 다음, 위 weight 파일과 본인이 detect할 파일을 넣은 후,

__*python detect.py --source FILENAME --weights ./best.pt*__

를 적어주면 된다.(window vscode 유저의 경우)

 아마 패키지를 다 깔지 않은 곳이라면, __*pip install tqdm*__으로 tqdm을 설치해줘야할 수도 있다.



tensorboard는 다음과 같은 명령어로 확인 가능.

```python
%load_ext tensorboard
%tensorboard --logdir /content/yolov5/runs/
```



그리고 좀 중요한걸론 이게 yolov5가 업데이트되면서 경로가 조금 바뀐 것 같다. 영상에서는 inference에 저장이 된다는데, 난 __*/content/yolov5/runs/detect/exp*__ 폴더에 저장이 됬다.

 아래는 val에 있는 데이터로 테스트한  케이스.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201127-7.png)



아래는 샘플 테스트.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201127-9.png)

이미지 출처: CQB, [https://worldstory12.tistory.com/170](https://worldstory12.tistory.com/170)

K5 사격, KBS뉴스 [https://news.kbs.co.kr/news/view.do?ncd=2799832](https://news.kbs.co.kr/news/view.do?ncd=2799832)

 보다시피 권총은 좀 잘 잡는데, 소총은 잘 못잡는 듯 하다. 아무래도 학습 데이터에 소총이 별로 없었나보다.



 아무튼 이런 방식으로 쉽게 할 수 있다.

 참고로 몇몇 기능이 안되는데, 이건 야매를 썼기 때문. class가 0부터 시작해야하는데, 1부터 시작하는 바람에 이걸 고치지 않는 다른 코드들에선 문제가 발생한다.(고쳤던 train.py를 제외한 다른 모든 .py 파일들에게서) plot이라던지 등의 케이스에선 잘 안되므로, 뭐 일단 학습이 됬고, 이걸 쓸 순 있다 정도로만 보면 될 것 같다.

 ~~*사실 코드 짜서 class를 1에서 0으로 바꾸는게 제일 마음이 편할 것 같은데, 그 코드짜는 것 보다 -1 두 개 붙이는게 더 쉬우니...*~~



 다음 포스팅은 내가 직접 데이터셋을 다 만들어서 직접 넣어보고 굴려보는 걸 해보자. 요새 통 이해가 잘 안되어서 좀 힘들었는데, 훌륭한 분의 유튜브를 보고 뭔가 탁탁 트이는 것 같다.



### Reference

YoloV5 Train Custom Data, [https://github.com/ultralytics/yolov5/wiki/Train-Custom-Data](https://github.com/ultralytics/yolov5/wiki/Train-Custom-Data)