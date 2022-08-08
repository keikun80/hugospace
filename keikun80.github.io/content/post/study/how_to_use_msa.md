---
# Common-Defined params
title: "MSA답게 쓰는 법"
date: "2022-08-03"
lastmod: "2022-08-03"
description: "Micro가 되어야 하는 것"
categories:
  - "study"
  - "architect"
  - "application"
tags:
  - "API"
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
왜 API를 쓰는가

우리는 task를 할당 할 때, (task는 URL로 들어오는 request 일 수 도 있고, cli 에서 호출하는 명령어일 수도 있다.)
일단 request에 의해 task가 작동하고 response를 return 하게 된다.

우리는 API를 호출 할 때, 한번에 하나의 task 만 요청할 수도 있고, 여러개의 task를 묶어서 요청할 수도 있다. 
한번에 하나 요청하는 것을 single이라 하고, 여러개를 요청 하는 것을 bulk라 지칭하겠다. 
이렇게 single과 bulk가 존재할 수 있는 이유는 for / while loop statement 덕분이며, 
이 때문에 multi task/single request(bulk), 와 single task/single request가 생기게 된다. 


이것을 실제 application에서 사용하게 될 때는, bulk를 이용해서 request /response 시간을 단축 시킬 것인가? 아니면 single request로 해서 한번에 처리되는 양은 줄이지만 request / response의 시간에 부담을 가져갈 것인가? 
라는 두가지중에 하나의 선택일 수 있다. 


우린 언제 bulk와 single을 구분해야 하는가? 
먼저, 내 의견으로는 bulk는 대용량 batch 작업에서 어울리는 방법이다. 

프로그램 내부에서 request / response 가 모두 일어나며, 결과는 별도로 저장된다. 
이 경우 실행시간의 길이는 프로그램 수행이 문제가 되지 않는다. 

그래서, API에서 bulk를 이용하면 아래와 같은 단점이 있따. 
 - 수행시간이 오래 걸린다. 
 - response time에 대한 예측을 하지 못한다. 
 - 중간에 오류가 나서 멈추게 되면 모든 작업이 rollback이 되어야 한다. 

single은 외부의 모든 request에 대해서 response를 줘야하며, 이때 시간은 reasonable 해야한다. 
하나의 요청을 보냈는데 response가 언제 올지 모른다면 이미 API의 자격이 없다고 생각한다. 
예로 로그인을 하는데 10초가 걸린다고 해보자. 누가 쓰겠는가 

