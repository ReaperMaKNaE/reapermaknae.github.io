---
layout : post
title : "Learning Event-Based Motion Deblurring"
date : 2021-05-21 +0900
description : Event Camera를 이용하여 image deblurring을 성공적으로 수행한 Learning Event-Based Motion Deblurring 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Learning Event-Based Motion Deblurring, CVPR 2020



### Abstract

 motion-blurred image에서 sharp video sequence를 복구하는 것은 blurring 때문에 뭉개진 motion information 때문에 매우 어려운 문제이다. 하지만, event-based camera의 fast motion은 높은 time rate로 event를 잡을 수 있어 효과적인 해결책으로 발견되고 있다. 이번 논문에서 우리는 event-based motion deblurring의 일련적인 formulation에서 시작하여 어떻게 end-to-end deep neural network architecture로 최적화 될 수 있는지에 대해 알려준다. 제안된 architecture는 principled manner로 global하고 local한 scale 모두를 포함하는 visual, temporal knowledge를 통합한 RNN이다. reconstruction을 향상시키고 효과적으로 stream of event에서 boundary prior를 추출하기 위해 우리는 differentiable directional event filtering module을 소개한다. 우리는 GoPro dataset과 DAVIS240C camera로 촬영된 새로운 dataset에서 실험했다. 제안한 방법은 reconstruction quality에서 SOTA를 찍었으며, real-world motion blur를 다루는 것을 더욱 잘 할 수 있게 되었다.



### Introduction

 camera의 exposure time동안 camera sensor들이 서로 다른 time stamp를 기록하고 이것이 축적되어 평균적인 signal을 내기에, exposure time이 필요한 최근 카메라들은 motion blur가 자주 일어난다. 이 반대 문제는 deblurring이라고 하는데, motion-blurred image를 바꾸고 새롭게 sharp하도록 recovery 시키는 과정이 computer vision에서 challenging한 task로 자리잡고 있다. 간단한 motion pattern이 잘 파악되고 있음에도 불구하고, 실제 세상에서 복잡한 motion pattern을 formulating하기란 훨씬 어려운 일이다.

 일반적인 motion blur를 다루기 위해, 최근 deep learning approach들은 blurred image를 다양한 sharp image나 blurred version을 이용하여 복원하는 것들을 제안했다. 몇몇 scenario에 대해서는 이러한 것들이 성공적이었으나, vehicle이나 drone-equipped, handheld 등의 camera에서 오는 심한 motion blur들에 대해서는 reconstruction을 지속적으로 실패하였다. 이러한 심한 손상을 받은 image의 경우 visual, temporal information의 loss로 인해 다루는 것이 거의 불가능하다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-1.PNG)

 computational architecture에 기대는 것에 반해, 이번 work은 data capture stage에서 이러한 문제를 완화시키기 위해 event-based camera를 적용했다. event camera들은 생물학에서 영감을 받은 센서로, pixel의 intensity변화 (event) 를 micro second 단위로 기록하면서 매우 적은 power consumption을 자랑하는 카메라이다. 이러한 sensor의 hybrid model은 image의 temporally calibrated된 event를 기록하게 된다. 결과적으로 이러한 data들은 자연적으로 motion deblurring을 할 수 있는 dense temporal information을 가지게 된다. Figure 1의 (a)와 (b)에서 image가 심각하게 blur가 된 사진과, temporally dense한 scene의 moving pattern인 event가 기록된 것을 확인할 수 있다.

 event-based motion deblurring의 높은 potential에도 불구하고, 중요한 문제는 pixel intensity가 어떤 threshold값을 기준으로 변화가 생기게 된다면 이러한 정보는 lossy하고 noisy한 신호들을 유발하게 된다. 이러한 discrete하고 inconsistent한sampling(threshold 값을 정해놓는 것을 말하는 듯 하다)은 texture와 contrast를 복원하기 힘들게 만든다. Deblurring의 최근 SOTA인 Figure 1의 (d)와 같이, 만족할만한 결과를 내기가 힘들게 된다. 우리의 solution은 event-based deblurring process에 깊이 들어가서 data의 불완전함을 억제하는 것이다.

 자세하게 말하자면, 이번 work은 event-based deblurring의 일련의 formulation에서 시작한다. deep network와의 optimization을 새로 융합함으로써, 우리는 새로운 trainable RNN end-to-end network를 제안한다 각각의 time step에서 coarse recontruction이 이전의 reconstruction뿐 아니라 local temporal event를 받게 된다. Fine detail이 이후에 network prediction을 받아서 global과 local scale에서 appearance와 temporal cue를 guide하게 된다. reconstruction의 질을 향상시키기 위해, 우리는 event에서 motion boundary를 잘 융합하고 sharp deblurring prior를 생산해내는 differentiable Directional Event Filtering(DEF) module을 소개한다. 이를 평가하기 위해 우리는 large outdoor dataset인 DAVIS240C camera를 이용한 dataset을 만들었다. 이러한 dataset과 synthetic GoPro dataset에서 우리의 방법이 여러 분야에서 SOTA를 달성한 것을 보여줄 것이다.

 우리의 contribution을 정리하면 아래와 같다.

- Event-based motion deblurring을 위한 새로운 RNN architecture 제안
- events for motion deblurring에서 sharp boundary prior를 뽑기 위한 directional event filtering을 제안
- future research를 위한, motion blur가 포함되어있는 새로운 event dataset



### Related Work

__Blind motion deblurring__

 blind motion deblurring은 blurring kernel에 대한 사전 지식 없이 blurry image를 복원하는 것을 목표로 한다. 초기의 work들은 color channel statistics, patch recurrence나 image singlal의 outlier등 최근의 image prior를 정의하기 위해 다양한 blurring-aware indicator들을 사용했다. 몇몇 work들은 motion kernel, restoration function, image prior들을 데이터에서 학습할 수 있도록 제안했다. 더욱 복잡한 motion pattern은 서로 다른 목표에서 해결이 되었다. Richer prior knowledge(음... 그 scene의 상황에 대해 더 많이 알고있는? 아무튼 이미지 뿐만이 아닌 것을 말하는 듯.)가 꽤나 유용한것으로 증명되었다.

 최근의 trend는 이러한 motion deblurring의 복잡함을 deep neural network로 해결하자는 움직임을 보여주고 있다. 다양한 방식의 효과적인 network들이 제안되었다. receptive field를 늘리거나, multi-scale fusion, feature distangling, recurrent refinement등이다. 여기엔 blurred image의 motion dynamics를 decoding하여 sharp video sequence로 변환하는 연구도 포함되어 있다. 이러한 advance에도 불구하고, real world lighting, texture와 motion등 blurred image에서 많이 잃게 되는 정보들의 조합은 여전히 plausible 복원이 힘들다.

__Event cameras__

 Event camera는 저전력으로 microsecond level에서 intensity의 변화를 감지하는 sensor로 만들어진 camera이다. 이는 다양한 vision task에서 적용이 되는데, visual tracking이나 stereo vision, optical flow estimation등이다. related branch는 오염된 event signal들 분석하여 고frame의 image sequence로 복원하는 연구가 있었다. 최근에는, double integral model을 이용하여 event-based motion deblurring을 공식화한 사레도 있다. 하지만 여전히 event camera의 noisy hard sampling mechanism이 noise를 계속 축적하고 scene의 detail등에 대한 loss가 일어나고 있다.

 이러한 work들은 event-to-video translation과 같은 data에서 만족할만한 결과를 학슴시킴으로써 불완전한 event sampling을 압도하는 work들과 직관을 공유한다.(뭐 대충 두 개가 비슷한 맥락의 연구다 라는 뜻인듯) future frame prediction을 해결한 사례에 반해, local motion cue에서 얻은 정보로 intensity image를 이용해 event로 변환하는 연구도 있었다. 대신에, 이번 work은 motion deblurring을 위해 local appearance/motion cue뿐 아니라 새로운 event boundary 모두 연구하게 된다.



### Learning Event-Based Motion Deblurring

 주어진 motion-blurred image에 대해서, 우리의 목표는 T frame에서 sharp한 video sequence를 뽑아내는 것이다. 우리는 events의 set이 $\mathbb{E}_{1\sim T}$ 라 할 때 exposure동안 image-event sensor가 1 ~ T초 동안 촬영한 hybrid 라고 하자. 각 event $\mathcal{E}_{x,y,t}$ 를 time point t에서 (x,y)에서의 event라고 하자. 여기서 t는 integer일 필요는 없지만, fractional 해야한다.(high temporal resolution을 위해서 - 분수꼴로는 나타낼 수 있어야한다라는 뜻) polarity $p_{x,y,t}$ 는 local intensity의 변화를 의미한다. 이를 정리하게 되면,

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-2.PNG)

  가 된다. 약간의 time period와 threshold를 두고 polarity를 위와 같이 구할 수 있다. 위 범위 내에 해당하지 않는다면 polarity는 모두 0이 된다.  최근 image 사이의 관계에서 intensity는 아래와 같은 관계를 가진다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-3.PNG)

 한가지 알아 두어야 할 것으로는, 식 (2)의 경우 시간 간격인 t와 threshold value가 0에 가까워질수록 approximation error가 작아지게 된다. 하지만, threshold value의 inconsistent가 다양한 noise를 유발하게 되고, 실제 환경에서는 이러한 가정이 불충분해서 detail과 contrast에 loss를 유발하게 된다. 이러한 문제를 해결하기 위해, 우리는 sequential deblurring process를 새롭게 설명함으로써 data에서 clean image를 reconstruction하는 것을 배우는 joint framework을 제안한다.

__Deep sequential deblurring__

 Event-assisted deblurring은 Maximum-a-Posteriori로 formulate될 수 있다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-4.PNG)

 식 (3)을 풀기 위해서 우리는 아래와 같은 simplification이 필요하다. joint posterior에 대해서, 우리는 식 (2)의 adjacne의 temporal reltation을 사용하고 Markov chain model을 가정하게 되면,

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-5.PNG)

 위와 같은 식이 나온다. 참고로 위 식에서

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-6.PNG)

 해당 부분은 Markov assumption을 의미하게 된다. 이러한 simplified model이 첫번째로 T에서의 intensity( $I_T$ ) 를 구하게 되고, 이후로 sequential reconstruction을 수행하게 된다. bayesian rule에 따르면, backward reconstruction step의 maximizer는 다음과 같다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-7.PNG)

 여기서, prior term은 latent image의 distribution을 이끌어 낸다. 이는 최근 event-based image reconstruction의 l1 gradient나 manifold smoothness와 같다. likelihood term을 설계하기 위해, 우리는 previous reconstruction의 initial estimate를 식 (2)를 응용하여 아래와 같이 만들었다.

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-8.PNG)

 위 식에서 $\bigodot$ 은 hadamard product를, 나머지는 아래 식과 같다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-9.PNG)

 time interval이 매우 작기 때문에, 우리는 small drift와 좋은 initialization을 만들 수 있는 constant threshold value( $\tau$ ) 를 사용했다. $I_i^*$ 를 풀기 위해, 몇몇 work들은 $\hat{I_i^*}$ 주변의 simple distribution center를 가정해서 식 (5)와 같은 likelihood term을 정의했다.(다른 논문에서는 Poisson distribution을 사용했다고 한다.) 이런 방법에서 식 (5)는 잘 연구된 denoising problem으로도 다루어질 수 있다.(음.. 해당 식을 이용한 future work을 제시하는 문장 같음)

 simple image prior를 사용하는 것 대신에, 우리는 최근 연구에서 사용한 결과를 빌려왔다. 통상적으로, 우리는 deep network N을 learned denoisor라 하면, 아래와 같다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-10.PNG)

 위 식에서는 prior term인 P가 explicit하게 계산이 되는 것이 아닌 implicit하게 계산이 되는 것임을 확인할 수 있다. parameter size 수를 줄이고 overfitting을 막기 위해, 우리는 각 deblurring step의 parameter를 아래 논문에서 따왔다.

Brandli, Christian, et al. "A 240× 180 130 db 3 µs latency global shutter spatiotemporal vision sensor." *IEEE Journal of Solid-State Circuits* 49.10 (2014): 2333-2341.

 남은 문제는 식 (4)를 푸는 건데, 어떻게 initial latent image( $I_T$ )를 얻느냐는 것이다. 우리는 blurred image ( $\bar{I}$ ) 가 rough하게 exposure process에서 instant image들의 평균과 같다고 두었다. 이를 (6)과 혼합하면,

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-11.PNG)

 위 식이 되고, 위 식에서 $\mathcal{B}$ 는,

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-12.PNG)

 위와 같다. 이는 blurred image의 initial estimation인 $\hat{I}_T$ 를 blurred image와 event에서 제공할 수 있게 한다. 그러므로, 우리는 $I_T$ 를 denoising problem으로 다룰 수 있게 되고, network가 이를 예측할 수 있도록 할 수 있다. 하지만, (8)에서 축적이 되는 operator들은 sequential deblurring step과는 달리 더욱 drift가 된다. 그래서 우리는 분리되고 더욱 강력한 network으로 initial estimation을 수정하게 된다.

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-13.PNG)

 이 때 수정에 사용되는 network은 위와 같다.

 Full deblurring process는 Algorithm 1에 소개되어 있다. 식 (7)에 의해, 최근 image는 image와 event에서 local하고 long-term cue를 조건으로 하게 된다.

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-14.PNG)



### Network Architecture

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-15.PNG)

 위 그림이 전체적인 Network을 보여준다. 위 그림에서 event를 가로지르고 global scene motion의 single representation을 만드는 __read network__ , appearance와 motion을 묶고 initial latent image를 만드는 __initialize network__ , 모든 latent image들을 순차적으로 deblurring하는 __recurrent process network__ 가 있다. read와 initialize network은 algorithm 1에서 $N_0$ 로 나타나있고, process network은 $N$ 으로 표시되어 있다.

 read network은 모든 event data를 읽고 global event motion을 account하는 joint representation을 생성한다. 이를 성취하기 위해, exposure동안의 event들은 같은 time interval을 두고 첫번째 bin에 넣어지게 된다.(Figure 2 참조) 각각의 time interval에서 event들은 8 equal-size chunk의 간격을 통해 stacked event frame으로 표시가 되고 각 chunk에서 event falling의 poliarity를 모두 더한 후 channel dimension으로 stack을 하게 된다. read network은 Conv block과 Conv LSTM을 포함한 recurrent encoder로 구성이 되어 있다.

 initialize network은 blurred image에서 appearance로 decode하고 latent image $I_T^*$ 를 풀기 위해 global motion을 couple한다. blurred image와 initial estimate을 input으로 받고 이들을 convolutional encoder로 받아서 read network에서 온 global motion feature를 축적한 encoding을 concat한 다음 deocder에 joint feature로 주게 된다.

 주어진 initial 결과에서, process network가 남은 latent image들을 순서대로 deblur하게 된다. i번째 step에서 process network은 image와 event-based observation 모두를 흡수한다. image part는

- initial estimate
- local historical image(Motion Compensation에서 온 것)
- boundary guidance map(Directional Event Viltering에서 온 것)

 을 포함한다. MC와 DEF는 나중에 간단하게 소개될 것이다. input image들은 conv layer와 concate에 의해 진행이 되고, latent fusion을 통해 read network에서 per-step event로 feature가 추출이 된다. fused feature는 temporal knowledge를 propagate하기 위해 다른 convolutional LSTM에 processed와 fed가 된다. 마지막으로, decoder는 joint feature를 받고 deblurred image를 내뱉게 된다.

__Motion compenstation__

 나는 이전의 deblurring result를 warp하고 i-th time step의 initialization을 만들기 위해 motion compensation module을 만들었다. 식 (6)은 event integration 으로 이를 이룬 반면에, 우리는 추가적인 guidance로 clean image를 직접적으로 warp한 flow field를 예측하는 더 효율적인 방법을 찾는다. motion compensation module은 이미 다른 논문에서 소개된 적이 있다. 효율을 위해서, 우리는 FlowNetS의 구조를 따왔고, warping은 differentiable spatial transformer layer로 실행되었다.

__Directional event filtering__

 initial estimate은 naive blurring model과 event의 noisiness때문에 blur를 해결하지 못하고 있었다. 우리는 이러한 문제를 넓은 범위의 image prior를 조사하는 것인 sharp boundary prior를 이용하는 것으로 완화시켰다.

 scene의 밝기 변화와 physical boundary들을 event가 보여준다. 하지만, scene boundary가 움직인다면, 어떤 특정한 시간대에서는 그들(scene boundary)들이 오로지 최근에 보여졌던 image들과만 spatial하게 align이 된다. 

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-16.PNG)

 위 그림은 해당 과정의 example을 보여준다. 서로 다른 time에 3장의 사진이 찍혔고, 그 사이에 카메라가 아래 위로 흔들렸다고 하자. 이는 적절한 space-time position에서 event를 sampling함으로써 scene boundary prior를 만들어낼 수 있다는 것을 보여준다. scene depth의 변화때문에, different scene들은 distinct motion을 가지게 되고, position-adaptive sampling은 필수적이게 된다.

 반면에, event가 sparse, noisy, non-uniformly distributed signal을 가진다면, robust sampling process가 어디에, 얼마나 sample을 해야하는지 결정을 해야 한다. 우리는 이러한 task를 differentiable sampling and filtering을 통한 data에서 학습했다. 각 image position p 에 대해서, temporal center인 c(p) 와 2k+1개의 filtering coefficients(여기서 k는 filtering kernel) 들은 event의 small network에서 예측이 될 수 있다. filter가 된 결과는 아래와 같다.

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-17.PNG)

 위 식에서 $\lambda$ 는 sampling stride를 말하고, $s(\cdot , \cdot)$ 은 space-time domain에서의 sampling function을 말한다. event의 stacked event frame representation인 $\mathbb{E}_i^{i+1}$ 을 위해 continuous sampling을 위한 trilinear kernel을 적용할 수 있다. 참고로 velocity d는 event의 density surface를 따라 filter를 하기 위해 space-time point에서 event의 local motion에서 방향을 따라간다.(p, c(p))

 local velocity를 구하기 위해, 우리는 motion compensation module에서 구한 flow vector를 재사용한다. 우리는 object velocity가 일정하다고 가정하자. 그럼 motion compensation은 각 time i에서 모든 position의 velocity( $d(p_0, i)$ )를 준다. 

 이로 인해 shift 된 위치는 다음과 같다.

![img18](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-18.PNG)

 위 식에서 $n(p_0)$ 는 다음을 의미한다.

![img19](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-19.PNG)

 하지만 intersected position은 완벽한 sampling을 보장하지 않는다. 그래서 우리는 주어진 target p에 대한 velocity를 Nadaraya-Watson estimator를 이용해서 다시 resample을 하게 된다.

![img20](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-20.PNG)

 위 식에서 kernel $\kappa$ 는 standard Gaussian이다. 이러한 움직임들은 surface rendering의 computer graphics에서 모으는 의미와 유사하다.

 식 (11)은 각 position p를 예측하기 위해 모든 $p_0$ 를 사용한다. 실제 환경에서는 우린 p 근처에 local $L \times L$ 의 window를 위치시켜서 sample한다. window size L은 maximal spatial displacement로, 실험 결과 20이면 충분하는 것을 알게 되었다. 모든 proposed step은 differentiable해서, end-to-end training이 가능하다.

__Loss Function__

 우리는 아래와 같은 joint loss function을 만들었다.

![img21](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-21.PNG)

 여기서 $L_{content}$ 는 photometric l1 loss를 의미한다. 결과의 sharpness를 위해, 우리는 adversarial loss도 추가했다. 우리는 PatchGAN distcriminator와 똑같은 것을 사용했고, original loss를 엄격하게 따랐다.

 flow network는 두개의 다른 loss term을 소개한다. 첫번째 flow loss는 photometric reconstruction loss로,

![img22](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-22.PNG)

 위 식에서 $\omega(\cdot , \cdot)$ 은 forward flow를 이용한 backward warping function이다. 그리고 $L_{tv}$ 는 아래 식과 같으며, flow field smoothing을 위한 total variation loss이다.

![img23](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-23.PNG)

 여기서 $\lambda_a$ 와 $\lambda_t$ 는 각각 0.01과 0.05로 설정했다.



### Experiments

__5.1. Experimental Settings__

__Dataset preparation.__

 우리는 2가지의 데이터셋을 이용했다. 첫번째는 GoPro dataset이고, 믿을만한 synthesize event를 위해 open ESIM event simulator를 사용했다. 

 real world scenario에서 event-based motion deblurring의 평가를 위한 large-scale dataset이 부족하여, 우리는 Blur-DVS라는, DAVIS240C camera를 이용해 도시 환경에서의 새로운 dataset을 만들었다. 이 카메라는 low frame rate APS인, 180x240 pixel의 intensity image를 뽑아내는 high speed event sensor를 가지고 있다. 따라서, APS(Active Pixel Sensor)는 빠른 움직임에서 motion blur를 만들게 된다. 우리는 evaluation을 위해 2개의 subset을 만들었다. slow subset은 느리고 안정적인, 상대적으로 정적인 환경에서 이루어져있어 motion blur가 좀 덜한 반면, fast subset은 빠른 환경에서 이루어져 있어 조금 더 real motion blur에 맞다. 하지만, fast subset에 한해서 available한 GT data가 없다는 것이 문제이다.

__Method comparsion.__

 우리는 결과를 최근에 나온 것들과 비교해봤다. DCP, MBR, FLO... 등등등. metric으로는 PSNR과 SSIM metric을 사용했다.

__Implementation details.__

- batch size: 2
- Adam Optimizer
- 400 epochs
- lr: 1e-4
- lr decay to 0 starting from 200th epoch
- scratch에서 train

__5.2. Comparisons with State-of-the-Art Models__

 GoPro dataset에 대한 결과를 아래와 같이 담았다.

![img24](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-24.PNG)

![img25](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-25.PNG)

 task는 두 가지가 있는데, image deblurring(middle frame에서 recovering 하는 것)과 video reconstruction(모든 sharp frame을 recover하는 것) 이다. 

 하지만 GoPro dataset은 약한 blur만 있다보니, event에서 오는 개선이 좀 덜하다. 따라서 우리는 새로운 dataset인 Blur-DVS dataset을 만들었다. 이 결과는 아래에 담겨져있다.

![img26](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-26.PNG)

![img27](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-27.PNG)

![img28](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-28.PNG)

 이와 같은 실험 결과들에서 우리는 motion deblurring을 explicit하게 모델링하고 deblurring prior를 강하게 소개하는 것이 학습의 어려움을 완화시키고 potential overfitting을 막아주는 것임을 확인할 수 있었다.

__5.3. Performance Analysis__

__Analysing different components.__

![img29](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-29.PNG)

![img30](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-30.PNG)

 Ablation study라고 보면 될 듯.

__Justification of the DEF module.__

![img31](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-31.PNG)

![img32](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-32.PNG)

__Low-light photography.__

![img33](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210521-33.PNG)



### Conclusion



### Reference

Jiang, Zhe, et al. "Learning event-based motion deblurring." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.

Brandli, Christian, et al. "A 240× 180 130 db 3 µs latency global shutter spatiotemporal vision sensor." *IEEE Journal of Solid-State Circuits* 49.10 (2014): 2333-2341.