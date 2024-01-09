---
# Common-Defined params
title: "OS 모니터링"
date: "2021-06-08"
lastmod: "2021-06-08"
description: "우리는 무엇을 봐야 하는가"
categories:
  - "study"
  - "cloud" 
tags:
  - "SRE"
  - "모니터링"
menu: main # Add page to  menu. Options: main, footer

# Theme-Defined params
comments: true # Enable/disable Disqus comments for specific page
authorbox: true # Enable/disable Authorbox for specific page
toc: true # Enable/disable Table of Contents for specific page
tocOpen: true # Open Table of Contents block for specific page
mathjax: true # Enable/disable MathJax for specific page
related: true # Enable/disable Related content for specific page
meta:
  - date
  - categories
featured:
  #url: image.jpg # relative path of the image
  #alt: A scale model of the Eiffel tower # alternate text for the image
  #caption: Eiffel tower model # image caption
  #credit: Unknown author # image credit
  #previewOnly: false # show only preview image (true/false)
--- 
OS를 모니터링 합시다. 

흔히 OS의 상태를 모니터링하는것과 어플리케이션의 상태를 모니터링하는 것을 헛갈릴 수 있습니다. 

먼저 OS를 왜 모니터링 해야하는지 확인해 보겠습니다. 

OS는 하드웨어를 관리하고 소프트웨어를 실행하기 위한 플랫폼 입니다. 

OS를 통해서 우리는 하드웨어의 상태를 모니터링 할 수 있고, 소프트웨어 실행 정보를 모니터링 할 수 있습니다. 

OS를 모니터링 하는 방법을 찾아보면 적절한 툴을 찾아서 설치하고 화면을 구성하고 보는 것으로 되어 있습니다. 

하지만 우리의 화면은 제한적이고 최대한 많은 정보를 봐도 즉시 알기 어렵고, 무엇을 봐야하는지 모를 때가 많이 있습니다. 

모니터링시에 즉시적으로 장애를 내거나 성능에 영향을 줘서 장애로 발전되는 시점에 지표들은 변하기 마련입니다. 

성능에 관한 지표로는 CPU Load (Usage) , Memory Usage , DISK I/O , DISK space 를 먼저 살펴 보겠습니다. 

먼저 CPU 보겠습니다.

CPU를 모니터링 할 때 많이 하는 실수 중에 하나는 vCPU, 혹은 CPU CORE 의 갯수를 모두 합한게 100% 인지 각각의 core 가 100% 전체 합산이 

"100 x  코어갯수 = 전체 가용량" 으로 나타나는지 확인을 해야합니다. 

보통 top 명령어로 터미널 상에서 CPU load도 확인할 수 있고, 기타 여러가지 방법이 있지만 먼저 차이점을 하나씩 보자면 

[ec2-user]# top 

![02%20OS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%2084e83adf8b2b4572942eb51badc80cf4/_2021-06-03_09.46.04.png](/static/monitoring/2021-06-03_09.46.04.png)

일반적인 top 명령으로 사용륭을 보면 , 1.70 이니까, 170% 라고 보이지만 htop 명령으로 다시 확인해보면, 6번 코어 하나만 쓰리는 것을 볼 수 있습니다. 

그리고 나머지 core들은 일이 없거나 아주 작습니다. 

[ec2-user]# htop

![02%20OS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%2084e83adf8b2b4572942eb51badc80cf4/_2021-06-03_09.46.21.png](/static/monitoring/2021-06-03_09.46.21.png)

위의 top 명령어의 결과로 보면, 1.70으로 무섭게 보일 수 있습니다, 

하지만 htop 명령으로 확인해보면 16개의 코어 중에서 하나의 코어만 엄청 바쁘고 나머지는 아주 한가합니다. 이때 우리가 느끼는 감정은.. "에이.... 뭐야"라는 수준입니다. 

이런일이 일어나는 이유는  우리가 예상을 할 때 CPU LOAD는 다음과 같이 될 겁니다

1. (core N * 100) / core N ⇒  전체 CPU의 사용률을 100%로 표기 합니다
2. core N * 100 ⇒ 전체 사용률은 core 수 곱하기 100이 됩니다, 예로 16코어라면 1600%까지가 정상 범위가 됩니다 

      이것을 우리는 Irix(solaris)모드라고 합니다. 

다시 top으로 돌아가서 

숫자키 1을 눌러서 코어 별로 표기할 수 있도록 하면, 모든 코어가 100%에 육박해서 작업을 하는 것으로 볼 수 있습니다. 

![02%20OS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%2084e83adf8b2b4572942eb51badc80cf4/_2021-06-03_10.30.23.png](/static/monitoring/2021-06-03_10.30.23.png)

이렇게 보면 load average 의 16.06 을 다시 봐도 이상하지 않습니다. 1 core = 1.0 이라고 생각하면 , 16은 irix(solaris)모드로 정상적인 범위입니다. 

이상한게 아닙니다. 

그리고 이걸 다시  htop 명령으로 확인하면 

![02%20OS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%2084e83adf8b2b4572942eb51badc80cf4/_2021-06-03_10.15.42.png](/static/monitoring/2021-06-03_10.15.42.png)

엄청 바쁘게 보입니다. 16코어를 모두 꽉 채워서 사용하고 있습니다. cpu를 100% 사용한다는건, 전혀 불안한 일이 아닙니다.

물론 서버의 역할에 따라서 꽉 채워 쓰면 안되는 서버도 있지만, 일단 지금의 주제로는 100%는 위험한건 아닙니다. 

그렇다면 우리가 보통 모니터링 하는 서버의 경우에 어떻게 보일까요 

![02%20OS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%2084e83adf8b2b4572942eb51badc80cf4/_2021-06-03_10.36.12.png](/static/monitoring/2021-06-03_10.36.12.png)

일단 100%를 한계로 했기 때문에 100%에서 멈춰있는 것으로 보입니다. system hang 일지도 모르고 뭔가 엄청난 큰 작업이 걸려서 

CPU가 힘을 쓰지 못하는 것처럼 보일 수 있습니다. 

그런데 이 그래프는 지금 (core N * 100) / core N 공식으로 표기 되고 있습니다. 그런데 이럴 때는 그냥 CPU 많이 쓰는구나. 해도 됩니다. 

CPU가 서버를 다운시키는 일은 거의 일어나지 않습니다 CPU가 바쁘면 그냥 바쁜겁니다. 느린게 아니라 바쁜건데, 

바빠서 응답을 늦게 하는 상황이 발생할 수 있습니다. 그래도 불안하다면, 하드웨어의 다른 부분을 모니터링하거나 , 서버에 들어가서 

SSH 접속은 잘되는지, 포트는 열려있는지 확인해 주시면 됩니다. 

메모리 모니터링

메모리는 중앙처리장치와 보조기억장치 사이에 위치하며 실행되는 프로그램이 주기억공간이라고 일단 글로는 많이 배워왔습니다. 

디스크에 그 어떤것이 있어도 실행되려면 메모리에 있어야하고, 메모리에서 만들어진 데이터는 디스크로 써져야 일이 끝납니다. 

그런데 이 메모리 문제는 서비스를 중단시킬 수 있고, 많은 경우 범인으로 지목되기도 합니다. 

터미널에서 메모리의 상태를 보려면 우리는 free 명령어를 사용하고 , free 명령어의 칼럼의 뜻을 알아야 합니다. 

![02%20OS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%2084e83adf8b2b4572942eb51badc80cf4/_2021-06-03_15.07.17.png](/static/monitoring/2021-06-03_15.07.17.png)

free 명령어를 실행하면 total, used, free ,shared, buffers, cached 항목이 있고 각각의 뜻을 살펴보면

[total] : 설치된 물리 메모리 / 설정된 스왑의 크기

[used] : total - (free + buffers + cached ) , 사용중인 물리 메모리 / 사용중인 스왑의 크기

[free] : total - (used + buffers + cached), 실제 남아 있는 공간 / 남아있는 스왑 크기

[shared] : tmpfs(메모리 파일 시스템), ramfs 등으로 사용되는 공간, 프로그램끼리의 공유메모리 공간

[buffers] : 커널 버퍼

[cached] : 페이지 캐시 와 slab으로 사용중인 메모리

[available] : 스왑하지 않고 할당 가능한 메모리의 사이즈

어플리케이션의 종류 별로 메모리의 상태를 보겠습니다. 

![02%20OS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%2084e83adf8b2b4572942eb51badc80cf4/_2021-06-03_16.36.44.png](/static/monitoring/2021-06-03_16.36.44.png)

JVM 기반 어플리케이션 서버

EC2인스턴스의 스펙은 8GB이므로 1024로 계산을 해보면 시스템의 물리 메모리는 760Mib 가 됩니다. 

그래서 total이 7.5GB, 이고 free가 4.2GB 로 있기 때문에 USED는 7.5-4.2 = 3.3GB 정도를 사용하고 있습니다. 그리고 해당 서버의 어플리케이션은 실행시에 JVM heap 사이즈를 최소 1GB ,최대 2GB 를 사용하고 있습니다.  OS가 통상적으로 2GB를 점유한다고 보기 때문에 해당 어플리케이션은 1GB를 점유한다고 예상할 수도 있고, 실제로 확인을 해봐도 됩니다. 

실제 확인은 jmap -heap <pid> 명령으로 해당 JVM의 heap 정보를 확인할 수 있습니다. ⇒ 추후 추가 

그리고 Python 기반 서버를 한번 보겠습니다

![02%20OS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%2084e83adf8b2b4572942eb51badc80cf4/_2021-06-03_15.57.08.png](/static/monitoring/2021-06-03_15.57.08.png)

Python 배치서버

Python으로 만들어진 어플리케이션이 구동되는 배치 서버 입니다. 이 작업은 대용량 (30GB~90GB)의 xml  파일을 CPU core 수만큼 분리해서 json으로 변환하는 역할을 수행하는 서버입니다. 

하지만 메모리는 물리 메모리의 거의 한계까지 접근하게 됩니다.  하지만 서버는 죽지 않습니다. 

사실 결론적으로 CPU와 메모리는 서버가 갑작스레 잠시 멈추게는 할 수 있지만 장애로는 이어지지 않을 수 있습니다.
222222222222222222222
하지만 이제 살펴볼 디스크는 시스템을 멈추게하고 사용자의 개입이 필요한 경우가 많습니다. 

디스크 모니터링 

디스크가 시스템에 영향을 미치는건 몇가지 경우가 있습니다. 

- / (루트) 디스크 full
    - 물리적인 파일 용량이 100%로 오는 경우
        - 특정 디렉토리에서 특정 시간을 기준으로 파일을 삭제하여 시스템 정상화
    - inode가 최대 허용치에 도달한 경우
        - inode의 개수가 full이 되면, 먼저 우리가 알고 있는  diskspace full과는 다른 양상을 보입니다.
        - diskspace의 free 사이즈를 확인하면 100% 가 아닐 수 있습니다.
        - 이미 열려있는 파일과 관련한 연산은 정상적으로 수행중입니다.
        - 새로운 파일을 생성하지 못하며, diskspace 가 없다는 에러메시지가 발생합니다.
        - 새로운 파일을 생성하지 못해서, 어플리케이션이 crash를 발생 시키거나 , 멈추거나 합니다.
- 디스크 속도 지연
    - 하드웨어에 문제가 있어서 실제로 디스크가 느릴 수 있습니다.
    - 소프트웨어 문제로 의도적으로 혹은 의도치 않지만  큰 파일을 느리게 생성하거나 읽을 수 있습니다.

Prometheus와 node_exporter를 이용한 inode 모니터링 : [https://www.robustperception.io/filesystem-metrics-from-the-node-exporter](https://www.robustperception.io/filesystem-metrics-from-the-node-exporter)