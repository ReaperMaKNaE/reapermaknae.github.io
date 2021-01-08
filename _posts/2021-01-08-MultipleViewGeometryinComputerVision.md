---
layout : post
title : "Multiple View Geometry in Computer Vision 요약"
date : 2021-01-08 +1040
description : 뮌헨공대에서 Daniel Cremers 교수님이 강의하신 Multiple View Geometry in Computer Vision 수업 강의자료를 받아 요약한 내용입니다.
tag : [ComputerVision]
---

### Multiple View Geometry in Computer Vision



 전체적인 내용을 정리하는 느낌이라 세세한 것들은 다루지 않았기에, 아래 내용을 슥 훑었을 때 "어? 이거 뭐더라?" 하면 다시 강의자료를 참고하는 것을 추천합니다.



### Chapter 1. Mathematical Background: Linear Algebra

__Vector Space__

Linear Dependency/Independency, Basis, Inner Product(Canonical and Kronocker), Cross Product, Stack of Matrix)

__Linear Transformations and Matrices__

기본적으로 'Groups'에 대한 내용. Affine Group, Orthogonal Group, Euclidean Group. SE(3)이나 SO(3) 등의 표현에 대해 이해가 가능.

__Properties of Matrices__

Range, Span, Null space, kernel, Rank, Eigenvalues/Eigenvectors, symmetric, Norm, skew-symmetric

__Singular Value Decomposition__

SVD라고도 하는데, 다음 블로그의 포스팅을 참조.(다크프로그래머님의 SVD 포스팅)

[https://darkpgmr.tistory.com/106](https://darkpgmr.tistory.com/106)

위 포스팅에 pseudo inverse에 대한 내용도 나오는데, 정말 개념을 잘 잡을 수 있으니, 조금 길더라도 정독하고 공부하는 것을 추천.



### Chapter 2. Representing a Moving Scene

__The Origins of 3D Reconstruction__

Euclidean motion or Rigid body Motion, Perspective projection

__3D Space & Rigid Body Motion__

translation과 rotation, twist, infinitesimal rotation

__The Lie Group SO(3)__

__The Lie Group SE(3)__

__Representing the Camera Motion__

__Euler Angles__

 위 내용들은 강의 자료를 참조 할 것. Exponential Map, Rodrigues' Formula 등에 대한 내용이 있음.



### Chapter 3. Perspective Projection

__Mathematical Representation__

projection의 표현이 어떤 것인지, SVD를 왜 쓰는지, Standard projection Matrix는 뭔지.

__Intrinsic Parameters__

Camera parameter에 대한 내용

__Spherical Perspective Projection__

강의 자료 참조

__Radial Distortion__

Camera의 focal length가 짧은 경우, distortion이 일어나는 것.

__Preimage and Coimage__

아래 발표자료 참고.

[https://slideplayer.com/slide/13567596/](https://slideplayer.com/slide/13567596/)

__Projective Geometry__



### Chapter 4: Estimating Point Correspondence

__From Photometry to Geometry__

__Small Deformation & Optical Flow__

__The Lucas-Kanade Method__

__Feature Point Extraction__

__Wide Baseline Matching__

여기서 NCC(Normalized Cross Correlation)의 개념이 나온다. 사실 지금은 별로 안쓰임.



### Chapter 5: Reconstruction From Two Views: Linear Algorithms

__The Reconstruction Problem__

Triangulation 사용, projection error를 줄이려는 시도

__Epipolar Constraint__

Essential Matrix의 등장.

__Eight-point Algorithm__

__Structure Reconstruction__

Homography 개념의 등장.

__Four Point Algorithm__

__The Uncalibrated Case__

Fundamental Matrix의 등장. 여튼 여태까지 쓴 방법으로는, 실제 3D Recon이 힘듬. noise가 심해서. projective recon정도가 한계수준.



### Chapter 6: Reconstruction from Multiple Views

__From Two Views to Multiple Views__

trifocal tensor등 이용.

__Preimage & Coimage from Multiple Views__

__From Preimages to Rank Constraints__

__Geometric Interpretation__

__The Multiple-view Matrix__

__Relation to Epipolar Constraints__

__Multiple-View Reconstruction Algorithms__

__Multiple-View Reconstruction of Lines__



### Chapter 7: Bundle Adjustment & Nonlinear Optimization

__Optimality in Noisy Real World Conditions__

__Bundle Adjustment__

__Nonlinear Optimization__

gradient descent, least squares estimation, newton methods(Hessian matrix 등장), the gauss-newton algorithm, the levenberg-marquadt algorithm



### Chapter 8: Direct Approach to Visual SLAM

__Direct Method__

real time dense geometry, dense RGB-D tracking, loop closure and global consistency, dense tracking and mapping, large scale direct monocular SLAM, direct sparse odometry(모두 강의하신 Cremers 교수님께서 참여함)



### Chapter 9: Variational Methods: A Short Intro

__Variational Method__

variational image smoothing, Euler-Lagrange Equation, gradient descent, adaptive smoothing



### Chapter 10: Variational Multiview Reconstruction

__Shape Representation and Optimization__

Explicit(control point, spline, etc), Implicit(indicator function, signed distance function, etc)

__Variational Multiview Reconstruction__

__Sparse Time Reconstruction from Multiview Video__

__Imposing Silhoutte Consistency__



### Reference

Multiple View Geometry In Computer Vision 강의자료, [https://vision.in.tum.de/teaching/online/mvg](https://vision.in.tum.de/teaching/online/mvg)

SVD, "선형대수학 #4 특이값 분해(Singular Value Decomposition, SVD)의 활용" [https://darkpgmr.tistory.com/106](https://darkpgmr.tistory.com/106)

Preimage and Coimage, "Multiple-View Geometry for Image-Based Modeling", [https://slideplayer.com/slide/13567596/](https://slideplayer.com/slide/13567596/)

