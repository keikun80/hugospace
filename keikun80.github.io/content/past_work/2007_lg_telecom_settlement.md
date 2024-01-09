---
# Common-Defined params
title: "2007년 LG텔레콤 정산 현행화 프로젝트"
date: "2023-01-04"
lastmod: "2031-01-03"
description: "2007년 LG텔레콤 차세대 프로젝트 중 정산현행화 프로젝트에서 RM-COBOL로 작성된 Legacy solution을 Oracle ProC기반의 in-house solution으로 변경하는 프로젝트"
categories: 
  - "history"
tags:
  - "이력" 
  - "과거"
menu: Past work # Add page to a menu. Options: main, footer

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

2007년에서 2008년 사이에 진행했던 업무로 프리랜서로 참여 했습니다. 
주 업무는 당시 LG텔레콤에서 차세대로 가지 못하는 정산 업무을 RM-COBOL기반의 솔루션에서 
ProC 기반 in-house로 현행화 하는 것 이었습니다. 

RM-COBOL로 쓰여있던 솔루션은 LG텔레콤이 비즈니스를 시작하는 시점에 도입된 솔루션이었고, 
새로운 기능을 추가하거나 성능을 높일 수 없던 것으로 기억합니다. 
이 때 들은 말로는 정산 배치를 구동 시키면 운영계에서 일주일 정도 걸렸던걸로 기억합니다. 

#### 현행화의 목표 
- 가독성이 좋고 유지보수가 쉽도록 코드와 비즈니스를 분리
- AS-IS 솔루션의 모든 기능의 완벽 이전
- TO-BE 기능 추가

이 때 초기에는 성능에 대한 수요제기는 크지 않았습니다. 
이유는 프로젝트 자체가  RM-COBOL로 되어 있는 코드를 분석하고 ProC로 포팅하는데 중점을 두고 있었습니다. 

#### 프로젝트 순서
1. AS-IS에서 특정 영역의 비즈니스 로직과 SQL문을 분석 추출
2. 비즈니스 로직과 SQL의 상관관계 매핑 
3. 코드성 정보의 조회 및 변경에 대한 부분 매핑
4. ProC로 비즈니스 로직 작성
5. AS-IS SQL을 TO-BE에서 정의된 table에 맞도록 SQL문 수정
6. 수정된 SQL이 변경된 table의 구조와 성능상의 이슈 검토
7. 검토된 SQL는 explain 결과 확인 후 코드에 적용
프로젝트는 각 정산 항목 별로 위의 과정을 반복해서 진행했습니다. 
이 프로젝트를 하면서 겪은 여러 경험 중 데이터 웨어하우스(Data warehouse, 이하 DW)를 이용하는 경험과 AIX C Compiler 의 free 함수 버그로 인한 운영서버가 다운된 경험이 제일 기억에 남습니다. 

이 때까지 저는 DW에 대해서 이름을 들어봤지만 사용해본 적이 없었습니다. 
하지만, 프로젝트 진행 중 접속료 정산 부분에서 총액은 맞는데 세부 항목의 비율이 AS-IS와 약간씩 다르게 나오는 현상을 발견했습니다. 다르게 나오는 데이터의 총량이 너무 많고 (월 20억 레코드) 특정 부분을 유추하기도 어려운 상황에서 이전달 데이터를 기준으로 legacy의 리포트와 현재 개발중인 정산의 리포트, 그리고 DW의 리포트까지 
모두 3개의 리포트를 두고 데이터를 추적한 결과 AS-IS 소스 분석 중 정체를 알 수 없던 플래그 하나가 해당 소스를 벗어나서 사용되는 플래그 였는데 이 플래그의 상태에 따라서 리포트의 데이터가 변할 수 있는 부분이었습니다.
대량의 데이터를 이용할 때, 정확한 값이 아니어도 추세와 대략적인 정보를 보기 위해서 구축되어 있던 DW의 도움을 받아서 해당 리포트에 대해서 보완할 부분을 찾을 수 있었고 이것은 해당 프로젝트를 하면서 기억에 남는 두가지 일화 중 한 가지 입니다. 

그리고 다른 한가지는 운영 중이던 서버가 코드 때문에 다운된 일이었는데, 발단은 운영 환경의 변화가 요인 이었습니다. 
프로젝트는 HP-UX의 C compiler로 진행이 되었는데, 막바지에 AIX의 C Compiler로 변경 되었습니다. 
이 때의 저는 C compiler는 모두 같은 compiler 라고 생각하고 있었고, 코드 자체도 특정 OS에 종속된 라이브러리가 없었고,  컴파일과 테스트 모두 이상이 없었기 때문에 실제 데이터가 많이 있는 운영서버에서 최종 테스트를 하기로 결정 하였습니다. 왜냐하면 테스트 서버에는 데이터가 1억건 미만으로 존재하지만 운영은 20억건이 존재하기 때문입니다.

하지만  AIX는 OS의 특성상 free 함수가 구동되어도 해당 어플리케이션이 running 상태이면 malloc으로 할당한 부분 중 일부를 cache 상태로 유지를 하는 특성이 있었는데, 테스트 데이터가 적을 때에는 메모리를 모두 차지하기 전에 어플리케이션이 종료되기 때문에 알 수 없었지만, 운영 서버에서는 데이터가 많이 있었기 때문에 
호스트OS가 메모리 부족으로 다운되는 현상이 발생 되었습니다. 
원인은 메모리 최적화를 위해서 가변개수로 fetch하고 fetch된 데이터를 이용하는 Thread가 종료되면 free로 반환을 하도록 한 부분이 있었는데 이 부분을 고정개수로 fetch하고 Thread 가 종료되면 해당 영역을 새롭게 0으로 세팅한 후 재활용 하도록 코드를 수정해서 문제를 극복 했습니다.
운영계에서의 구동 첫날이어서 밤새 모니터링을 하기 위해 사무실에 있었는데 오전 04시에 이상을 발견하고 
나머지 서버들에서 구동중이던 어플리케이션을 종료하고, 서버를 리부팅 한 후 원인을 파악하고 코드 개선과 보고까지 총 4시가 정도가 소요된 사건이었습니다. 
원인과 결과 그리고 개선 사항과 그 개선 사항이 반영된 변경 코드의 diff 내역까지 첨부해서 제출 했었고 
나머지 모든 코드에 대해서도 malloc/free 부분에 대한 재검토를 진행 했었습니다. 
