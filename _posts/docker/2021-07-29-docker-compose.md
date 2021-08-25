---
title:  "[Docker] 도커 컴포즈(docker-compose)"

categories:
  - Docker
tags:
  - [Docker, docker compose]
toc: true
toc_sticky: true
---

### docker-compose.yml 형식

```bash
#버전 정의
# 버전에 따라서 지원하는 형식이 다릅니다.
version: '3.0'

#service 정의
# docker-compose로 생성 할 container의 옵션을 정의 합니다.
# service안의 container들은 하나의 project로서 docker-compose로 관리 됩니다.
services:
    #생성 할 container 이름을 지정 합니다.
    container-이름:
        #container 생성시 사용 할 이미지 지정
        image: node-server:v1
        #build 옵션
        # docker-compose build 옵션에서 사용 됩니다.
        # dockerfile에 명시된 FROM의 image를 사용하여 위에 명시된image 이름과 tag로 새로운 image를 생성 합니다.
        build:
            #dockerfile의 위치를 지정 합니다.
            context: .
        #container port mapping 정보
        ports:
            - "80:80"
        #EXPOSE
        # 도커 컨테이너가 실행되었을 때 요청을 기다리고 있는(Listen) 포트를 지정합니다. 여러개의 포트를 지정할 수 있습니다.
        expose:
            - "80"
        #환경 변수 리스트를 정의
        environment:
            UPDATE_URL: "https://www.doitnow-man.com"
```

 

- **버전 확인**

```
docker-compose --version
```

 

- **docker-compose up**

컨테이너를 생성 및 실행 합니다.

```
docker-compose up [옵션] [서비스명]
```

| 옵션      | 설명                       |
| --------- | -------------------------- |
| -d        | 백그라운드 실행            |
| --no-deps | 링크 서비스 실행하지 않음  |
| --build   | 이미지 빌드                |
| -t        | 타임아웃을 지정(기본 10초) |

참고로 특정 서비스들의 경우 백그라운드로 실행하지 않으면 컨테이너가 생성 및 실행되며 바로 종료될 수 있습니다.



* **docker-compose -f**

Docker Compose는 기본적으로 커맨드가 실행하는 디렉토리에 있는 `docker-compose.yml` 또는 `docker-compose.yaml`를 설정 파일로 사용합니다. 다른 이름이나 경로의 파일을 Docker Compose 설정 파일로 사용하고 싶다면 `-f` 옵션으로 명시를 해줍니다.

```bash
$ docker-compose -f docker-compose-local.yml up
```

`-f` 옵션은 여러 개의 설정 파일을 사용할 때도 사용할 수 있습니다. 이 때는 나중에 나오는 설정이 앞에 나오는 설정보다 우선하게 됩니다.

```bash
$ docker-compose -f docker-compose.yml -f docker-compose-test.yml up
```

 

* **docker-compose -down**

`docker-compose down` 커맨드는 `docker-compose up` 커맨드와 정반대의 동작을 합니다. Docker Compose에 정의되어 있는 모든 서비스 컨테이너를 한 번에 정지시키고 삭제합니다.

```bash
$ docker-compose down
Stopping django-app_web_1 ... done
Stopping django-app_db_1  ... done
Removing django-app_web_1 ... done
Removing django-app_db_1  ... done
Removing network django-app_default
```



- **docker-compose ps**

현재 동작중인 컨테이너들의 상태를 확인할 수 있습니다.

```bash
docker-compose ps
```



![img](https://blog.kakaocdn.net/dn/co9RwX/btqBpPJMoV5/K7J4ko12wZ3gtxRkMZdp81/img.png)



 

- **docker-compose logs**

컨테이너들의 로그를 출력 합니다.

```bash
docker-compose logs
```



![img](https://blog.kakaocdn.net/dn/9IIKG/btqBp56Ewvg/lMs8tb0LqkW9jJRF4j57uk/img.png)



 

- **docker-compose run**

docker-compose up 명령어를 이용해 생성 및 실행된 컨테이너에서 임의의 명령을 실행하기 위해 사용 합니다.

컨테이너들을 모두 삭제할 경우 docker-compose start가 아닌, docker-compose up으로 다시 컨테이너들을 생성 해주어야 합니다.

```bash
docker-compose run
```

 

만약 특정 서비스에서 /bin/bash를 실행시켜 쉘 환경으로 진입하고 싶다면 아래와 같은 명령어를 이용하면 됩니다. 참고로 서비스명과 컨테이너명은 다른겁니다.

서비스명은 docker-compose.yml의 services: 밑에 작성한 서비스 이름 입니다.

```bash
# docker-compose run [서비스명] [명령]
docker-compose run redis /bin/bash
```

 

- **docker-compose (start / stop / pause / unpause / restart)**

여러개의 서비스 또는 특정 서비스를 시작 / 정지 / 일시정지 / 재시작을 할 수 있습니다.

```bash
# 서비스 시작
docker-compose start

# 서비스 정지
docker-compose stop

# 서비스 일시 정지
docker-compose pause

# 서비스 일시 정지 해제
docker-compose unpause

# 서비스 재시작
docker-compose restart
```

각각의 설정 뒤에 서비스명을 붙이면 특정 서비스만 제어할 수 있습니다. (ex. docker-compose restart [서비스명])



* **docker-compose exec**

`docker-compose exec` 커맨드는 실행 중인 서비스 컨테이너를 대상으로 어떤 명령어를 날릴 때 사용합니다.

```bash
$ docker-compose exec db psql postgres postgres
psql (12.2 (Debian 12.2-2.pgdg100+1))
Type "help" for help.

postgres=#
```



- **docker-compose rm**

docker-compose로 생성한 컨테이너들을 일괄 삭제 합니다. (삭제 전, 관련 컨테이너들을 종료 시켜두어야 합니다.)

```bash
docker-compose rm
```

 

- **docker-compose kill**

실행중인 컨테이너를 강제로 정지 시킵니다. 

-s옵션을 사용하여 시그널을 지정해줄 수 있는데, 아래 코드에서는 SIGINT를 사용 하였습니다. -s 옵션을 사용하지 않고 docker-compose kill만 사용할 경우 SIGKILL 이 전송 됩니다.

kill 뒤에 서비스를 지정하여 특정 서비스만 kill할 수 있습니다.

```bash
# docker-compose kill [옵션]
docker-compose kill -s SIGINT
```

 

- **docker-compose down**

네트워크 정보, 볼륨, 컨테이너들을 일괄 정지 및 삭제 처리 합니다.

```bash
docker-compose down
```

만약 docker-compose down --rmi all 명령을 사용한다면 모든 이미지까지 삭제 합니다.

 

- **docker-compose port**

서비스 프라이빗 포트 번호의 설정을 확인할 수 있습니다.

```bash
# docker-compose port [서비스명] [프라이빗 포트 번호]
docker-compose port nginx 80
```

 

- **docker-compose config**

docker-compose 구성 파일의 내용을 확인할 수 있습니다. docker-compose.yml의 내용을 출력 해주므로 많이 쓸일은 없습니다.

```bash
docker-compose config
```



공식 홈페이지 : https://docs.docker.com/compose/reference/