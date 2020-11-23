---
layout : post
title : "Object Detection Dataset 정리"
date : 2020-11-24 +0900
description : Object Detection
tag : [ComputerVision]
---

### Object Detection Dataset

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

 아래는 위 과정을 마친 후 conda list를 했을 때 나오는 패키지들.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201124-3.jpg)

(vscode의 경우 처음에는 ctrl+shift+p를 누른 후 select interpreter로 경로를 찾아주고 pylint를 설치해야 아마도 제대로 될 가능성이 높다. interpreter를 제대로 설정했고, __*activate 가상환경 이름*__ 을 잘 했는지 확인)

(계속 작성중)



### Error which I have faced



지정된 프로시저를 찾을 수 없습니다.

__*Error loading OSError: [WinError 127] "C:\Users\82104\anaconda3\envs\torchvisionTutorial\lib\site-packages\torch\lib\caffe2.dll" or one of its dependencies.*__




 위와 같은 에러는 python 3.8로 업그레이드하면서 해결(기존 python 3.7)



### Reference

파이토치에서 지원하는 torchvision dataset

[https://pytorch.org/docs/stable/torchvision/datasets.html?highlight=dataset](https://pytorch.org/docs/stable/torchvision/datasets.html?highlight=dataset)

파이토치 dataset format [https://pytorch.org/docs/stable/_modules/torch/utils/data/dataset.html#Dataset](https://pytorch.org/docs/stable/_modules/torch/utils/data/dataset.html#Dataset)