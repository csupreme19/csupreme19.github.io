---
layout: post
title: Pod OOMKilled 원인 찾기
feature-img: assets/img/titles/kubernetes-logo.png
thumbnail: assets/img/contents/kpot-1.png
author: csupreme19
tags: [Kubernetes, K8S, Pod, JVM, Heap, Memory]

---

# Kubernetes Pod OOMKilled 원인 찾기

![kubernetes-logo.png]({{ "/assets/img/titles/kubernetes-logo.png"}})

Kubernetes Pod OOMKilled 원인 찾는 과정을 정리한 문서이다.

---

## OOMKilled 원인을 찾아서...

### 발단

![kpot-1.png]({{ "/assets/img/contents/kpot-1.png"}})

오전 1시 50분 production 운영 cluster에 올라간 Pod중 하나가 죽었다는 알림이 발생하였다.

보통 k8s cluster에서는 controller-manager과 kube-scheduler에 의해 자동 복구 되므로 큰 이슈는 없으나 추후 문제를 최소화하기 위하여 기록해본다.

상태는 `OOMKilled`로 파드에 Out Of Memory가 발생하여 재시작됨

#### 1. 해당 시점 로그 확인

![kpot-2.png]({{ "/assets/img/contents/kpot-2.png"}})

Elastic Stack을 구축하여 메트릭과 로그를 수집하고 있으므로 Kibana에서 Application Log를 살펴보았다.(stdout)

유의미한 로그는 찾기 힘들지만 1시 50분에 재구동된 사실을 double-check 할 수 있었다.

#### 2. 해당 시점 메트릭 확인

![kpot-3.png]({{ "/assets/img/contents/kpot-3.png"}})

메모리 사용량이 지속적으로 누적되어 100%를 초과하여 아웃 오브 메모리가 발생하였다.

#### 3. 상세 내용 확인

상세 내용을 확인하기 위하여 OOMKilled가 발생한 해당 서비스 deployment에 Java APM Agent를 적용 후 경과를 관찰하였다.

![kpot-4.png]({{ "/assets/img/contents/kpot-4.png"}})

![kpot-5.png]({{ "/assets/img/contents/kpot-5.png"}})

System memory usage, Heap memory, Thread count가 지속적으로 상승하는 것을 확인하였다.

Java heap 메모리 누수로 인한 것으로 결론지었다.

하지만

### 메모리 사용량이 이상하다?

![kpot-6.png]({{ "/assets/img/contents/kpot-6.png"}})

위 메모리 사용량의 경우 `kubernetes.pod.memory.usage.limit.pct` 메트릭을 이용하여 확인하였는데.

해당 메트릭은 일부 파드의 경우 위와 같이 메모리가 99.9% 사용량(1.5GB)을 유지하지만 Pod 상태에는 문제가 없다는 것을 발견하였다.

99.9%가 유지되는 것도 이상하지만 OOMKilled가 발생하지 않고 정상적으로 작동하는 것이 이상하다고 생각하여 아래와 같이 생각해보았다.

1. 메트릭 정보가 잘못된 경우
2. `kubernetes.pod.memory.usage.limit.pct` 메트릭이 메모리 사용량만을 나타내는 것이 아닌 경우

#### kubectl top 확인

```sh
$ kubectl top pod -n production | g
{서비스명}-prd-deploy-86975ffd89-kvdr4	54m	298Mi
{서비스명}-prd-deploy-86975ffd89-n4r85	43m	254Mi
```

실제 pod의 메모리 사용량으로 산정되는것과 다른 것을 확인할 수 있었다.

![kpot-7.png]({{ "/assets/img/contents/kpot-7.png"}})

kubernetes metricbeat에 따르면 kubernetes 메모리 관련 메트릭은 위와 같다.

kubelet은 내부적으로 cAdvisor를 사용하여 자원 사용량을 모니터링한다.

> [cAdvisor](https://github.com/google/cadvisor)는 리눅스 cgroup의 메트릭 정보를 수집하는 오픈소스이다.

이중 위 빨간 박스의 3가지 메트릭은 cAdvisor의 프로메테우스 형식의 메트릭에 대응된다.

- `kubernetes.pod.memory.usage.bytes` = `container_memory_usage_bytes`
- `kubernetes.pod.memory.working_set.bytes` = `container_memory_working_set_bytes`
- `kubernetes.pod.memory.rss.bytes` = `container_memory_rss`

- `container_memory_working_set_bytes`

최근 할당된 메모리, 더티 메모리, 커널 메모리를 포함하는 메모리 사용량

- `container_memory_rss`

swap 캐시 메모리를 포함하는 메모리 사용량

![kpot-8.png]({{ "/assets/img/contents/kpot-8.png"}})

실제 메트릭 정보를 살펴보면 Pod의 메모리 사용량은

`kubernetes.pod.memory.working_set.bytes`, `kubernetes.pod.memory.rss.bytes`에 근접한다는 것을 확인할 수 있다.

### 메모리 확인 결과

`kubernetes.pod.memory.working_set.bytes`이 limit에 도달하거나 `kubernetes.pod.memory.rss.bytes`가 limit에 가깝다면 OOMKilled의 대상이될 가능성이 높아진다.

kubectl pod 조회 결과 나타나는 메모리 사용량은 `kubernetes.pod.memory.working_set.bytes`와 동일하다.

따라서 두 메트릭을 조회하여 limit에 근접하는 경우 OOMKilled 가능성이 높음을 확인할 수 있다.

### 메모리 누수 Pod 확인 결과

모든 pod를 수동으로 확인 결과 사용 메모리가 꾸준히 우상향하는 Pod 리스트

1. {서비스1}

![kpot-9.png]({{ "/assets/img/contents/kpot-9.png"}})

2. {서비스2}

![kpot-10.png]({{ "/assets/img/contents/kpot-10.png"}})

3. {서비스3}

![kpot-11.png]({{ "/assets/img/contents/kpot-11.png"}})

4. {서비스4}

![kpot-12.png]({{ "/assets/img/contents/kpot-12.png"}})

5. {서비스5}

![kpot-13.png]({{ "/assets/img/contents/kpot-13.png"}})

6. {서비스6}

![kpot-14.png]({{ "/assets/img/contents/kpot-14.png"}})

7. {서비스7}

![kpot-15.png]({{ "/assets/img/contents/kpot-15.png"}})

8. {서비스8}

![kpot-16.png]({{ "/assets/img/contents/kpot-16.png"}})

9. {서비스9}

![kpot-17.png]({{ "/assets/img/contents/kpot-17.png"}})

10. {서비스10}

![kpot-18.png]({{ "/assets/img/contents/kpot-18.png"}})

### JVM Heap 메모리 확인 결과

![kpot-5.png]({{ "/assets/img/contents/kpot-5.png"}})

![kpot-19.png]({{ "/assets/img/contents/kpot-19.png"}})

파드 메모리 확인 결과 

system memory와 thread count는 지속적으로 증가하지만 heap memory는 약 250mb 도달시 GC되는 것을 확인할 수 있었다.

따라서 JVM Heap Memory 문제는 아니라고 판단하여 OS 레벨을 확인해보았다.

### OS memory leak?

top, free 명령어를 이용하여 메모리 모니터링을 해보았으나 유의미한 값을 찾지는 못하였다.

### 다시 JVM으로...

APM에서 수집한 JVM 메트릭 정보를 다시 확인해보니 Non-Heap Memory와 thread count가 지속적으로 증가하는 것을 확인하였다.

#### Thread count 증가

![kpot-20.png]({{ "/assets/img/contents/kpot-20.png"}})

#### Non Heap memory 증가

![kpot-21.png]({{ "/assets/img/contents/kpot-21.png"}})

106mb에서 107.5mb로 점진적으로 증가한 모습을 보였다.

확인 결과 2개의 파드를 제외하고 모두 non-heap memory가 증가되는 것으로 보임

---

## 후기

Memory 누수가 어디서 발생하는지는 찾았지만 JVM 메모리 모니터링 및 디버깅을 통하여 deep dive로 직접적인 원인을 찾지 못하여 찜찜하다.

현재 다른 업무로 인하여 관심에서 멀어졌으나 언제 한번 기회가 된다면 다시 찾아봐야겠다.

> [https://stackoverflow.com/a/39330016](https://stackoverflow.com/a/39330016)
>
> Java 8 부터 Class Metadata가 Non-Heap 메모리에 저장되어 그럴 수 있다고 하는데... 한번 살펴봐야겠다.


---

## Reference

1. [https://www.magalix.com/blog/memory_working_set-vs-memory_rss](https://www.magalix.com/blog/memory_working_set-vs-memory_rss)
2. [https://www.getoutsidedoor.com/2021/03/30/kubernetes-pod-memory-monitoring/](https://www.getoutsidedoor.com/2021/03/30/kubernetes-pod-memory-monitoring/)
3. [https://engineering.linecorp.com/ko/blog/prometheus-container-kubernetes-cluster/](https://engineering.linecorp.com/ko/blog/prometheus-container-kubernetes-cluster/)
4. [https://stackoverflow.com/a/39330016](https://stackoverflow.com/a/39330016)