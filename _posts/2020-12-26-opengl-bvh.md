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

![CubbyFlow](https://github.com/utilforever/cubbyflow)에서 어떻게 하면 저 mesh 덩어리를
real-time에 진짜 물처럼 실감나게 만들 수 있을까를 고민하고 있었는데, 아래의 요소들이 구현되어야 한다.

1. 물 수면에서 수면 아래 바닥과 주변 environment map이 비쳐보일 것.
2. 아래 그림처럼 `Caustic` 효과가 나타날 것
![Water caustic](https://miro.medium.com/max/578/1*mLV4Jxoe56pROvQCR8DzGQ.png)
3. 물은 투명하므로 environment map과 근처 scene object들이 투과되어 보일 것.

위와 같은 효과들을 mesh shader로도 구현이 가능하겠지만, 구글링을 하던 중 `Path Tracer`로 구현한 결과물을 보니 
`Path Tracer`를 구현해보기로 마음을 굳혔다.
![Path tracing water](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F68a9ee3b-550c-41c7-90aa-589bd76c73de%2FUntitled.png?table=block&id=4d3b0cf1-8e4a-4a13-806a-756d4586ddb6&width=3070&userId=68fbb75e-8f7a-4af4-ba3b-0eb97516ef61&cache=v2)

`Path Tacer`를 real-time에 돌리기 위해서는 acceleration structure가 필수 불가결하다고 판단하여 BVH 구조를 찾아 공부했다.


---
### reference
1. ![https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F68a9ee3b-550c-41c7-90aa-589bd76c73de%2FUntitled.png?table=block&id=4d3b0cf1-8e4a-4a13-806a-756d4586ddb6&width=3070&userId=68fbb75e-8f7a-4af4-ba3b-0eb97516ef61&cache=v2](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F68a9ee3b-550c-41c7-90aa-589bd76c73de%2FUntitled.png?table=block&id=4d3b0cf1-8e4a-4a13-806a-756d4586ddb6&width=3070&userId=68fbb75e-8f7a-4af4-ba3b-0eb97516ef61&cache=v2)
2. ![https://www.scratchapixel.com/lessons/advanced-rendering/introduction-acceleration-structure/introduction](https://www.scratchapixel.com/lessons/advanced-rendering/introduction-acceleration-structure/introduction)