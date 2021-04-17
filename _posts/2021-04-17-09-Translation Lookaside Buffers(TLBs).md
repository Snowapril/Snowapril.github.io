---
layout: post
title:  "09-Translation Lookaside Buffers(TLBs)"
date:   2021-04-17 14:38:27 +0900
categories: operating-system
tags: operating-system os computer-science
comments: true  
---

### The problem

- Address translation 이 너무 느림
⇒ simple linear page table이 memory lookup 비용을 두배로 늘임
- **Goal : Make address translation fast**

### TLB(Translation Lookaside Buffer) - HW cache

- MMU(memory management unit) chip의 일부분
- virtual-to-physical address translation을 위한 HW cache

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled.png)

### TLB Organization

TLB는 hardware상에서 구현되어 있음

- 한번에 제한된 갯수만큼의 page만 처리함
⇒ TLB 하나에 16~256 entry가 일반적
- 대개 fully associative method
- Replacement policy : **LRU (Least recently used)**
- TLB는 PFN뿐만 아니라 PTE 전체를 cache 해온다.

### Address Translation with TLB

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%201.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%201.png)

```cpp
VPN = (VirtualAddress & VPN_MASK) >> VPN_SHIFT;
(Success, TlbEntry) = TLB_lookup(VPN);
if (Success) { //When TLB Hit
	if (CanAccess(TlbEntry.ProtectBit)) {
		offset = VirtualAddress & OFFSET_MASK;
		PhysAddr = TlbEntry.PFN << PFN_SHIFT | offset;
		AccessMemory(PhysAddr);
	} else {
		RaiseException(PROTECTION_ERROR);
	}
} else {
	PTEAddr = PTBR + sizeof(PTE) * VPN;
	PTE = AccessMemory(PTEAddr);
	...
	TLB_Insert(VPN, PTE.PFN, PTE.ProtectBits);
	RetryInstruction();
}
```

### Accessing an array example)

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%202.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%202.png)

- VPN이 0~15까지 있으니 VPN은 $2^4$이고, page table에는 16개의 PTE가 존재함
- offset이 $2^4$이니, page의 크기가 16byte이다.
- VPN과 offset이 각각 4bit이니, address space가 8bit이다.

### Locality

- Temporal Locality : 최근에 access 했던 instruction 또는 data item이 가까운 미래에 다시 access할 가능성이 높음

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%203.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%203.png)

- Spatial Locality : 만약 program이 address x에 access했다면, 곧 x와 가까운 memory에 access할 가능성이 크다.

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%204.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%204.png)

### Who handles the TLB Miss?

x86 CISC에서는 hardware-managed TLB

- hareware는 page table이 memory에서 어디에 있는지 알고있어야함
- hardware가 page table로 가서 correct한 page-table entry를 찾아서 적절한 translation을 추출해서 instruction을 update하고 retry해야함

RISC에서는 software-managed TLB

- TLB Miss가 생기면, hardware는 exception을 발생시킴
⇒ TLB Miss에 대한 Trap handler code가 OS에 존재함

### TLB Control flow algorithm(OS Handled)

```cpp
VPN = (VirtualAddress & VPN_MASK) >> SHIFT;
(Success, TlbEntry) = TLB_Lookup(VPN);
if (Success) {
	if (CanAccess(TlbEntry.ProtectBit)) {
		offset = VirtualAddress & OFFSET_SHIFT;
		PhysAddr = (TlbEntry.PFN << PFN_SHIFT) | offset;
		Register = AccessMemory(PhysAddr);
	} else {
		RaiseExceptipn(PROTECTION_FAULT);
	}
} else {
	RaiseException(TLB_MISS);
}
```

### TLB Issue : Context switching

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%205.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%205.png)

Context switch마다 TLB를 flushing하는 것은 cost가 너무 높다

### To solve problem : address space identifier(ASID)

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%206.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%206.png)

### Another issue : share a page

Prcess 1과 Process2가 physical page 101을 공유할 때,

P1은 이 page를 10번 VPN에, P2는 50번 VPN에 저장할 수 있다

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%207.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%207.png)

Page공유도 간단함

### TLB Replacement Policy

LRU(Least recently used)

- 최근에 쓰지 않은 entry를 evict해버림
- LRU로 memory reference stream에서 localiy의 장점을 최대한 이용함

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%208.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%208.png)

- 총 11번의 TLB Miss

### TLB Performance

- TLB는 많은 performance problems의 원인임
⇒ Performance metric : hit rate, lookup latency
- Increase TLB reach(== #TLB entries * Page size)
⇒ Increase the TLB size
⇒ Increase the page size
- Use multi-level TLBs
- **TLB friendly하게 algorithm과 data structure를 작성하는게 매우 중요**
⇒ TLB hit를 최대한 많이

![09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%209.png](09-Translation%20Lookaside%20Buffers(TLBs)%2017548a69cc5345a9a2af72851107c196/Untitled%209.png)

### Summary

Dedicated on-chip TLB를 address-translation cache로 사용

- 대부분의 memory reference가 main memory에 있는 page table에 접근하지 않고 handling될 수 있음
- TLB는 locality를 추구하므로, temporal locality,와 spatial locality 신경씀