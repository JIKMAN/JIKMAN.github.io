---
title:  "[Mongo DB] Mongo DB install"

categories:
  - mongo DB
tags:
  - [mongodb]
toc: true
toc_sticky: true
---

### Ubuntu 서버에 mongoDB 설치

- 우분투 몽고DB 서버와 연결
  - `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`
- 몽고DB 리스트 가져오기
  - `echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`
- 로컬환경 서버 reload
  - `sudo apt-get update`
- MongoDB 패키지 설치
  - `sudo apt-get install -y mongodb-org`
  - 

> **mpngodb-org 패키지**
>
>  A metapackage that will automatically install the four component packages, mongodb-org-server, mongodb-org-mongos, mongodb-org-shell, and mongodb-org-tools

> **mongoDB 저장공간**
>
>  The MongoDB instance stores its data files in /var/lib/mongodb and its log files in /var/log/mongodb by default, and runs using the mongodb user account. You can specify alternate log and data file directories in **/etc/mongod.conf**.

- MongoDB 시작
  - sudo service mongod start
- Verify that MongoDB has started successfully
  - Verify that the mongod process has started successfully by checking the contents of the log file at /var/log/mongodb/mongod.log for a line reading
  - [initandlisten] waiting for connections on port
  - where is the port configured in /etc/mongod.conf, 27017 by default.
- MongoDB 종료
  - sudo service mongod stop



### **mongod: unrecognized service** 오류 발생시

* Mongo DB 설치 후 ```sudo service mongod start``` 과정에서 **mongod: unrecognized service** 오류 발생시

  * `$ sudo vi /etc/init.d/mongod` 로 들어가서
  * [https://github.com/mongodb/mongo/blob/master/debian/init.d](https://github.com/mongodb/mongo/blob/master/debian/init.d) 해당 링크의 내용 붙여넣기
  * `cd /etc/init.d` 로 이동
  * `$ chmod 755 mongod` 명령어로 권한 설정
  * `sudo service mongod start` 명령어로 실행