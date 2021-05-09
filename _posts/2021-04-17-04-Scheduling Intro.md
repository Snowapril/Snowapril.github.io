---
layout: post
title:  "04-Scheduling Intro"
date:   2021-04-17 14:38:27 +0900
categories: computer-science
tags: operating-system os computer-science
comments: true  
use_math: true
---

### CPU Scheduling

runnable processes에서 다음에 실행할 process를 결정하는 **Policy**

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled.png)

### Basic approaches

**Non-preemptive(비선점형) scheduling**

- 실행중인 process가 CPU를 내놓을 때까지 scheduler가 기다림
- Process가 협력적이어야함

**Preemptive scheduling(선점형) scheduling**

- 모든 현대 scheduler들은 대부분 선점형임
- Scheduler가 process를 interrupt(timer interrupt)시키고 context switch를 강요할 수 있음

### Terminology

- **Workload** : set of job descriptions (e.g. arrival time, run time, etc..)
- **Scheduler** : logic that decides when jobs run
- **Metric** : measurement of scheduling quality (turnaround time, response time..)

# Scheduling Introduction

### Workload assumption

1. 각 Job이 동일한 시간만큼만 수행함
2. Job들이 똑같은 시간에 도착함
3. 한번 시작하면 끝까지 수행함
4. Job들이 CPU만 사용함(I/O 작업 안함)
5. 각 job의 run-time을 미리 알고있음

### Metric

- **Performance Metric : Turnaround time**
⇒$T_{turnaround}=T_{completion}-T_{arrival}$
- **Fairness** : CPU를 각 Job들이 얼마나 공평하게 나눠서 사용하는가
⇒Performance와 Fairness는 종종 상충됨

### 1) FIFO

Frist come, First served (FCFS)

- simple and easy to implement
- Real-world scheduling 방식 (supermarket)
- 비선점형(Non-preemptive)
- Job들이 공평하게(Fairness) 다뤄짐 : **no starvation**

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%201.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%201.png)

 Average turnaround time = $[(10-0)+(20-0)+(30-0)]/3=20sec$

### Why FIFO is bad ?

**Convoy effect** : 실행시간이 짧은 process들이 긴 process의 종료를 마냥 기다리는 현상

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%202.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%202.png)

 Average turnaround time = $[(100-0)+(110-0)+(120-0)]/3=110sec$

### 2) Shortest Job First (SJF)

assumption (1), 각 Job들이 동일한 시간만큼 수행된다는 가정 완화(relaxed)

Non-preemptive scheduler임

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%203.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%203.png)

Average turnaround time = $[(10-0)+(20-0)+(120-0)]/3=50sec$

**assumption(2), 각 job들이 동시에 도착한다는 가정을 완화하면,**

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%204.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%204.png)

Average turnaround time = $[(100-0)+(110-10)+(120-10)]/3=103.33sec$

**또 Convoy effect가 발생했음**

### 3) Shortest Time to Completion First(STCF)

**SJF에 Preemptition(선점형)을 더했음**

- assumption(3), 한번 시작하면 끝날때까지 CPU 독점한다는 가정을 완화
- Preemptive Shortest Job First(PSJF) 라고 불리기도 함

새로운 Job이 system에 도착하면,

- remaning jobs과 new job 중에 남은 실행시간이 짧은걸 선택

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%205.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%205.png)

Average turnaround time = $[(120-0)+(20-10)+(30-10)]/3=50sec$

### 새로운 Metric : Response Time

Job이 도착하고, 처음 scheduled되기까지의 시간

$T_{response}=T_{firstrun}-T_{arrival}$

### 1) Round Robin (RR) scheduling

**Time slicing scheduling**

- Job을 time slice만큼 실행하고 next job으로 switch하면서 run queue에 있는 job들이 다 끝날때까지 반복함
⇒ run queue : circular FIFO queue
⇒ Time slice는 scheduling quantum이라고 불리기도 함
- **Time slice의 길이는 timer-interrupt 주기의 배수여야함**
- Preemptive, No starvation
- **RR은 fairness(good at response time)이지만, performance(turnaround time)은 대게 안좋음**

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%206.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%206.png)

$T_{averageturnaroundtime}=[(5-0)+(10-0)+(15-0)]/3 =10sec$

$T_{average response}=[(0-0)+(5-0)+(10-0)]/3 =5sec$

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%207.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%207.png)

$T_{averageturnaroundtime}=[(13-0)+(14-0)+(15-0)]/3 =14sec$

$T_{average response}=[(0-0)+(1-0)+(2-0)]/3 =1sec$

### Time slice의 길이가 Critical함

Shorter time slice

- better response time
- context switch의 비용이 전체 성능을 dominate 함

Longer time slice

- Worse response time
- context switch의 비용을 Amortize(상쇄)
- time slice 길이 정하는게 system designer에게 trade-off임

### Incorporation I/O

assumption(4) 을 완화해서 모든 program이 I/O를 수행한다고 하자

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%208.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%208.png)

이렇게 Job을 분할해서 CPU를 할당하는 것 보다는 아래처럼

![https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%209.png](https://snowapril.github.io/assets/img/post_img/04-Scheduling%20Intro%203ba9c2ac1b4641afb10edd727ee7ab56/Untitled%209.png)

B가 I/O를 수행할 때는 다른 Job에게 CPU 제어권을 넘기는게 더욱 효율적임

Job 이 I/O request를 하면,

- I/O completion까지 Blocked state임
- Scheduler는 다른 job을 CPU로 schedule해야함

I/O가 끝이나면,

- Interrupt가 raised 됨 (I/O가 끝난 Process)
- 그러면 OS가 그 process를 blocked 에서 ready state로 바꿔줌