---
layout: post
title: Kubernetes Pod 재시작 장애 확인하기
feature-img: assets/img/titles/kubernetes-logo.png
thumbnail: assets/img/titles/kubernetes-logo.png
author: csupreme19
tags: [Kubernetes, Pod, Fail]

---

# Kubernetes Pod 재시작 장애 확인하기

[Container restart policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)

강제로 죽는 pod를 배포 후 해당 파드가 어떻게 restart 되는지 살펴보았다.

---

## 실험

### 실험 절차



#### 1. Pod 배포

```yaml
# alias k=kubectl

# Declarative way
$ vim dummy-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: dummy-pod
spec:
  containers:
    - name: dummy-pod
      image: ubuntu
  restartPolicy: Always
  
$ k create -f dummy-pod.yaml
pod/dummy-pod created

# Imperative way
$ k run dummy-pod --image ubuntu
pod/dummy-pod created
```



#### 2. Pod 상태 확인

```sh
$ k get po
NAME                       READY   STATUS              RESTARTS   AGE
dummy-pod                  0/1     ContainerCreating   0          2s

$ k get po
NAME                       READY   STATUS      RESTARTS   AGE
dummy-pod                  0/1     Completed   1          10s

$ k get po
NAME                       READY   STATUS      RESTARTS   AGE
dummy-pod                  0/1     Completed   2          32s

$ k get po
NAME                       READY   STATUS      RESTARTS   AGE
dummy-pod                  0/1     Completed   3          61s

$ k get po
NAME                       READY   STATUS      RESTARTS   AGE
dummy-pod                  0/1     Completed   4          118s

$ k get po
NAME                       READY   STATUS             RESTARTS   AGE
dummy-pod                  0/1     CrashLoopBackOff   4          2m34s

$ k get po
NAME                       READY   STATUS      RESTARTS   AGE
dummy-pod                  0/1     Completed   5          3m33s
```

10, 30, 60, 110, 210초에 각각 restart되는 것으로 확인

공식 문서에 따르면 지수 백오프 지연(10초, 20초, 40초, ...)로 10초로 시작하여 2배씩 재시작 간격이 증가한다고 함(최대 300초(5분))

지수배는 아니지만 비슷하게 재시작 된 것을 확인할 수 있었다.



#### 3. 결론

아래와 같이 다양한 기준을 적용하여 파드가 계속 재시작중인지 restart 횟수 만으로 판별 가능
- 파드 배포 후 1분 내 2번 이상(10, 30, 60초)
- 파드 배포 후 3분 내 4번 이상(10, 30, 60, 110, 210초)


---

## Reference

1. [Container restart policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)