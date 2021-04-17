---
layout: post
title:  "01-Introduction to Operating System"
date:   2021-04-17 14:38:27 +0900
categories: operating-system
tags: operating-system os computer-science
comments: true  
---

Running program executes an instruction from memory

1. The processor fetches an instruction from memory
2. **Decode** : Figure out which instruction this is
3. **Execute** : add two numbers, access memory, check a condition etc..
4. The processor moves on to the next instruction and so on

![01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled.png](01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled.png)

### Operating system (OS)

responsible for 

1. program 실행을 쉽게 만듬
2. program들 간에 memory를 sharing
3. program이 device들과 interact할 수 있게
- OS는 **system**이 **correctly**하고 efficiently하게 동작하도록 보장하는 일 함

### Virtualization

OS는 physical resources를 virtual form으로 변환시켜줌
⇒ processor, memory, disk..

- virtual form은 general, powerful and easy-to-use
- 때때로, OS를 virtual machine이라 하기도 함

### System call

system call으로 user가 OS에게 뭐좀 해달라고 할 수 있음

- OS는 interface를 제공함 (APIs, standard library)
- typical OS는 몇백개의 system call을 가짐 (run program, access memory,device)

### OS는 resource manager이다

OS는 CPU, memory, disk와 같은 resources를 관리함

OS는 아래를 허용한다

- many programs to run → sharing the CPU
- many programs concurrently access their own instructions & data → sharing the memory
- many programs to access devices → sharing disks

### Virtualizing CPU

system은 매우 많은 수의 virtual CPU를 가짐

- CPU 하나를 무한한 수의 CPU를 가진 것처럼 보이게 함
- 많은 program이 동시에 실행되고 있는 것처럼 보이게 함

![01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%201.png](01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%201.png)

Application들이 자기만의 고유한 CPU를 가지고 있는 것처럼 보이게함

![01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%202.png](01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%202.png)

![01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%203.png](01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%203.png)

processor가 하나밖에 없지만, 4개의 program이 동시에 실행되는 것처럼 보임

### Virtualizing Memory

physical memory는 array of bytes임 (각각의 byte는 addressable함)

program은 모든 자료구조를 memory에 보관함

- Read memory(load)
⇒ data를 access할 수 있는 address를 명시함
- Write memory(store)
⇒ data와 data를 쓸 address를 명시함

![01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%204.png](01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%204.png)

![01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%205.png](01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%205.png)

각 process는 자기의 고유한 virtual address space에 access함

### Concurrency의 problems

OS는 많은 일을 한번에 수행함

![01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%206.png](01-Introduction%20to%20Operating%20System%206791cdd11eb24c8f89bea270fb4a6a38/Untitled%206.png)

shared counter 변수를 증가시키는 건 아래 3개의 instruction으로 구성

1. Load the value of the counter from memory into register
2. Increment it
3. Store it back into memory

위의 3개의 동작이 atomically하게 실행되진 않으므로 concurrent 문제가 생김

### Persistence

DRAM(main memory)과 같은 device는 volatile(휘발성)

Hardware와 software는 data를 persistently하게 저장해야함

- hardware : I/O device such as HDD, SSD
- software : 
⇒ filesystem(hardware disk를 abstraction한 것)이 disk를 관리함
⇒ filesystem이 아무 file이나 store하는 것을 책임짐

disk에 write하기 위해서 OS가 하는 일이 무엇일까?

- 새로운 data가 reside할 곳을 disk에서 찾아냄
- 해당 storage device에 I/O request를 issue함

File system은 disk write 하는 동안 **system crashes를 handling함**

- journaling 또는 copy-on-write
- writes to disk를 ordering함

### 운영체제의 Design goal

- Build up abstraction
- Provide high performance ⇒ OS의 overhead를 최소화
- Application들 사이의 protection (isolation)
- High degree of reliability
- Energy efficiency, security, mobility