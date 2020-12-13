---
layout: post
title:  "About C++ name mangling"
date:   2020-12-11 21:38:27 +0900
categories: cpp
tags: c c++ abi name-mangling
comments: true  
---

**name mangling**이란 [1](https://en.wikipedia.org/wiki/Name_mangling)Wikipedia의 정의에 따르면 아래와 같다.

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

compiler는 symbol name을 생성할 때, function name과 function signature를 고려하여 symbol을 만든다
간단한 예를 들자면, 아래와 같은 함수가 있다고 하자
```c++
int add(int a, int b);
```
function name과 function signature를 이용하여 `addii`와 같은 형식으로 만들어낸다.

또한 compiler마다 mangling 규칙이 다른데 아래의 표를 참고해보자
![compiler name mangling rules](https://snowapril.github.io/assets/img/post_img/2020-12-11-mangling-rules.jpg)

그렇다면 function overloading가 존재하지 않는 C에서는 name mangling이 필요할까?
C에서는 function overloading이 없으므로 하나의 binary에는 동일한 function name을 가진 구현이 하나만 존재하므로,  
mangled된 function name이 필요하지 않다. 그렇지만 위의 add라는 함수는 C에서 compile을 거치면 _add로 변한다.

function overloading을 하지 않아도 function signature에 따라 name mangling 되므로,  
C++에서 정의한 함수를 C에서 쓰려고 하면 linking error가 발생한다.   
이런 경우에 사용할 수 있는게 `extern "C"`이다. 

---
### reference
1. https://en.wikipedia.org/wiki/Name_mangling
2. https://spikez.tistory.com/19