---
layout: post
title:  "Tangent Space"
date:   2021-03-30 14:38:27 +0900
categories: real-time-rendering
tags: tangent-space graphics linear-algebra 
comments: true  
---

# Tangent Space

Advanced shading techniques are bale to make use of texture maps that store finely detailed geometric information. 

![https://snowapril.github.io/assets/img/post_img/tangent_space_post.png](https://snowapril.github.io/assets/img/post_img/tangent_space_post.png)

Texture map example

In the coordinate system of a texture map, the x and y axes are aligned to the horizontal and vertical directions in the 2D image, and the z axis points upward out of the image plane. 

In order to perform shading calculations that use geometric information stored in a texture map, we need a way to transform between the coordinate system of the texture map and object space. This is done by identifying the directions in object space that correspond to the coordinate axes of the texture map which vary from triangle to triangle. The x and y axes of the texture map point along directions that are tangent to the surface in object space, and at least one of these vectors needs to be calculated ahead of time.

As with normal vectors, we calcualte an average unit-length tangent vector **t** for each vertex in a triangle mesh. Although it may not be strictly true for the specific texture mapping applied to a model, we assume that the two tangent directions are perpendicular to each other, so a second tangent direction **b** called the *bitangent vector* can be calculated with a cross product. The three vectors **t**, **b**, and **n** form the basis of the tangent frame at each vertex, and the coordinate space in which the **x**, **y**, and **z** axes are aligned to there directions is called *tangent space*. We can transform vectors from tangent space to object space using the 3x3 matrix like below.

![https://snowapril.github.io/assets/img/post_img/tangent_space_post1.png](https://snowapril.github.io/assets/img/post_img/tangent_space_post1.png)

Since this matrix is orthogonal, the reverse transformation from object space to tangent space is the transpose. The name *TBN matrix* is often used to refer th either one of these matrices. 

Let **p0**, **p1**, and **p2** be the three vertices of a triangle, wound in counterclockwise order, and let (ui, vi) represent the texture coordinates associated with the vertex **pi**. The values of u and v correspond to distances alogn the axes **t** and **b** that are aligned to the **x** and **y** directions of the texture map. This means that we can express the difference between two points with known texture coordinates as like below.

![https://snowapril.github.io/assets/img/post_img/tangent_space_post2.png](https://snowapril.github.io/assets/img/post_img/tangent_space_post2.png)

And **t** and **b** can be derived from the above equation.

![https://snowapril.github.io/assets/img/post_img/tangent_space_post3.png](https://snowapril.github.io/assets/img/post_img/tangent_space_post3.png)

(e1 = p1 - p0, e2 = p2 - p0)

We can nudge them the rest of the way by applying *Gram-Schmidt orthonormalization*. First, assuming the vertex normal vector **n** has unit length, we make sure the vertex tangent vector **t** is perpendicular to **n** by replacing it with

![https://snowapril.github.io/assets/img/post_img/tangent_space_post4.png](https://snowapril.github.io/assets/img/post_img/tangent_space_post4.png)

using the rejection operation. The vertex bitangent vector **b** is then made perpendicular to both **t** and **n** by calculating

![https://snowapril.github.io/assets/img/post_img/tangent_space_post5.png](https://snowapril.github.io/assets/img/post_img/tangent_space_post5.png)

These vectors along with n now form a set of unit-length orthogonal axes for the tangent frame at a vertex. Since the vectors are orthogonal, it is not necessary to store all three of the vectors. Just the normal vector and the tangent vector will always suffice, but we do need one additional bit of information. The tangent frame can form either a right-handed or left-handed coordinate system, and which one is given by the sign of detminant(*TBN matrix*). Calling the sign of this determinant 

**σ**, we can reconstitute the bitangent with the cross product.

![https://snowapril.github.io/assets/img/post_img/tangent_space_post6.png](https://snowapril.github.io/assets/img/post_img/tangent_space_post6.png)

One possible way to communicate the value of **σ** to the vertex shader is by extending the tangent to a four-component vertex attribute and storing **σ** in the w coordinate.

---
### Reference
1. Foundations of Game Engine Development : Volume2 Rendering