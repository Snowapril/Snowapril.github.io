---
layout: post
title:  "Visual Studio Runtime Library Option(/MT, /MD)"
date:   2020-12-17 21:55:27 +0900
categories: language
tags: c cpp visualstudio runtime-library
comments: true  
---

오랜만에 Visual Studio를 이용해서 프로젝트를 하는 중, cmake에서 GLFW를 add_subdirectory를 이용해 빌딩하는데 자꾸 
`unresolved external error`가 나서 도통 이유를 알 수가 없었다.

구글링을 하던 중, 나와 똑같은 case에서 동일한 error가 발생해 [질문글](https://discourse.glfw.org/t/could-not-link-with-glfw-on-windows/1584)을 올렸는데 결론은 GLFW가 default로 debug mode에서는 `/MDd`, release mode에서는 `/MD` option으로 build 하기 때문에 내 프로젝트에서 GLFW를 사용하려면 동일한 option을 줘야한다는 것이었다.

일단 그대로 적용시켜 성공적으로 build가 되긴 했지만, 그래서 `/MD`는 뭐고 `/MT`는 뭘까? 그리고 왜 이것 때문에 linking error가 발생했던 것일까?

일단 Runtime library linking에 대해 어떻게 external references를 해결하는가에 따라 4가지 옵션이 존재한다.
DLL로 runtime library를 제공하는지 static하게 프로그램에 link하는지에 따라 `/MD`, `/MT`로 나뉘고 debug 옵션을 주는가 안주는가에 따라서 `/MDd` 그리고 `/MTd`로 나뉜다.

각각의 옵션에 대한 자세한 설명은 microsoft document에 아래와 같이 나와있다.
> **/MD** : Causes the application to use the multithread-specific and DLL-specific version of the run-time library. Defines _MT and _DLL and causes the compiler to place the library name MSVCRT.lib into the .obj file.
> Applications compiled with this option are statically linked to MSVCRT.lib. This library provides a layer of code that enables the linker to resolve external references. The actual working code is contained in MSVCR versionnumber.DLL, which must be available at run time to applications linked with MSVCRT.lib.

> **/MDd** : Defines _DEBUG, _MT, and _DLL and causes the application to use the debug multithread-specific and DLL-specific version of the run-time library. It also causes the compiler to place the library name MSVCRTD.lib into the .obj file.

> **/MT** : Causes the application to use the multithread, static version of the run-time library. Defines _MT and causes the compiler to place the library name LIBCMT.lib into the .obj file so that the linker will use LIBCMT.lib to resolve external symbols.

> **/MTd** : Defines _DEBUG and _MT. This option also causes the compiler to place the library name LIBCMTD.lib into the .obj file so that the linker will use LIBCMTD.lib to resolve external symbols.

정리하자면, `/MD` 옵션은 application이 multithread & DLL-specific한 runtime-library인 MSVCRT.lib에 linking됨을 의미한다. MSVCRT.lib는 linker가 external references를 해결할 수 있는 layer of code를 제공하는데, 실제 동작 code는 runtime에 program에 linking되는 MSVCR(version number).dll에 들어있다. actual working code가 program에 적재되는 것이 아니라 사용자 PC에 dll 형태로 존재하다가, run-time에 linking 되기 때문에 program의 용량도 적고 compile 시간도 단축되는 장점이 있다.

`/MT` 옵션은 application이 multithreaded & static한 run-time library인 LIBCMT.lib에 linking 되는데, linker가 program build때 external symbols를 해결하기에 program이 필요로 하는 runtime library code를 실행파일에 포함시켜 용량이 커지고 compile 시간도 늘어난다. 대신 실제 동작 코드를 사용자 PC의 dll에 의존하는 것이 아니라 program에 shipping 해버리므로 program이 동작하는 environment에 구애받지 않고 독립적으로 실행이 가능하다. 

아래는 `/MD`, `/MDd`, `/MT`, `/MTd` 각각의 옵션에 따라 linking 되는 library의 종류이다.

| Standard C++ Library | Characteristics | Option | Preprocessor Directives |
|:---:|:---:|:---:|:---:|
|LIBCMT.lib|Multithreaded, static link|/MT|_MT|
|LIBCMTD.lib|Multithreaded, static link|/MTd|_MT, _DEBUG|
|MSVCRT.lib|Multithreaded, dynamic link(import library for MSVCR(version number).dll)|/MD|_MD, _DLL|
|MSVCRTD.lib|Multithreaded, dynamic link(import library for MSVCR(version number).dll)|/MDd|_MD, _DEBUG, _DLL|

또한, MS document 보다보면 아래와 같은 문구가 있다.
> All modules passed to a given invocation of the linker must have been compiled with the same run-time library compiler option (/MD, /MT, /LD).

왜 내 project에서 glfw library를 build 할 때 unresolved external error가 발생했는지 알 수 있는 대목이다. 나는 CMakeLists.txt에서 `/MT`로 build option을 주었는데, glfw에서는 default로 `/MD` option을 사용하여 build 했기 때문에 glfw를 사용하는 application을 build 할 때 external reference를 resolve 하려는데 glfw.lib에서는 external reference를 해결하지 못하므로 발생한 linking error이다.

---
### reference
1. [https://discourse.glfw.org/t/could-not-link-with-glfw-on-windows/1584](https://discourse.glfw.org/t/could-not-link-with-glfw-on-windows/1584)
2. [https://diehard98.tistory.com/entry/MSDN-MT-MTd-MD-MDd-C-Runtime-Library-Option](https://diehard98.tistory.com/entry/MSDN-MT-MTd-MD-MDd-C-Runtime-Library-Option)
3. [https://stackoverflow.com/questions/919267/whats-the-difference-between-mtd-and-mdd-in-code-generation-property-section](https://stackoverflow.com/questions/919267/whats-the-difference-between-mtd-and-mdd-in-code-generation-property-section)
4. [https://m.blog.naver.com/milennium9/20153887924](https://m.blog.naver.com/milennium9/20153887924)
5. [https://docs.microsoft.com/ko-kr/cpp/build/reference/md-mt-ld-use-run-time-library?view=msvc-160](https://docs.microsoft.com/ko-kr/cpp/build/reference/md-mt-ld-use-run-time-library?view=msvc-160) 