---
layout: post
title:  "Devstack을 이용한 OpenStack 환경 구축"
date:   2020-10-03 19:15:27 +0900
categories: openstack
tags: devstack openstack single-node undercloud ubuntu googlecloud gce
comments: true  
---
# Devstack을 이용한 openstack 설치

아래와 같이 Google Cloud에서 4core, 16GB RAM 스펙의 Ubuntu 18.04 VM을 생성했다.

![GCE Specc](https://snowapril.github.io/assets/img/post_img/gce-spec.png)  

Google Cloud VPC 네트워크에서 고정 IP 주소 아래와 같이 할당받았다.

Horizon 대시보드 접속을 위해서 Public IP가 필요한데, VM을 해제/할당할 때마다 google cloud 방화벽을 수정하는 번거로움을 덜 수 있다.

![GCE Dashboard](https://snowapril.github.io/assets/img/post_img/gce-dashboard.png)  

Devstack은 sudo가 허용된 non-root user 계정에서 실행되어야 한다.

아래의 명령어로 stack 사용자를 생성하고, sudo 권한을 부여하고, stack 계정으로 전환한다.

```
$ sudo useradd -s /bin/bash -d /opt/stack -m stack
$ echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
$ sudo su - stack
```

그리고 Devstack의 stable/pike 브랜치를 clone한다.

```
$ git clone -b stable/pike https://opendev.org/openstack/devstack
$ cd devstack
```

devstack/samples에 있는 local.conf를 상위 디렉토리로 복사하고, vim을 이용해 conf 파일 내부 비밀번호를 수정한다. 중요한 것은, cloud 환경을 이용하여 openstack 서버를 구축하였으므로, LOCAL\_IP에 VM의 public IP를 추가해줘야 한다.

```
$ cp samples/local.conf ./local.conf
$ vim ./local.conf
```

![local.conf](https://snowapril.github.io/assets/img/post_img/localconf.png)  

이제 devstack의 openstack 설치 자동화 스크립트인 stack.sh를 실행시켜 openstack 설치를 완료한다. 설치는 보통 15 ~ 20분이 소요된다.

```
$ FORCE=yes ./stack.sh
```

![devstack install](https://snowapril.github.io/assets/img/post_img/devstack-install.png)  

30분정도 경과된 후 설치가 완료되었다.

완료된 후 나오는 host IP address는 internal address로, dashboard는 http://**public-ip**/dashboard 에 접속하면 확인가능하다.

![Horizon](https://snowapril.github.io/assets/img/post_img/horizon.png)  

이후, openstack을 가지고 놀다가 의도치 않은 방향으로 흘러가 devstack 설치 이후 지점으로 복원하고 싶을 때를 대비하여 즉시 VM의 스냅샷을 떴다.

![Snapshot](https://snowapril.github.io/assets/img/post_img/snapshot.png)  

출처 : [https://docs.openstack.org/devstack/latest/](https://docs.openstack.org/devstack/latest/)