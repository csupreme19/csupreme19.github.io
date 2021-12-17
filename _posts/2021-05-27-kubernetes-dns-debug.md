---
layout: post
title: Kubernetes DNS Debug
feature-img: assets/img/titles/kubernetes-logo.png
thumbnail: assets/img/titles/kubernetes-logo.png
author: csupreme19
categories: DevOps Kubernetes
tags: [Kubernetes, K8S, DNS, Proxy, Debug]

---

# Kubernetes DNS Debug

![kubernetes-logo.png]({{ "/assets/img/titles/kubernetes-logo.png"}})

[Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)

kubenretes DNS 서비스 접근시 디버깅 과정 정리

---
## Kubernetes DNS Debugging

kubenretes cluster 내부에서 DNS 이용하여 서비스 접근시 디버깅 과정 정리

### 디버깅 순서

#### 1. 실 서비스 내부 파드 접속

```shell
$ kubectl exec -it yourservice-prd-deploy-6fddbc47c6-sljr9 -n production sh
```

#### 2. `/etc/resolv.conf` 설정 확인

```shell
# pod 내에서 진행
$ cat /etc/resolv.conf
nameserver 10.96.0.10
search production.svc.cluster.local svc.cluster.local cluster.local
options ndots:5

# kubernetes cluster에서 진행
$ kubectl get svc -n kube-system kube-dns
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   14d
```

kube-dns 서비스의 Cluster IP와 `/etc/resolv.conf`의 nameserver 주소가 일치하는지 확인

#### 3. CoreDNS pod running 확인

```shell
# kubernetes cluster에서 진행
$ kubectl get po -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
coredns-74ff55c5b-knln5                    1/1     Running   0          3h57m
coredns-74ff55c5b-q9z5f                    1/1     Running   0          3h58m
```

DNS Daemonset 정상 배포 & Pod Running 확인

#### 4. `nslookup`으로 DNS lookup 정상 작동 확인

```shell
# pod 내에서 진행
$ nslookup kubernetes.default
nslookup: can't resolve '(null)': Name does not resolve

Name:      kubernetes.default
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
```

#### 5. 접속하려는 서비스 이름 확인

```shell
# kubernetes cluster에서 진행
$ kubectl get svc -n kong -o wide
NAMESPACE              NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kong                   kong-kong-proxy              NodePort    10.104.186.93    <none>        80:30080/TCP,443:30443/TCP   10d
```

#### 6. 서비스 DNS lookup 확인

```shell
# pod 내에서 진행
$ nslookup kong-kong-proxy.kong.svc
nslookup: can't resolve '(null)': Name does not resolve

Name:      kong-kong-proxy.kong.svc
Address 1: 10.104.186.93 kong-kong-proxy.kong.svc.cluster.local
```

서비스의 경우 `my-svc.my-namespace.svc.cluster-domian.example`의 형태로 DNS 이름이 정해진다.

> 예) 
>
> - `kong-kong-proxy.kong.svc.cluster.local`
> - `kong-kong-proxy.kong.svc`

#### 7. curl 설치 및 요청

```shell
# pod 내에서 진행
$ apk add curl	# curl 설치
$ curl -v kong-kong.proxy.kong.svc
* Connected to kong-kong-proxy.kong.svc.cluster.local (10.104.186.93) port 80 (#0)
> GET / HTTP/1.1
> Host: kong-kong-proxy.kong.svc.cluster.local
> User-Agent: curl/7.64.0
> Accept: */*
>
< HTTP/1.1 404 Not Found
< Date: Thu, 27 May 2021 06:36:46 GMT
< Content-Type: application/json; charset=utf-8
< Connection: keep-alive
< Content-Length: 48
< X-Kong-Response-Latency: 1
< Server: kong/2.3.3
<
* Connection #0 to host kong-kong-proxy.kong.svc.cluster.local left intact
{"message":"no Route matched with those values"}/
```
> alpine 이미지의 경우 curl 패키지 없어서 설치해야함
>

서비스 접속 확인


---

## Reference

1. [Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)