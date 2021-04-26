---
layout: post
title:  "03-Limited Direct Execution"
date:   2021-04-17 14:38:27 +0900
categories: computer-science
tags: operating-system os computer-science
comments: true  
---

### Direct execution

그냥 program을 CPU 위에서 direct하게 수행함

![https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled.png](https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled.png)

running program에 제한이 없으면 OS는 anything에 대한 control이 없음. 그냥 library가 되는거임

### Problem 1 : Restricted operation

Process가 I/O request와 같은 제한된 operation을 수행하고 싶을 때.

Solution : **Protected control transfer**

- User mode : application은 hardware resource에 full access가 없음
- Kernel mode : OS는 machine의 full resources에 access 가능함

### System call

kernel이 특정한 functionality를 uesr program에게 조심스럽게 노출시킴

- access file system
- create & destroy process
- communicate with other processes
- allocate more memory

**Trap instruction**

- kernel로 jump
- privilege level을 uesr mode에서 kernel mode로 격상

**Return-from-trap instruction**

- trap을 호출한 user program으로 return 함
- privilege level을 다시 user mode로 격하시킴

![https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%201.png](https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%201.png)

trap은 OS 내부에서 어떤 코드가 수행될지 어케알까

- **trap table, trap handler**
⇒ table에 여러 종류의 트랩이 존재하는데, 각 trap마다 trap-handler를 저장해놓음. trap-handler는 OS에서 trap에 대해 수행되는 코드부분

**System-call number**

- system call 마다 할당되어있음, trap-number랑은 다름
- **user code는 원하는 system-call number를 register에 넣는 것을 맡음**

### Limited Direction Execution Protocol

![https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%202.png](https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%202.png)

![https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%203.png](https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%203.png)

Direct execution에서 uesr mode와 kernel mode로 제한을 둬 Limited direct execution

### Problem2 : Process들 사이의 switching

어떻게 CPU 제어권을 OS가 다시 가져와서 다른 process로 넘겨줄까?

- Cooperative approach : wait for system calls
- Non-cooperative approach : The OS takes control

### Cooperative approach : wait for system calls

process가 자발적 & 주기적으로 yield같은 system call로 CPU를 양보함

- 양보 받으면 OS는 이제 다른 task를 수행함
- Application이 illegal한걸 수행하면 또 OS에 transfer control
- **problem : process가 무한loop에 빠지면 재부팅해야함**

### Non-cooperative approach : OS takes Control

**Timer interrupt** 

- booting 중에 OS가 timer를 시작시킴
- 주기적으로 timer interrupt가 발생함
- interrupt가 발생하면
⇒ 현재 실행중인 process가 일시정지되고
⇒ process의 state를 저장함
⇒ 미리 configured 된 interrupt handler를 실행시킴

### Saving & Restoring Context

**Scheduler가 decision making**

- 현재 process를 계속 실행할지, 다른걸로 switching 할지 결정
- switch하기로 결정하면 OS가 **context switch**를 수행함

### Context Switch

Context-switch는 매우 빈번하게 일어나므로 overhead를 줄이기 위해 **머신에 최적화된 assembly로 구현됨**

- 현재 process의 몇몇 register값들(PC, general purpose registers, kernel stack pointer)을 Kernel stack에 **Save**함
- Kernel stack에 곧 실행될 process의 registers 를 **Restore**함
- 곧 실행될 Processs를 위해 Kernel stack으로 switching 됨

![https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%204.png](https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%204.png)

### Limited Direction Execution Protocol (With Timer interrupt)

![https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%205.png](https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%205.png)

![https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%206.png](https://snowapril.github.io/assets/img/post_img/03-Limited%20Direct%20Execution%20649153b885da45b4824dab9eb67f15f7/Untitled%206.png)

### Concurrency에 대한 걱정

interrupt 또는 trap handling 중에 다른 interrupt가 발생하면 어떻게 될까?

OS는 이런 situtations를 아래와 같이 대처함

- interrupt processing 중에는 **interrupt를 disable함**
- 정교한 **Locking schemes**로 internal data structure로의 concurrent access를 보호함