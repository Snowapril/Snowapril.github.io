---
layout: post
title:  "About C++ name mangling"
date:   2020-12-11 21:38:27 +0900
categories: cpp
tags: c c++ abi name-mangling
comments: true  
---

**name mangling**이란 [[1]](https://en.wikipedia.org/wiki/Name_mangling)Wikipedia의 정의에 따르면 아래와 같다.

> 컴파일 과정에서 식별자(*Identifier*)는 같지만 다른 namespace에 속해 있거나,   
> 다른 시그니쳐를 가지는 경우(*ex. function overloading*) 고유한 이름을 가지게 하기 위해  
> 호출 규약(*calling convention*) 등에 따라서 함수나 변수의 이름을 컴파일러가 변경하는 것

이를 설명하기 위해서는 C++에서의 **function overloading**을 살펴봐야 한다.
**function overloading**이란 이미 존재하는 함수와 동일한 이름으로 다른 시그니쳐(signature)를  
가지는 함수를 정의하는 것을 나타낸다.

```c++
int add(int a, int b);
int add(int a, int b, int c);
float add(float a, float b, float c);
```

위의 예와 같이 add라는 naming 하나로 아래의 세가지 예시에 해당하는 코드를 모두 구현할 수 있다.
* int 2개를 더하여 반환
* int 3개를 더하여 반환
* float 3개를 더하여 반환  

그렇다면 compiler는 코드에서 add를 호출할 때, 어떻게 적절한 add 구현체를 찾아 호출 해주는지가 의문일 것인데,
이를 가능하게 해주는 것이 `name mangling`이다.

compiler는 symbol name을 생성할 때, function name과 function signature를 고려하여 symbol을 만든다.
간단한 예를 들자면, 아래와 같은 함수가 있다고 하자
```c++
int add(int a, int b);
```
function name과 function signature를 이용하여 `addii`와 같은 형식으로 만들어낸다.
실제로는 아래 그림과 같이 컴파일러마다 mangling rule이 다르다.
![compiler name mangling rules](https://snowapril.github.io/assets/img/post_img/2020-12-11-mangling-rules.jpg)

그렇다면 function overloading이 존재하지 않는 C에서는 name mangling이 필요할까?
C에서는 function overloading이 없으므로 하나의 binary에는 동일한 function name을 가진 구현이 하나만 존재하므로,
mangled된 function name이 필요하지 않다. 그렇지만 경우에 따라 함수에 대한 추가적인 정보를 제공하기 위해 `_add`, `_add@4`, `@add@4`와 같이
mangling되기도 하는데 자세한 내용은 이 글의 범위를 벗어나니 생략한다.

function overloading을 하지 않아도 function signature에 따라 name mangling 되므로,  
C++에서 정의한 함수를 C에서 쓰려고 하면 linking error가 발생한다.   

이런 경우에 사용할 수 있는게 `extern "C"`이다. [microsoft docs](https://docs.microsoft.com/en-us/cpp/cpp/extern-cpp?view=msvc-160)에 의하면 extern "C" 와 extern "C++"의 역할은 아래와 같다.

> extern specifies that the linkage conventions of another language are being used for the declarator(s). 

간단하게는, C++ 파일에서 extern "C" scope 안에서 정의된 내용들은 compiler를 통해 name mangling 하지 않고 C-style symbol로 바뀌게 된다.
아래는 extern "C"를 이용해 함수를 선언하는 헤더파일의 예이다.

```c++
//vec_calculation.hpp
#ifndef VEC_CALCULATION_HPP
#define VEC_CALCULATION_HPP

#include <vec.hpp>

extern "C"
{
    float dot(vec3 v1, vec3 v2);
}

#endif //end of VEC_CALCULATION_HPP
```

하지만 위의 코드로는 vec_calculation.hpp 파일을 C에서는 include 하지 못하는데, C에서는 extern "C"를 지원하지 않기 때문이다.
따라서 predefined macro인 __cplusplus를 이용해 아래와 같이 수정할 수 있다.

```c++
//vec_calculation_modified.hpp
#ifndef VEC_CALCULATION_MODIFIED_HPP
#define VEC_CALCULATION_MODIFIED_HPP

#include <vec.hpp>

#ifdef __cplusplus
extern "C"
{
#endif

float dot(vec3 v1, vec3 v2);

#ifdef __cplusplus
}
#endif

#endif //end of VEC_CALCULATION_MODIFIED_HPP
```

언급했듯이 function name이 name mangling 되는 순간은 linking 할 때가 아니고, compile 할 때이므로, header 파일에서만 extern "C"로
감싸놓는다 해도 source 파일에서 extern "C" 처리를 해놓지 않으면 소용이 없다.

vec_calculation_modified.hpp만 extern "C" 처리를 하고 vec_calculation_modified.cpp 에서는 extern "C" 처리를 하지 않는다면,
`dot`을 호출하는 C 파일에서는 name mangling 되지 않은 dot의 symbol을 찾는데, dot의 body는 C++ compiler에 의해서 name mangling 되어버리므로
C 파일에서 호출한 dot은 구현체가 없는 상황이 발생해버린다. 따라서 linker는 error를 뱉어낸다.

```c++
//vec_calculation_modified.cpp
#include <vec.hpp>
float dot(vec3 v1, vec3 v2)
{
    return v1.x * v2.x + v1.y * v2.y + v1.z * v2.z;
}
```

source 파일에서도 extern "C"를 감싸면 문제가 해결되지만, 이 방법보다도 함수의 선언부가 포함된 header 파일인 vec_calculation_modified.hpp을
source 파일에서 include 해주면 compiler가 알아서 `dot`의 body도 name mangling 되지 않게 해준다.

---
### reference
1. [https://en.wikipedia.org/wiki/Name_mangling](https://en.wikipedia.org/wiki/Name_mangling)
2. [https://spikez.tistory.com/19](https://spikez.tistory.com/19)
3. [https://docs.microsoft.com/en-us/cpp/cpp/extern-cpp?view=msvc-160](https://docs.microsoft.com/en-us/cpp/cpp/extern-cpp?view=msvc-160)
