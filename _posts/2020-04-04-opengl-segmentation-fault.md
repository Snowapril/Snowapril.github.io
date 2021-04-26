---
layout: post
title:  "OpenGL Segmentation Fault [Solved]"
date:   2020-04-04 20:38:27 +0900
categories: real-time-rendering
tags: opengl debug gdb backtrace
comments: true  
---
CubbyFlow에서 Visualization 파트를 맡게 되어서 CubbyRender branch에서 opengl 개발중에 있는데, 
OpenGL 개발은 혼자서 실험적으로 해본적 많지만, 이렇게 큰 프로젝트에서 메인으로 맡게된 것은 처음이었다.


OpenGL 뿐만 아니고, Framework를 잘 설계해서 DirectX, Vulkan 등의 연동도 고려하는 중이라서 architecture를 크게크게 그리고 있는데 이슈가 발생했다. 


Application을 생성하고 Shader, Vertex Buffer 등의 생성 그리고 메인 루프까지는 아무 문제없이 동작한다.
그런데 프로그램이 종료될 때 Segmentation Fault가 발생한다. 
 

Visual Studio에서 벗어나 Ubuntu server에서 gdb로 디버깅 하는 경험은 적은지라 조금 해멨는데, segmentation fault가 발생했을 때 어느 부분에서 발생한 것인지를 알아야하는데 이 방법을 몰랐다.
 

일단은, backtrace를 이용해서 segmentation fault가 일어났을 때의 frame stack을 살펴보았다.
```
Thread 1 "GL3Examples" received signal SIGSEGV, Segmentation fault.
0x00007ffff5613bc0 in ?? ()
(gdb) bt
#0  0x00007ffff5613bc0 in ?? ()
#1  0x0000555555688737 in CubbyFlow::CubbyRender::GL3VertexArrayObject::onRelease (this=0x555555c2eee0)
    at ../Experimental/Sources/CubbyRender/GL3/Buffer/GL3VertexArrayObject.cpp:57
#2  0x000055555564a2bf in CubbyFlow::CubbyRender::InputLayout::release (this=0x555555c2eee0)
    at ../Experimental/Sources/CubbyRender/Framework/Buffers/InputLayout.cpp:66
#3  0x0000555555688562 in CubbyFlow::CubbyRender::GL3VertexArrayObject::~GL3VertexArrayObject (this=0x555555c2eee0, __in_chrg=)
    at ../Experimental/Sources/CubbyRender/GL3/Buffer/GL3VertexArrayObject.cpp:33
#4  0x00005555556871eb in __gnu_cxx::new_allocator::destroy (
    this=0x555555c2eee0, __p=0x555555c2eee0) at /usr/include/c++/7/ext/new_allocator.h:140
#5  0x00005555556870b9 in std::allocator_traits<std::allocator >::destroy (__a=..., __p=0x555555c2eee0) at /usr/include/c++/7/bits/alloc_traits.h:487
#6  0x0000555555686d6e in std::_Sp_counted_ptr_inplace<CubbyFlow::CubbyRender::GL3VertexArrayObject, std::allocator, (__gnu_cxx::_Lock_policy)2>::_M_dispose (this=0x555555c2eed0) at /usr/include/c++/7/bits/shared_ptr_base.h:535
#7  0x00005555556045a0 in std::_Sp_counted_base<(__gnu_cxx::_Lock_policy)2>::_M_release (this=0x555555c2eed0)
    at /usr/include/c++/7/bits/shared_ptr_base.h:154
#8  0x00005555555f8a11 in std::__shared_count<(__gnu_cxx::_Lock_policy)2>::~__shared_count (this=0x7fffffffd2c8, __in_chrg=)
---Type  to continue, or q  to quit---
    at /usr/include/c++/7/bits/shared_ptr_base.h:684
#9  0x00005555555f47f4 in std::__shared_ptr<CubbyFlow::CubbyRender::InputLayout, (__gnu_cxx::_Lock_policy)2>::~__shared_ptr (this=0x7fffffffd2c0, 
    __in_chrg=) at /usr/include/c++/7/bits/shared_ptr_base.h:1123
#10 0x00005555555f4834 in std::shared_ptr::~shared_ptr (this=0x7fffffffd2c0, __in_chrg=)
    at /usr/include/c++/7/bits/shared_ptr.h:93
#11 0x00005555555e2dc7 in RunExample1 (application=std::shared_ptr (use count 2, weak count 0) = {...}, resX=800, 
    resY=600, numberOfFrames=60) at ../Experimental/Examples/GL3Examples/main.cpp:99
#12 0x00005555555e557a in main (argc=1, argv=0x7fffffffe328) at ../Experimental/Examples/GL3Examples/main.cpp:200
```
`frame stack`을 살펴보니 main 함수에서 RunExample1을 수행하고 끝날 때, `shared_ptr<InputLayout>`의 `ref count`가 0이 되면서 `InputLayout`의 `destructor`가 호출되고, `destructor`가 `virtual`로 선언되었기에 `GL3VertexArrayObject`의 `destructor`도 호출되고 결국은 `GL3VertexArrayObject`의 `onRelease` method까지 이어진다. 

`GL3VertexArrayObject`의 `onRelease` method의 코드는 다음과 같다.
```[c++]
void GL3VertexArrayObject::onRelease() 
{
    if (static_cast<int>(_vertexArrayID))
        glDeleteVertexArrays(1, &_vertexArrayID);
}
```
정말 단순한 코드다 `_vertexArrayID`가 0이 아니라면 `glDeleteVertexArrays`를 호출하여 _vertexArrayID에 할당된 opengl object를 해제한다. 결국은 `segmentation fault`가 `glDeleteVertexArrays`에서 발생한 것인데... 뭐가 문제일까?

문제는 RunExample1 함수에서의 OpenGL Object들의 `destroy` 순서에 있었다. 
Application 객체가 `terminate method`를 호출하면 `glfw`를 이용하여 `glfwWindow`를 삭제하고 `opengl context`를 파괴한다.
문제는 다른 `opengl object`들이 해제 되기도 전에 `context`가 없어져버리니, `glDelete call`들에서 프로그램이 터져버리는 것이다.