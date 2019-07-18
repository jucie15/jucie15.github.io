---
title: "[Django] 크롤링 시 Malformed attribute selector at position Error 해결하기"
categories:
  - Django
  - Crawling
tags:
  - Django
  - Crawling
  - Error
  - TroubleShooting
---

- ##### 크롤링 자체코드만 실행했을 때에는 잘 동작하던 코드가 장고서버내에서 실행하니 에러가 났다!!

![Django-Malformed-error](../_imgs/Django-Malformed-error.png)

- ###  TroubleShooting

``` python
for a_tag in list_soup.select('ytd-thumbnail a[href^=/watch]') # <- 해당 부분에서 에러 발생

##### 해결법! ->

for a_tag in list_soup.select('ytd-thumbnail a[href^="/watch"]') # /watch 해당 부분에 ""를 감싸주면 해결~ 
```

