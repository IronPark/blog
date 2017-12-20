---
title: "Kubernetes는 뭔가요?"
date: 2017-12-20T22:27:04+09:00
draft: false
categories: ["Development","GCP"]
tags: ["kubernetes","docker","vm","cloud"]
---
부제 : 클라우드를 항해하는 히치하이커를 위한 안내서

![kubernetes 로고](/img/kubernetes.jpg)
## Kubernetes는 "조타수" 다
Kubernetes(쿠버네티스) 는 그리스어로 조타수를 뜻하는데. Docker가 각종 컨테이너를 얹은 선박이라면 Kubernetes는 **전체 선박을 총괄지휘하는 일등 항해사** 라고나 할까? 그래서 그런지 Kubernetes가 하는 일을 Container Orchestration이라고 표현하곤 한다.

## 하는일이 뭐야?
> Kubernetes is an open-source system for automating deployment, scaling and management of containerized applications that was originally designed by Google and donated to the Cloud Native Computing Foundation. [wikipedia](https://en.wikipedia.org/wiki/Kubernetes)

위키피디아에서는 " *Kubernetes Google이 설계하여 Cloud Native Computing Foundation에 기증한 컨테이너 응용 프로그램의 배포, 확장 및 관리를 자동화하는 오픈 소스 시스템* "이라고 정의하고 있다.

사실 사전적 정의는 그 안에 함축되어있는 표현이 많이 들어있기 때문에 관련 사전지식이 충분치 않은 경우 크게 와닿지 않을 것이다.

## 가상화 시스템
![vmware 로고](/img/vmware-logo.png)
Kubernetes가 뭔지에 대해 알려면 먼저 컨테이너 시스템과 Docker에 대한 이해가 필요한데. 몇 년 전부터 빠르게 퍼진 새로운 가상화 형식으로 서버를 관리하는 새로운 형식으로 이해하면 될 듯하다.

한 운영체제에서 격리된 환경을 만들려고 하면 **전/반** 가상화를 통해 서로 격리된 VM(가상머신) 인스턴스를 만들어 각 인스턴스에 알맞는 운영체제를 설치하고 관리했다. 간단히 예를 들자면 리눅스 위에 게스트 OS 형태로 여러 리눅스를 동작시키는 형태로..

이렇게 운영체제 위에 새로운 운영체제를 가상화된 시스템 위에 설치를 하는 일은 성능적인 면에서 **필시 비용 효율적일 리가 없을진대 슬기롭던 호모 사피엔스 사피엔스 들은 어째서? 왜? 이런 일을 저질렀단 말인가..**

바로 한 시스템 위에서 각종 시스템을 같이 운영할 때 종종 일어나는 다양한 형태의 **사이드-이펙트 를 피하기 위해서이다.** 다양한 프로그램을 노트북에서 어느 정도 개발을 하다 보면 특정 라이브러리를 업데이트했다가 생각지 못했던 다른 프로그램의 다른 라이브러리에서 오류를 뿜어낸다거나, 특정 프로그램과 생각지도 못한 콜라보레이션 으로 메모리 충돌을 일으켜 시스템이 다운된다거나.. 여러 가지 형태로 **당신의 컴퓨터에게 괴롭힘을 당한 적이 있을 것이다.**

## Docker와 컨테이너
![docker 로고](/img/docker-facebook.png)

Docker를 상용하는 목적 또한 위의 시나리오에 한정한다면 거의 90% 이상 아니 100% 일치한다. 그렇다면 대체 기존 가상화 시스템과 무엇이 그렇게 다르길래 사람들이 많이 사용하고, 언급하는 것 일까.

![vmware 로고](/img/what-is-docker.png)

그것은 바로 컨테이너 시스템으로 기존의 VM(가상머신)을 이용한 시스템에서는 가상화/에뮬레이션된 하드웨어 환경 그 위에서 동작하는 Guest 운영체제 그리고 또다시 그 위에서 각종 서비스를 구현했다면, Docker 는 같은 Host 위에서 얇은 추상 계층을 제공하여 **"개발된 서비스가 포함된 운영 환경" 만을 소프트웨어적으로 격리해버린다.** 이때 이 운영 환경을 **"컨테이너"** 라고 지칭한다.

당연히도 이런 방식은 VM을 이용하는 방식보다 가볍고 빠를 수밖에 없다. 효율적인 컨테이너 시스템의 개발은 서비스를 배포하는 방식 또한 효율적으로 바꾸어 버리는데. 그냥 가볍게 새로운 컨테이너 이미지를 docker 서버에 배포(교체)하는 방식으로 업데이트를 진행하는 것으로.

## 다시 Kubernetes로

그래 Docker가 좋은 것은 알겠는데 그럼 거기서 Kubernetes는 또 왜 튀어 나오는 것인가? 앞서 말했듯 Docker는 하나의 호스트 위에서 다양한 컨테이너를 실어서 사용한다. 그렇다면 하나의 호스트가 아닌 2개 이상 아니 100개 1000개 이상의 호스트 시스템 위에 각각 Docker 시스템이 설치되어 운영되어야 하는 상황을 생각해 보자.

컨테이너를 하나,둘 운영하다 보면 특정 호스트에 여유 자원 (CPU,램 등.)이 부족한 경우가 생기고 이런 경우 다른 비교적 자원이 풍부한 호스트 위에 컨테이너를 배포해야 할 것이다.

그런데 새로운 컨테이너를 배포 할 때마다 호스트별 자원 상황을 확인하고 배포하는 작업을 일일이 수동으로 한다면 그것은 서버를 운영하는 입장에서는 지옥과 같을 것이다. 그래서 이런 것들을 자동으로 관리해주어 **우리 서비스의 순항을 돕는 똑똑한 조타수가 Kubernetes다.**

## 그것만?

똑똑한 조타수가 하는 일이 각각의 선박에 최대 적재량을 계산하여 컨테이너를 쌓는 일을 하는 것 뿐이라면 우리의 순항을 돕는다고 할 수 있을까? 과적재를 막는 것이라면 차라리 안전관리 요원이나 선박 매니저라는 표현이 더 어울릴 텐데 말이다.

사실 Kubernetes가 하는 일은 실로 만능이라고 할 수 있는데. 간단히만 살펴보자면 다음과 같다.

1. **[Automatic binpacking]**
   - 유휴 자원등을 계산하여 자동 컨테이너 배치
2. **[Self-healing]**
   - 컨테이너의 생사 유무를 판단하여 자동 재시작
3. **[Horizontal scaling]**
   - CPU등의 자원을 계산하여 수평적으로 컨테이너 확장/축소
4. **[Service discovery and load balancing]**
   - Round-robin 형식의 로드벨런싱 제공
5. **[Automated rollouts and rollbacks]**
   - 무중단 서비스 배포
6. **[Secret and configuration management]**
   - 보안이 중요한 요소를 숨긴 상태에서 배포 및 업그레이드


이외에도 여러 Docker Pool 을 운용하고 있을때 대부분의 귀찮은 일을 대신 처리해주는데 이쯤이면 **똑똑한 조타수 라고 할 수 있겠다.** (아니 충직하고 일잘하는 하인인가?)

다음에는 GCP (Google Cloud Platform) 에서 Kubernetes 를 사용하는 방법에 대해 알아보자.


## 참고자료
- https://blog.2dal.com/2017/03/07/kubernetes/
- https://en.wikipedia.org/wiki/Kubernetes
- https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html
- https://www.docker.com/what-container
- http://www.giljae.com/2016/06/kubernetes.html
