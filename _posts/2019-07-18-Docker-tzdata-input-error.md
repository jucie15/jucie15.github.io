---
title: "[Docker] image build 시 tzdata 입력 창 뜨는 문제(Feat. Postgresql)"
categories:
  - Docker
tags:
  - Docker
---

### Refer

- ##### [askUbuntu](https://askubuntu.com/questions/909277/avoiding-user-interaction-with-tzdata-when-installing-certbot-in-a-docker-contai)

###Trouble Shooting

```shell
ENV DEBIAN_FRONTEND=noninteractive 
# ENV를 쓸 경우 해당이미지에 종속되고 자식이미지들에게 계속 영향을 줄 수 있어 비추천한다.
ARG DEBIAN_FRONTEND=noninteractive # ARG는 빌드시에만 사용되는 변수로 추천!
```

