---
title: SRE는 무엇을 해야하는가?
description: SRE 업무의 개요와 해야한다고 생각되는 일
date: 2023-05-18T02:07:18.285Z
preview: ""
draft: true
tags:
  - SRE
categories:
  - architect
slug: what-does-sre-do
keywords:
  - SRE
---
### SRE? Service? Site? 
근래에 들어 Site와 Service의 구분이 애매해지긴 했어도 엔지니어들은 구분해서 사용해야 한다고 생각합니다. 
Site는 고객(Enduser:최종사용자)에게 Service를 제공하는 공간의 개념이고,  
Service는 고객(Enduser:최종사용자)가 액션을 통해서 대가를 얻는 것이라 볼 수 있습니다. 
이 Service를 제공하기 위해서 데이터를 조작해야 하는데 데이터가 저장된 곳이 Storage, 조작을 위한 이동이 
Transaction, 조작이 Application 입니다. 

범위를 좁혀 나가면 아래의 순서로 내려가게 됩니다. 

__Site > Services > Application > Transacton > Storage__

그렇다면 site reliability engineering에서 reliablility를 확보하기 위한 엔지니어링이 무엇인지 접근을 해야합니다.  
구글이 정의하는 SRE란
*"SRE는 안정적인 프로덕션 시스템을 실행하기 위한 직무, 사고방식, 엔지니어링 방식 집합입니다. Google Cloud는 도구, 전문 서비스, 기타 리소스를 통해 SRE 원칙을 구합니다."*
으로 되어 있습니다. 

이중에서 중요한건 사고방식과 엔지니어링 방식이라고 생각합니다. 
도구는 내 아이디어를 보다 쉽고 효율적으로 표출 할 수 있도록 해주는 것이지 도구가 엔지니어링을 리드하지 못합니다. 
사고방식이 엔지니어링을 리드하고 도구가 엔지니어링을 도와주는 것입니다. 

이러한 사고방식에 대한 전환이 없는 상태에서는 단순히 SRE직무를 할당하고 도구와 사람을 투입했다고 해서 SRE가 가능해지지 않습니다. 
개인적인 생각으론 Kubernetes가 SRE의 사고방식과 엔지니어링이 결합되어 나온 SRE가 만들고 보급한 최고의 툴이라고 생각합니다. 
순수하게 필요에 의해서 나온 툴이라는 의견을 갖고 있습니다. 

신뢰성을 확보하기 위해서는 
