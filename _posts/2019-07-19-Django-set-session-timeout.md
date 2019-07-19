---
title: "[Django] 장고에서 Session 만료 시간 관리하는 방법"
categories:
  - Django
tags:
  - Django
  - Session
---

### Refer

- ##### [Django-Docs-Sessions](https://docs.djangoproject.com/en/2.2/topics/http/sessions/)

### How To

장고에서 세션을 관리할 수 있게 다양한 방법을 제공해준다.

`set_expiry()`등 처럼 함수로도 제공되니 필요하신 분들은 [장고 문서](https://docs.djangoproject.com/en/2.2/topics/http/sessions/)를 참고해보면 좋을 듯 하다.

(저는 간단하게 변수설정으로 세션 만료시간을 설정해 봤어요. ㅎㅎ)

``` python
# settings.py
SESSION_COOKIE_AGE = 3600 # 세션만료 시간을 1시간으로 설정
SESSION_EXPIRE_AT_BROWSER_CLOSE = True # 브라우저가 꺼지면 세션 만료
SESSION_SAVE_EVERY_REQUEST = True # 사용자로부터 요청이 있을 때마다 세션 갱신
```

