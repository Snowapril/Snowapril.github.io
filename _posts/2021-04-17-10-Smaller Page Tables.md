---
layout: post
title:  "10-Smaller Page Tables"
date:   2021-04-17 14:38:27 +0900
categories: computer-science
tags: operating-system os computer-science
comments: true  
---

### Paging : Linear Tables & Smaller Tables

system의 process 하나마다 page table을 하나씩 가지니 너무 커서 memory를 너무 많이 잡아먹음

- 32bit address space에서 4KB page와 4byte PTE 일 때
⇒ page가 4KB 라는 것은 offset이 12bit이고, 32bit address space이니 VPN이 20bit이다.
⇒ PTE 4byte가 $2^{20}$개 있으니 4MByte
- 32bit address space에서 16KB page와 4byte PTE일 때
⇒ page가 16KB이니 offset이 14bit이고, VPN은 18bit이다
⇒ PTE 4byte가 $2^{18}$개 있으니 1MByte

page size를 크게 하니 결과적으로 page table 크기는 줄었지만, **internal fragmentation**이 생긴다.

### Problem

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled.png)

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%201.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%201.png)

전체 virtual address space에서 **대부분의 Virtual address는 UNUSED 공간**임에도 통째로 page table로 만드니 낭비가 너무 큼  

### Hybrid approach : Paging and Segments

Segmentation

- virtual address를 segments로 divide 한다
- 각 segment는 variable한 length를 가짐 (base-and-bound)

Paging

- 각 segment를 fixed-sized page로 divide함
- 각 segment마다 page table이 존재함
- 각 segment는 base register와 segment의 bound(limit) 를 tracking 한다
- base register를 segment 자체를 가리키는게 아니라, **그 segment에 해당하는 page table의 physical address를 가리킴**
- bounds register는 page table의 끝을 가리키는데 사용함

### Simple example for hybrid approach

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%202.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%202.png)

### TLB Miss on Hybrid approach

Hardware가 page table으로부터 physical address를 가져옴

```cpp
SN = (VirtualAddress & SEG_MASK) >> SEG_SHIFT;
VPN = (VirtualAddress & VPN_MASK) >> VPN_SHIFT;
PTEAddr = Base[SN] + sizeof(PTE) * VPN;
```

### Problem of hybrid approach

- 만약 엄청 큰 heap을 할당했는데 sparsely used라면, 결국에 또 page table을 낭비하는 꼴이 된다
- Segment를 variable size로 divide하여 또 external fragmentation 발생함

### Multi-level Page Tables

- Linear page table을 tree 같은 형태로 구성함
    - page table을 page-sized units으로 chop-up
    - 만약 page-table entries의 모든 page가 invalid하다면, 그 page에 대한 page table은 할당하지 않는다
    - page-table의 page가 valid 하다면, 새로운 구조체, **page directory**를 사용함

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%203.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%203.png)

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%204.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%204.png)

### Multi-level Page Tables : Page Directory entries

Page directory는 page-table의 page마다 하나의 page directory entry를 가진다

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%205.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%205.png)

- PDE는 valid bit와 PFN을 가진다.

### Multi-level Page tables의 Pros & Cons

Pros

- 사용하는 address space의 크기에 비례해서 page-table을 할당한다
⇒ space를 훨씬 절약
- OS는 page table을 grow하거나 할당해야 할 때, 그냥 next free page를 가져옴 ⇒ management가 편하다

Cons

- Multi-level table은 time-space trade-off의 한 예이다.
⇒ Page directory까지 읽어야해서 reference를 한번 더함
- **복잡하다**

### Multi-level page table : Level of indirection

- 원래는 PTBR이 page table을 직접 가리키고 있었지만, multi-level page table 방식은 PTBR이 page directory를 가리키고, page directory entry가 page table을 가리키는 indirection이 생김

### Detailed Multi-level example)

- Address space 16KB = $2^{14}$byte = 14bit virtual address
- Page size 64byte = 6bit offset
- VPN : 14 - 6 = 8bit
- offset = $2^6$
- Page table entry : page 개수만큼 있으므로, 256개의 PTE
- Page directory는 page-table의 page 개수 만큼 entry가 필요함
⇒ PTE size가 4byte라고 가정하자
⇒ page table의 크기는 4byte * 256 = 1KB가 된다
⇒ page의 크기가 64byte이므로, $2^{10}/2^6=2^4$로, Page마다 16개의 PTE를 가질 수 있음 : VPN으로 4bit가 필요하다
- page-directory entry가 invalid하면 raise exception

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%206.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%206.png)

만약 PDE가 valid 하다면, 이 PDE가 가리키는 page table의 page에서 PTE를 fetch 해봐야함. 즉, 위의 그림에서 PDE가 valid하면 그 PDE의 PFN으로 page table에 가서 VPN의 하위 4bit로 indexing하고 offset을 적용한다.

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%207.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%207.png)

### More than two level

좀더 deep한 tree를 구성하는 것도 가능하다

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%208.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%208.png)

- Virtual address 30 bit
- VPN : 21bit ⇒ $2^{21}$개의 page, PTE
- offset : 9bit ⇒ $2^9$byte = 512byte page size

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%209.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%209.png)

- PTE size를 4byte라고 가정하면, page size가 512 byte이므로, page마다 128개의 PTE가 존재함. 이를 indexing 하기 위한 VPN이 7bit 필요
- Page directory index가 14bit로, $2^{14}$개의 entry를 가지는데, PDE가 4byte라고 하더라도 $2^{16}$이고, 이는 page size 512byte에 다 못들어감
- 이걸 해결하기 위해서 tree의 level을 한단계 더 deep 하게 들어감

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%2010.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%2010.png)

### Multi-level page table control flow

```cpp
VPN = (VirtualAddress & VPN_MASK) >> VPN_OFFSET;
(Success, TlbEntry) = Tlb_Lookup(VPN);
if (Success) {
	if (CanAccess(TlbEntry.ProtectBit)) {
		offset = VirtualAddress & OFFSET_MASK;
		PhysAddr = TlbEntry.PFN << PFN_SHIFT | offset;
		Register = AccessMemory(PhysAddr)
	} else {
		RaiseException(PROTECTION_FAULT);
} else {
	PDIndex = (VPN & PD_MASK) >> PD_SHIFT;
	PDEAddr = PDBR + PDIndex * sizeof(PDE);
	PDE = AccessMemory(PDEAddr);
	if (PDE.Valid == false) {
		RaiseException(SEGMENTATION_FAULT);
	} else {
		PTIndex = (VPN & PT_MASK) >> PT_SHIFT;
		PTEAddr = (PDE.PFN << SHIFT) + PTIndex * sizeof(PTE);
		PTE = AccessMemory(PTEAddr);
		if (PTE.Valid == false)
			RaiseException(SEGMENTATION_FAULT);
		else if (PTE.ProtectBit == false)
			RaiseException(PROTECTION_FAULT);
		else {
			TLB_Insert(VPN, PTE.PFN, PTE.ProtectBits);
			RetryInstruction();
		}
	}
}
```

### Inverted Page Tables

- 이때까지는 각 process마다 page table을 가졌는데, 각각의 physical page가 어느 virtual page에 mapping 되어있는지 정보를 가지고 있음
- 각 entry가 어떤 process가 이 page를 사용하고 있는지, 그 process의 어떤 virtual page가 이 physical page에 mapping 되어있는지 담고있음
- 어떤 process의 VPN을 변환하려면 이 inverted page table을 전체 search하며 일치하는것을 찾아야함
- hashing을 활용하기도 함

Pros & Cons

- page table을 저장하는데 필요한 memory를 극적으로 줄일 수 있다
- TLB Miss 일 때 inverted page table을 전체 다봐야해서 시간이 늘어남

![10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%2011.png](10-Smaller%20Page%20Tables%20032cb87b951f41cdadd8768ee391ecda/Untitled%2011.png)

### Summary

To smaller tables

- bigger page size 
⇒ table은 작아지지만 internal fragmentation
- hybrid approach
⇒ paging with segmentation
⇒ 근데 variable sized segment로 인해 external fragmentation
⇒ 큰 heap을 할당했을 때 UNUSED가 많아 낭비됨
- Multi-level page tables
- **모두 time & space tradeoff는 존재한다**