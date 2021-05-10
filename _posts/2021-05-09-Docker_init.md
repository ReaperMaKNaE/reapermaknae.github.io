---
layout : post
title : "Docker 빠르게 시작하자"
date : 2021-05-09 +0900
description : CUDA의 알 수 없는 오류로 받던 고통을 docker로 해결한 기록입니다.
tag : [ComputerVision]
---

### Docker 시작



1. 도커설치 및 버전 확인

```shell
curl -s https://get.docker.com/ | sudo sh
sudo systemctl --now enable docker
docker -v
```



2. 도커 관리자 권한 (해당 bash에서만 유효함. 필요없으면 그냥 3번 이후의 과정은 다 sudo 붙이면 됨)

```shell
sudo usermod -aG docker $USER
sudo su - $USER
```

 

3. 도커 이미지 확인 및 컨테이너 확인

```shell
docker images
docker ps
docker ps -a
```

docker images는 docker images 확인

docker ps는 현재 실행중인 컨테이너 확인

docker ps -a는 모든 컨테이너 확인.

물론 지금은 깐게없으니 아무것도 안뜸.



4. nvidia 버전을 설치

이걸 까는 이유는 이걸 설치해야 쿠다의 도커이미지를 받았을 때 nvidia-smi치면 나옴. 이거 안깔면 쿠다의 도커이미지 받아도 nvidia-smi, nvcc -V 다 먹통임.

[https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)

 위 홈페이지의 instruction을 따라가면 됨.

일단 그대로 옮겨오면,

```shell
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
   
sudo apt-get update
sudo apt-get install -y nvidia-docker2
sudo systemctl restart docker
```



5. 쿠다 image 다운로드

[https://gitlab.com/nvidia/container-images/cuda/blob/master/doc/supported-tags.md](https://gitlab.com/nvidia/container-images/cuda/blob/master/doc/supported-tags.md)

위 링크를 들어가면 nvidia에서 제공하는 이미지를 받을 수 있다.

예로 들어보자. ubuntu 18.04 , cuda 10.2, cudnn7 을 받고 싶다. 하면,

```shell
docker pull nvidia/cuda:10.2-cudnn7-devel-ubuntu18.04
```

 보면 devel, base, runtime 등등 여러개가있는데, 그냥 devel받자. devel이 full-package 버전이라고 보면된다.

 위에껄 하고 다시 이미지를 확인하면, image가 받아진 걸 확인할 수 있다.

```shell
docker images
```

 이미지를 기반으로 우리는 컨테이너를 만들것이다.



6. 도커 컨테이너 생성 및 접속

자 이제 컨테이너를 만든다. 이미지는 뭐고 컨테이너는 뭔데요? 는 나중에 답변예정. 일단 따라하자

```shell
nvidia-docker run --name [container] -it --shm-size=8G -v [host_root:container_root] [image name] /bin/bash
```

 위와 같이 입력하면 컨테이너가 만들어지고 바로 시작 가능하다. 지금 시작이 된건지 꺼진건지 확인하려면, shell을 한개 더 키고 아래 명령어 입력하면 STATUS 란에 Up~~ 어쩌고가 뜬다.

```shell
docker ps -a
```



-  [container]는 가상환경 이름 만드는 것 처럼 쓰는 거다. 좋아하는 이름 아무거나 쓰자.

- shm-size는 이거 안쓰면 dataloader가 용량이 딸려서 못불러온다. 무조건 써주자. 16G도 될듯.

- -v의 경우는 볼륨이라고 하는데, 그냥 우분투의 mount와 유사.

- host_root는 마운트할 폴더의 위치(나는 그냥 우분투에 마운트된 하드디스크를 통째로 마운트해서 썼다.)

- container_root는 아무렇게나 해도되는데, 나는 그냥 /dataset 이렇게 적어놓는다.

이러면 컨테이너 들어가자마자 ls칠 때 /dataset 폴더가 생기는데, 그 dataset 폴더 안에 마운트가 뙇! 하고 되어있다. 데이터셋이고 뭐고 그냥 싹 다 들어가니 걱정 ㄴㄴ(일단 1.3TB까진 잘 되는거 확인함)

- image name은 image name이다. 위에서 받은 image를 치게되면 image의 이름이 나온다.(무슨 7fa928dv9 이런식으로) 이걸 치면 됨. 이것도 좀 바꾸는 방법이 있다는데, 추후 추가 예정. 즉, 나로 예를 들면,



```shell
nvidia-docker run --name recon -it --shm-size=8G -v /home/byeonginjoung/recon:/dataset 798a1b666e92 /bin/bash
```

 를 치면 된다. 그럼 컨테이너 생성 및 접속 끝.



 바로 컨테이너 들어와진 것이다.



7. 컨테이너 내 환경 구축

일단 아나콘다 설치한다 생각하고,

```shell
apt-get update
apt-get install -y wget vim git gcc build-essential
apt-get install libglib2.0-0 libgl1-mesa-glx
wget https://repo.anaconda.com/archive/Anaconda3-2019.03-Linux-x86_64.sh
bash Anaconda3-2019.03-Linux-x86_64.sh
```

 하면 설치 끝. libgl ~~~는 opencv 관련된건데, 이거 없으면 그... 아무튼 안됨. 아무튼 하면 된다.(만약 이거 안깔리면, 콘다 다 깔고 환경 하나 만든 다음, pip install opencv-python 을 한 후에 libgl들을 설치하면 된다. )

근데 콘다 명령어가 안먹을테니,

```shell
vim ~/.bashrc
export PATH=~/anaconda3/bin:$PATH
source ~/.basrhc
```

 가운데 export 어쩌고를 bashrc 안 아무데나 적어두고 source로 한번 켜주자. 그럼 conda 환경 설정 완료.

 대충 recon이라는 환경을 만든다 치자.

```shell
conda create -n recon python=3.6
```

 만들었으면 이제 이것저것 깔아서 하면 된다.

파이토치 링크: [https://pytorch.org/](https://pytorch.org/)

detectron2 : [https://detectron2.readthedocs.io/en/latest/tutorials/install.html](https://detectron2.readthedocs.io/en/latest/tutorials/install.html)

 여기까지 대략 10~15분 정도 걸린다.



8. 아니 그래서 이미지랑 컨테이너가 뭔데요

git clone ~~해서 어떤거 불러오면, git에 올라가있는 애는 우리가 커밋 푸쉬 하지 않는이상 절대 안바뀐다.

이렇게 원본이 되는 것들이 있는데, 얘내들이 이미지임.

그리고 우리가 이걸 마음대로 바꿀수 있는 '로컬'을 컨테이너라고 보면 된다.

그래서 image는 대충 환경구성(우분투, 센토스, 윈도우 뭐 아무거나. 추가적으로 쿠다를 지원하기도 한다.)의 베이스를 가져오는 거고, 우리가 그 이미지를 베이스로 컨테이너를 만든다.(베이스 환경에서 우리만의 환경을 추가로 구축하는 것이 컨테이너)

그리고 그 컨테이너는 아무리 지지고 볶아도, 원래의 이미지는 안변한다. 우리가 뭔짓을 하던. 

근데 우리가 만든 환경을 저장하고싶다? 그럼 컨테이너를 이미지로 다시 만드는 과정이 있다. 이걸 일종의 깃 푸쉬 과정과 비슷하다고 보면 된다.

 자세한 내용은 검색해보면 나옴.



9. 도커 컨테이너 잘못만듬. 어떡함..

컨테이너 그냥 삭제하고 다시만드셈. 물론 환경구축 다시해야하는 문제가 있는데, 이게 마음편하다. 일단은 나도 공부중이라, 해당 컨테이너를 만들었을 때 컨테이너의 환경만 바꾸는 법을 좀 알아봐야할듯

```shell
docker rm [container name]
```

 만약 뭐 실행중이다, 그러면 stop하고 rm해야한다.

```shell
docker stop [container name]
```



10. 추가 응용

보니까 openssh-server를 깔아서 각 컨테이너마다 서버를 열 수 있는 것 같다. 하지만 이건 귀찮으니 생략. 뭐 어짜피 서버에서 잘 굴리고있는데 이걸 또 서버로 만들 필요까지야...



 여튼 CUDA11.3을 원래 깔아놨었는데, 너무 에러가 자주뜨는게 고통스러워서 그냥 컨테이너를 새로 만들었다. 덕분에 아주 깔끔하게 잘 되고 있다.(CUDA 10.2, CUDNN 7 사용)



 cuda 버전 바꾸려면 시간이 정말 많이 드는데, 이렇게 하면 그나마 좀 편한듯.



11. 컨테이너 나가려면?

```shell
exit
```

 위 명령어를 쳐주자. 그럼 container에서 나갈 수 있다. 작업했던 환경(pytorch, conda 등 설치한 패키지들)은 저장이 되니 걱정할 필요가 없다.

 참고로 여러개의 bash에서 하나의 container에 접속중인 경우, 모든 bash에서 container를 나가야(혹은 bash를 끄거나) Exited 라고 뜬다. 



12. 그럼 어떻게 다시 시작해요?

```shell
docker ps -a
```

 를 쳐보면, 나갔던 컨테이너는 STATUS에 Exited라고 뜬다. 해당 container에 다시 접속을 하고 싶다면,

```shell
nvidia-docker start [container name]
nvidia-docker exec -it [container name] /bin/bash
```

 로 들어가서 다시 작업을 해주면 된다.

 만약 다른 bash에서 컨테이너에 접속하고 있다면, STATUS에 Up이라고 뜨고 있다. 이럴 때엔 그냥,

```shell
nvidia-docker exec -it [container name] /bin/bash
```

를 쳐주면 접속된다.



