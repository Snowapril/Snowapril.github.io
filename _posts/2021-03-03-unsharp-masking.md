---
layout: post
title:  "OpenGL Unsharp Masking in framgment shader"
date:   2021-03-03 14:38:27 +0900
categories: real-time-rendering
tags: opengl image-processing glsl unsharp-mask filter
comments: true  
---

`GPU-Zen : Adanced Rendering Techniques`의 챕터1을 구현해보고자 책에서 Github에 공유한 코드를 보는데 다른 부분들은
다 이해가 가는데 Post-processing에서 Unsharpen은 제대로 알지를 못했다.

```c++
void calculateAO(
    const in vec2 tc,
    const in float threshold,
    const in float aoCap,
    const in float aoMultiplier,
    const in float depth,
    const in sampler2D depthTexture,
    inout float ao,
    inout int pixelsCaculated)
{
    float d = depthAt(tc, depthTexture);

    if (abs(d-depth) < threshold)
    {
        // map to AO amount
        ao += min(aoCap, max(0.0, depth - d) * aoMultiplier);
        // propagate to sum
        ++pixelsCaculated;
    }
}

float unsharpmaskingValue(
    const in vec2 texCoord,
    const in sampler2D depthTexture)
{
    //------------------생략------------------
    float depth = depthAt(texCoord, depthTexture);

    int iterations = 6;
    int kernelSize = 16;

    if (depth != 1.0) {
        for (int i = 0; i < iterations; i++) {

            // Iterate over kernel
            for (int j = 0; j < kernelSize; j++) {
                calculateAO(
                    texCoord + poisson[j] * t,
                    threshold,
                    aoCap,
                    aoMultiplier,
                    depth,
                    depthTexture,
                    ao,
                    pixelsCaculated);
            }

            t *= 2.0;
            aoMultiplier *= 1.5;
        }

        ao /= float(pixelsCaculated);
    }

    return 1.15 - ao;
}
```

코드만 보면 대충, 주어진 texture coordinate의 주변 pixel들의 AO를 계산해서 depth의 변화가 있는 부분을 강조하는 듯한 느낌이다.

---

Unsharp Mask란 이름과 반대로 이미지를 sharpening 할 때 사용하는 기법이다. texture 와 detail을 강조하고 싶을 때 사용하는 것이 sharpening으로, Unsharp Mask는 흔히 사용되는 sharpening의 기법이다.

Unsharp mask는 크게 두가지 step으로 나뉜다. 
### 1. Create Unsharp Mask
주어진 Texture의 blurred된 버전을 생성하여 원본에서 빼주게 되면 Edge detection이 가능하다. 
아무래도 edge 근처에서 pixel color의 값 변화가 급격하게 일어나므로 가능한 것 같다.
### 2. High Contrast Original Image 와 Unsharp Mask 그리고 Original Image를 더해준다.
![Unsharp Mask](https://snowapril.github.io/assets/img/post_img/unsharp_mask.png)  
`STEP2`는 higher 3개의 이미지를 mask-overlay를 이용해 합친다.

Texture의 resolution은 변하지 않는데 어떻게 final image가 sharpening 된걸까?
![Unsharp Mask Graph](https://snowapril.github.io/assets/img/post_img/unsharp_mask_graph.png)  
Edge를 Ideal step으로 transformation 한게 아니라, transition의 light 부분과 dark 부분을 과장하여 
표현함으로서 acutance(첨예도?)를 높여 sharpening 한다.

왜 이런 Light의 overshoot과 Dark의 undershoot이 sharpness를 높이는데 도움이 될까? 
Unsharp Mask는 사람의 Visual system에서 일어나는 `Trick`을 이용한다.
![Mach band](https://cdn.cambridgeincolour.com/images/tutorials/usm_gradient2.png)
사람의 시각 시스템은 위와 같이 sharp의 변화의 Edge에서 `Mach bands`가 보이는데, 
이게 edge에서의 detail을 분간하는데 영향을 준다.

![Mach band graph](https://cdn.cambridgeincolour.com/images/tutorials/usm_gradploteye.png)
위의 그림에서 Brighness가 변화하는 edge에서의 exaggeration을 보자.

Unsharp Mask를 사용할 때엔 세가지 parameter에 대해서 이해해야 한다. 
1. `Amount` : How much contrast is added at the edges.
2. `Radius` : Controls the amount to blur the original for creating the mask.
3. `Threshold` : sets the minimum brightness change that will be sharpened.

물론, 아무렇게나 적용한다고 visually correct하진 않다. visual artifact를 발생시키는 경우가 있는데, 
light와 dark의 over/undershoot을 많이하여 너무 sharpening을 많이하거나, 
Red object와 Gray background인 Image의 경우가 대표적이다. 자세한건 아래 [1](https://www.cambridgeincolour.com/tutorials/unsharp-mask.htm)링크를 참조하자.

---

Unsharp mask의 전반적인 이론에 대해서는 이정도로 넘어가며, Graphics에서 depth buffer를 이용해서 
어떻게 구현할 것인가에 대해서는 다음 논문을 참고한다 [Luft et al. 2006].

Depth buffer에 unsharp mask를 적용하게 되면 우리는 추가적인 semantic information으로
scene의 object들 사이 spatial relation을 알 수 있다. 

예를들어, Blue background 앞에 있는 Blue object에 Unsharp mask를 이용해 Local contrast enhancement를 하면
color contrast의 결핍으로 이미지가 변하지 않지만, Depth buffer를 이용한 technique은 추가적인 
depth information(spatial relation)으로 color altering이 가능하다.

아래 그림은 [AttributeVertexClouds](https://github.com/snowapril/AttributeVertexClouds)의 
Unsharp masking을 적용하기 전과 적용 후 비교샷이다.

![Before](https://snowapril.github.io/assets/img/post_img/2021-03-09-unsharp-masking-before.png)![After](https://snowapril.github.io/assets/img/post_img/2021-03-09-unsharp-masking-after.png)

---
### Reference
1. [https://www.cambridgeincolour.com/tutorials/unsharp-mask.htm](https://www.cambridgeincolour.com/tutorials/unsharp-mask.htm)
2. ![Luft, T., Colditz, C., and Deussen, O. 2006. Image enhancement by unsharp  masking the depth buffer. ACM Trans. Graph. 25, 3, 1206–1213.](https://dl.acm.org/doi/10.1145/1141911.1142016)