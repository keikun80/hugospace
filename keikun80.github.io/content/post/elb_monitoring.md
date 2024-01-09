---
# Common-Defined params
title: "AWS ELB 모니터링"
date: "2021-05-21"
lastmod: "2021-05-21"
description: "우리는 무엇을 봐야 하는가"
categories:
  - "study"
  - "cloud" 
tags:
  - "SRE"
  - "모니터링"
menu: main # Add page to a menu. Options: main, footer

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
### 목적 

우리는 무엇을 봐야 하는가. 
우리는 무엇을 "위해서" 모니터링을 하는지 고려를 해봐야 합니다. 
목적이 없다면 단지, CPU ,RAM, Network IO , DISK IO 등의 기계적인 수치를 보는 것을 모니터링이라고 믿을 수 있습니다.
모니터링의 목적을 대부분의 사람에게 물어본다면 , 장애예방, 장애탐지, 다운타임 최소화, 의사결정, 자동화 등 이 목적이라고 할 것입니다. 
모니터링의 목적에 따라 모니터링을 하는 대상과 방법이 달라져야 합니다. 

### 대상

모니터링의 목적에 따라서 하드웨어와 소프트웨어를 분리해서 모니터링 해야합니다.
장애 탐지를 위해서는 장애 상황을 가정해서 어떤 지표를 봐야할지 고려를 해야 합니다. 
저는 여기서 API의 정상적인 작동이 저의 목표 입니다. 그래서 API의 정상적인 작동을 모니터링을 하는 것이 목적입니다. 
API가 서비스 되는 과정을 그려보겠습니다. AWS의 Best Paractice 에 약간의 상황을 더합니다. 
![api-gateway-sample.png](/static/monitoring/api-gateway-sample.png)
외부에서 통신을 할 수 있도록 Elastic IP (Static IP)를 갖는 network load balancer (이하, NLB) 와 3개의 path 조건을 갖는 Application Load Balancer (이하, ALB)에 연결되어 있는 2대의 API gateway EC2 인스턴스를 모니터링 한다고 가정하겠습니다. 

* 목적 : API의 정상적인 운영
* 목표 : API의 장애 탐지 시간 줄이기
* 시스템 구성 : NLB, ALB, API Gateway, ALB, API Container, AWS AuroraDB,  외부 서비스


장애를 탐지하고 싶고 그 탐지된 장애가 어디에서 발생한 것인지 알아야 할 때, 봐야 하는 곳과 내용이 서로 다릅니다. 

1. Network Load Balancer  : 전체 트래픽의 In/Out을 확인 할 수 있습니다. 
2. Gateway Load Balancer : 외부에서 들어온 트래픽이 context-path에 따라서 분기됩니다.
3. API gateway : 트래픽에 따른 하드웨어의 처리량을 모니터링 할 수 있습니다. 
4. Api Load Balancer : 지정된 context-path에 따라 분기됩니다. 
5. API service (ECS) : 컨테이너의 개수 , CPU,RAM등의 하드웨어 리소스 확인 
6. AuroraDB : API가 사용하는 Database 모니터링
7. External Link : 외부 연결 리소스에 대한 모니터링

## AWS ELB 모니터링

먼저 Network Load Balancerd에서 확인 할 수 있는 메트릭 중에서 network IO를 확인 할 수 있습니다. 

- metric : NetworkELB > processbytes

    ![2021-05-24_15.39.47.png](/static/monitoring/_2021-05-24_15.39.47.png)

    OPENAPI EDGE NLB의 processed Bytes

Metric으로 processbytes를 선택한 이유는 , edge NLB를 통해서 전송되는 모든 트래픽은 API service가 주고 받은 모든 데이터의 트래픽과 비슷해야 하고, 요청이 많아 질수록 

응답량도 많아지기 때문에 트래픽의 전체 추이를 볼 수 있습니다. 

그 다음으로 바로 연결되어 있는 ALB를 모니터링 할 수 있습니다. 

- metric : ApplicationELB > requestCount

![_2021-05-24_15.40.02.png](/static/monitoring/2021-05-24_15.40.02.png)

OPENAPI ALB의 REQUEST 카운트

위의 그래프로 보면 NLB의 처리량과 ALB의 처리량은 비례관계에 있음을 알 수 있습니다. 

그렇다면 API의 정상적인 작동을 확인하려면 우리는 API 가 정상의 경우에 2XX, 3XX, 4XX를 리턴할 것이고,  장애가 생긴 경우 5XX를 리턴할 것이라는 것을 알고 있습니다. 

([HTTP 코드](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) 참조)

그렇다면 하나의 Metric이 더 필요합니다. 위와 같이 처리건수와 함께 HTTP코드별로 건수를 확인하면 API의 정상작동을 알 수 있을것 같습니다. 

- Metric : ApplicationELB > HTTPCode_target_2xx , HTTPCode_target_4xx, HTTPCode_target_5xx

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_15.43.23.png](/static/monitoring/2021-05-24_15.43.23.png)

OPENAPI ALB의 코드별 응답 수

그림 "OPENAPI ALB의 REQUEST 카운트"와 그림 "OPENAPI ALB의 코드별 응답 수" 를 비교해보면 대부분의 요청은 2xx 코드를 리턴하는 것을 알 수 있습니다. 

여기까지보면 API Gateway는 정상적으로 작동되는 것을 볼 수 있습니다.  혹은 target group 별로 Grouping 해서 확인 할 수 있습니다. 

이 때, ALB에서  target이 보낸 HTTPCode_Target_5xx가 의미하는 것은, ALB와 target은 연결이 되어 있는 상태이고, target의 return 값이 5xx 에러이므로, 

이 경우에 우리는 target의 상태를 살펴봐야 함을 알 수 있습니다. 

 target은 정상적으로 작동 중이지만, target이 처리하는 내용이 이상이 있을수 있습니다. 

그리고 ALB에서도 오류가 생길 수 있기 때문에, ALB 자체의 응답코드도 모니터링 합니다. 

ALB에서 ELB_5xx에러가 증가를 한다는 것은 특히, 503에러의 경우는 target에 연결이 되지 않는 경우 혹은 ALB에서 지정하지 않은 PATH로 접근시에도발생하는 경우가 많이 있기 때문에 반드시 "알람" 을 설정해서 target에 대한 모니터링을 해야합니다. 

⇒ [정적임계값을 기반으로 cloudwatch 알람설정](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/ConsoleAlarms.html)

이 때, ALB는 가용 영역마다 존재하기 때문에, 가용영역의 문제점을 알기위해선 각 영역을 분리해야 합니다. 

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_15.48.17.png](/static/monitoring/2021-05-24_15.48.17.png)

OPENAPI ALB자체의 5xx 에러

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_15.48.30.png](/static/monitoring/2021-05-24_15.48.30.png)

OPENAPI ALB자체의 5xx에러

그리고 마지막으로 각 ELB 에 있는 Target group에서 Instance 의 healthy 상태를 모니터링 해야 합니다. 

아래의 그림은 ALB에 설정된 PATH별로 연결되어 있는  target group에 속한 인스턴스의 health check 상태를 모니터링 합니다. 

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_15.27.49.png](/static/monitoring/2021-05-24_15.27.49.png)

인스턴스 헬스 카운트 예제

이 때, 모니터링 Metric은  HealthyHost의 개수가 Auto Scale Group을 이용하는 경우는 min 미만으로 떨어질 경우 "알람", desire 초과 인 경우도 "알람"을 설정하여, 

호스트의 수가 변동되는 것에 대한 "알람"을 받을 수 있어야 합니다. 

여기까지가  위에서 서술한 1번 NLB와 2번 ALB의 모니터링 내용입니다. 

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_14.01.15.png](/static/monitoring/2021-05-24_14.01.15.png)

openapi edge NLB와 ALB의 모니터링 화면 

### EC2 인스턴스의 모니터링

EC2인스턴스 모니터링은 기존에 익숙한 하드웨어 모니터링 방식을 따릅니다. 

prometheus와 node_exporter를 이용해서 데이터를 콜렉션하여 사용 합니다. 

서버를 모니터링 하는 방법 및 지표는 너무 많은 자료가 있고, 각자 많은 경험이 있을테니, 성능 및 안정성에 영향을 미치는 꼭봐야 하는 부분만 짚어보도록 하겠습니다. 

1. CPU 
    1. 참고로 CPU가 100%에 도달한 상태에서 작업이 진행되어도 서버가 죽지 않습니다. ⇒ 단지 너무 느려지면서, 제때 응답을 하지 못하는 겁니다. 
    2. CPU에 대한 LOAD 테스트를 진행하기 전, 서버별로 주로 수행하는 작업이 현재 스펙에서 CPU를 최대 몇 %까지 쓰게 할건지 먼저 정해야 합니다. (임계값 설정)
    3. 해당 서버의 목적에 맞는 어플리케이션을 구동하고 LOAD 테스트를 진행 합니다. 
    4. 서버의 목적에 따라, 50% , 70%, 85% 로 구분하여 하드웨어 스펙을 결정합니다. 

     

2. MEMORY 
    1. MEMORY는 100% 차지하면 시스템이 다운될 수 있습니다. 
    2. OS영역은 약 2GB의 물리 메모리를 필요로 합니다. 
    3. 메모리의 여유 용량은 계산하기 까다롭습니다. (기준이 되는 REDHAT 리눅스도 5,6,7 버전에 따라서 약간씩 다릅니다.)
    4. JVM 설정을 기준으로 최대값에 물리 메모리의 1/2, 최소값에 1/4을 할당하길 권고 합니다 ([oracle: Tunning JVM](https://docs.oracle.com/cd/E21764_01/web.1111/e13814/jvm_tuning.htm#PERFM159))
    5. SWAP영역은 구동 되는 어플리케이션에 따라서 설정을 할 수도 있습니다. (Amazon Linux 는 기본적으로  swap이 없습니다.)

        ⇒ [Redhat linux의 스왑 가이드](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-swapspace)

        [메모리별 스왑 권장표](https://www.notion.so/77418354952a45238e785f6f1d518acc)

3. DISK IO / ACTIVITY
    1. 로그를 저장하는 공간은 생각보다 빨리 없어집니다. 
    2. 로그는 반드시 logrotate 등으로 관리 해야 합니다. 
    3. 디스크에서 read /write 모두 늦어지면 시스템에 문제가 있습니다. 
    4. read/write가 오랫 동안 멈춰있으면 시스템 죽을 수 있습니다. 
    5. 디스크의 남은 공간을 모니터링 하고 read/write  시간 지연도 모니터링 해야합니다. 

마지막으로 위의 모든것을 볼 수 있다면 그 어떤 툴이라도 상관없습니다. 

하지만 저는, Grafana, prometheus, node_exporter 씁니다. 

### 뒤쪽 또 다른 ALB(실제 API에 연결되어 있는 ALB)

API Gateway는 다시 API service 를 제공하는 ECS와 ALB로 연결되어 있습니다. 

하지만 경로 및 설정이 Edge와는 다릅니다. 그리고 우리는 아직 우리의 목표인 빠른 장애 탐지를 위해서는 이곳의 ALB도 모니터링 해야 한다는 것을 알고 있습니다. 

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_14.55.45.png](/static/monitoring/2021-05-24_14.55.45.png)

API ALB의 응답 코드별 카운트

그래프의 모양으로 보아 그림 "openapi ALB의 target group별 응답 코드 카운트"에 있는 전체의 수치와 그림 "API ELB의 응답 코드별 카운트"의 수치가 거의 비슷한 느낌적 느낌이 들고 있습니다.

이러한 그래프를 볼 때는 추세를 봐야하지, 숫자를 맞추려고 하면 안됩니다. 

초보 분들이 숫자가 맞지 않으면 답답해 하거나, 잘못되었다고 생각하는데, 시간의 범위에 따라서 집계를 하면 항상 오차는 발생하고 그 이유는 여러 가지가 있지만 

일단 컴퓨터의 시간은  ms(밀리세컨), ns(나노세컨)단위로 측정이 되고, 기록되는 시간이 서로 다른 관계로 정확한 수치는 맞지 않습니다. 

단 실험 계획을 수립해서 정확한 시간 동안 요청 / 응답 카운팅을 하거나, 외부 요소를 모두 배제한 상태에선 숫자를 맞출 수 있습니다.

이곳에서도 ALB에 연결되어 있는 target group에 속한 인스턴스의 개수를 추적할 수 있도록 화면을 추가 합니다. 

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_15.34.41.png](/static/monitoring/2021-05-24_15.34.41.png)

API ALB에 연결되어 있는 인스턴스의 개수 모니터링

그리고, ALB 자체의 오류코드도 모니터링 합니다. 

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_15.41.00.png](/static/monitoring/2021-05-24_15.41.00.png)

Internal API 자체의 5xx 에러

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_15.41.11.png](/static/monitoring/2021-05-24_15.41.11.png)

Internal API 자체의 5xx 에러

AWS의 네트워크 자원에 대한 모니터링 화면을 만들어 볼 수 있습니다. 

![01%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%203ff2c72412804fb2a73bef77e6c09ffd/_2021-05-24_15.53.44.png](/static/monitoring/2021-05-24_15.53.44.png)