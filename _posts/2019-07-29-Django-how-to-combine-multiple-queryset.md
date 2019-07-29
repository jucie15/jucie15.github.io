---
title: "[Django] 2개 이상의 다른 쿼리셋을 하나의 리스트로 합치기"
categories:
  - Django
tags:
  - Django
  - List
  - Queryset
  - Chain
---

### Refer

- ##### [Stackoverflow](https://stackoverflow.com/questions/431628/how-to-combine-2-or-more-querysets-in-a-django-view/)

### How To

파이썬 내장 라이브러리인 `itertools` 에 `chain` 이라는 좋은 함수가 있고 해당 함수를 통해 간단하게 리스트로 합칠 수 있다.

``` python
from itertools import chain
page_list = Page.objects.all()
article_list = Article.objects.all()
post_list = Post.objects.all()

result_list = list(chain(page_list, article_list, post_list)) # 하나의 리스트로 합치기

# 정렬하기
result_list = sorted(
    chain(page_list, article_list, post_list),
    key=lambda instance: instance.created_at)
```

