---
layout: post
title:  "OpenGL BVH(Bounding Volume Hierarchy)"
date:   2020-12-26 13:38:27 +0900
categories: opengl
tags: opengl path-tracer bvh acceleration-structure fluid
comments: true  
---
그동안 BVH에 대해서는 온갖 article들을 보다보니 인지는 하고 있었지만,
rasterization만 하다보니 정작 사용할 일은 없었다.

[CubbyFlow](https://github.com/utilforever/cubbyflow)에서 어떻게 하면 저 mesh 덩어리를
real-time에 진짜 물처럼 실감나게 만들 수 있을까를 고민하고 있었는데, 아래의 요소들이 구현되어야 한다.

1. 물 수면에서 수면 아래 바닥과 주변 environment map이 비쳐보일 것.
2. 아래 그림처럼 `Caustic` 효과가 나타날 것
![Water caustic](https://miro.medium.com/max/578/1*mLV4Jxoe56pROvQCR8DzGQ.png)
3. 물은 투명하므로 environment map과 근처 scene object들이 투과되어 보일 것.

위와 같은 효과들을 mesh shader로도 구현이 가능하겠지만, 구글링을 하던 중 `Path Tracer`로 구현한 결과물을 보니 
직접 구현해보기로 마음을 굳혔다.
![Path tracing water](https://pbs.twimg.com/media/B7xk6R7CcAAaYX_?format=jpg&name=small)

`Path Tacer`를 real-time에 돌리기 위해서는 acceleration structure가 필수 불가결하다고 판단하여 BVH 구조를 찾아 공부했다.

Ray tracing은 왜이렇게 느릴까? "Accelerated Ray Tracing" (1983) 후지모토의 말에 따르면, ray tracing method의 speed는 거의 전적으로
Scene complexity에 달려있다. 

> "The calculation speed of the ray-tracing method is undoubtedly one of the basic problems that must be dealt with. Why is raytracing so computationally expensive? The main cause was clearly identified at the very moment ray tracing first entered the field of computer graphics. According to Whitted, for simple scenes 75 percent of the total time is spent on calculating intersections between rays and objects. For more complex scenes the proportion goes as high as 95 percent. The time that must be spent calculating the intersections is directly related to the number of objects involved in the-scene".

위의 후지모토의 말을 인용하자면, 간단한 scene에서의 ray tracing은 전체 소요 시간의 75%가 ray와 object들 사이의 intersection test에 할애된다. 
더 복잡한 scene에서는 95% 까지도 차지한다. 그러므로, ray tracing의 속도를 개선하기 위해서는 intersection test의 수를 가능한 줄이는 것이 좋은데
이를 위해서 필요한게 `Acceleration Structure` 이다. 

teapot의 예를 들어 설명하자면, teapot의 32개의 `Bezier patches`를 각각 tight하게 감싸는 bounding box를 생성하여 teapot의 
모든 vertices에 대해 intersection tests 하는게 아닌, bounding box들에 대해서 먼저 intersection tests를 하고 hit 했을 경우에만 
ray와 교차한 bounding box에 포함된 vertices에 intersection tests를 한번 더 한다. 

![Teapot bounding volume](https://snowapril.github.io/assets/img/post_img/2020-12-26-bounding-volume.gif)

이런 간단한 아이디어에서 비롯한 결과물은 bounding volume을 도입하기 전과 후에 이론적으로 38배까지 속도 향상이 가능하다.

구현이 간단한만큼 단점도 존재하는데, 이와 같은 기법은 bounding volume을 추적하는게 object 자체를 ray-tracing 하는 것보다 빠를 경우에만 유효하다.
또한, loosely tight한 bounding-box으로 인해 intersecion test를 통과한 bounding-box 안에서 object에 대한 intersection test는 miss인 경우가 많다.
![Teapot bounding box tight](https://snowapril.github.io/assets/img/post_img/2020-12-26-bbox-tight.png)

이러한 단점들을 극복하기 위해 1986년 Kay와 Kajiya는 새로운 bounding-volume 생성 알고리즘을 고안했는데 그 내용은 아래와 같다.

![BVH Projection](https://snowapril.github.io/assets/img/post_img/2020-12-26-bvh-projection.png)
object의 vertices들 중 하나를 `P(x,y,z)` 라고 했을 때, normal vector가 (A, B, C)인 평면의 방정식 Ax + By + Cd + d = 0

$$ x = y^2 $$

---
### reference
1. [https://twitter.com/hb3p8/status/557431407350665218?s=20](https://twitter.com/hb3p8/status/557431407350665218?s=20)
2. [https://www.scratchapixel.com/lessons/advanced-rendering/introduction-acceleration-structure/introduction](https://www.scratchapixel.com/lessons/advanced-rendering/introduction-acceleration-structure/introduction)