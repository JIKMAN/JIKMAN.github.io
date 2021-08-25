---
title:  "[k8s] 1-2. wordpress 배포"

categories:
  - Kubernetes
tags:
  - [kubernetes, k8s, minikube]
toc: true
toc_sticky: true
---

## wordpress 배포

### yml 파일로 배포

```bash
$ kubectl apply -f wordpress-k8s.yml
```

```yaml
# wordpress-k8s.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - image: mysql:5.6
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: password
          ports:
            - containerPort: 3306
              name: mysql

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - image: wordpress:5.5.3-apache
          name: wordpress
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_PASSWORD
              value: password
          ports:
            - containerPort: 80
              name: wordpress

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
```



* 배포 상태 확인

```bash
$ kubectl get all
```

```bash
NAME                                  READY   STATUS    RESTARTS   AGE
pod/wordpress-5f59577d4d-l8pb2        1/1     Running   0          10m
pod/wordpress-mysql-545d9c6dc-qqxpm   1/1     Running   0          10m

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP        2d
service/wordpress         NodePort    10.99.32.84     <none>        80:32494/TCP   10m
service/wordpress-mysql   ClusterIP   10.98.119.178   <none>        3306/TCP       10m

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress         1/1     1            1           10m
deployment.apps/wordpress-mysql   1/1     1            1           10m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/wordpress-5f59577d4d        1         1         1       10m
replicaset.apps/wordpress-mysql-545d9c6dc   1         1         1       10m
```

* ip 확인

```bash
$ minikube service wordpress --url
🏃  Starting tunnel for service wordpress.
|-----------|-----------|-------------|------------------------|
| NAMESPACE |   NAME    | TARGET PORT |          URL           |
|-----------|-----------|-------------|------------------------|
| default   | wordpress |             | http://127.0.0.1:33815 |
|-----------|-----------|-------------|------------------------|
```

![wordpress](https://camo.githubusercontent.com/4e1409952e81dc476f8a04c455878e8bd854fa2829172b1b1e2761d5a8d59950/68747470733a2f2f73756269637572612e636f6d2f6b38732f6173736574732f696d672f776f726470726573732e61393134373136372e706e67)

* 리소스 제거

```bash
$ kubectl delete -f wordpress-k8s.yml
```

