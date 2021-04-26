---
layout: post
title:  "CLI 환경에서의 OpenGL Debugger"
date:   2020-03-25 20:38:27 +0900
categories: real-time-rendering
tags: opengl cli debugger apitrace headerless
comments: true  
---
CLI 환경에서 OpenGL Debugger나 API Tracer를 찾아보려고 오늘 하루종일 끙끙 앓으며 찾아 다녔다.


자주 사용되는 [apitrace](https://github.com/apitrace/apitrace)로 opengl api call tracing을 해보려고 해봤는데, headless server라서 display가 없는게 
문제가 되는지 아무런 `.trace` 파일도 생성되지 않는다. 


Valve의 `vogl`가 command line interface를 제공하는 것 같아서 빌드해보려 했지만, ubuntu 18.04 64 bit에서 이슈가 있는지
```[bash]
list index: 1 out of range (-1, 0)
```
라는 cmake 에러메시지가 발생한다.

이외에 여러 다른 툴들을 시도해봤지만 제대로 동작하는 것 같지가 않다. 답답한 마음에 reddit에 질문 글을 올렸는데, 한 유저가 Renderdoc을 사용해보라고 써보진 않았지만 command line 버젼도 제공한다고 하여 바로 시도해봤다.


https://www.reddit.com/r/opengl/comments/fom100/need_command_line_debugger_for_headless_server/
결과는 역시나 아무런 caputre 파일이 생성되지가 않는다. 내가 잘못 사용하고있는건가 싶어 코드를 까봤는데 역시나 디스플레이를 여는 부분이 있다. 이러니 아무런 반응이 없지.. 


결국은 apitrace로 돌아왔다. Documentation을 보면 Linux 사용자를 위해 `LD_PRELOAD`변수에 `libGLX`, `libEGL` wrapper로 덮어 씌워 실행시킨 사용자 OpenGL 프로그램을 `apitrace`로 분석하는 방법을 설명해주고 있다.


하지만 나같이 `Headless server`에서 OSMesa dependency로 `opengl context`를 생성하는 사용자를 위한 옵션은 없었다. 분명 나같은 사용자가 있을거 같아서 issue를 뒤져봤는데 있었다.
https://github.com/apitrace/apitrace/issues/525
개발자 말로는 해당 이슈가 자신의 우선순위에 있지 않아서 구현하지를 못할 것 같다며, 
하지만 `OSMesa support`를 구현하는 것은 쉽다며 그 방법을 설명해주며 사용자의 몫으로 넘겼다.
`Pull request`를 봐도 아직 osmesa support를 구현한 사람은 없는듯 하다.