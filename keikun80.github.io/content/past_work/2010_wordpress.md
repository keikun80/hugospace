---
# Common-Defined params
title: "Wordpress(PHP)기반 호텔 예약 관리 개발"
date: "2023-01-04"
lastmod: "2031-01-03"
description: " 2010~2015년 사이에 만들었던 사이트들 모음"
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

PHP 5 ~ 7 시절에는  wordpress같은 CMS를 이용한 웹개발이 많았던 시절 이었습니다. 
소비자는 저렴한 가격의 웹사이트를 원했고, 공급자도 저렴하게 웹사이트를 쉽게 찍어낼 수 있으면서
고급스러운 관리자 기능을 제공할 수 있었고, 무엇보다 웹호스팅 중 제일 저렴한게  LAMP(Linux + apache + mysql + php) 였기 때문에 전체적으로 공급가를 낮출 수 있었습니다. 

이 시절에는 wordpress로 기본적인 웹사이트의 골격을 잡고 세부적인 기능은 플러그인을 그대로 사용하거나, 
고객의 요청에 따라서 플러그인을 개발하기도 했습니다. 

단가는 플러그인 개발 유무에 따라서 편차가 심했는데 제일 기억에 남는 플러그인은 호텔이 예약관리 플러그인 있었습니다. 

#### 호텔 예약 관리 개발
이유는 당시에는 호텔의 객실 관리에 대한 지식 없었던 때라 먼저 예약이나 방청소 등의 호텔의 업무를 알아야 했습니다. 그래서 먼저 호텔의 관계자와의 인터뷰를 통해서 체크인/체크아웃 시간에 일어나는 일을 청취했고 
고객들이 예약을 했을 때 고려해야 하는 사항을 조사했습니다. 
이 때 만들어진 사항을 기반으로 wordpress에 있는 예약 플러그인을 조사했고, 하나의 예약 플러그인을 fork 해서 개발 할 수 있었습니다. 
객실 소개 페이지 템플릿을 만들고 그 템플릿 안에 예약 폼을 연결했습니다. 

##### 전제조건
 - 객실의 종류는 상품의 종류이고 객실 수 재고 관리와 같다.
 - 따라서 객실은 상품으로 취급된다. 
 - 다른 서비스의 경우도 재고 유무에 따라 예약 가능 여부를 판별한다.
 - 예약은 지정 날짜 D-3일까지 예약을 취소 할 수 있다. 
 - 체크인 시간까지 고객이 도착하지 않거나 별도의 연락이 없다면 예약은 취소 된다. 

##### 구현
- 객실 소개 상세 페이지
- 객실 관리자 페이지는 이커머스용 상품 관리 플러그인 사용
- 예약정보는 여권번호 및 이름만 사용
- 예약 시 해당 SKU -1 시키고 예약 레코드 생성
- 객실  SKU 수량은 변경 사항 발생 시 마다 프런트 데스크에서 수정 

##### 제한사항
먼저 이때는 호텔스닷컴과 아고다 등의 예약 서비스가 한창 사용이 되던 시기였지만 각 업체별로 솔루션을 공급했기 때문에, 통합 보다는 work-in(직접 방문)고객을 관리하기 위해서 내부적으로 직원들끼리 공유하기 위한 솔루션이었습니다. 따라서 구현시에 외부 API연동은 고려되지 않았습니다. 
