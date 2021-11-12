---
title:  "[Mongo DB] 계정 및 role 생성하기"

categories:
  - mongo DB
tags:
  - [mongodb]
toc: true
toc_sticky: true
---

### MongoDB의 Authentication

```bash
security:

  authorization: enabled
```

mongod.conf 파일에 해당 부분을 활성해 주면 MongoDB 서버 간의 통신을 위한 내부 인증뿐만 아니라 사용자의 로그인을 위한 인증까지 모두 활성화 됩니다.

해당 기능을 활성화 한 상태에서 MongoDB를 구동하게 되면 단순히 OS 셸에서 mongo 명령어로 접속했을때 MongoDB로 접속은 가능하나, 내부에 있는 데이터 확인이나 모든 use나 로그인 관련 명령어를 제외하고는 동작하지 않게 됩니다.

## admin 계정 생성

일반적인 계정 생성을 위한 명령어는 아래와 같습니다.

```bash
$ mongo
MongoDB shell version v4.2.6
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("c8dfe45a-84c0-48c3-99e4-39b65a85780c") }
MongoDB server version: 4.2.6
> use admin
switched to db admin
> db.createUser(
...   {
...     user: "admin",
...     pwd: passwordPrompt(), // or cleartext password
...     roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
...   }
... )
Enter password:
Successfully added user: {
  "user" : "admin",
  "roles" : [
    {
      "role" : "userAdminAnyDatabase",
      "db" : "admin"
    },
    "readWriteAnyDatabase"
  ]
}
> db.auth("admin",passwordPrompt())
Enter password:
1
```

계정을 생성하기 위해서 use admin 명령을 통해 admin DB로 이동했는데, 이 admin db가 인증 데이터베이스가 되는 것입니다. 꼭 admin 데이터베이스를 사용하지 않고 자신이 생성한 데이터베이스를 사용해도 됩니다.

 

## MongoDB의 역할(Role)

일반 계정은 admin 권한을 가져서는 안됩니다. 단순히 데이터를 읽어가는 read 권한이나, 개발에 필요한 readWrite 권한만 부여한다던지 등 각 유저 용도에 맞는 권한을 부여해야 합니다.

- **read** : 모든 비시스템 컬렉션 및 system.indexes, system.js, system.namespace와 같은 시스템 컬렉션의 데이터를 읽을수 있는 권한
- **readWrite**: 동일한 권한을 부여하면, 모든 비시스템 컬렉션 및 system.js 컬렉션의 데이터를 수정할 수 있는 권한
- **dbAdmin**: 스키마 관련 작업, 인덱싱, 통계 수집과 같은 관리 작업에 대한 권한. 사용자와 역할 관리에 대한 권한은 없음
- **userAdmin**: 현재 데이터베이스에서 역할과 사용자를 생성하고 수정
- **dbOwner**: readWrite, dbAdmin, userAdmin 역할이 부여한 권한을 모두 포함
- **cluseterManager**: 클러스터상에서 관리와 모니터링 작업을 할 수 있는 권한
- **clusterMonitor**: MongoDB 클라우드 매니저와 옵스 매니저 모니터링 에이전트 같은 모니터링 도구에 대한 읽기 전용 접근 권한
- **hostManager**: 서버를 모니터링하고 관리할 수 있는 권한
- **clusterAdmin**: cluseterManager, clusterMonitor, hostManager 역할이 부여한 권한과 dropDatabase 권한을 포함
- **backup**: MongoDB 클라우드 매니저 백업 에이전트 혹은 옵스 매니저 백업 에이전트를 사용하는 권한이나, mongodump를 사용해 mongod 인스턴스를 백업하는 권한
- **restore**: system.profile 컬렉션 데이터를 포함하지 않는 백업으로부터 데이터를 복원하는데 필요한 권한
- **readAnyDatabase**: local과 config를 제외한 모든 데이터베이스에서 read와 동일한 권한과 클러스터 전체에 대한 listDatabases 권한을 가짐
- **readWriteAnyDatabase**: local과 config를 제외한 모든 데이터베이스에서 readWrite와 동일한 권한과 클러스터 전체에 대한 listDatabases 권한을 가짐
- **userAdminAnyDatabase**: local과 config를 제외한 모든 데이터베이스에서 userAdmin과 동일한 권한을 가진다. (사실상 superuser)
- **dbAdminAnyDatabase**: local과 config를 제외한 모든 데이터베이스에서 dbAdmin과 동일한 권한을 가지며 클러스터 전체에 대한 listDatabases 권한을 가짐
- **root**: 모든 리소스에 대한 접근 권한을 가짐



MongoDB의 매뉴얼 참조 - [https://docs.mongodb.com/manual/reference/privilege-actions/index.html](https://docs.mongodb.com/manual/reference/privilege-actions/index.html)



 

## 새로운 사용자 정의 역할 만들기

아래는 새로운 역할(role)을 만드는 명령입니다.

```bash
db.createRole(
  {
  role: "serviceDev",
  privileges: [],
  roles: [
     { role: "readWrite", db: "test01"}
    ]
  }
)
```

역할을 만들때 두 가지 방법이 있습니다. 위의 예제처럼 `roles` 필드에 기존의 역할을 맵핑 시키는 방법이 있고, 아래처럼 `privileges` 필드에 액션을 직접 부여해서 만드는 방식이 있습니다.

```bash
db.createRole(
  {
  role: "serviceDev",
  privileges: [
           { resource: { db: "test01", collection: "" }, actions: [ "find", "update", "insert", "remove" ]},
  roles: []
  }
)
```



역할은 `db.createRole()` 명령을 이용하며, 생성된 사용자 역할에 새로운 액션을 추가하거나 제거하는 작업은 `db.grantPrivilegesToRole()` 명령과 `db.revokePrivilegesFromRole()` 명령으로 가능합니다.

 

## 사용자 정의 역할을 이용해 계정 생성하기

아래는 개발용 계정을 만드는 예제입니다.

우선 Role를 만들어줍니다.

```bash
mongo> use admin

mongo> db.createRole(
  {
  role: "DevOps",
  privileges: [],
  roles: [
     { role: "readWrite", db: "test01"}
    ]
  }
)
```



개발용 계정은 DB와 컬렉션에대한 정보를 읽을수 있어야 하며, 새로운 컬렉션 및 도큐먼트를 생성할 수 있어야 합니다. 그래서 readWrite role를 계정이 접속하여 사용하고자 하는 DB에 부여 했습니다.

```bash
db.createUser(
  {
     user: "User_name",
     pwd: "password",                                // 혹은 passwordPrompt()
     roles: [ {role: "DevOps", db: "test01"} ]
  }
)
```





DevOps라는 사용자 정의 역할을 이용해 유저를 생성했습니다. passwordPrompt()를 이용하면, 직접 패스워드를 넣을수 있는 입력하는 암호가 노출되지 않는 보안 Prompt를 사용할 수 있습니다.

이렇게 생성된 계정으로 client tool에서는 계정, 패스워드, 인증DB 정보만 가지고 접근할 수 있고, 터미널에서는 아래와 같은 명령으로 접속을 합니다.

```bash
$ mongo -u "User_name" --authenticationDatabase "admin"

Enter password:
```

 

**참고**

MongoDB Manual: [https://docs.mongodb.com/manual/](https://docs.mongodb.com/manual/)

[https://rastalion.me/mongodb%EC%9D%98-%EC%9D%B8%EC%A6%9D%EA%B3%BC-%EA%B6%8C%ED%95%9C/](https://rastalion.me/mongodb%EC%9D%98-%EC%9D%B8%EC%A6%9D%EA%B3%BC-%EA%B6%8C%ED%95%9C/)