---
title:  "[k8s] 2. 쿠버네티스 커맨드"

categories:
  - Kubernetes
tags:
  - [kubernetes, k8s]
toc: true
toc_sticky: true
---

# K8s Command

| 명령어     | 설명                                                         |
| ---------- | ------------------------------------------------------------ |
| `apply`    | 원하는 상태를 적용합니다. 보통 `-f` 옵션으로 파일과 함께 사용합니다. |
| `get`      | 리소스 목록을 보여줍니다.                                    |
| `describe` | 리소스의 상태를 자세하게 보여줍니다.                         |
| `delete`   | 리소스를 제거합니다.                                         |
| `logs`     | 컨테이너의 로그를 봅니다.                                    |
| `exec`     | 컨테이너에 명령어를 전달합니다. 컨테이너에 접근할 때 주로 사용합니다. |
| `config`   | kubectl 설정을 관리합니다.                                   |

### kubectl api-resources

: 쿠버네티스 전체 오브젝트 및 약어 정보 확인

```bash
$ kubectl api-resources

NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
secrets                                                                       true         Secret
serviceaccounts                   sa                                          true         ServiceAccount
services                          svc                                         true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
controllerrevisions                            apps                           true         ControllerRevision
daemonsets                        ds           apps                           true         DaemonSet
deployments                       deploy       apps                           true         Deployment
replicasets                       rs           apps                           true         ReplicaSet
statefulsets                      sts          apps                           true         StatefulSet
tokenreviews                                   authentication.k8s.io          false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io           true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io           false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling                    true         HorizontalPodAutoscaler
cronjobs                          cj           batch                          true         CronJob
jobs                                           batch                          true         Job
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest
leases                                         coordination.k8s.io            true         Lease
endpointslices                                 discovery.k8s.io               true         EndpointSlice
events                            ev           events.k8s.io                  true         Event
ingresses                         ing          extensions                     true         Ingress
ingressclasses                                 networking.k8s.io              false        IngressClass
ingresses                         ing          networking.k8s.io              true         Ingress
networkpolicies                   netpol       networking.k8s.io              true         NetworkPolicy
runtimeclasses                                 node.k8s.io                    false        RuntimeClass
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io      true         RoleBinding
roles                                          rbac.authorization.k8s.io      true         Role
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
csidrivers                                     storage.k8s.io                 false        CSIDriver
csinodes                                       storage.k8s.io                 false        CSINode
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment
```

* 특정 오브젝트 설명 보기

```bash
# 특정 오브젝트 설명 보기
$ kubectl explain pod
```



### get nodes

: node들의 정보를 가져옴

```bash
$ kubectl get nodes
```

```bash
# 자세한 정보 확인
$ kubectl get nodes -o wide

NAME           STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
controlplane   Ready    master   30m   v1.18.0   172.17.0.23   <none>        Ubuntu 18.04.5 LTS   4.15.0-122-generic   docker://19.3.13
node01         Ready    <none>   29m   v1.18.0   172.17.0.24   <none>        Ubuntu 18.04.5 LTS   4.15.0-122-generic   docker://19.3.13
```

```bash
# 원하는 노드의 자세한 정보
$ kubectl describe node {노드명}
```





### get pods

: pod들의 정보를 가져옴

```bash
$ kubectl get pods
```



#### log 조회

```bash
# Pod 조회로 이름 검색
$ kubectl get pod

# 조회한 Pod 로그조회
$ kubectl logs wordpress-5f59577d4d-8t2dg

# 실시간 로그 보기
$ kubectl logs -f wordpress-5f59577d4d-8t2dg
```



### --help

: 도움말을 보여줌

```bash
# 전체 명령어 도움말
$ kubectl --help

# logs 명령어 도움말
$ kubectl logs --help 
```



### Run

: 컨테이너를 실행할 pod를 생성한다.

```bash
# webserver라는 이름의 pod 생성 (namespace를 따로 지정해 주지 않았기 때문에 defalut namespace에 생성됨)
$ kubectl run webserver --image=nginx:1.14 --port 80
pod/webserver created 

$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
webserver   1/1     Running   0          11s

# 해당 pod의 정보 확인
$ kubectl describe pod webserver
```



### create deployment

```bash
# mainui 라는 이름의 deploymnet를 생성
# httpd image를 사용한 container를 포함하고 있으며
# 해당 deployment가 replica set 3개를 보장해 줌
# yaml파일로 deployment를 생성하는 방법도 있음
$ kubectl create deployment mainui --image=httpd --replicas=3
```

> #### deployment
>
> kubernetes Deployment는 애플리케이션 인스턴스들의 생성과 업데이트를 책임진다.
>
> Deployment를 생성하면 쿠버네티스 마스터는 클러스터에 존재하는 각각의 노드위에 애플리케이션 인스턴스를 스케줄링 한다.
>
> 애플리케이션 인스턴스들이 생성되면 쿠버네티스 Deployment 컨트롤러는 이러한 인스턴스들을 지속적으로 모니터링 한다.
>
> Deployment 컨트롤러는 노드 호스팅이 다운되거나 삭제가 된다면 인스턴스를 바꿔치기 한다. 이는 기계 고장 또는 장비 유지 보수에 대한 자가치유 메커니즘을 제공한다.
>
> Deployment의 생성과 관리는 쿠버네티스 command line interface로 사용한다. (kubectl)
>
> kubectl은 kubernetes API를 사용하여 cluster와 통신한다.
>
> Deployment를 생성할 때 애플리케이션을 구동하기 위한 복제 숫자 및 명시적인 컨테이너 이미지가 필요하다.
>
> Deployment Scaling up은 가용한 자원의 노드에 pods을 생성하고 스케줄링 하는 것을 보장한다.
>
> 쿠버네티스는 autoscaling을 제공한다.



### yaml 파일로 deployment 생성하는 방법

* deployment 구성 yaml 파일 생성

```yaml
# nginx-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

* 명령어를 통해 deployment 생성

```bash
$ kubectl apply -f ./nginx-deployment.yaml
```

* 생성된 것을 확인

```bash
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           22s
```



### deployment 수정

```bash
$ kubectl edit deployments {디플로이먼트명}

# vi editor가 뜸, replicas 등 수정 가능
```



### 동작중인 파드 수정

```bash
$ kubectl edit pod {pod명}
```



### pod 내용 확인

```bash
$ kubectl get pod -o wide
$ kubectl get pod -o yaml
$ kubectl get pod -o json

# 여러 TYPE 입력
$ kubectl get pod,service
#
$ kubectl get po,svc

# Pod, ReplicaSet, Deployment, Service, Job 조회 => all
$ kubectl get all

# Label 조회
$ kubectl get pod --show-labels

# 과정을 실시간으로 확인
$ kubectl get pods -o wide --watch

# namespace와 함께 확인
$ kubectl get pods --all-namespaces
```



### 컨테이너  명령어 전달

```bash
# Pod의 컨테이너에 접속
$ kubectl exec -it wordpress-5f59577d4d-8t2dg -- bash

# multipod일 경우
$ kubectl exec -it wordpress-5f59577d4d-8t2dg -c {컨테이너명} -- bash
```



##  설정 관리 (config)

kubectl은 여러 개의 쿠버네티스 클러스터를 컨텍스트context로 설정하고 필요에 따라 선택할 수 있습니다. 현재 어떤 컨텍스트로 설정되어 있는지 확인하고 원하는 컨텍스트를 지정합니다.

```sh
# 현재 컨텍스트 확인
kubectl config current-context

# 컨텍스트 설정
kubectl config use-context minikube
```



### port-forward

```bash
$ kubectl port-forward {pod명} 8080:80
```



### dry-run

: run을 통해 실행할 수 있는 상태인지 아닌지를 확인

```bash
# test01 이라는 pod를 생성할 수 있는지
$ kubectl run test01 --image=nginx:1.14 --port 80 --dry-run
```

* 실행할 수 있는 상태인지를 yaml format으로 확인

```bash
$ kubectl run test01 --image=nginx:1.14 --port 80 --dry-run -o yaml
```

* 확인한 것을 yaml파일로 저장

```bash
# test-pod.yaml 파일 생성
$ kubectl run test01 --image=nginx:1.14 --port 80 --dery-run -o yaml > test-pod.yaml
```

* 해당 yaml 파일로 pod를 생성

```bash
$ kubectl create -f test-pod.yaml

# test01 pod가 생성됌
pod/test01 created
```



#### Delete

```bash
# Pod 조회로 이름 검색
$ kubectl get pod

# 조회한 Pod 제거
$ kubectl delete pod/wordpress-5f59577d4d-8t2dg
```



#### alias 설정

`kubectl`명령어를 `k`로 줄여쓰면 편합니다.

```sh
# alias 설정
alias k='kubectl'

# shell 설정 추가
echo "alias k='kubectl'" >> ~/.bashrc
source ~/.bashrc
```



