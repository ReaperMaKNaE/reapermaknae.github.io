---
layout : post
title : "On the Continuity of Rotation Representations in Neural Networks"
date : 2021-02-15 +1200
description : On the Continuity of Rotation Representations in Neural Networks 논문의 간단한 리뷰입니다.
tag : [PaperReview]
---

### On the Continuity of Rotation Representations in Neural Networks, CVPR 2019



__Summary__

 일단 시작하기전에, Topology(위상학)에 대한 내용이 정말 많이 나온다. 용어는 대부분 영어로 적긴 했으나, 몇몇 topology 관련된 내용들은 아래에 도움이 될만한 link를 해당 문단 아래에 달아 두었다. 하지만 이러한 내용을 일일이 읽으면서 찾기는 힘드니, 아래 link를 한번씩 읽고 난 다음 논문을 접하는 쪽이 훨씬 편하다. ~~더럽게 어렵다~~

Homeomorphism, wikipedia, [https://en.wikipedia.org/wiki/Homeomorphism

Quotient space, 리브레위키, [https://librewiki.net/wiki/%EB%AA%AB%EA%B3%B5%EA%B0%84](https://librewiki.net/wiki/%EB%AA%AB%EA%B3%B5%EA%B0%84)

Hausdorff space, wikipedia, [https://en.wikipedia.org/wiki/Hausdorff_space](https://en.wikipedia.org/wiki/Hausdorff_space)

Embedding, stackexchange, [https://math.stackexchange.com/questions/2996272/slice-chart-condition-proof-topological-embedding](https://math.stackexchange.com/questions/2996272/slice-chart-condition-proof-topological-embedding) - 이건 누군가가 그냥 질문글로 올린건데, topology 책에서 embedding에 대한 부분을 긁어와서 모르는 걸 그대로 다 올리는 바람에 대충의 설명이 들어가있다.

limitation of quaternion, gamedev.net, [https://gamedev.net/forums/topic/378542-quaternion-limitations/3498917/](https://gamedev.net/forums/topic/378542-quaternion-limitations/3498917/) - 해당 내용은 quaternion의 한계에 대해 설명하고 있다. 대충 quaternion의 경우 모든 rotation에 대한 표현이 가능하다고 나와있지만, 한바퀴를 돌리게 되면 결국 같아지는 것은 똑같다. 그래서 내가 몇바퀴를 돌렸는지를 알 수가 없기 때문에 continuous하지 않다고 하는 것을 말하는 것 같다.

Gram-Schmidt process

Stiefel manifold

수학에서 backslash의 의미, wolfram, [https://mathworld.wolfram.com/Backslash.html](https://mathworld.wolfram.com/Backslash.html) - 여기서는 set difference를 의미하는 것 같다. 예를 들자면, $A \backslash B = \{ x : x \in A \;and \;x \notin B\}$ 를 의미한다. 물론 맞는지는 나도 모른다.

stereographically projection, wikipedia, [https://en.wikipedia.org/wiki/Stereographic_projection](https://en.wikipedia.org/wiki/Stereographic_projection)



 여튼 모르겠는거 천지라서 수학에 자신이 없다면 결과표만 보고 "아, 5D와 6D representation이 더 좋구나"를 이해하면 될 것 같다.(neural network에서 학습시키기에)

 결국 6D representation에서 우리가 원하는 rotation을 뽑아내는 것은, DISN에 나와있어서 그 표현을 봐야할 것 같다.



### Abstract

 NN에서, 같은 공간의 표현을 다양하게 표현하는 작업은 가치가 있다. 예를 들어, 3D rotation은 quaternion이나 Euler로 나타내어 질 수 있다. 이 논문에서 우리는 DNN에서 조금 더 효과적으로 학습이 될 수 있는 continuous representation을 말한다. 우리는 homeomorphism(위상 동형)과 embedding(위상 공간 내에서의 표현)과 같은 topological concept와 연관지었다. 그리고 우리는 2D, 3D, n-dimensional rotation에서 어떤 것이 continuous하고 어떤 것이 discontinuous한 지에 대해서 조사한다. 3D rotation의 경우에는 4차원 표현이나 그 아래 차원의 모든 표현이 real Euclidean space가 discontinuous한 것을 설명한다. 그러므로, 현재 널리 사용되고 있는 quaternion이나 euler angle이 NN에서 학습하기엔 discontinuous하고 어렵다. 우리는 3D rotation이 5D나 6D에서는 continuous하다는 것을 보여준다.(이러한 방법은 학습에 더욱 효과적이다.) 그리고 우리는 $SO(n)$ group에서 n dimensional rotation 표현을 generalize한 expression을 보여준다. 우리의 main focus는 rotation이기 때문에, orthogonal group이나 비슷하게 변형된 다른 group에도 적용이 가능하다는 것을 보여준다. 마지막으로 실증적인 결과를 보여주어, 우리의 continuous rotation representation이 몇몇 graphics와 vision에서 문제되어 왔던 것들에서 효과적임을 보여준다.(simple auto-encoder sanity test, 3D point cloud의 rotation estimator, 3D human pose의 inverse kinematic solver 등등)



### 1. Introduction

 최근, 3D 연구가 많이 이루어지고 있다. 이러한 연구들에서 중심이 되는 것으로는 3D, 혹은 4D representation등을 기반으로 rotation을 측정하는 방법이 있다.(quaternion, axis-angle, euler angle 등)

 하지만, 3D rotation에서, 위와 같은 rotation representation들이 문제를 발생시킨다는 것을 발견했다. 잘 훈련된 network더라도 어떤 한 각도에 대해서는 큰 오차를 output으로 계속 내뱉었다. 이는 모든 model에 해당하는 것이 아닌, full rotation space가 필요한 경우를 말한다.(특히 mesh쌓거나 하는 등의 경우를 말하는 듯 하다) 우리는 이러한 문제가 discontinuity에서 온다는 것이라고 생각을 하게 되었다.

 이를 바탕으로 우리는 section 3에서 NN에서의 continuity representation에 대한 정의를 소개하고, 간단한 2D rotation을 소개하여 이 정의를 보여준다. 그리고 이러한 개념을 homeomorphism이나 embedding과 같은 위상학적인 개념의 키에 연결하게 된다.

 section 4에서는 rotation representation의 continuity를 이론적으로 분석한 것을 보여준다.

 마지막으로 section 5에서는 우리의 idea를 test하게 된다.

 또한 우리는 3 by 3 rotation matrix로 바로 regression을 하게 되는 경우, 우리가 제안한 6D rotation representation보다 오차가 크다는 것을 확인하게 된다. 추가적으로, inverse 혹은 forward kinematics와 같은 application을 위해, orthogonal matrix를 만드는 것이 network에서 중요할 수 있다. 그래서 우리는 network에서 orthogonalization procedure를 요구한다. 특히, 우리가 Gram-Schmidt orthogonalization을 사용하는 경우, 6D representation에 딱 맞게 되었다.



### 2. Related Work

__Neural network approximation theory.__

__Continuity for rotations.__

__Neural networks for 3D shape pose estimation.__

 posenet 같은 경우도 90도와 180도 사이만 잘되고, 그 밖은 힘듬.

__Neural networks for inverse kinematics.__



### 3. Definition of Continuous Representation

__Terminology.__

 matrix를 표현할 때에는 $M$ 이라 표현하고, $(i,j)$ 에 위치한 entry는 $M_{ij}$ 로 표현한다. $SO(n)$은 special orthogonal group에 n dimensional rotation의 공간이라고 하자. 이 group은 $n\times n$ real matrices set과 $MM^T = M^T M = I$ , $det(M) = 1$의 관계를 가진다. group operation은 multiplication이며, rotation의 합으로 표현된다. 우리는 n dimensional unit sphere를 $S^n = \{ x \in \mathbb{R}^{n+1} : ||x|| = 1 \} $ 로 표현한다.

__Motivating example: 2D rotations.__

 우리는 이제 2D rotation의 표현으로 접어든다. $M \in SO(2)$ 에 속하는 어떤 2D rotation이더라도 아래와 같이 표현이 가능하다.
$$
M =  \begin{bmatrix} cos(\theta) & -sin(\theta) \\ sin(\theta) & cos(\theta)
\end{bmatrix}
$$
 우리는 $R = [0, 2\pi]$ 내에 존재하는 경우, $ \theta \in R$ 을 선택함으로써, $M \in SO(2) $ 로 표현할 수 있다. 하지만 이러한 특정 representation은 직관적으로 continuity에 문제가 있게 된다. 만약 우리가 original space $SO(2)$에서 angular representation space $R$로 mapping하는 $g$를 정의한다면, 이러한 mapping은 discontinuous하다. 특히, $g$의 한계는 identity matrix로, zero rotation인 경우 undefined가 된다. 이럴 경우 rotation이 0인지 $2\pi$인지 알 수 없기 때문이다. 이러한 현상이 아래 그림과 같게 된다.(어떤 물체가 0근처에서 회전을 하게 되는 경우, 이를 mapping하는 결과는 Figure 1의 좌측과 같아 지는데, 한 물체가 continuous한 게 아니라 나뉘어져 있는 것 때문에 discontinuous하다는 것을 말하는 것 같다. 이러한 것이 결국 mapping g가 discontinuous하다는 것을 의미함)

![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210215-1.PNG)

 따라서, 우리는 이러한 형태(위에서 예로 든 경우 mapping  $g$)가 nueral network에서 학습되기 힘든 구조임을 말한다. 대조적으로, 만약에, 2D rotation을 $M$의 first column인 $[cos(\theta ), sin(\theta )]^ T$로 표현한다면, 이는 continuous가 된다.

__Continuous representation:__

 이제 우리는 무엇이 continuous한 representation인지 정의한다. 우리는 우리의 표현을 아래 Figure 2와 같이 나타낸다.

 ![img2](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210215-2.PNG)

 $R$을 Euclidean topology의 real vector space의 subset이라 하면, 이 $R$을 representation space라고 하자.  우리가 만든 network은 $R$에서 intermediate representation을 의미한다. 이 neural network은 figure 2에서 왼쪽에 있다. 우린 이 neural network에 나중에 짧게 다시 언급할 것이다. $X$를 topological space라고 하자. 그러면 우리는 $X$를 original space라고 할 수 있다. 우리가 만들어낸 representation space $R$은 original space $X$로 mapping이 가능하다. 여기서, mapping function $f$와 $g$를 각각 $f: R \to X$ , $g: X \to R $ 으로의 mapping이라고 하자. 그러면 우리는 $(f, g)$ 를 말할 수 있게 되고, $f$는 $g$의 inverse가 된다. ($x \in X, f(g(x)) = x$)

__Connection with neural networks:__

 다시 figure 2의 왼쪽 neural network로 돌아가보자. 그리고, 학습이 왼쪽에서 오른쪽으로 진행이 된다는 것을 상상해보자. 그러면 neural network는 왼쪽에서 input signal을 받고, 오른쪽으로 $R$에서의 representation을 output으로 내뱉는다. 그리고 이는 mapping $f$를 이용하여, original space $X$에 위치하는 element를 얻게 된다. 여기서 mapping $f$는 training이건 inference time이건 forward pass의 부분으로 사용되는 mathematical function이다. 전형적으로 training time동안은 우린 original space $X$ 에서의 오차를 loss로 사용한다.

 이제 우리는 왜 mapping $g$가 continuous한지 알아볼 시간이다. Figure 1의 우측 그림과 같은, original space $X$ 에서 연결된 set $C$ 가 있다고 가정해보자. 만약 우리가 $C$를 representation space인 $R$로 mapping을 하고, 이 때 $g$가 continuous하다고 가정하면, set $g(C)$는 여전히 연결된 상태로 남아있게 된다. 그러므로, 만약 우리가 continuous training data를 가지고 있다면, 이는 효과적으로 continuous training signal을 만들게 된다. 반대로, 만약 $g$가 Figure 1의 좌측처럼 continuous하지 않다면, connected set은 representation space에서 끊어지게 될 것이다. 이것은 network의 training signal을 discontinuous하게 만든다. 우리는 Euclidean topology space에서, neural network의 unit들이 전형적으로 continuous하다는 것을 상기해보자. 그러면 우리는 network unit들이 continuity를 가지기 때문에, Euclidean topology에서 representation space R이 필요하다는 것을 알게 된다.

(이게 제대로 이해한 건지 모르겠는데, Euclidean topology space에서는 neural network의 unit들이 continuous하다고 한다. 왜냐하면 그래야 학습이 잘 되기 때문. 결국 얘들이 continuous한지는 알 수가 없는데, 아무튼 continuous하다고 가정하고 mapping $f$와 $g$를 찾으면 언젠간 나오겠지 하는 마인드로 찾은 것 같다.)

__Domain of the mapping $f$:__

 추가적으로, neural network에서, neural network의 output이 lie되는 경우(??) set의 거의 모든 부분이 정의되는 것은 분명히 mapping $f$에 있어서 이윤이 있다. 이는 $f$가 original space $X$로 back project되는 임의의 representation으로 map이 가능하게 한다. (? 전체적으로 문장의 해석이...)

__Connection with topology:__

 $(f,g)$가 continuous representation이라고 하자. $g$는 compact topological space에서 Hausdorff space로의 continuous one-to-one function이다. 만약 우리가 $g$의 codomain을 $g(X)$로 두면, mapping의 결과는 homeomorphism이다. homeomorphism은 continuous bijection이고 continous inverse이다. geometric한 직감으로 봤을 때, homeomorphism은 자주 continuous하고, invertible stretching하며, 한 space에서 다른 space들로의 bending으로 설명이 된다.(같은 point들로 project되는 set들의 수...?) 여하튼, 두 가지의 space가 topologically equivalent한다면, 이는 homeomorphism이 그들 사이에 존재한다고 할 수 있다. 추가적으로, $g$는 original space $X$ 에서 representation space $R$로 가는 topological embedding이다. 우리는 여기서 $g$의 inverse를 $f$로 정의했음을 기억하자. 표현은 대충 $f|_{g(X)}$ 이다. 반면에, original space X가 representation space $R$의 그 어떤 면과도 homeomorphic 하지 않다면, $(f,g)$의 표현은 continuous하지 않게 된다. 우리는 이후에 4차원이나 그 이하의 차원으로 설명하는 rotation matrix가 continuous한 representation이 없다는 것을 잠시 후에 설명할 계획이다.



### 4. Rotation Representation Analysis

__4.1. Discontinuous Representations__

__Case 1: Euler angle representation for the 3D rotations.__

 3D rotation의 표현으로, original space가 $X = SO(3)$이라고 해보자. 그러면 우리는 Euler angle이 쉽게 discontinuity하다는 것을 알 수 있다.(이후에 뭔갈 설명하는데 짐벌락을 설명하는 것 같음)

__Case 2: Quaternion representation for the 3D rotations.__

 original space $X=SO(3)$을 가정하고, representation space $Y=\mathbb{R}^4$ 라고 하자.(quaternion을 표시하기 위해) 그러면 우리는, representation space $g_q(M)$을 다음과 같이 define할 수 있다.
$$
g_q(M)=\begin{cases}[M_{32} - M_{23}, M_{13}-M_{31}, M_{21}-M_{12}, t]^T \qquad if \quad t \neq 0 \\ [\sqrt{M_{11}+1}, c_2 \sqrt{M_{22}+1}, c_3 \sqrt{M_{33}+1}, 0 ]^T \qquad if \quad t = 0 \end{cases} \qquad (2)
$$

$$
t=Tr(M) +1, \; c_i = \begin{cases} 1 \qquad if \quad M_{i,1} + M_{i,2}>0 \\ -1 \qquad otherwise \end{cases} \qquad (3)
$$

 유사하게, original space $SO(3)$ 로의 mapping은 다음과 같다.
$$
f_q([x_0, y_0, z_0, w_0])=\begin{bmatrix} 1-2y^2 -2z^2 & 2xy-2zw & 2xz+2yw\\2xy+2zw & 1-2x^2 -2z^2 & 2yz - 2xz \\ 2xz - 2yw & 2yz+2xw & 1-2x^2 - 2y^2  \end{bmatrix}
$$

$$
where \qquad (x,y,z,w) = N([x_0, y_0, z_0, w_0]) \qquad (4)
$$

 normalization function은 $N(q) = \frac{q}{||q||}$ 로 된다. matrix $M$에 대해서 axis-angle representation의 개념으로 확장한다면, 모든 $M \in SO(3)$에 대해서, $f(g(M)) = M$이다.

 하지만, 우리는 representation이 continuous하지 않다는 것을 찾았다. Geometrically, 이것은 $R_{\pi}= \{M \in SO(3): Tr(M) = -1 \}$로 정의된 180도 내에서만 유효하다. 특히, Equation 2에 의하면, $t \neq 0$일 때, $g_q$의 한계는 180도, 즉 $[0,0,0,0]$ 인 경우에 생기게 된다. $Tr(M)$이 -1에 가까워져서 $R_{\pi}$가 180도에 가까운 위치에 해당하게 되면 $g_q$는 $[0,0,0,0]$을 가리켜야하는데, 문제는 이 때 $g_q$가 $[0,0,0,0]$이 아닌 다른 값을 가지게 된다.(즉, $[x,y,z,w] \neq [0,0,0,0] \quad where \quad t \sim 0$ )

우리의 정의대로라면, continuous representation의 정의는 representation space $Y$ 에 대해서 Euclidean topology를 필요로 한다는 것이었다. 이는 우리가 다음 문단에서 설명할 real projective space $\mathbb{R}P^3$ 의 quaternion을 이용한 topology와는 반대되는 것이다. 비슷한 방법으로, 우리는 axis-angle 방법 역시 180도 근처에서 문제를 일으킨다는 것을 확인할 수 있다.

__Representations for the 3D rotations are discontinuous in four or fewer dimensions.__

(이 문단은 솔직히 모르면 그냥 아~ 그렇구나 하고 넘어가는게 좋다. 왜냐면 나도 이해를 잘 못하겠다.)

 3D rotation group $SO(3)$는 real projective space $\mathbb{R} P^3 $와 homeomorphic이다. space $\mathbb{R} P^n$이 $\lambda \neq 0$ 을 만족하는 $x \sim \lambda x$의 관계를 가진 $\mathbb{R} ^{n+1}$ 의 quotient space라고 하자. 그럼, 직관적으로 space $\mathbb{R}P^3$가 quotient space를 통한 적절한 topology를 이용한다면 $\mathbb{R}^4$에서 homogeneous coordinate이 될 수 있다고 할 수 있다. 

 아래 논문에 따르면, 우리는 $\mathbb{R}P^3$(혹은 $SO(3)$)이 Euclidean topology에서는 $\mathbb{R}^5$ 안에서 embed가 가능하지만, $\mathbb{R}^d$ 에서 $d<5$인 경우에는 embed하지 않다는 것을 나타내고 있다.

 D. M. Davis. Embeddings of real projective spaces. Bol. Soc. Mat. Mexicana (3), 4:115–122, 1998.

 embedding의 정의에 따르면, $SO(3)$에서는 $d<5$인 $\mathbb{R}^d$의 그 어떤 subset에도 homeomorphism이 없다는 것을 알 수 있다. 하지만 continuous representation은 이러한 것들을 필요로 한다. 그러므로, 우리는 5차원 미만의 표현(quaternion을 포함 - quaternion이 4차원 표현이기 때문) 으로는 continuous representation을 찾을 수 없다는 것을 알 수 있다.

(쉽게 말해, 이 문단을 제대로 이해하고 싶으면 가운데에 적힌 저 1998년에 쓰인 논문을 읽고 와야한다.)

__4.2. Continuous Representations__

 이번 section에서 우리는 n dimensional rotations인 $SO(n)$에서 2가지의 continuous representation을 발전시킨다. 그리고 어떻게 3D rotation이 6D 혹은 5D continuous rotation representation으로 나타날 수 있는지 설명한다.

__Case 3: Continuous representation with $n^2 -n $ dimensions for the n dimensional rotations.__

 이전까지 다루었던 representation들은 전부 not-continuous하다.(분명 어딘가에서 half-sphere만 적용이 된다고 하여 quaternion을 사용하더라도 반만 썼다는 것을 봤다. -> PoseNet이다! $q$가 $-q$와 같다고 말하는데, 이게 위에서 설명한 그것과 같은지를 모르겠다. loss 계산하면 같아져서 그런가? 나중에 선배님께 물어보기)

 여하튼 continuous한 rotation representation은 identity mapping인데, 실제 세상에서 size $n \times n$ 의 경우, orthogonalization을 필요로 할 수 있다.(Gram-Schmidt process와 같은) 이러한 것을 바탕으로, representation에 orthogonalization을 수행하는 것을 목적으로 했다. original space $X=SO(n)$ 와, representation space $R=\mathbb{R}^{n\times(n-1)} \backslash D$ ($D$는 추후 짧게 설명 예정 - 즉, 6D rotation에 속하면서 임의의 representation인 $D$에는 속하는 $R$을 말하는 것 같음. 이걸 왜 굳이 나누는지는 아래에서 증명을 하면서, 이래서 나눴다! 라고 하는데, 무슨 말인지 이해를 못해서 그냥 넘어갔다.) 를 가정하자. 

 그러면 우리는 mapping $g_{GS}$를 마지막 column을 떨쳐내는 simple한 형태인, 아래와 같이 표현할 수 있다.
$$
g_{GS} \begin{pmatrix} \begin{bmatrix} | & & | \\ a_1 & ... & a_n  \\ | & & |\end{bmatrix}\end{pmatrix} = \begin{bmatrix} | & & | \\ a_1 & ... & a_{n-1} \\ | & & | \end{bmatrix} \qquad (5)
$$
 위에서 $a_i$ ( $i = 1, 2, ..., n$ )은 column vector들을 의미한다. 위 식에서 $g_{GS}$는 Stiefel manifold를 의미한다.

 여기서 $f_{GS}$ 를 이용해 original space로 옮기게 되면, 우리는 아래와 같은 Gram-Schmidt처럼 생긴 process를 밟게 된다.
$$
f_{GS} \begin{pmatrix} \begin{bmatrix} | & & | \\ a_1 & ... & a_{n-1} \\ | & & |  \end{bmatrix} \end{pmatrix} = \begin{bmatrix} | & & | \\ b_1 & ... & b_n \\ | & & | \end{bmatrix} \qquad (6)
$$
 가 되고, 여기서 $b_i(i = 1, 2, 3, ... n)$는,
$$
b_i=\begin{bmatrix} \begin{cases} N(a_1) \qquad \qquad \qquad \qquad \quad \; if \quad i = 1\\N(a_i - \sum_{j=1}^{i-1}(b_j \cdot a_i)b_j) \qquad if \quad 2 \le i <n \\ det \begin{bmatrix}| & & | & e_1 \\ b_1 & ... & b_{n-1} & \vdots \\ | & & | & e_n \end{bmatrix} \quad if \quad i=n \end{cases} \end{bmatrix} \qquad (7)
$$
 여기서 $N(\cdot)$은 normalization function을 말하고 이전과 같다. $e_1, ..., e_n$은 Euclidean space의 n canonical basis vector라고 볼 수 있다. 보통의 Gram-Schmidt process와 $f_{GS}$의 다른 점은, 마지막 column이 n 차원에서 cross product의 일반화로 계산된 것이다. 그래서 분명하게 말하자면, $g_{GS}$는 continuous하다. 모든 matrix $M \in SO(n)$에 대해서 확인하기 위해($f_{GS}(g_{GS}(M)) = M$ 이 성립하는지 확인하기 위해) 우리는 Gram-Schmidt process가 첫 n-1 성분의 column을 고치지 않는 것을 보여주기 위해 matrix $M$의 column들의 orthonormal basis vector들의 성질과 귀납법을 사용할 수 있다. __(이거 논문 쓴 사람이 fundamental한 수학의 엄청난 고인물인 것 같다. 뭐 어떻게 했는데? 라고 묻고 싶은데, 답변을 듣는다고 내가 이해할 만한 것 같지는 않다. 대충 그렇대 하고 넘기자.)__ 최근, 우리는 Bloom이 쓴 *Linear algebra and geometry* 의 Theorem 5.14.7(?? 일단 도서관 책엔 없어서 pass)과 같은 것들을 이용해서 우리는 generalized cross product를 위한 theorem을 사용할 수 있다.

 여튼 결과적으로.... 잘 모르겠다. 하... 뭐지?

__6D representation for the 3D rotations:__

 3D rotation에 대해서, Case 3는 우리에게 6D representation을 주었다.($n^2-n$에서 $n$에 3을 대입하면 ok) 이렇게 되면, 식 (7)은 단순히 $b_1 \times b_2 $로 간소화 되게 된다. 우리는 이걸 appendix에서 자세하게 설명할 계획이다.(근데 찾아봐도 appendix가 없다..?) 우리는 여기서 continuous하다는 것을 알 수 있는게, $b_1 \times b_2$의 결과가 3 by 3 matrix로 나오는 것이다. 하지만 대조적으로 우리는 3 by 3 matrix를 prediction하도록 하고, post process나 network 내부에서 orthogonalization될 수 있도록 한다. 만약 network에서 진행이 된다면, matrix의 마지막 3 components는 Gram-Schmidt process에 의해 사라지게 된다.(equation 7에서) 그래서 결과적으로 3 by 3 matrix가 남게 된다. 결과적으로 이 3 by 3 은 9개의 output을 의미하는데, 여기서 6D representation으로 쓰이는 것과 쓸모 없는 3개의 term들이 같이 생기게 된다. 만약 postprocess로 진행한다면, 이러한 현상은 forward kinematics와 같은 것으로 막을 수 있지만, error가 section 5에서 관찰할 수 있듯 더 커지게 된다.

__Group operations such as multiplication:__

 original space가 rotation group과 같은 group이라고 하고, 두 representation을 단순히 곱하고 싶다고 하자.($r_1, r_2 \in R$) 일반적으로, 우리는 original space로의 mapping을 처음에 시도할 수 있고, 두 요소들을 더한 다음, back mapping을 해서 구하게 된다.($r_1 r_2 = g(f(r_1)f(r_2))$) 하지만, 제안된 representation에 의하면, 우리는 다음과 같은 몇 computational efficiency를 얻을 수 있다. equation (5)에서 몇 column을 drop하기 때문에(버리기 때문에), $f(r_2)$를 계산할 때에는 단순히 마지막 column을 버리고 곱하기에 $n\times n$ matrix 연산이 $n \times (n-1) $ matrix 연산으로 변하게 된다.

__Case 4: Further reducing the dimensionality for the n dimensional rotations.__

 $n \ge 3 $ 인 경우에 대해서, 우리는 continuous representation을 유지하며 차원을 줄일 수 있다. 직관적으로, 적은 dimensional representation은 쓸모 없는 것들을 쳐내기에 학습하기에 조금 더 쉬울 수도 있다. 하지만 우리는 실험에서 차원을 줄이는 것이 Gram-Schmidt와 같은 representation보다 성능이 좋지 않다는 것을 알게 되었다. 하지만 우리는 여전히 이러한 representation을 발전시키려 하는데, 이는 discontinuous한 것들 보다는 성능이 좋기 때문이다.

 우리는 normalization과 합쳐진 한 가지 혹은 그 이상의 stereographic projections를 사용해서 dimension reduction을 수행할 수 있다. 우리는 이를 아래 그림으로 묘사했다.

![img3](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210215-3.PNG)

 우리가 처음 input point를 normalize한다고 하고, 이를 sphere에 project한다고 하자. (뭐 여튼 이렇게 하는게 stereographically projection 이라고 하나보다. 차원을 한 단계 낮추는 projection인 것을 말하는 것 같은데, 왜 저렇게 하는지는 나도 잘 모르겠다. 맨 윗 글의 위키피디아를 읽어도 똑같은 말을 한다.)

 아무튼 이러한 것을 normalized projection이라고 하고, $P : \mathbb{R}^m \rightarrow \mathbb{R}^{m-1} $이라고 둘 수 있다. 그리고 이는,
$$
P(u) = [ \frac{v_2}{1-v_1}, \frac{v_3}{1-v_1},..., \frac{v_m}{1-v_1}]^T, v=u/||u|| \qquad (8)
$$
 위와 같이 되고, 이와 반대로 stereographic un-projection을 아래와 같이 설정해보자.
$$
Q(u) = \frac{1}{||u||} \begin{bmatrix} \frac{1}{2} (||u||^2 -1 ), u_1, ... , u_{m-1}\end{bmatrix}^T \qquad (9)
$$
 un-projection은 back sphere 표현이 아님을 알아두어야 한다. (왜 이렇게 설정했는지 조차 모르겠다.) 그럼 우리는 이걸 1부터 n-2의 normalized projection을 하면서, continuity를 유지할 수 있다.(라고 하는데, 그냥 한차원 정도 낮추는 것 까지는 위상학적으로 봐줄 수 있기 때문에, 5D representation도 가능하다 라고 생각하고 넘어가는 것이 마음이 편할 것 같다.)

 좀 간단하게 설명하기 위해서, 우리는 stereographic projection의 한가지 경우로 설명할 것이다. (라고 설명은 하나, 아래 부분은 도저히 이해가 안된다. 아래 그림에서 뭐 설명을 시도하려고 하나본데, 내 머리로는 도저히 불가능하다. 그래서 pass)

![img4](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210215-4.PNG)

 __Other groups: O(n), similarity transforms, quaternions.__

 이번 논문에서 우리는 rotation의 표현에 대해 집중했다. 하지만, 우리는 이전의 표현들이 orthogonal group인 $O(n)$으로 일반화될 수 있다는 것을 기억해야한다. .... 이후 또 알 수 없는 내용들이 가득하다. 그래서 pass. 그냥 결과만 보자.



### 5. Empirical Results

__5.1. Sanity Test__

__5.2. Pose Estimation for 3D Point Clouds__

__5.3. Inverse Kinematics for Human Poses__

![img5](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20210215-5.PNG)

 위 그래프를 해석하자면, 좌측의 mean errors는 학습 도중에 error의 평균이 얼마나 줄어드는지를 말하는 것이고(x축이 시간에 대한 항 이라고 보면 된다), 우측의 percentile of errors는 어느 각도에서 error가 가장 일어나는지를 말한다.(x축이 모든 error에서 분포가 어떻게 되는지를 말함. 예를 들어, 5D와 6D representation을 제외한 나머지 representation들은 baseline 각도가 100도를 초과하는 경우 100%에 육박하는 에러를 나타낸다.)

 loss를 계산하는 방법은 상황에 따라 다른 loss를 적용했다고 한다. Sanity test의 경우는 geodesic error를 적용했고, 나머지 2가지 종류는 L2 loss를 이용했다고 한다.



### 6. Conclusion



### 7. Acknowledgements



### Reference

Zhou, Yi, et al. "On the continuity of rotation representations in neural networks." *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 2019.

Homeomorphism, wikipedia, [https://en.wikipedia.org/wiki/Homeomorphism

Quotient space, 리브레위키, [https://librewiki.net/wiki/%EB%AA%AB%EA%B3%B5%EA%B0%84](https://librewiki.net/wiki/%EB%AA%AB%EA%B3%B5%EA%B0%84)

Hausdorff space, wikipedia, [https://en.wikipedia.org/wiki/Hausdorff_space](https://en.wikipedia.org/wiki/Hausdorff_space)

Embedding, stackexchange, [https://math.stackexchange.com/questions/2996272/slice-chart-condition-proof-topological-embedding](https://math.stackexchange.com/questions/2996272/slice-chart-condition-proof-topological-embedding)

limitation of quaternion, gamedev.net, [https://gamedev.net/forums/topic/378542-quaternion-limitations/3498917/](https://gamedev.net/forums/topic/378542-quaternion-limitations/3498917/)

수학에서 backslash의 의미, wolfram, [https://mathworld.wolfram.com/Backslash.html](https://mathworld.wolfram.com/Backslash.html)

stereographically projection, wikipedia, [https://en.wikipedia.org/wiki/Stereographic_projection](https://en.wikipedia.org/wiki/Stereographic_projection)

 D. M. Davis. Embeddings of real projective spaces. Bol. Soc. Mat. Mexicana (3), 4:115–122, 1998.

