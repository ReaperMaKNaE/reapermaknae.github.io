---
layout : post
title : "STTR"
date : 2021-01-18 +1631
description : Revisiting Stereo Depth Estimation From a Sequence-to-Sequence Perspective with Transformers 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### Revisiting Stereo Depth Estimation From a Sequence-to-Sequence Perspective with Transformers, CVPR 2020



SUMMARY: Transformer를 이용해서 depth를 추출해내는 network. 이름은 STTR.

 feature extractor는 hourglass model을 사용함. 여기서 local, global context에 대한 vector를 뽑아냄.

 이후 attention을 이용한 transformer로 들어가게 됨. transformer에서는 self and cross attention을 진행함.

 여기에서 나온 결과를 optimal transport로 넘기게 된다. 이 때 optimal transport는 regularization을 위해 sinkhorn algorithm을 사용함.

 여기서 나온 결과는 disparity와 occlusion 정보가 담긴 matrix로, adjustment를 진행한 후, 최종 disparity & occlusion을 제시하게 됨.  결국은 input으로 2 RGB image를 받고, depth map을 내뱉음. cost volume 없이! pose도 없이! 어짜피 물체간의 상대적인 위치로 depth를 판별하기 때문에, descriptor간의 차이가 어떻게 되느냐가 depth를 결정하는 요소가 됨.



### Abstract

 Stereo depth estimation은 epipolar line 위의 pixel들 사이에서 matching을 이루는 연관성이 중요하다. 각 pixel들을 전부 matching하는 것이 아니라, 우리는 dense pixel matching의 cost volume construction을 대신하기 위해, position information과 attention을 이용한 sequence-to-sequence correspondence perspective에서 이 문제를 다시 살펴보았다. 이러한 접근은 STTR(STereo TRansformer)로 이름을 지었고, 다음과 같은 이점이 있다.

- disparity range가 고정되어 있는 제한을 풀어버림
- occluded region을 확인하고 강한 estimation confidence
- matching process동안 유일한 constraint을 도입하는 것

 우리는 STTR이 real-world dataset이나 synthetic dataset이나 잘 작동하는 것을 확인했다.(fine tuning 없이)



### Introduction

 Stereo depth estimation은 3D information의 recon을 가능하게 해서 상당히 흥미롭게 다루어져 왔다. 이것을 끝내기 위해, left and right camera의 image 사이에서 pixel을 matching하는 방법이 3D scene reconstruction에서 주로 사용되어 왔다. 최근 deep learning base로 stereo depth estimation을 하는 방법이 많이 나와있지만, 여전히 많은 도전들이 남아있다.

 좋은 성능을 보이는 모델들이 많이 나왔지만, 그들은 pre-specified disparity range의 한계가 있다. 이는 cost volume이란 방법에 의존해있기 때문인데, 이러한 것들은 여러 candidate을 match해서 코스트를 계산하고 disparity value를 aggregated sum으로 예측했다. 하지만 이런 방법은 memory에 전혀 도움이 되지 않고, disparity range를 manual하게 수정해주어야 한다는 단점이 있다. 대부분의 learning-based architecture는 maximum disparity를 192로 제한을 하고 있다. 이러한 disparity range는 여러 방면에서 충분하지만, 우리가 section 4에서 설명할 camera resolution, proximity, configuration of the stereo camera 등에 의존한다. 더욱이, 자율주행 자동차와 같은 경우에서 이것은 충돌을 피하기 위해 가까운 물체를 인식하는 것은 중요하기에 fixed range assumption을 relax하는 것은 필요하다.

 또, geometric properties나 constraint는 learning base인 경우에도 실패하는 상황이 존재한다. Stereo depth estimation의 경우, occluded region은 valid disparity를 가지지 않는다. 몇몇 알고리즘은 piece-wise smoothness assumption을 통해 occlude region의 disparity를 구하기도 했으나, 항상 valid한 것은 아니었다. disparity value에 confidence를 강티 주는 방법도 있으나, 이러한 information을 가지고 있지는 않다. 한 개의 pixel은 다른 image의 여러 점에 해당할 수 없다는 제약이 있는데, 이러한 제약은 애매모호한 지역, 혹은 large textureless area에선 꽤나 유용했으나 대부분의 방법에서는 이를 도입하지 않았다.

 이전에 언급한 문제들은 점점 커졌다. feature matching하면서 현재의 view에서도 cost volume을 만드는 것이다. 하지만 이러한 문제들은 epipolar line에서 sequence-to-sequence matching의 disparity estimation의 접근으로 이러한 challenge를 피할 수 있었다. 이러한 방법들은 새로운 것이 아니라, 1999년에 이미 소개되었다. 하지만 pixel intensity와 matching criteria의 유사성에만 사용이 되어서, 그 성능이 제한되었다. 최근에 attention base로 발전한 network들은 feature descriptor 사이의 관계를 잡아냈고, 이러한 관점에서 다시 우리가 시도하게 만들었다. 우리는 최근의 Transformer architecture에 영감을 받아 새로운 end-to-end training stereo depth estimation network인 STTR을 제안한다. STTR의 main advantage는 pixel-wise correlation을 계산하지, cost volume을 만들지 않는다는 점이 중요하다. 그 덕분에, STTR은 위에서 언급한 모델들과 성능에서 큰 차이 없이 단점을 완화할 수 있다. 우리는 refinement 따로 없이 synthetic image에 대해서만 학습을 했음에도 competitive한 performance를 synthetic, real image benchmark에서 보여주었다.

 다음 기술적인 이점들이 STTR을 만들어주었다.

- Patch-wise correlation 대신에, matching process동안 feature를 구분해주는 alternating self- and cross-attention 방식을 사용하여 pixel의 유사성을 계산한다.
- optimal transport theory를 통한 matching uniqueness 제약을 도입하고, large textureless region에서 애매모호함을 해결하기 위해 feature descriptor에 상대적인 pixel distance information을 제공한다.
- 우리는 network를 흔하게 발생하는 인공적인 stereo 문제(예를 들면 rectification error나 occlusion)에도 적응하기 위해 다양한 비대칭적인 확장을 통해 학습시켰다.
- STTR의 실행에 있어서 memory, parameter efficient한 방법을 창안한다.



### Related Work

__2.1. Stereo Depth Estimation__

 일반적으로 stereo depth estimation에는 2가지의 중요한 step이 있다. feature matching과 matching cost aggregation이다. 

 전통적인 방법도 있고 end-to-end 방법을 이용한 방법도 있다. 전자는 일단 전반적으로 성능이 제한되거나 좋지 못했고, end-to-end 방법은 대부분 cost volume base이기 때문에 computation의 문제에 걸려있다.

 이러한 방법 말고 MRF와 같은 방법을 이용해서 접근한 방법들도 있었으나, 그 어떤 방법들도 non-learning base인 work에서도 성공했던 sequential nature나 geometric properties를 exploit하는 것에 실패했다.

 모든 learning based method에서, maximum disparity를 설정하는 것은 memory와 computation을 고려해야했다. 각 pixel은 cost volume의 matching으로 만들어지는데, pre-defined range의 disparity에 의해 cost volume base의 approach는 간단하게 match가 되질 않았다. 이러한 한계는 서로 다른 scene과 stereo camera configuration에 따라서 network를 generalization 해야하는 필요성에 놓이게 되었다. 더욱이, 대부분의 learning based network는 occlusion을 자세하게 다루지 않는다. 

__2.2. Comparison of STTR to Previous Paradigms__

__STTR vs 3D convolution-based Stereo Depth Networks__

__STTR vs Correlation-based Stereo Depth Networks__

__2.3 Attention Mechanism and Transformer__

 Attention은 NLP에서 효과적인 tool로 입증이 되었다. 최근, 이러한 attention은 object detection, panoptic segmentation, SLAM등과 같은 CNN base 모델의 computer vision task에도 적용이 되고 있다. 이러한 attention 기반으로 접근한 것들은 아래와 같다.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-14.PNG)



### The Stereo Transformer Architecture

 I _ h와 I _ w를 각각 image의 height, width라고 함. 그리고 feature descriptors의 channel dimension은 C라고 하자.

__3.1 Feature Extractor__

 우리는 효과적으로 global context를 얻기 위해 hourglass-shaped architecture를 사용했다. 각 픽셀에서의 feature descriptor는 각각 size C _ e를 가진 vector e _ I 로 표현한다.(encode에서 local, global 둘 다) 마지막 feature map은 input size와 똑같다.

__3.2 Transformer__

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-15.PNG)

 우리는 2가지 type의 attention module을 소개할건데, 하나는 self-attention이고, 다른 하나는 cross-attention이다. 우리는 여기서 N-1 번의 iteration을 돌았다. 이러한 교차적인 형태는 section 3.2.2에서 설명이 되겠지만 image context와 relative position에 대해서 feature descriptor를 지속적으로 update하게 해준다.

 마지막 cross-attention layer에서, 우리는 raw disparity를 estimate하기 위해 추가적인 pixel을 사용한다. uniqueness constraint를 준수하기 위한 optimal transport와 검색 영역을 줄여줄 attention mask를 추가했다.

__3.2.1 Attention__

 attention module은 query와 key vector 사이의 attention을 계산하고, 이후 value로 weight를 준다. mutli-head attention을 적용했고, 여기에 사용할 channel dimension은 feature descriptor의 channel dimension에 head의 개수를 나누었다. 그러므로, 각 head는 서로 다른 semantic meaning을 가질 수 있고, similarity가 각 head에서 계산이 될 수 있다. 각 attention head h에 대해서, feature descriptor input에 대해 결과로 나오는 query, key, value는 다음과 같다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-16.PNG)

 normalize는 softmax를 사용해서 다음과 같이 된다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-17.PNG)

그리하여 output은 아래와 같다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-18.PNG)

 마지막단계에 output으로 나가게 되는 value는 residual connection으로 이루어졌기에 다음과 같은 형태로 나가게 된다.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-20.PNG)

__3.2.2 Relative Positional Encoding__

 large textureless area에서는 pixel간 similarity를 찾는 것이 애매모호할 수 있다. 하지만 이러한 similarity의 애매함은 edge와 같은 명백한 feature들을 찾는 것을 이용해서 해결할 수 있다. 그러므로, 우리는 data-dependent spatial information을 제공한다. 즉, positional encoding이다. 우리는 절대적인 pixel location보다는 상대적인 pixel distance를 encode하는 것을 선택했다. 이는, invariance의 shift 때문이다. 평범한 Transformer에서는 absolute positional encoding e _ p 가 바로 더해진다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-19.PNG)

 이러한 경우엔, equation 2 에서 i번째 pixel과 j번째 pixel 사이의 attention은 다음과 같다. 여기서 term (a)를 보면 data-data component이고, term (b)와 (c)는 data-position, (d)는 image content를 전반적으로 무시하고 빠져야 하는 term이다. 대신에, relative positional encoding을 쓰고 (d)를 지우게 되면, 아래와 같다. (position끼리 query, key value를 연산해서 계산해봤자 별로 의미가 없음. content가 어떤 위치에 존재하느냐가 더 중요하기 때문)

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-21.PNG)

 e _ p,i-j는 i번째 pixel과 j번째 pixel 사이의 positional encoding을 의미하는데, e _ p,i-j와 e _ p,j-i는 다르다는 것을 알아야한다. 직관적으로, attention은 content similarity와 relative distance에 의존한다. (이 부분은 이해가 잘 가지 않는다.)

 그런데 이렇게 연산하는 것은 문제가 많다. 그래서 효율적인 연산을 썼음. 자세한건 appendix에 있음.(appendix를 보면 이렇게 하면 행렬이 간단해져서 연산이 쉬워짐.)

__3.2.3 Optimal Transport__

 stereo matching에서 uniqueness constrain을 거는 것은 이미 시도가 되었다. 하지만 이러한 시도는 gradient flow를 막았었다. 반면에, entropy-regularized optimal transport가 soft한 assignment와 differentiability로 이상적인 대체재로 떠올랐다. 두 marginal distributions a 와 b의 cost matrix M에 대해서, entropy-regularized optimal transport는 아래 식을 푸는 것으로 optimal coupling matrix T를 찾는 것을 시도했다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-22.PNG)

 위 식에서 E(T)는 entropy regularization이다. 만약 두 marginal distributions a, b가 uniform이라면, T는 soft uniqueness constraint와 mitigate ambiguity를 impose하는 assignment problem의 최적이 된다. 그래서 위 식의 해법은 iterative Sinkhorn algorithm에서 찾을 수 있다. 직관적으로, T는 equation 2의 softmaxed attention과 유사하게 pairwise matching probability를 말한다.

 occlusion때문에 몇몇 pixel은 match가 되지 못한다. SuperGlue를 따라서, 우리는 cost matrix에 dustbin을 추가해서 match가 되지 못하는 것들은 버리게 되었다.

 STTR에서, cost matrix M은 optimal transport가 attention value를 normalize하지 않도록 softmax없는 cross-attention module에 의해 계산된 attention의 음수값으로 세팅된다.

__3.2.4 Attention Mask__

 어떤 물체가 카메라에 비해서 상대적으로 우측에 위치한다면, 상대적으로 stereo camera의 오른쪽 image에 해당 물체가 더 오래 맺히게 된다. 그러므로 마지막 cross-attention layer에서는 left image에서 각 pixel이 right에 조금 더 집중하도록 한다. 이러한 constraint를 impose하기 위해, 우리는 diagonal binary mask를 사용했다. (Figure 2 참조)

__3.2.5 Raw Disparity and Occlusion Regression__

 대부분의 이전 work들은 모든 disparity candidate의 weight sum으로 사용이 되었다. 우리는 대신에 winner take all 접근 방식을 사용해서 regress disparity를 대신한다. 이 방식은 multi-model distribution에 robust하다.

 raw disparity는 optimal transport assignment matrix T에서 가장 match가 잘될 것 같은 location을 찾는 것에서 계산이 되고, k로 notation한다. 그리고 그 결과는 화면 근처에 3px로 만들게 된다. 이러한 3px window를 포함해서 re-normalization이 실행된다. candidate disparity들의 weighted sum은 raw disparity d ~ raw를 regress한다. 따라서, 우리는 아래와 같은 식을 얻는다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-23.PNG)

 occlusion probability의 역으로, 이러한 3px window의 probability는 현재 assignment에 대한 confidence를 나타낸다. 그러므로, 그 형태는 아래와 같다.

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-24.PNG)

__3.3 Context Adjustment Layer__

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-25.PNG)

 epipolar line위에서 regress되는 raw disparity와 occlusion map은 multiple epipolar line에서 부족한 context를 보여준다. 이를 완화하기 위해, 우리는 bundle adjustment를 쓴다. context adjustment는 위 그림에 나와있다.

 raw disparity와 occlusion map은 처음에 channel dimension을 따라서 left image에 concatenate 된다. occlusion은 Conv를 통하고, disparity는 residual block을 지속적으로 통과한다.

__3.4 Loss__

 우리는 assign matrix T에 대해서, occlusion때문에 Relative Response loss인 L _ rr을 적용한다.

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-26.PNG)

 위 식에서 t star는 matching probability를 의미하고, d _ gt는 GT disparity를 의미한다. interp는 linear interpolation을 말한다. raw와 final disparity 둘 다 에서 smooth L1 loss를 적용했다. final occlusion map에는 binary-entropy loss를 적용해서, total loss는 아래와 같다.

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-27.PNG)

__3.5 Memory Efficient Implementation__

 메모리 얼마나 아꼈는가가 나옴. 960*540에 channel이 128이면 8GB정도라고함.



### Experiments, Results, and Discussion

__Datasets__

Scene Flow, KITTI 2015, Middlebury 2014, SCARED 사용했음

__Hyperparameters__

채널은 128, sinkhorn algorithm은 10번의 iteration 등등.

__4.1 Ablation Study__

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-28.PNG)

__Attention Mask__

__Optimal Transport__

__Context Adjustment Layer__

__Relative Positional Encoding__

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-30.PNG)

__Attention Span__

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-29.PNG)

__4.2 Scene Flow Benchmark Result__

![img18](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-31.PNG)

__4.3 Cross-Domain Generalization__

![img19](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-32.PNG)

__Feature Distribution__

![img20](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-33.PNG)

__4.4 KITTI Benchmark Result__

![img21](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-34.PNG)

__4.5 Shortcomings in Challenge Design__

__4.6 Loose Analogy to Biological Stereo Vision__



### Conclusion



### Appendix

__A. Efficient Implementation of Attention with Relative Positional Encoding__

![img22](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210119-35.PNG)

__B. UMAP of Feature Map__

__C. Evolution of Feature__

__D. Evolution of Attention__



### References

Li, Zhaoshuo, et al. "Revisiting Stereo Depth Estimation From a Sequence-to-Sequence Perspective with Transformers." *arXiv preprint arXiv:2011.02910* (2020).

