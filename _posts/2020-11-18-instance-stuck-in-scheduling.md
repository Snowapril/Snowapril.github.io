---
layout: post
title:  "OpenStack Instance stuck in Scheduling"
date:   2020-11-18 19:59:27 +0900
categories: openstack
tags: devstack openstack single-node undercloud ubuntu googlecloud gce nova
comments: true  
---
Openstack에서 Private network의 instance와 ssh 연결 테스트를 위해 demo tenant 위에 cirros instance를 생성 중에, **openstack server create **명령어를 넣은지 꽤 되었음에도, instance의 status가 BUILD로 찍히고 있다.

![instance-build-stuc](https://snowapril.github.io/assets/img/post_img/instance-build-stuc.png)  

Cirros image이므로 setup에 그리 오래 걸리지 않을터인데 한참동안 BUILD status에 멈춰있고, horizon dashboard에서 봐도 SCHEDULING에서 계속 로딩 중이다. 

서비스 목록에서 nova 관련 서비스의 상태를 출력해보니 아래와 같다.

![openstack-service-list-instance-build](https://snowapril.github.io/assets/img/post_img/openstack-service-list-instance-build.png)  

n-cond-cell1, n-sch, n-super-cond 3개의 nova service가 failed 상태에 있었다. 

일단 생각나는 조치로 서비스 재시작을 해주었더니 예상외로 별 문제없이 복구되었다.

```
sudo systemctl restart devstack@n*
```

![openstack-service-list-instance-build-again](https://snowapril.github.io/assets/img/post_img/openstack-service-list-instance-build-again.png)  

이후 다시 인스턴스를 만들었더니 별 이상없이 잘 동작한다.

![openstack-build-stuck-solved](https://snowapril.github.io/assets/img/post_img/openstack-build-stuck-solved.png)  