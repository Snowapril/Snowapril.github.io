---
layout: post
title:  "OpenGL Postprocessing quad vertices input trick"
date:   2021-03-08 14:38:27 +0900
categories: opengl
tags: opengl post-processing quad trick glsl
comments: true  
---

`GPU-Zen : Adanced Rendering Techniques`의 Post-processing의 sample code들을 보고있는데 신선한 충격을 받은 대목이 있다.

Post-processing을 구현하는 일반적인 루틴은, framebuffer와 color attachment를 이용해 texture에 main frame을 그려내고
해당 color attachment를 texture로 binding하는데 one-to-one mapping이기 때문에 아래와 같은 형태로 fragment shader로 넘겨줘야한다.
![VS to FS](https://snowapril.github.io/assets/img/post_img/2021-03-08-vs-to-fs.png)  

이를 위해서 triangle-strip으로 4개의 position과 texcoord를 담은 vertices를 넘겨주거나, position input을 이용해 texcoord를 계산하는 것 까지는 normal한 approach라고 생각된다. 

더 나아가 gl_VertexID와 gl_FragCoord라는 내장변수를 이용해 vertices를 아예 upload하지 않고 계산하는 고수의 방법도 있다.

하지만 GPU-Zen의 sample 코드에서는 이런 상식적인 일들에서 벗어나 vertex shader에서 내보낸 output variables들이 fragment shader의 각 fragment들에서 보간된다는 점을 이용한다.
![VS to FS interpolation](https://snowapril.github.io/assets/img/post_img/2021-03-08-vs-to-fs-interpolated.png)  

4개의 position vertices를 넘기는게 아닌 3개의 position vertices를 넘기며, 그림과 같이 interpolation 되므로 fragment shader에서는
다른 방법들과 똑같다.

---
### Reference
1. `GPU-Zen : Adanced Rendering Techniques`