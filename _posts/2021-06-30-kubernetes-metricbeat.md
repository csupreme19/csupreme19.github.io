---
layout: post
title: Kubernetes Metricbeat 적용하기
feature-img: assets/img/titles/kubernetes-logo.png
thumbnail: assets/img/contents/km-1.png
author: csupreme19
categories: DevOps Elastic
tags: [Kubernetes, Helm, K8S, Metricbeat, ArgoCD, GitOps, ELK, Elastic, Elasticsearch, Metric, Monitoring, Observability]

---

# Kubernetes Metricbeat

![kubernetes-logo.png]({{ "/assets/img/titles/kubernetes-logo.png"}})

Kubernetes 클러스터, 노드, 파드의 메트릭 정보 수집을 위한 `kube-state-metrics`와 metricbeat를 배포하여 메트릭 정보를 수집, 모니터링한 경험을 작성해봤어요.

---

## 개요

Kubernetes 메트릭 정보 수집을 위해 Metricbeat를 DaemonSet 형태로 띄워 메트릭을 수집해야 하는 상황이에요.


---
## `kube-state-metrics`
kube-system의 `kube-state-metrics` deployment 필요해요.

helm-chart를 이용하여 구성할 예정이에요.

GitOps인 ArgoCD를 활용하여 구성한다고 가정하였으며 일반 디렉토리 생성하여 구성하여도 동일해요.

### `kube-state-metrics` using helm

[helm-charts/kube-state-metrics](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-state-metrics)

#### 1. GitLab ArgoCD repo에 kube-system 디렉토리 생성(새 네임스페이스이므로)

#### 2. kube-system/kube-state-metrics 디렉토리 생성

#### 3. 위 helm 링크에서 helm chart 다운받아 kube-state-metrics에 넣기

#### 4. `values.yaml` 수정

worker 노드에 파드를 띄우기 위해 아래 nodeSelector 지정했어요.

(zone과 role에 대한 label이 노드에 이미 지정되어 있다고 가정)

```yaml
nodeSelector:
 zone: private
 role: kong
```

#### 5. ArgoCD New App 생성(kube-system)

#### 6. ArgoCD sync 진행

#### 7. 배포 확인

---
## Metricbeat-kubernetes

kubernetes 메트릭을 모니터링하기위해 metricbeat를 DaemonSet 형태로 k8s cluster에 띄워야해요.

<br>

### Metricbeat-kubernetes DaemonSet 배포

#### 1. GitLab ArgoCD repo kube-system/metricbeat-kubernetes 생성

#### 2. `metricbeat-kubernetes.yaml` 다운로드

```shell
# ELK 버전 7.12.1
$ curl -L -O    https://raw.githubusercontent.com/elastic/beats/7.12/deploy/kubernetes/metricbeat-kubernetes.yaml
```

#### 3. 위에서 받은 `metricbeat-kubernetes.yaml` gitlab repo에 파일 추가

#### 4. `values.yaml` 수정

```yaml
# Elasticsearch 호스트 지정
 - name: ELASTICSEARCH_HOST
   value: elasticsearch
 - name: ELASTICSEARCH_PORT
   value: "9200"
 - name: ELASTICSEARCH_USERNAME
   value: elastic
 - name: ELASTICSEARCH_PASSWORD
   value: changeme
```

#### 5. ArgoCD sync 진행

#### 6. 배포 확인

---
## Kibana 대시보드 확인

![km-1.png]({{ "/assets/img/contents/km-1.png"}})

---
## 참고사항

### 수집 영역
  - kubelet
  - kube-state-metrics
  - apiserver
  - controller-manager
  - scheduler
  - proxy

<br>

### Metricsets
사용 가능한 Metricset들 리스트
```
    apiserver
    container
    controllermanager
    event
    node
    pod
    proxy
    scheduler
    state_container
    state_cronjob
    state_daemonset
    state_deployment
    state_node
    state_persistentvolumeclaim
    state_pod
    state_replicaset
    state_resourcequota
    state_service
    state_statefulset
    state_storageclass
    system
    volume
```


---

## Reference

1. [Run Metricbeat on Kubernetes](https://www.elastic.co/guide/en/beats/metricbeat/current/running-on-kubernetes.html)

2. [Kubernetes 통합 가시성 자습서: 메트릭 수집 및 분석](https://www.elastic.co/kr/blog/kubernetes-observability-tutorial-k8s-metrics-collection-and-analysis)
3. [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics#kubernetes-deployment)
4. [helm-charts/kube-state-metrics](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-state-metrics)