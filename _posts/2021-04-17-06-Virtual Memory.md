---
layout: post
title:  "06-Virtual Memory"
date:   2021-04-17 14:38:27 +0900
categories: computer-science
tags: operating-system os computer-science
comments: true  
---

### Memory virtualization

- OS가 physical memory를 virtualize
- OS가 각 process에게 **illusion memory space** 제공
⇒ CPU 가상화는 각 process가 자기만의 illusion CPU가 있는 것처럼
- 각 process가 자기가 memory 전체를 다 쓰는 것처럼 보이게함

### Goal

- **Transparency**
    - process는 memory가 shared된다는 것을 몰라야 함
    - programming에 편의적인 **abstraction**을 제공
- **Efficiency**
    - space : minimize fragmentation due to variable-sized requests
    - time : get some hardware support
- **Protection**
    - protect process and OS from another processes
    - **Isolation** : process가 다른 process들에 영향주지않고 지혼자 fail함
    - cooperating processes가 memory의 일부분을 공유함

### Multiprogramming and Time sharing

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled.png)

- 이로 인해 protection issue가 중요해졌음
⇒ errant memory accesses from other processes

### **Address space**

- OS가 physical memory에 대한 abstraction을 생성함
    - address space는 running process에 대한 모든 정보를 보관
    - program code, heap, stack and etc 로 구성됨

    ![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%201.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%201.png)

address space가 16KB == $2^{14}$byte
⇒ byte마다 번호를 매기기 위해서는 14개의 bit가 필요함
⇒ 14bit address space

- **Code** : instruction이 머무는 곳
- **Heap** : dynamically allocate memory(malloc, new)
- **Stack** : store return addresses or values, local variables, arguments

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%202.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%202.png)

### Virtual address

process에 있는 모든 address는 **virtual** 
⇒ OS가 virtual address를 physical address를 변환해줌

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%203.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%203.png)

### Memory virtualizing with efficiency and control

- memory virtualizing은 Limited direct execution과 같은 전략을 취함
- memory virtualizing은 **hardware support**를 통해 efficiency와 control
⇒ ex : registers, TLB, page-tabe

### Address translation

- hardware가 virtual address를 physical address로 변환함
- OS가 주소변환에 필요한 정보를 hardware에 제공해줌
- Assumption(가정)
1. user의 address space가 physical memory에 contiguously하게 있다
2. address space의 크기가 physical memory size보다 작다
3. 각 address space는 모두 같은 크기를 가짐

### Address transliation 예제

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%204.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%204.png)

x = x + 3이 아래의 세 assembly code로 변환됨

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%205.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%205.png)

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%206.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%206.png)

1. fetch instruction at address 128
2. execute this instruction ⇒ load from address 15KB
3. fetch instruction at address 132
4. execute this instruction ⇒ no memory reference
5. fetch instruction at address 135
6. execute this instruction ⇒ save to address 15KB

### Relocation address space

OS는 process가 physical memory의 0 위치에 못올라오게 하는걸 원함
⇒ (virtual) address space 는 address 0에서 시작함
⇒ address space의 주소가 physical address로 변환될때 relocation 해야함

### A single relocated process

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%207.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%207.png)

### Static relocation

- Software-based relocation
⇒ OS가 program이 memory에 loading되기 전에 rewrites함
⇒ static data와 functions의 address를 바꿈

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%208.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%208.png)

**Pros** 

- hardware support가 필요없다

**Cons**

- no protection enforced
⇒ process가 OS나 다른 process의 process를 침범할 수 있다.
⇒ no privacy : 아무 memory address나 읽을 수 있음
- Cannot move address space after it has been placed
⇒ external fragmentation으로 new process를 allocate하기 힘들다

### Dynamic relocation

Hardware-based relocation

- MMU(memory management unit)가 모든 memory reference instructions에서 address translation을 수행함
- Hardware에 의해 protection이 enforced됨 
⇒ virtual address가 invalid하면 MMU가 raise exception
- OS가 현재 process의 valid address space에 대한 정보를 MMU에 전달

### Base & Bounds register

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%209.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%209.png)

### Dynamic(hardware-based) relocation

- program이 시작될 때, OS가 process를 physical memory의 어디에 load시킬지 결정함
    - **base register** 값 결정

    ![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2010.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2010.png)

    - 모든 virtual address는 **bound register**보다 크거나 음수아면 안됨

    ![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2011.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2011.png)

### Relocation and Address translation (by MMU)

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2012.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2012.png)

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2013.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2013.png)

address 128의 instruction은 아래와 같이 변환됨
⇒ 128 + 32KB(base-register) = 1024 * 32 + 128

execute this instruction
⇒ address 15KB에서 값을 불러와야함
⇒ 15KB + 32 KB = 47KB

### Two ways of bound registers

1. address space의 size를 bound reigster로 사용
2. physical memory에서 address space가 끝나는 지점을 bound 

### OS issues for memory virtualizing

- OS가 base-bounds register approach를 위해서 뭔가 action을 취해야함
- Three ciritical junctures(고비) :
    - when a process starting : finding space for address spcae in physical memory
    - when a process is terminated : reclaming the memory for use
    - when context switch occurs : saving & restroing base-bound pair

### OS issues : When a process starting

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2014.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2014.png)

- OS가 새로운 address space를 위해서 room을 찾아줘야함
⇒ **free list** : 사용중이지 않은 physical memory의 범위 list

### OS issues : When a process is terminated

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2015.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2015.png)

- OS는 memory를 다시 free list로 넣어줘야함

### OS issues : When context switch occurs

![06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2016.png](06-Virtual%20Memory%209706154001414b829e00e641a976460d/Untitled%2016.png)

OS가 base-and-bound pair를 save & restore 해줘야함 
⇒ process structure 또는 process control block (PCB) 안에 save&restore