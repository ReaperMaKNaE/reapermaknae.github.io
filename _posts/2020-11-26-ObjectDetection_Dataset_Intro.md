---
layout : post
title : "Object Detection Dataset 정리"
date : 2020-11-26 +0900
description : Object Detection
tag : [ComputerVision]
---

### Object Detection Dataset with Pytorch Tutorial

 직접 코드를 좀 돌려보면서 해보고 하려고 했는데, 아무리 해도 좀 안됬다.

 결국 jpynb 파일을 받아서 쓰는것으로 결정. 원래 목적은 dataset이 어떤 식으로 쓰이는지 알고, 내가 원하는 dataset을 넣어서 학습시키고 원하는 것을 얻어내는 것이기에 일단 이걸로라도 대충 공부를 하는 것으로 했다.





### 이전에 작성하던 기록. 아래는 그냥 기록용으로만 남겨둡니다.

### 참조하지마세요!



 Object detection에서 주로 사용되는 dataset의 종류에는 대표적으로 Pascl VOC, COCO, ImageNet, OpenImages, Yolo, MNIST, FashionMNIST, LSUN Classification, CIFAR10, STL10, SVHN, PhotoTour 등의 형태가 있다.

 coco는 x,y,z,h의 value가 픽셀로 나타나와있는 반면, yolo의 경우는 이미지 크기에 대한 center x, center y, w, h의 비율 값으로 구성됨.



 먼저, pytorch에서 사용되는 dataset을 알아보자.

 Torchvision object detection finetuning tutorial에서 PennFudan 데이터셋을 받자.

[https://pytorch.org/tutorials/intermediate/torchvision_tutorial.html](https://pytorch.org/tutorials/intermediate/torchvision_tutorial.html)

 나의 경우, 아나콘다에서의 개발환경은 python3.8, numpy, pillow, 그리고 pytorch, torchvision, cpuonly -c pytorch 였다.

 모든 것을 순서대로 기록하면,

__*conda create -n torchvision python==3.8*__

__*conda install numpy pillow*__

__*conda install pytorch torchvision cpuonly -c pytorch*__

그리고 pycocotools가 필요하다. 이걸 윈도우에 깔려면 여러것들이 좀 필요하다.

__*conda install git cython*__



![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201124-3.png)

(vscode의 경우 처음에는 ctrl+shift+p를 누른 후 select interpreter로 경로를 찾아주고 pylint를 설치해야 아마도 제대로 될 가능성이 높다. interpreter를 제대로 설정했고, __*activate 가상환경 이름*__ 을 잘 했는지 확인)



 그리고 문제는 pycocotools를 설치해야하는데, window 버전이 없다고 뜬다. 물론 어떤 사람이 pycocotools를 window버전으로 만들어서 다시 쓴 경우가 있는데, 문제는 이렇게까지 할 바에야 그냥 파이토치에서 올려놓은 주피터노트북 파일로 공부를 하는게 낫다는 생각이 들어, 그쪽으로 방향을 틀었다. 후.... 아무튼 나중에 도움이 될 수도 있다고 생각해서 여기까지 겪으며 얻은 에러는 아래에 기록해둔다.



### Error which I have faced



지정된 프로시저를 찾을 수 없습니다.

__*Error loading OSError: [WinError 127] "C:\Users\82104\anaconda3\envs\torchvisionTutorial\lib\site-packages\torch\lib\caffe2.dll" or one of its dependencies.*__



 위와 같은 에러는 python 3.8로 업그레이드하면서 해결(기존 python 3.7)



 다른 것으로는 

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201126-1.png)

 references/detection/engine.py, utils.py, transforms.py를 받으라는데, 내가 받은 파일들 내에선 이러한 것들이 없었다. 그래서 어디서 받으란거지? 했는데, 다행히 아래 파이토치 discuss에서 찾을 수 있었다.

[https://discuss.pytorch.org/t/object-detection-finetuning-tutorial/52651/9](https://discuss.pytorch.org/t/object-detection-finetuning-tutorial/52651/9)

 위 내용에서 답변에 따르면, 그냥 github page에서 받아오면 되는 모양.

[https://github.com/pytorch/vision/tree/master/references/detection](https://github.com/pytorch/vision/tree/master/references/detection)

 위 페이지에서 engine.py, transforms.py, utils.py를 얻을 수 있다. 복사 붙여넣기로 같은 폴더에 집어넣자.





### Reference

파이토치에서 지원하는 torchvision dataset

[https://pytorch.org/docs/stable/torchvision/datasets.html?highlight=dataset](https://pytorch.org/docs/stable/torchvision/datasets.html?highlight=dataset)

파이토치 dataset format [https://pytorch.org/docs/stable/_modules/torch/utils/data/dataset.html#Dataset](https://pytorch.org/docs/stable/_modules/torch/utils/data/dataset.html#Dataset)