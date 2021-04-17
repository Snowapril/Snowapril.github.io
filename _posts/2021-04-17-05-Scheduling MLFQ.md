---
layout: post
title:  "05-Scheduling MLFQ"
date:   2021-04-17 14:38:27 +0900
categories: operating-system
tags: operating-system os computer-science
comments: true  
---

### General CPU Scheduler

Goals 

- Optimize turnaround time, Minimize response time

Challenge

- No priori knowledge on the workloads

어떻게 scheduler가 jobs의 특성을 학습하고 더 나은 결정을 할수있을까?

- Past로부터 학습하고 Future를 예측함

### MLFQ(Basic rules)

MLFQ는 몇개의 distinct한 queue를 가짐

- 각 Queue는 다른 priority level을 가짐

![05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled.png](05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled.png)

ready to run인 job이 single queue에 있을 때

- higher queue(우선순위가 높은 queue)에 있는 Job이 선택됨
- 같은 Queue에 있는 job들은 RR 방식으로 scheduling 됨
- **Rule1) If Priority(A) > Priority(B), A runs**
- **Rule2) If Priority(A)==Priority(B), A&B run in RR**

MLFQ는 observed behaviour로 jobs의 priority를 다르게 함

일반적인 workload는 interactive job과 CPU-intensive job이 섞여있음

interactive jobs

- short-running, require fast response time
- repeatedly relinquishes the CPU while wating IOs
- **Keep its priority high**

CPU-intensive jobs

- CPU를 intensively 하게 long periods 사용함
- response time은 별로 신경 안씀
- **Reduce its priority**

![05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%201.png](05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%201.png)

### MLFQ : How to change priority

MLFQ Priority adjustment algorithm:

- **Rule 3) Job이 system에 도착하면 highest priority에 놓음**
- **Rule 4a) Job이 time slice 전체를 사용하면, priority를 하나 낮춤**
- **Rule 4b) Job이 time slice를 다 사용하기 전에 CPU를 양도하면 우선순위 그대로 둠**

이런 방식으로 MLFQ는 SJF를 approximate 함

### Ex1) Single long-running job

![05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%202.png](05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%202.png)

### Ex2) Long running CPU-intensive job and Short interactive job

![05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%203.png](05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%203.png)

### Ex3) Long running CPU-intensive job과 I/O를 많이하는 Short interactive job

![05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%204.png](05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%204.png)

### **Problems with basic MLFQ**

- **Starvation**
⇒ system에 interactive job이 너무 많으면, Long-running jobs은 CPU time을 못받음
- Game the scheduler (스케줄러 특성을 악용)
⇒ 일부로 Time-slice를 99%만 쓰고 I/O operation issue하는 식으로 악용
- Program may change its behaviour over time
⇒ CPU bound process가 I/O bound process로 바꿈

### **The Priority Boost**

- **Rule5) After some time period S, 모든 jobs을 system의 가장 높은 queue로 옮김**

![05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%205.png](05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%205.png)

Gaming of scheduler는 어떻게 방지할까?

Rule4a)와 Rule4b)를 합침 **Gaming tolerance**

- Rule4) Job이 해당 Level에서의 time allotment를 다 쓰면 priority를 낮춤

![05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%206.png](05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%206.png)

### Tuning MLFQ and other issues

- Lower priority, Longer Quanta (priority마다 다른 time-slice 시간)

![05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%207.png](05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%207.png)

### MLFQ : Summary

![05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%208.png](05-Scheduling%20MLFQ%20c4558ae88d794ef191e6119ec3b4becf/Untitled%208.png)