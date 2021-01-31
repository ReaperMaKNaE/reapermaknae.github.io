---
layout : post
title : "Simplified Kruppa Algorithm"
date : 2021-01-25 +2200
description : Intrinsic Camera parameter를 estimating하는 Kruppa Algorithm 관련 포스팅입니다.
tag : [ComputerVision]
---

### Simplified Kruppa Algorithm



 시작에 앞서, 이 포스팅은 아래 논문과 사이트를 참조하였습니다.

Lourakis, Manolis IA, and Rachid Deriche. *Camera self-calibration using the Kruppa equations and the SVD of the fundamental matrix: The case of varying intrinsic parameters*. Diss. INRIA, 2000.

[http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/FUSIELLO3/node4.html](http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/FUSIELLO3/node4.html)



 일단 notation은 다음과 같다. M은 3D point를 의미하고, 이를 projection한 point는 m이라 하면,
$$
{s\widehat{m} = P\widehat{M}}
$$
 이 되고, 여기서 s는 nonzero scalar factor, P는 3x4 projection matrix이다. 만약 homography의 이미지 2장을 상상해보면, 아래와 같이 정리할 수 있다.
$$
s_1 \widehat{m_1} = P_1 \widehat{M} \quad where \quad P_1 = [A|0]
$$
$$
s_2 \widehat{m_2} = P_2 \widehat{M} \quad where \quad P_2 = [A'R|A't]
$$

 R과 t를 각각 Rotation, translation으로 한다고 하고, A는 camera parameter matrix를 의미하며, 그 의미는 아래와 같다.
$$
{A = \begin{bmatrix}
\alpha_u & -\alpha_u cot \theta & u_0\\ 
0 & \alpha_v/sin\theta & v_0\\ 
0 & 0 & 1
\end{bmatrix} \quad where \quad \alpha_u, \alpha_v = focal\; distance}
$$
$$
{A' = \begin{bmatrix}
\alpha'_u & -\alpha'_u cot \theta' & u'_0\\ 
0 & \alpha'_v/sin\theta' & v'_0\\ 
0 & 0 & 1
\end{bmatrix}\quad where \quad \theta = angle\; between\; the\;two\;images}
$$

그리고, aspect ratio라고 알려진 값은 아래와 같다.
$$
{\frac{\alpha_v}{\alpha_u}=aspect\;ratio}
$$
 또한 통상적으로, 일반적인 카메라에서 세타는 90도에 해당한다.

 위의 notation을 참조했을 때, Kruppa는 다음과 같은 notation을 추가했다.
$$
{if \; K = \begin{bmatrix}
K_1 & K_2 & K_3\\ 
K_2 & K_4 & K_5\\ 
K_3 & K_5 & 1
\end{bmatrix}, \; then\; A =\begin{bmatrix}
\sqrt{K_1 - K_3 ^2 - \frac{(K_2 - K_3 K_5 )^2}{K_4 - K_5 ^2}} & \frac{K_2 - K_3 K_5}{\sqrt{K_4 - K_5 ^2}}& K_3\\
0& \sqrt{K_4 - K_5^2}& K_5 \\
0&0 &1
\end{bmatrix}}
$$
 위 식에서 K는 Kruppa matrix를 말하고, A는 camera parameter matrix를 말한다.

 Kruppa는 위 식을 해결 할 방법을 제안했지만, 이러한 제안은 계속 개선이 되면서 현재는 Hartley에 의해 제안된 Simplified Kruppa equation이 사용되고 있다.

 이 방식은 Fundamental matrix에 SVD를 취한 꼴에서 K를 구하는 방법으로, 아래와 같다.
$$
{F = UDV^T \quad \quad where \quad U = \begin{bmatrix}
u_1^T\\
u_2^T\\
u_3^T
\end{bmatrix}\quad 
V =\begin{bmatrix}
v_1^T\\
v_2^T\\
v_3^T
\end{bmatrix} \quad D = diag(r, s, 0)}
$$
 위 식에서 K를 어떻게 쿵짝쿵짝하면(자세한건 논문 참조), 아래와 같은 식이 마법같이 튀어나온다.
$$
{\frac{r^2 v_1^TKv_1 }{u_2^TK'u_2}=-\frac{rsv_1^TKv_2}{u_2^TK'u_1}=\frac{s^2 v_2^T K v_2}{u_1^T K' u_1}}
$$
 K 와 $K'$은 한 쌍의 image를 촬영한 camera가 서로 다르다고 가정했을 경우, 각 camera에 해당하는 kruppa matrix이다. 그런데 위 식을 풀려면, 무언가 아다리가 안맞다. 미지수는 5개이고 식은 3개인데, unique solution을 어떻게 찾을 수 있는가?

 일단 constraint를 걸게 된다. 두 이미지를 촬영한 카메라는 같다고 가정하고, 그 카메라의 kruppa matrix를 K로 통일한 후, aspect ratio를 1이라고 가정하며, focal length의 x, y가 같고 skew part(K2)가 0이라고 한다면, kruppa matrix는 단 2개의 미지수만 남게 된다.
$$
{A = \begin{bmatrix}
\sqrt{K_1 - K_3 ^2}&0&K_3\\
0&\sqrt{K_1 - K_3^2 }&K_3\\
0&0&1
\end{bmatrix}\quad \because K_3 = K_5, \;K_1 = K_4, \; K_2 = K_3 K_5}
$$
 위 식을 바탕으로, 
$$
{K= \begin{bmatrix}
K_1&K_3^2 & K_3\\
K_3^2&K_1 &K_3\\
K_3&K_3&1
\end{bmatrix}}
$$
 위 값으로 initial value를 잡고, 여기서 LM을 쓰던 다른 iteration방법을 쓰던, cost를 어떻게든 도입해서 최적화되는 값을 찾아가는 방향이다. 여러 paper들이 이를 이용해서 camera parameter를 측정했는데, 그 정확도가 꽤나 괜찮은 편이었던 것으로 기억한다.



### Reference

Lourakis, Manolis IA, and Rachid Deriche. *Camera self-calibration using the Kruppa equations and the SVD of the fundamental matrix: The case of varying intrinsic parameters*. Diss. INRIA, 2000.

[http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/FUSIELLO3/node4.html](http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/FUSIELLO3/node4.html)