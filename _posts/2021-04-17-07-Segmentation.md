---
layout: post
title:  "07-Segmentation"
date:   2021-04-17 14:38:27 +0900
categories: operating-system
tags: operating-system os computer-science
comments: true  
---

### Inefficiency of base and bound approach

![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled.png)

- address space 내부에 **Big chunk of "free" space**
- "free" space가 physical memory도 차지함

### Segmentation

- Address space를 logical segments로 나눔
    - Segment는 그냥 address space에서 특정 길이의 연속된 부분
    - 각 segment가 address space의 logical entity와 상응됨
    - logically-difference segment : **code, heap, stack**
- 각 segment는 독립적임
    - 같은 process의 segment라도 physical memory의 다른위치에 놓임
    - grow or shrink
    - **segment마다 base and bound register가 각각 존재**

### Placing segment in physical memory

![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%201.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%201.png)

![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%202.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%202.png)

- Physical address = offset + base register address
⇒ segment 방식에서는 address space에서의 virtual address가 아니고, **각 segment의 시작지점에서의 offset을 사용함**
- offset = physical address - base register address

### Address translation on segmentation

![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%203.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%203.png)

- virtual address 100의 offset은 100이다
⇒ virtual address 100은 code segment에 속하므로, offset도 100임
⇒ virtual address 100의 physical address는 32KB + 100 = 32868

![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%204.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%204.png)

- virtual address 4200의 offset은 4200 - 4KB = 104 이다
⇒ virtual address 4200의 physical address는 34KB + 104 = 34920

### Segmentation fault or violiation

![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%205.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%205.png)

- 만약 위의 상황에서 heap의 end보다 큰 7KB 같은 illegal address를 reference하면, OS가 segmentation fault를 발생시킨다
⇒ hardware detects that address is **out of bounds**

### Referring to segment

- Explicit approach : virtual address의 상위 몇 bits로 address space를 segments로 chop-up 한다

![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%206.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%206.png)

example ) virtual address 4200  = (01000001101000)
⇒ segment bits : 01 == heap, offset인 1101000을 heap의 base register에서 더함

![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%207.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%207.png)

```cpp
// get top 2 bits of 14-bit virtual address
segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT
// now get offset
offset = (VirtualAddress & OFFSET_MASK);
if (offset >= Bounds[segment])
	RaiseException(PROTECTION_FAULT);
else
	PhysAddr = Base[segment] + offset;
	register = AccessMemory(PhysAddr);
```

### Referring to stack segment

- Stack은 거꾸로 사람
- Extra hardware support가 필요함
    - hardware가 segment가 어느 방향으로 자라는지 체크함
    ⇒ 1이면 positive direction, 0이면 negative direction

    ![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%208.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%208.png)

위의 segment register table로 virtual address 15KB를 referring 할 때,
⇒ 11110000000000, segment bit : 11, offset bit : 110000000000 = 3KB
⇒ 12bit offset이므로, max segment size가 4KB.
⇒ grows positive가 0이므로 correct negative offset = 3KB - 4KB
⇒ Physical address = 28KB + (-1KB) = 27KB

### Support for sharing

- Segment는 address space들 사이에서 shared될 수 있음
    - **Code sharing**은 오늘날에도 잘 사용됨
    ⇒ 메모장을 여러개 실행해도, 기반이되는 코드는 다 중복됨
    - hardware support에 의해 가능함
- sharing을 위해서는 protection bits를 위해 extra hardware support 필요
    - 몇몇 bits를 더 추가해서, permission (read,write,execute) 표시

    ![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%209.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%209.png)

### Find grained and Coarse grained

- Coarse-grained는 segmentation을 적은 수로 한다는 것
⇒ e.g. code, heap, stack
⇒ segmentation 하나하나가 크고 수가 적어서 관리는 편함
- Find-grained segmentation은 early-system에서 좀더 flexible한 address space를 허용했음
⇒ To support many segments, hardware support with a segment table is required
⇒ segment size가 작고 많아 유연하게 사용가능하지만 관리가 힘들다

### OS support : Fragmentation

- **external fragmentation** : physical memory의 little hole들이 새로운 segment를 할당하는 것을 어렵게 만듬
⇒ ex) 전체 free space를 합치면 24KB가 나오지만, contiguous 하지않아서 OS가 20KB request를 수행할 수 없음
- **compaction** : rearranging the existing segments in physical memory
    - compaction은 굉장히 cost가 비쌈
    ⇒ 1) Stop running process
    ⇒ 2) Copy data to somewhere
    ⇒ 3) Change segment register value

    ![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%2010.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%2010.png)

### Segmentation의 Pros & Cons

Pros

- address space의 sparse한 allocation이 가능함
⇒ stack과 heap이 independently하게 grow함
⇒ internal fragmentation이 없음. 이전에는 stack과 heap사이의 빈공간
- fast, easy, and well-suited to hardware
- easy to share segments
⇒ 동일한 memory translation을 base/bound pair에 적용 가능함
⇒ segment level에서의 code/data sharing (e.g. shared libraries)
- supports dynamic relocation of each segment

Cons

- 각 segment의 contents는 여전히 contiguously하게 존재해야함
⇒ **external fragmentation(base&bound에서는 internal)**
⇒ large segments를 위한 physical memory가 충분하지 않을 수 있음
- Segmentation으로는 여전히 충분한 만큼 flexible하지 않다
⇒ 만약 한 logical segment에서 large하지만 sparsely-used heap인 경우, 실제로 사용되는 메모리는 일부분이지만 전체 heap이 메모리에 모두 올라와있어야 한다는 것이 비효율적이다
⇒ ex) 전화번호부 프로그램의 경우, 모든 전화번호를 malloc()으로 할당했는데 실제 사용되는 전화번호가 몇개밖에 안되면 굉장히 비효율적

![07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%2011.png](07-Segmentation%20d44a82ab894a4603bc8dce2df8f143a9/Untitled%2011.png)