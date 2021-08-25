---
title:  "[k8s] 1-1. minikube 환경 구성"

categories:
  - Kubernetes
tags:
  - [kubernetes, k8s, minikube]
toc: true
toc_sticky: true
---

## 실습환경

* 쿠버네티스(minikube) 설치

```bash
# docker 사용시 설치 필요
curl -fsSL https://get.docker.com/ | sudo sh
sudo usermod -aG docker $USER

# install minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/
```



* 기본 명령어

```bash
# 버전확인
minikube version

# 가상머신 시작
minikube start --driver=docker

# driver 에러가 발생한다면 virtual box를 사용
minikube start --driver=virtualbox

# 특정 k8s 버전 실행
minikube start --kubernetes-version=v1.20.0

# minikube 상태확인
minikube status

# minikube 실행
minikube start

# 특정 k8s 버전 실행
minikube start --kubernetes-version=v1.20.0

# 특정 driver 실행
minikube start --driver=virtualbox --kubernetes-version=v1.20.0

# minikube ip 확인 (접속테스트시 필요)
minikube ip

# minikube 종료
minikube stop

# minikube 제거
minikube delete
```

---

## 다양한 운영환경

- [kubeadm(opens new window)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [kubespray(opens new window)](https://github.com/kubernetes-sigs/kubespray)
- [Amazon EKS(opens new window)](https://aws.amazon.com/ko/eks)
- [Google Kubernetes Engine(opens new window)](https://cloud.google.com/kubernetes-engine)
- [Azure Kubernetes Service](https://docs.microsoft.com/ko-kr/azure/aks/)

##  다중노드

최신 minikube 버전을 사용한다면, 다중노드를 구성할 수 있습니다.

```sh
# 가상머신 시작 (단일노드, 기본)
minikube start

# 가상머신 시작 (단중노드)
minikube start -n 2
```

## 멀티 프로필

하나의 개발 PC에 여러개의 minikube를 사용할 수 있습니다.

```sh
# 가상머신 시작
minikube start # 기본 profile - minikube로 생성

# 두번째 가상머신 시작
minikube start -p hellowlrd # helloworld 라는 이름의 profile로 생성

# profile 목록 확인
minikube profile list

# 현재 사용중인 profile 확인
minikube profile

# 다른 profile로 변경
minikube profile hellowlrd # helloworld로 변경
minikube profile minikube # minikube로 변경

# 가상머신 제거
minikube delete # 현재 사용중인 profile의 가상머신 제거
```

### Kubectl install

```sh
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

# test
kubectl version
```



#### 클러스터 세부사항

```bash
# 클러스터 세부사항 조회
$ kubectl cluster-info
```

```bash
Kubernetes master is running at https://172.17.0.26:8443
KubeDNS is running at https://172.17.0.26:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

>**KubeDNS** 란?
>
>클러스터의 서비스와 DNS파드를 관리하며, 개별 컨테이너들이 DNS 네임을 해석할 때 DNS 서비스의 IP를 사용하도록 Kuberlets를 구성한다.
>
>이를 통해 쿠버네티스에서는 **클러스터 내부에서만 사용가능한 DNS**를 설정하여 사용할 수 있다.
>
>또한 특정 PoD에 문제가 생기거나 배포가되어 재생성될 때 IP가 변경되더라도 KubeDNS에서 **자동으로 변경된 포드의 IP를 등록**하는 역할도 수행한다.



```bash
# 클러스터 노드 정보 조회
$ kubectl get nodes
```

```bash
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   10m   v1.17.4
```

mimikube master노드가 생성된 것을 확인할 수 있다.