---
title:  "[Docker] 호스트(local)와 컨테이너 간 파일 전송"

categories:
  - Docker
tags:
  - [Docker]
toc: true
toc_sticky: true
---

도커에서 컨테이너로 파일을 전송하려고 한다면 docker cp 를 이용하면 된다.

예를들어 리눅스 컨테이너에 파일을 전송하고자 한다면 



1. 호스트 -> 컨테이너 파일 전송

```bash
docker cp /path/foo.txt mycontainer:/path/foo.txt
```





2. 컨테이너 -> 호스트 파일 전송

```bash
docker cp mycontainer:/path/foo.txt /path/foo.txt
```



