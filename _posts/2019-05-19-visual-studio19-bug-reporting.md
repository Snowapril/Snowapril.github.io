---
layout: post
title:  "Visual Studio 2019 버그리포팅한 이야기"
date:   2019-05-19 20:38:27 +0900
categories: dev-issue
tags: visualstudio19 bug-report bug report compile
comments: true  
---

군입대를 2개월가량 남긴채 너무 할짓이 없어 "ToonEngine" 프로젝트의 Compile time JSON parsing 기능을 구현하는 중에 constexpr vector class를 만들고 컴파일을 하니 아래와 같은 에러메시지가 발생했다.

```[c++]
1>C:\Users\sinji\Desktop\Github\ToonEngine\ToonEngine\ToonResourceParser\Sources\main.cpp(6): fatal error C1001: 컴파일러에서 내부 오류가 발생했습니다.
1>(컴파일러 파일 ‘msc1.cpp’, 줄 1527)
1> 이 문제를 해결하려면 위 목록에 나오는 위치 부근의 프로그램을 단순화하거나 변경하십시오.
1>자세한 내용을 보려면 Visual C++ [도움말] 메뉴에서 [기술 지원] 명령을
1> 선택하거나 기술 지원 도움말 파일을 참조하십시오.
1>'C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.20.27508\bin\HostX86\x64\CL.exe’에서 내부 컴파일러 오류
1> 자세한 내용을 보려면 Visual C++ [도움말] 메뉴에서
1> [기술 지원] 명령을 선택하거나 기술 지원 도움말 파일을 참조하십시오.
1>“ToonResourceParser.vcxproj” 프로젝트를 빌드했습니다. - 실패
```

**"fatal error c1001 : 컴파일러에서 내부 오류가 발생했습니다" **
비록 내가 개발경력이 짧지만, 이런 오류메시지는 처음 보았다. 
내가 TMP 에 익숙하지 않아 코드를 잘못짜서 뭔가 엉켰나? 싶었지만 별 대단한 코드를 작성한 상태도 아니고, 정말 간단하게 template class에 constexpr constructor의 선언과 정의 이게 다였다. 
이런저런 시도를 해봤지만 계속 같은 오류메시지만 나와서 google에 "visual studio fatal error c1001"을 검색해보니 
stackoverflow에서 이 오류메시지는 컴파일러 내부의 오류로, 버그리포팅을 해야한다고 말하고있었다.


버그리포팅을 하기 전에, 같은 이슈로 누군가가 이미 버그리포팅을 하였는지 Visual Studio Developer Community에서 글을 찾아보았다. 다행히(?) 나와같은 이슈를 작성한 사람은 없었다. 


버그리포팅은 생전 처음해보는지라 어떻게해야 "개발자들이 쉽게 디버깅 할 수 있도록" 버그리포팅을 작성해야하는지 알아봐야 했지만, 군입대가 얼마 안남은 나는 그냥 최대한 정중하게 에러메시지, 에러코드, 다른 컴파일러에서의 동작여부만 첨부하고 글을 게시했다.


[bug report page](https://developercommunity.visualstudio.com/content/problem/517958/fatal-error-c1001-when-i-define-constexpr-construc.html)

3일간 댓글이 달리지않길래, 내가 이상하게 해놓고 버그리포팅이랍시고 글을 올린걸까 생각했지만
4일째에 댓글이 달리고, 그후로 clang 컴파일러 버젼을 공유한 후, 6일째에 "A fix for this issue has been internally implemented and is being prepared for release. We'll update you once it becomes available for download." 라는 댓글이 달렸다. 


내가 정말 애용하는 visual studio에, 정말 사소한 이슈지만 무언가 기여했다는 것이 군입대 전에 보란찬 일을 한것같아 기분이 좋았다. 


버그리포팅을 한지 8일째에 "A fix for this issue has been released! Install the most recent release from https://visualstudio.microsoft.com/downloads/ . Thank you for providing valuable feedback which has helped improve the product." 라는 댓글이 달렸고, 즉시 VS2019 버전 업데이트를 한 후 컴파일을 돌려보니 잘 동작하였다.


버그리포팅을 해보고 깨달은 것은, 내가 운이 좋은 케이스라는 것이다. 정말 우연히 constexpr constructor를 가진 template class를 작성하였고, 또 우연히 이러한 코드에서 clang 19.20.27508.1 version에 버그가 존재했고, 하필이면 또 이런 이슈를 아무도 버그리포팅하지 않았다. 이런 세번의 행운이 겹쳐서 버그리포팅에 성공하게 된 것이다.