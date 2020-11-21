---
layout: post
title:  "Cinder Volume Service state down 해결하기"
date:   2020-10-03 19:25:27 +0900
categories: openstack
tags: devstack openstack single-node undercloud ubuntu googlecloud gce cinder
comments: true  
---
# OpenStack cinder.volume down 해결

User VM 생성 테스트를 하던 도중, instance 생성중에 Volume 생성에서 error가 발생했다.

그래서 OpenStack CLI에서 volume 목록을 확인해보니, 아래와 같이 status에 error가 찍혀있다.

![openstack-volume-list](https://snowapril.github.io/assets/img/post_img/openstack-volume-list.png)  

openstack volume service list 명령어로 cinder-volume service을 확인해보니까 아래처럼 상태가 down..

![openstack-volume-service-list](https://snowapril.github.io/assets/img/post_img/openstack-volume-service-list.png)  

바로 systemctl 목록에서 cinder.volume의 service 이름을 확인하고 재시작을 해봤는데, 잠깐 state가 up으로 돌아왔다가 다시 down됐다.

![openstack-service-list](https://snowapril.github.io/assets/img/post_img/openstack-service-list.png)  
![openstack-volume-service-list-after-restart](https://snowapril.github.io/assets/img/post_img/openstack-volume-service-list-after-restart.png)  
![openstack-volume-service-list-die-again.png](https://snowapril.github.io/assets/img/post_img/openstack-volume-service-list-die-again.png)  

journalctl 명령어를 통해 cinder.volume의 log를 확인해봤더니, **lvmdriver-1 is uninitialized** 라는 debug 내용이 보인다.

![journalctl-cinder](https://snowapril.github.io/assets/img/post_img/journalctl-cinder.png)  

devstack을 설치하면 **stack-volumes-lvmdriver-1**라는 볼륨 그룹이 생성된다고 한다. 하지만, vgdisplay를 통해 확인해본 결과 모종의 이유로 생성이 안된 것을 확인할 수 있다.

![vgdisplay](https://snowapril.github.io/assets/img/post_img/vgdisplay.png)  

default와 lvmdriver-1의 backing file을 이용하여 볼륨 그룹을 직접 생성해준다.  
이후, c-vol.service를 재시작 해준 다음 openstack volume service list를 통해 cinder-volume의 state가 up으로 돌아온 것을 확인할 수 있다.

![cinder-solved](https://snowapril.github.io/assets/img/post_img/cinder-solved.png)  

그리고 다시 journalctl 명령어로 c-vol.service에서 생성되는 로그를 한동안 지켜봤는데, 아까와 같은 lvmdriver-1 is uninitialized 와 같은 로그는 보이지 않고, cinder-volume의 state가 up으로 지속되는 것으로 보아 순조롭게 해결된 것으로 보인다.

![cinder-solved-result](https://snowapril.github.io/assets/img/post_img/cinder-solved-result.png)  

reference :

[devstack环境问题：cinder-volume服务状态down · Issue #16 · Xphobia/blog](https://github.com/Xphobia/blog/issues/16)