---
layout : post
title : "SuperGlue"
date : 2021-01-18 +1631
description : SuperGlue, Learning Feature Matching with Graph Neural Networks 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### SuperGlue: Learning Feature Matching with Graph Neural Networks, CVPR 2020



SUMMARY: feature extraction을 front라 하고, bundle adjustment나 pose estimation을 back이라고 한다면, SuperGlue는 그 중간에 위치한다. 이 친구들은 visual descriptor와 position에 대한 정보를 받아서 Attentional Aggregation을 통해 두 이미지의 feature를 서로 연결하고, 이렇게 연결한 matching descriptor들의 score를 matrix로 기록한다. 이 matrix에는 각 score 뿐 아니라, dustbin score라 하여 얼마나 많은 점들이 dustbin(다른 이미지와 매칭이 되지 않는 점들을 버리는 곳)으로 가는지 결정하고, 이를 Sinkhorn Algorithm을 통해서 normalization을 진행한다. 그리고 각 점에 따라 partial assignment를 진행한다. 이렇게 되면 각 feature가 어디와 어떻게 연결되는지 알 수 있게 된다.



### Abstract

 이 논문에서는 correspondences는 찾고, non-matchable point는 reject하여 local feature를 찾는 neural network을 소개한다. 과제들은 graph neural network로 예측된, differentiable optimal transport problem의 cost를 해결하여 estimate된다. 3D Scene과 feature assignment를 같이 하는 것으로 superglue를 적용한, flexible context aggregation method를 소개한다. traditional한 방법들과 비교하였을 때 우리의 학습 방법은 end-to-end이고, pose estimation과 같은 것들에서 SOTA를 찍었다.(indoor, outdoor 모두) 



### Introduction

 point간의 연관성을 찾는 것은 3D structure와 camera pose에서 중요하다. 주로 사용되는 곳은 SLAM이나 SfM이다. 이러한 연관성은 local feature를 matching하는 것에서 시작한다. 하지만 이러한 것들은 큰 view-point, light change, occlusion, blur와 같은 상황에서 심각해진다.

 이번 paper에서 우리는 feature matching problem에 있어서 새로운 길을 제시한다. 간단한 heuristics and tricks에 의해 따라오는 local feature들의 학습 대신에 우리는 SuperGlue라고 불리는 novel neural architecture를 이용해서 pre-existing local feature에서 matching process를 학습하는 것을 제안한다. SLAM에 있어서 사례를 보면, visual feature extraction을 front-end로 사용하고, 이후 bundle adjustment나 pose estimation 같은 것들을 back-end로 쓰는데, 우리는 그 중간에 위치한다. 즉, middle-end인 셈.

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-18.PNG)

  이번 work에서는, learning feature matching이 local feature의 set 사이에서 부분적인 assignment를 찾는 것이다. 우린 matching problem에서 classical한 graph-based 전략이 dfferentiable하게 해결될 것이라고 생각하고 있다. GNN(Graph Neural Network)에 의해 optimization의 cost function이 predict 되었다. Transformer의 성공에 영감을 받아, cost function은 keypoint들의 spatial relationship과 그들의 appearance 사이에서 저울질을 하는 것을 사용한다. 이러한 형태는 prediction의 assignment 구조를 강요한다.(?? assignment 구조라는게 occlusion과 non-repeatable keypoint를 복잡하게 다루는 cost 방식이 아닌 다른 어떤 것을 말하는 듯. Transformer를 읽어봐야 이해가 올 듯 하다.) 우리의 방법은 image pair에서 end-to-end로 학습하는 것을 말한다. 우리는 large dataset에서 pose estimation을 하고 이를 학습하게 된다. 이러한 방법은 여러 군데에서 쓰일 수 있다. 아래 그림 참조.

![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-19.PNG)



### Related Work

__Local feature matching__

 local feature matching은 일반적으로,

- interest point 찾기
- visual descriptors 계산
- NN search로 매칭
- incorrect match들 filter로 날리기
- geometric transformation 예측

  이러한 방법을 응용한 것은 traditional한 방법이나 CNN을 이용한 방법 등등 여러개가 존재하지만, 모두 한계가 있음. assignment structure나 visual information을 무시함. 그래서 point cloud나 dense matching등의 내용이 생기긴 했으나, 여전히 같은 한계에 부딪치게 됨. 그에 반대로, 우리의 모델은 context aggregation, matching, filtering을 한 개의 end-to-end architecture에서 구현하였음.

__Graph Matching__

 위와 같은 문제는 NP hard와 같은 quadratic 배치 문제에 주로 사용이 되었으나, 복잡하고, 비싸고, impractical solver에 의존해야했음. 이러한 방법들이 많이 나와있으나, 가볍지 않음. 반면에 우리가 사용하는 SuperGlue는 dNN에서 flexible한 cost를 학습할 수 있음.

__Deep learning for sets__

 point cloud와 같은 deep learning set은 permutation equivalent나 invariant function을 설계하는 것을 목표로 한다. 우린 attention을 graph에 multiple types of edge와 적용해서 복잡한 local feature set의 추론을 학습했다. 



### The SuperGlue Architecture

__Motivation__

 3D world는 어쩔땐 부드럽고, 어쩔 땐 planar해서, single epipolar line에 있더라도 correspondence나 pose들은 서로 다르다.(지구의 형태에 의한 자연적인 distortion을 의미한다) 더욱이, 2D keypoint들은 3D point의 projection이므로, 다음과 같은 physical constraint에 제한된다.

- keypoint는 최대 1개의 correspondence를 다른 이미지에서 가진다.
- 몇몇 keypoint들은 occlusion이나 detector의 failure로 match되지 않는다.

 그래서 효율적인 model은 same 3D point에 대한 correspondence를 찾고, 어떤 것들이 match 되지 않는지에 대해 아는 것이다. 이걸 우리는 SuperGlue로 만들었고, 그 architecture는 아래와 같다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-20.PNG)

__Formulation__

 local feature인 (p,d)에서 p를 keypoint position이라 하고, d를 visual descriptor라 하자. 여기서 detection confidence를 c라고 했을 때, p_i := (x,y,c)_i 이다. 여기서 visual descriptor는 SuperPoint나 SIFT등과 같은 것들에서 뽑아낼 수 있다. Image A는 M개, Image B는 N개의 local feature를 가지고 있다고 하자.

__Partial Assignment__

 위에서 제한되었던 physical constraint 2개는, 두 keypoint사이에서 partial 배치에서 오는 correspendence에 의해 생기게 된다. downstream task을 합치고, 더 나은 interpretability를 위해, 각 correspondence는 confidence value를 가지게 된다. 우리는 각 partial soft assignment Matrix를 다음과 같이 정의 한다.

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-21.PNG)

 우리의 목표는 두 local feature set에서 assignment P를 예측하는 것이다.

__3.1 Attentional Graph Neural Network__

 keypoint의 위치와 그것의 visual apperance에 있어서, 이러한 것들을 합한 contextual cue들은 얼마나 구별이 잘 되느냐에 따라 증가한다는 것을 직감적으로 알 수 있다. 우리는 이러한 spatial, visual relationship을 salient, self-similar, statistically co-occurring 혹은 adjacent중 하나 같은 것들로 고려할 수 있다. 하지만 반면에, 두 번째 이미지에서 keypoint를 알고 있다는 것은 애매하지 않은 점들의 geometric, photometric transformation 예측이나 candidate match와 같은 것들로 인해 애매한 것들을 해결하는 방식을 도와줄 수 있다.

 만약 애매한 keypoint들을 match할 때라면, 사람은 이미지를 앞뒤로 움직이며 본다. 이는 전체적인 이미지에서 contextual cue를 찾은 이후, 다시 돌아와서 그 이미지에 대한 파악을 하게 된다. 이러한 힌트는 특정한 위치에서 주의를 기울일 수 있는 것으로, 이를 iterative하게 반복하면 알 수 있게 된다.

 우리는 SuperGlue의 첫번째를 Attentional Graph Neural Network로 설정했다. 초기 local feature를 따라서, matching descriptors를 계산한다. (서로의 feature를 communicate하도록 해서) 알게 되겠지만, feature aggregation이 image에서 전반적으로 이루어지는 것은 robust matching에 활력을 불어넣어준다.(Figure 3에서 첫번째 파트를 보면, 같은 image에서 나온 feature 지들끼리의 관계와, 다른 이미지에서 나온 feature와의 관계를 따로 파악을 한다. 이러한 형태를 나중에 합치는 것을 말하는 듯 하다.)

__Keypoint Encoder__

 각 keypoint i에 대한 initial representation ^(0)x_i 는 visual appearance와 location을 합친 것이다. 우리는 MLP를 이용해서 이러한 keypoint position을 high-dimensional vector로 합쳤다.

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-22.PNG)

 이러한 encoder는 attention과 함께라면, graph network가 appearance와 position 둘 다 추론 가능하도록 한다.

__Multiplex Graph Neural Network__

 우린 한 개의 완성된 graph를 고려했다. 이 graph의 node는 두 image의 keypoint이다. 이러한 graph는 2가지 타입의 undirected edge를 가진다. 바로 multiplex graph이다. intra-image edge(혹은 self edge)에서는 같은 image내에 있는 모든 keypoint들이 서로를 연결한다. inter-image edge(혹은 cross edge)에서는 다른 image에 있는 모든 keypoint들과 연결한다. 우리는 이러한 두 타입의 edge에서 정보를 propagate하기 위해 message passing formulation을 사용했다. multiplex Graph Neural Network의 결과는 각 node의 high-dimensional state와 시작해서 서로 node끼리 주고받은 message를 합침으로써 각 layer의 updated된 representation을 계산한다. (여튼 각 노드끼리 어떤 관계를 가지게 되는지를 나타내는 방법이 필요한데, 이 방법이 multiplex graph가 딱 맞아 떨어진단 것 같다)

 이 때 image A의 layer l에 있는 element i를 ^(l)x_i^A라고 하자. 그러면 \epsilon에서 i로 보내는 message는(여기서 \epsilon은 edge를 말한다. 즉, node를 말하는 듯) 모든 keypoint들이 연결된 결과이다. residual message passing은 다음과 같다.

![img6](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-23.PNG)

 위에서 대괄호 내부의 표시는 concatenation을 의미한다. 유사한 방법으로, 다른 image인 B에서도 위와 같은 과정이 진행된다. 

__Attentional Aggregation__

 attention은 message를 만드는 것에 있어서 aggregation과 연산을 모두 행한다. self edges는 self-attention을 base로, cross edge는 cross attention을 base로 한다. 

 데이터베이스 retrieval과 유사하게, query는 q, value는 v, key는 k가 된다. message는 value의 weight average가 된다.

![img7](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-24.PNG)

 여기서 attention weight alpha는 아래와 같이 계산된다.(attention에서 계산하는 것과 같음)

![img8](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-25.PNG)

 따라서, 위 두가지를 합쳐 다시 쓰면, 그 결과는 아래와 같다.

![img9](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-26.PNG)

  SuperGlue는 retrieve, attend를 모두 할 수 있다. 이는 appearance와 keypoint location이 모두 representation x로 encode되어 있기 때문이다. 이러한 점들은 주변 점들과의 관계나 salient keypoint와의 관계를 나타내어 준다. 이는 geometric transformation을 알 수 있게 해주는 역할을 하게 된다. 그래서 결국 final matching descriptor는 linear projection인 다음과 같다.

![img10](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-27.PNG)

 이러한 점들을 합쳐서 visualize를 하면 다음과 같다.

![img11](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-28.PNG)

__3.2 Optimal matching layer__

 Figure 3에서 두 번째 main block은 optimal matching layer로, partial assignment matrix를 만들어내는 것이다. 이를 우리는 P 라고 하기로 했다. 이 친구는 traditional한 score matrix로 계산이 가능하다. 이는 linear assignment problem과 같다.

__Score Prediction__

 모든 M*N potential match를 각각 representation해서 build하는 것은 매우 비싸다.(prohibitive라고 적어놨는데, 저걸 다 각각 적어서 쓰는건 말도 안되는 메모리양을 먹기 때문에 그렇게 표현한 듯 하다) 그래서 우리는 대신에 matching descriptor의 similarity인 pairwise score로 표현하기로 했다.

![img12](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-29.PNG)

 위에서 <>의 표시는 inner product이다. visual descriptor와는 다르게, matching descriptor들은 normalize되지 않고, confidence를 prediction하기 위해 학습 도중에는 feature에 따라 그 magnitude가 바뀐다.

__Occlusion and Visibility__

 network가 몇몇 keypoint를 suppress하기 하기 위해, 우리는 each set에 dustbin을 더해 augment해서 unmatched keypoint들이 서로 연결되도록 한다. 이러한 방법은 graph matching에서는 흔히 있어왔고, dustbin도 이미 SuperPoint에서 사용이 되었다. (SuperPoint에서는 detection을 하지 못한 cell을 처리하기 위해서 사용했다고 함) 우리는 S를 S bar로 확장했다. (새로운 column과 row를 추가하는데, 이는 point to bin 과 bin to bin score에 해당한다)

![img13](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-30.PNG)

 만약 image A 안의 keypoint들이 B의 single keypoint로 연결이 되거나 dustbin으로 연결이 된다면, 각 dustbin은 그들의 각 set에서 가지고 있는 수많은 match들 만큼 가지게 된다. 즉, A의 image에는 N개의 dustbin이, B image에는 M개의 dustbin이 있다.(keypoint의 수와 같다는 뜻) 우리는 여기서 각 keypoint가 dustbin과 얼마나 match가 되었는지를 행렬 a와 b로 나타내고, 이는 다음과 같다.

![img14](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-31.PNG)

__Sinkhorn Algorithm__

 optimal transport에 해당하는 optimization problem의 해답은, a와 b, S bar 사이의 discrete distribution이다. 이는 entropy-regularized formulation 형태로, desired soft assignment의 결과를 낳게 되며, GPU를 이용하면 효율적으로 계산할 수 있다. 이 때 사용하는 알고리즘이 Sinkhorn algorithm이다. 이것은 hungarian algorithm의 differentiable version이며, bipartite matching에서 주로 사용된다. 이는 보통 score function의 exponential을 iteratively normalizing하는 것을 포함한다. (이 과정은 softmax의 row, column과 유사하다) T iteration이 지나게 되면, 우리는 dustbin의 수를 많이 줄이고 P를 복구한다.

(matching이 얼마나 잘되었느냐를 score로 매기게 되는데, 이 때 매긴 score는 keypoint가 버려지느냐(dustbin으로 가느냐), dustbin이 또다른 dustbin으로 가느냐를 저장하는 row, column을 추가해서 S bar로 확장하게 된다. 그리고 이 때 각 image에서 버려지는 point들을 a, b로 표시한다면, 이 때 dustbin을 최소화시키면서 P를 얻게 된다는 것 같다.)

__3.3 Loss__

 graph neural network와 optimal matching layer는 모두 differentiable하다. 이는 모두 matching에서 visual descriptor로 backpropagation이 가능하다. 여기서 GT를 만드는 것은 pose, depth map, homography등을 이용한 상대적인 transformation으로 사용이 가능하다. 

![img15](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-32.PNG)

 그래서 결국엔 dustbin을 최소화시키고 정확도를 올리게 되는 방법이다.

__3.4 Comparisons to related work__

 다른 network들과의 차이점을 보여주고 있다는데, 다른 것들을 몰라서 보는게 의미가 있나 싶다. 고로 생략.

__SuperGlue vs. Instance Normalization__

__SuperGlue vs. ContextDesc__

__SuperGlue vs. Transformer__



### Implementation details

 우리가 만든 SuperGlue는 그 어떤 local feature detector와 descriptor와도 잘 어울리는데, 특히 SuperPoint와 잘 어울린다.

__Architecture details__

D는 SuperPoint와 같은 256을 썼고, L은 9, attention head는 4개, T는 100을 썼다.

__Training details__



### Experiments

__5.1 Homography estimation__

__Dataset__

Oxford and Paris dataset 사용했음.

__Baselines__

__Metrics__

__Results__

![img16](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-33.PNG)

__5.2 Indoor pose estimation__

__Dataset__

ScanNet을 사용했음.

__Metrics__

__Baselines__

__Results__

![img17](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-34.PNG)

__5.3 Outdoor pose estimation__

__Dataset__

PhotoTourism dataset을 썼음.

__Results__

![img18](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-35.PNG)

__5.4 Understanding SuperGlue__

__Ablation study__

![img19](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210118-36.PNG)

__Visualizing Attention__



### Conclusion



### References

Sarlin, Paul-Edouard, et al. "Superglue: Learning feature matching with graph neural networks." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2020.

