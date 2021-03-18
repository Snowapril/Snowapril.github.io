---
layout: post
title:  "Contribution on OpenXR Sample app with Window support"
date:   2021-03-18 14:38:27 +0900
categories: openxr
tags: vulkan openxr ar opensource contribute window cross-platform msvc
comments: true  
---

오늘은 Github 오픈소스에 처음으로 제대로 된 contribution을 하게 되어 그 경험을 이야기하려고 합니다.

요즘 XR에 대한 관심이 많아지면서 Khronos group의 OpenXR에 대한 Webinar 영상을 보게되었는데 <br>

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/7eLQXMpwzsQ?start=1532" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

영상의 중간쯤에 Steve Winston이라는 분께서 openxr의 sample code를 설명하며 시연을 하는 부분이 있다.
친절하시게도 OpenXR을 이용한 XR application 개발이 생소한 사람들을 위해 사이트를 운영중이신데, 링크를 들어가면 
OpenXR Demo에 대한 github link와 build 하는 방법 등을 설명해놓으셨다.
[https://www.holochip.com/](https://www.holochip.com/) 

나도 직접 sample을 build 해보고 싶기도 하고, 시연영상 중에 조금씩 보인 glTF와 Vulkan 코드를 더 자세히 보고싶어서 
내 PC 환경에서 빌드해보려는데, 이곳저곳에서 에러가 많이발생했다. 하나하나 정리해보자면

1. MSVC에서 허용하지 않는 non-standard 구문의 사용 (non-constant array initialization, designated structure initialization, ..)
2. std::wstring을 std::string으로 direct하게 assign하려고 하여 wrong type error
3. window platform detection을 위한 잘못된 매크로 사용 (WIN32 또는 WIN64인데 WINDOW로 사용)
4. spir-v shader 코드를 binary 모드로 읽어서 vkCreateModule에 전달해줘야 하는데, 일반 text 모드로 읽어서 전달.

더 자세하게는 [Pull-Request](https://github.com/Holochip-Public/OpenXRSamples/pull/2)를 확인해보는게 좋을 것 같다.

window msvc에서 동작하게끔 위의 사항들을 수정하여 sample을 돌려봤는데 매우 정상적으로 동작했다.
이러한 경험을 그냥 묻고 지나가기 보다는 sample demo repository에 window msvc support로 
pull-request를 올려보면 좋을 것 같다는 생각이 들어 해보았고, 2~3일만에 approve되어 main에 병합되었다.
