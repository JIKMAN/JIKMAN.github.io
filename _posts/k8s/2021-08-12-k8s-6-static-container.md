---
title:  "[k8s] 6. Static Container"

categories:
  - Kubernetes
tags:
  - [kubernetes, k8s, Static Container]
toc: true
toc_sticky: true
---

## Static Container

k8s 작동 원리는 API 서버에 명령을 요청해서 control plane에서 worker node에 pod를 할당하게 됩니다.

그러나 __Static Container__를 이용하면 API 서버 없이 특정 노드에 있는 kubelet 데몬에 의해 컨테이너를 직접 관리할 수 있습니다.

노드의 /etc/kubernetes/manifests/ 디렉토리에 k8s yaml 파일을 저장하면 적용됩니다.

1. 노드에 접속 후

`/var/lib/kubelet/config.yaml`을 살펴보면 아래와 같은 경로로 staticPod 경로가 구성되어 있다.

```bash
staticPodPath: /etc/kubernetes/manifests
```

* static pod 디렉토리 구성

2. staticPodPath`/etc/kubernetes/manifests`  에 yaml파일을 저장하면

3. kublet 데몬을 통해 pod가 실행된다. (k8s apiserver를 거치지 않음)

4. kublet이 실행 중일 경우 재시작한다.

   ```bash
   systemctl restart kubelet
   ```

   

#### yaml 파일 예시

```yaml
# apiserver에서 사용되던 pod yaml 파일과 같다..

apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
```



__참조__

https://kubernetes.io/ko/docs/tasks/configure-pod-container/static-pod/