---
layout: post
title: Elastic APM Server 구축하기
feature-img: assets/img/titles/elastic-logo.png
thumbnail: assets/img/contents/eas-1.png
author: csupreme19
tags: [Elasticsearch, Elastic, ELK, APM, ArgoCD, GitOps, Helm]

---

# Elastic APM Server 구축하기

![elastic-logo.png]({{ "/assets/img/titles/elastic-logo.png"}})

Elastic APM Server 구축기

---

## 개요

![eas-1.png]({{ "/assets/img/contents/eas-1.png"}})

[APM Server Overview](https://www.elastic.co/guide/en/apm/server/7.12/overview.html)

APM Server는 APM Agent로부터 메트릭, 로그 정보등을 수집하고 엘라스틱서치 Document로 변환하여 인덱스에 저장하는 역할을 한다.

### APM?

APM(Application Performance Monitoring)

어플리케이션 성능 지표 등을 모니터링하는 것을 말한다.

어플리케이션 레벨의 메트릭 정보, 로그 정보, 트랜잭션 추적, API 레이턴시 등 인프라/컨테이너 레벨에서는 알기 힘든 정보들을 가시화해준다.

Java APM을 예로들면 아래와 같은 정보들을 모니터링 가능하다.

- HTTP Requests/Responses
- Latency
- Throughput
- Transactions
- Error rate
- Dependencies
- JVM Metrics
- DB Queries

---

## Elasitc APM 서버 설치
### apm-server 설치(deb)
#### 1. 설치

```shell
$ cd /root
$ curl -L -O https://artifacts.elastic.co/downloads/apm-server/apm-server-7.12.1-amd64.deb
$ dpkg -i apm-server-7.12.1-amd64.deb
```
#### 2. apm-server.yml 설정

```shell
$ vim /etc/apm-server/apm-server.yml
```
```yaml
#-------------------------- Elasticsearch output --------------------------
output.elasticsearch:
  hosts: ["http://{엘라스틱서치1}:9200", "http://{엘라스틱서치2}:9200", "http://{엘라스틱서치3}:9200"]
  enabled: true
  protocol: "http"
  username: "elastic"
  password: "changeme"
```
#### 3. 환경설정

```shell
$ sudo -u apm-server apm-server setup
Index setup finished.
Loaded Ingest pipelines
```
> debian 패키지로 설치한 경우 공식문서에서는 non-root user로 실행권장(`apm-server`라는 유저가 자동 생성됨)
#### 4. apm-server 실행

```shell
$ sudo -u apm-server apm-server -e
```
#### 5. 접속 확인

```shell
$ curl -ivs localhost:8200
```

### apm-server 설치(k8s)

[charts/apm-server](https://hub.kubeapps.com/charts/elastic/apm-server/7.12.1)

헬름 차트를 사용하여 K8S에 배포

#### 1. gitlab 접속

#### 2. argocd git 저장소

폴더 elastic/apm-server 생성

> ArgoCD와 같은 GitOps 미적용시 일반 디렉토리로 대체

#### 3. 헬름 차트 다운로드

[helm-charts/apm-server](https://github.com/elastic/helm-charts/tree/v7.12.1/apm-server)

git clone 또는 helm pull

```sh
$ git clone git@github.com:elastic/helm-charts.git
```

```sh
$ helm repo add elastic https://helm.elastic.co
$ helm pull elastic/apm-server
```



#### 4. helm template에 secret 추가

```shell
$ cd elastic/apm-server/templates
$ vim secret.yaml
```
```yaml
{% raw %}
{{- if .Values.secret }}
---
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Values.secret.name }}
  labels:
    app: {{ .Chart.Name }}
    release: {{ .Release.Name | quote }}
    heritage: {{ .Release.Service }}
type: {{ .Values.secret.kind }}
data: 
  {{- range $key, $value := .Values.secret.data }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
{{- end -}}
{% endraw %}
```
> elastic apm-server 헬름 차트에서 secret 타입 템플릿을 제공하지 않으므로 직접 작성하는 과정이다.


#### 5. `values.yaml` 수정

```yaml

---
# Allows you to add config files
apmConfig:
  apm-server.yml: |
    apm-server:
      host: "0.0.0.0:8200"
    queue: {}
    output.elasticsearch:
      hosts: ["http://{엘라스틱서치1}:9200", "http://{엘라스틱서치2}:9200", "http://{엘라스틱서치3}:9200"]
      username: "${ELASTICSEARCH_USERNAME}"
      password: "${ELASTICSEARCH_PASSWORD}"

# Extra environment variables to append to the DaemonSet pod spec.
# This will be appended to the current 'env:' key. You can use any of the kubernetes env
# syntax here
extraEnvs:
  - name: 'ELASTICSEARCH_USERNAME'
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: username
  - name: 'ELASTICSEARCH_PASSWORD'
    valueFrom:
      secretKeyRef:
        name: elastic-credentials
        key: password

# secret은 새로 만든 템플릿의 value이므로 추가
# Allows kubernetes secret
secret:
  name: elastic-credentials
  type: Opaque
  data:
    username: ZWxhc3RpYw==
    password: Szg2eWN5aEFERUtvV1Vid2dPMGo=

# 노드 선택을 위한 셀렉터
nodeSelector: 
  zone: private
  role: worker
  
```
#### 6. git commit & push

#### 7. ArgoCD new App

- Application Name: apm-server
- Project: default
- Repository URL: ssh://git@{gitlab 주소}:30722/{group명}/{gitops 저장소 주소}.git
- Path: elastic/apm-server
- Cluster URL: https://kubernetes.default.svc
- Namespace: elastic

8. #### ArgoCD Sync

   ![eas-2.png]({{ "/assets/img/contents/eas-2.png"}})

   > ArgoCD와 같은 GitOps 미사용시 해당 헬름 upgrade하여 직접 배포
   >
   > ```sh
   > $ helm install apm-server elastic/apm-server -f values.yaml
   > ```
   >
   > 

#### 9. 배포 확인

```shell
$ kubectl exec -it apm-server-apm-server-6ccd949d46-hfh29 bash -n elastic

bash-4.2$ curl -i localhost:8200
HTTP/1.1 200 OK
Content-Type: application/json
X-Content-Type-Options: nosniff
Date: Thu, 22 Jul 2021 08:05:23 GMT
Content-Length: 125

{
  "build_date": "2021-04-20T19:55:39Z",
  "build_sha": "32f34ed4298d648bf9476790f2a8a54d72805bb6",
  "version": "7.12.1"
}
```

---

## Reference

1. [APM Server Overview](https://www.elastic.co/guide/en/apm/server/7.12/overview.html)
2. [charts/apm-server](https://hub.kubeapps.com/charts/elastic/apm-server/7.12.1)
3. [helm-charts/apm-server](https://github.com/elastic/helm-charts/tree/v7.12.1/apm-server)

