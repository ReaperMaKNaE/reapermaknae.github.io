---
layout : post
title : "Server를 포맷하고 난 이후 machine learning을 위한 환경 구축 가이드"
date : 2021-05-09 +0900
description : 포맷하고 난 이후 다시 server로 돌릴 수 있는 과정을 기록한 내용입니다.
tag : [ComputerVision]
---

### Server 포맷 이후



 연구실에서 새로운 PC를 받거나, 문제가 생기게 되면 포맷을 하게 되는 경우가 많은데, 그 이후 다시 서버로 돌려놓는데에 걸리는 시간이 꽤나 걸려서 이를 해결하고자 기록으로 남겨둔다.



 참고로, 서버는 Linux Ubuntu 1804, 클라이언트는 windows 10 기준이다.



일단 가장 먼저 할 것은, 포맷하고 난 이후 인터넷을 접속하고 update 및 openssh-server를 깔아야한다. 그 외 몇가지 필요한 툴들을 설치.

```bash
sudo apt-get update
sudo apt-get install openssh-server gcc build-essential vim
```

 나는 추가적으로 vim을 깔긴한다. 맨 마지막에 vim은 생략가능하니 nano나 vi등 다른 툴을 쓰는 사람은 다른 것으로 대체하거나 생략 가능.



 설치를 하고나면 자동적으로 시작이 되나, 혹시 모르니,

```bash
service ssh start
service ssh status
```

 위 두가지로 제대로 돌아가고있는지 확인하자. 제대로 돌아간다면 active(running) 비슷하게 녹색 글씨로 뜨게 된다.



### ssh 연결용 key 등록



 서버를 바꾸면 문제가 되는게 바로 key다. 포맷하고 다시 연결하면 기존 클라이언트에 저장된 key가 서버의 key와 다르므로 서로 연동이 되질 않는다. 따라서, 다시 key를 만들어줘야한다.



 나는 vscode와 putty를 사용하는데, putty는 따로 뭐 할게 없어서 괜찮지만, vscode의 경우에는 키를 따로 등록해줘야한다. 그 과정은 다음과 같다.



 일단 key를 만들어야한다. window의 실행에서 powershell을 키고, 

```bash
ssh-keygen -t rsa -b 4096
```

 을 쳐서 key를 만든 다음,

```bash
scp C:\Users\(클라이언트계정이름)\.ssh\id_rsa.pub (서버계정이름)\(서버ip):id_rsa.pub
```

 를 이용해 서버로 보낸다.

근데 키를 만들면서 가끔

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
~~~
Host key verification failed.
lost connection
```

 이따구로 뜰 때가 있다.

이럴 때엔,

```bash
ssh-keygen -R (서버IP)
```

 로 키를 지우고 다시 만들어주면 된다.



 이후 putty를 이용해 서버에 접속하건, 서버에 직접 들어가서,

```bash
cd ~
mkdir .ssh
chmod 700 .ssh
cat id_rsa.pub >> .ssh/authorized_keys
```

 를 등록해주면 된다.



 그리고 vscode의 ssh설정에서 config file을 열고,

```bash
Host (원하는 이름 아무거나)
  HostName (서버ip)
  User (서버계정)
  IdentityFile ~/.ssh/id_rsa
```

을 입력한 뒤, ssh로 접속을 하면 된다.



 ### CUDA



 다음은 필수인 CUDA 설치이다.

 Nvidia CUDA install guide를 직접 참고하는 것이 좋긴한데, 최근 11.3을 깔았다가 그 이전의 모든 버전이 제대로 동작하지 않아서 결국 docker로 해결했던 기억이 있다.

 그래서 과거의 명작인, CUDA 10.2 + cudnn 7.6.5를 기준으로 설명한다.(글 작성일 당시에는 CUDA 10.2 + cudnn 8 도 지원하는 것 같다.) - 물론 10.2 가이드랑 똑같다.

 __참고로 docker를 쓸 사람은, cudnn은 설치 안해도된다. 하지만 CUDA는 필수다.__

 아나콘다를 쓰는 경우 cudatoolkit만으로도 해결이 되긴하나, 일단 나는 기본 베이스에 CUDA를 깔긴 한다.

 그래서 10.2 + cudnn 7을 어떻게 설치하는데요?

```bash
lspci | grep -i nvidia
```

 위 명령어로 nvidia GPU가 있는지 확인한다.

 가이드에는 gcc --version으로 gcc를 확인하라는데, 뭐 다 깔려있을테니 생략.

 이후 다음 명령어로 kernel 업데이트.

```bash
sudo apt-get install linux-headers-$(uname -r)
```

 이후 CUDA 10.2 설치.

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-10-2-local-10.2.89-440.33.01/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
```

이제 끝으로, 경로만 추가해주면 된다.

vim으로 bashrc를 열고, 

```bash
vim ~/.bashrc
```

아래를 아무데나 추가해준다. 맨 위에넣든 맨 아래에 넣든 상관 없음.

```bash
export PATH=/usr/local/cuda-11.3/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-11.3/lib64\
                         ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-11.3/lib\
                         ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

 그리고 업데이트를 해준다.

```bash
source ~/.bashrc
```

 그리고 설치가 된 것을 확인해보자.

```bash
nvidia-smi
nvcc -V
```

 만약 무슨 에러가 뜬다면, 재부팅 한 이후 다시 시작해보자.

ssh 연결의 경우 재부팅을 하더라도 연결은 살아있기 때문에 다시 연결이된다.

```bash
sudo reboot now
```

 이후 다시 설치 확인을 해보면 잘 동작하는 것을 확인할 수 있다.



이제 여기서 두갈래로 나뉜다. docker를 쓰느냐, cudnn을 쓰느냐.

매나 같은 맥락이긴한데, 난 cudnn을 직접 설치하는 편을 선호하기에 cudnn 설치를 먼저 가이드 한다.

 참고로 cudnn은 왜쓰나요? 없어도 되는데요 할 수 있는데, 실제로도 맞다. 그런데 최근 나오는 code들이 대부분 안쓰면 안되더라. 이건 뭐 나도 nvidia에서 developer가 아니기때문에 알 방법이 없긴 하지만, 아무튼 그냥 깔면 된다.



### cuDNN 설치



 cuDNN은 network 설치를 지원하지 않는다. 그래서 클라이언트(윈도우)에서 받은 다음 scp로 보내주는게 낫다.

[https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn)

 위 홈페이지로 들어가서, nvidia 로그인을 해야한다. 구글계정으로 로그인이 가능하다. 나는 구글계정으로 로그인을 했다. 이후 동의하고 보면 cudNN의 최신 버전 밖에 안보이는데, 그 아래에

__Archived cuDNN Releases__ 를 누르면 cuDNN의 과거 버전들도 볼 수 있다.

난 여기서 7.6.5 for CUDA 10.2를 골랐다.

근데 누르면 뭔가 많이 뜬다.

linux 기준으로,

__cuDNN Library for Linux__  - 필수

__cuDNN Runtime Library for Ubuntu18.04 (Deb)__  - 조건에 따라서.

__cuDNN Developer Library for Ubuntu18.04 (Deb)__  - 조건에 따라서.

__cuDNN Code Samples and User Guide for Ubuntu18.04 (Deb)__  - sample 돌려보고 싶은 사람만.

그리고 이걸 scp로 일일이 다 보내준다. 따로 윈도우에서 다운로드 위치를 변경하지 않았다면,

```bash
scp C:\Users\(클라이언트계정이름)\Downloads\cudnn어쩌구저쩌구 (서버계정)@(서버ip):/home/(서버계정)
```

대충 이렇게 보내주면 된다.

이후엔 cuDNN install 가이드를 따라가면 되는데,

밑에 두개는 왜 받아요? 할 수 있다.

그 이유는 과정에서 설명.

일단, 받았으면 풀어야한다.

```bash
tar -xzvf cudnn-10.2-linux-x64-v7.6.5.32.tgz
sudo cp cuda/include/cudnn*.h /usr/local/cuda/include 
sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda/lib64 
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

이후,

```bash
sudo dpkg -i libcudnn7_6.5.32-1+cuda10.2_amd64.deb
sudo dpkg -i libcudnn7-dev_7.6.5.32-1+cuda10.2_amd64.deb
```

 추가로 sample도 있는데, 이건 단순 check용이라 제꼈다. guide의 cuDNN check를 해보고 싶다면 받아도 무관.

그리고 차례대로,

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin 
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/ /"
sudo apt-get update
sudo apt-get install libcudnn7=7.6.5.32-1+cuda10.2
sudo apt-ge tinstall libcudnn7-dev=7.6.5.32-1+cuda10.2
```

 이러면 끝난다.

 밑에 간단하게 sample로 test를 하는게 있는데, 해보고 싶은 사람은 하면 된다.

이후 scp로 옮겼던 cuDNN 설치 파일은 다 쓸데가없으니 삭제해도 무방.



### Docker 시작



먼저 도커를 깐다.

```shell
curl -s https://get.docker.com/ | sudo sh
sudo systemctl --now enable docker
docker -v
```



__이후 도커 관리자 권한__ (해당 bash에서만 유효함. 필요없으면 그냥 3번 이후의 과정은 다 sudo 붙이면 됨)

```shell
sudo usermod -aG docker $USER
sudo su - $USER
```

 

__도커 이미지 확인 및 컨테이너 확인__

```shell
docker images
docker ps
docker ps -a
```

docker images는 docker images 확인

docker ps는 현재 실행중인 컨테이너 확인

docker ps -a는 모든 컨테이너 확인.

물론 지금은 깐게없으니 아무것도 안뜸.



__nvidia 버전을 설치__

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



__쿠다 image 다운로드__

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



__도커 컨테이너 생성 및 접속__

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



__컨테이너 내 환경 구축__

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



__아니 그래서 이미지랑 컨테이너가 뭔데요?__

git clone ~~해서 어떤거 불러오면, git에 올라가있는 애는 우리가 커밋 푸쉬 하지 않는이상 절대 안바뀐다.

이렇게 원본이 되는 것들이 있는데, 얘내들이 이미지임.

그리고 우리가 이걸 마음대로 바꿀수 있는 '로컬'을 컨테이너라고 보면 된다.

그래서 image는 대충 환경구성(우분투, 센토스, 윈도우 뭐 아무거나. 추가적으로 쿠다를 지원하기도 한다.)의 베이스를 가져오는 거고, 우리가 그 이미지를 베이스로 컨테이너를 만든다.(베이스 환경에서 우리만의 환경을 추가로 구축하는 것이 컨테이너)

그리고 그 컨테이너는 아무리 지지고 볶아도, 원래의 이미지는 안변한다. 우리가 뭔짓을 하던. 

근데 우리가 만든 환경을 저장하고싶다? 그럼 컨테이너를 이미지로 다시 만드는 과정이 있다. 이걸 일종의 깃 푸쉬 과정과 비슷하다고 보면 된다.

 자세한 내용은 검색해보면 나옴.



__도커 컨테이너 잘못만듬. 어떡함..__

컨테이너 그냥 삭제하고 다시만드셈. 물론 환경구축 다시해야하는 문제가 있는데, 이게 마음편하다. 일단은 나도 공부중이라, 해당 컨테이너를 만들었을 때 컨테이너의 환경만 바꾸는 법을 좀 알아봐야할듯

```shell
docker rm [container name]
```

 만약 뭐 실행중이다, 그러면 stop하고 rm해야한다.

```shell
docker stop [container name]
```



__추가 응용__

보니까 openssh-server를 깔아서 각 컨테이너마다 서버를 열 수 있는 것 같다. 하지만 이건 귀찮으니 생략. 뭐 어짜피 서버에서 잘 굴리고있는데 이걸 또 서버로 만들 필요까지야...



 여튼 CUDA11.3을 원래 깔아놨었는데, 너무 에러가 자주뜨는게 고통스러워서 그냥 컨테이너를 새로 만들었다. 덕분에 아주 깔끔하게 잘 되고 있다.(CUDA 10.2, CUDNN 7 사용)



 cuda 버전 바꾸려면 시간이 정말 많이 드는데, 이렇게 하면 그나마 좀 편한듯.



__컨테이너 나가려면?__

```shell
exit
```

 위 명령어를 쳐주자. 그럼 container에서 나갈 수 있다. 작업했던 환경(pytorch, conda 등 설치한 패키지들)은 저장이 되니 걱정할 필요가 없다.

 참고로 여러개의 bash에서 하나의 container에 접속중인 경우, 모든 bash에서 container를 나가야(혹은 bash를 끄거나) Exited 라고 뜬다. 



__그럼 어떻게 다시 시작해요?__

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



이상 끝. 고생하셨습니다.