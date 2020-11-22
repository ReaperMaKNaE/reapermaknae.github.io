---
layout : post
title : "Object Detection Using ImageAI"
date : 2020-11-22 +0900
description : ObjectDetection
img : 20201122-2.jpg
tag : [ComputerVision]
---

### Object Detection using ImageAI

 Opensource로 나와있는 object detection중에서 다루기가 쉬운 것으로 알려진 ImageAI를 사용해서 Object detection을 진행해보자.

 OlafenwaMoses의 GitHub와 제너럴공국님이 작성하신 [ImageAI 기본사용법 <1. Object Detection>] 내용을 바탕으로 진행했다.

 [OlafenwaMoses/ImageAI](https://github.com/OlafenwaMoses/ImageAI)

 [제너럴 공국님이 작성하신 포스팅](https://generalthird.tistory.com/22)



 다른 케이스를 찾아봤었는데, tensorflow 버전이나 numpy 버전등이 안맞아서 잘 되는게 없었다. 제너럴공국님의 포스팅대로 진행하면 대부분 문제없이 진행이 되리라 예상한다.

 나의 경우는 두가지 문제가 발목을 잡았다.

 첫번째는 import cv2를 하면 발생하는 numpy error.

 정확한 에러명은 다음과 같다.

__*ImportError: numpy.core.multiarray failed to import*__

 해당 내용은 numpy가 제대로 설치가 안 된 경우로,

__*conda install -c conda-forge numpy*__

 명령어를 이용하여 해결할 수 있었다.

 이 해결법은 아래 stackoverflow를 참조했다.

[https://stackoverflow.com/questions/20518632/importerror-numpy-core-multiarray-failed-to-import](https://stackoverflow.com/questions/20518632/importerror-numpy-core-multiarray-failed-to-import)



 두번째는 __*DLL load failed: 지정된 프로시저를 찾을 수 없습니다.*__ 였다.

 이는 아래와 같은 명령어로 해결했다.

__*pip install protobuf==3.6.0*__

 이 역시 stackoverflow에서 참고하여 해결할 수 있었다.

[https://stackoverflow.com/questions/52092810/tensorflow-error-dll-load-failed-the-specified-procedure-could-not-be-found](https://stackoverflow.com/questions/52092810/tensorflow-error-dll-load-failed-the-specified-procedure-could-not-be-found)



 해당 문제가 모두 해결이되었으면, 직접 적용해보자.

 일단 아래 이미지는 친구가 찍은 사진이다. ~~(어딘지모름, 친구가 찍은것이니 저작권 걱정도 없다!)~~

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201122-1.jpg)

 위 사진에 적용을 한다면, 

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201122-2.jpg)

 위와 같이 적용된다!

 마찬가지로 영상에도 적용이 가능하다.

 영상에 적용하려면, 일단 [OlafenwaMoses](https://github.com/OlafenwaMoses)/ImageAI 에서 다음 링크를 눌러 확인했을 때, 마크다운 문서로 작성된 내용 중에서 RetinaNet을 다운받는다.

[https://github.com/OlafenwaMoses/ImageAI/blob/master/imageai/Detection/VIDEO.md](https://github.com/OlafenwaMoses/ImageAI/blob/master/imageai/Detection/VIDEO.md)

 이후 받은 RetinaNet 파일(파일 이름은 resnet50_coco_best_v2.0.1.h5)과 같은 장소에 아래 코드를 이용한다.(모두 위 링크에 나와있다!)

```python
from imageai.Detection import VideoObjectDetection
import os

execution_path = os.getcwd()

detector = VideoObjectDetection()
detector.setModelTypeAsRetinaNet()
detector.setModelPath( os.path.join(execution_path , "resnet50_coco_best_v2.0.1.h5"))
detector.loadModel()

video_path = detector.detectObjectsFromVideo(input_file_path=os.path.join(execution_path, "video1.mp4"),
                                output_file_path=os.path.join(execution_path, "video1_detected")
                                , frames_per_second=24, log_progress=True)
print(video_path)
```

 나의 경우 넣는 비디오의 이름을 video1으로 하였고, 확장자가 mp4였다.

 출력물의 경우 video1_detected의 이름으로 설정하였고, 확장자는 avi로 된다.

 ![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201122-3.png)

 다 되면 위와 같이 링크가 뜬다!

 프레임의 경우 20프레임으로 설정했는데, 보통 대부분 파일의 우클릭 설정에서 원본의 프레임을 확인할 수 있다. 이 프레임을 맞춰주면 똑같은 길이의 영상을 선택할 수 있으니, 위 코드에서

__*frames_per_second=24*__ 

이 부분은 꼭 확인후 고치자.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201122-4.png)

 나의 경우 원본 동영상이 24.00프레임/초 였기에, 24로 했다.

 그리고 object detection의 결과의 링크.

 [영상을 보려면 이 글을 누르면 된다.](https://youtu.be/vuaI_eMFSuM)

 나름 잘 잡아오는 걸 확인할 수 있다.



### 추가적인 Object Detection 내용 정리(개인적인 기록)

IoU : Intersection over Union object detection

교집합 영역 넓이 / 합집합 영역 넓이

(object detection 에서 주로 사용됨)

object detection의 경우 feature extractor, classifier, regressor, IoU, NMS 등등이 복합적으로 얽혀져 있는 형태.

 해당 내용은 아래를 참고했다.

[https://tutorials.pytorch.kr/beginner/deep_learning_60min_blitz.html](https://tutorials.pytorch.kr/beginner/deep_learning_60min_blitz.html)



 차후로는 custom dataset을 이용해서 object detection을 해 볼 예정.