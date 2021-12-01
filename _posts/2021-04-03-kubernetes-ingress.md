---
layout: post
title: Kubernetes Nginx Ingress 적용기
feature-img: assets/img/titles/nginx-logo.svg
thumbnail: assets/img/contents/ki-1.png
author: csupreme19
tags: [Kubernetes, K8S, Ingress, Ingress Controller, Certificate, TLS, SSL, Nginx, Security]

---

# Kubernetes Nginx Ingress 적용기

![nginx-logo.svg]({{ "/assets/img/titles/nginx-logo.svg"}})

[Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

본 문서는 온프레미스 환경에서의 Ingress, SSL 인증서 적용 내용을 정리하였다.

nginx ingress controller를 이용하여 jenkins 서비스에 연결하는 과정을 정리하였다.

---
## 선행 사항
- Let's Encrypt 인증서 발급
- Kubernetes 환경 구축
- Ingress controller 설치

---
## Ingress 개요
<div class="mermaid">
flowchart LR
  A[User]
  subgraph Kubernetes Cluster
  subgraph Ingress
  B[Service: Ingress Contoller]
  J[Pod: Ingress Controller]
  F[Resource: Ingress]
  end
  subgraph Deployment1
  C[Service1]
  G[Pod1]
  end
  subgraph Deployment2
  D[Service2]
  H[Pod2]
  end
  subgraph Deployment3
  E[Service3]
  I[Pod3]
  end
  end
  A -.Ingress.-> B
  B---J
  C---G
  D---H
  E---I
  B---F
  B --> C & D & E
</div>



외부에서 클러스터에 접근할 때 요청받는 것이 Ingress이다. 쉽게 얘기하면 일반적인 proxy, gateway라고 보면 된다.

동일하게 로드밸런서, 서비스 메쉬의 역할도 수행하며 경우에 따라서는 Blue Green 배포나 Canary 배포도 가능하다.

외부 → Ingress → Service → Pod

내부 서비스에 SSL 인증서를 적용하는 것이 아닌 Ingress에 인증서를 적용한다.

이와 같이 적용하면 서비스 종속성 없이 앞단에서 인증서 적용이 가능하므로 뒷단의 서비스가 추가되어도 쉽게 적용이 가능하다.

### `Ingress Controller`와 `Ingress Resource`의 차이

![ki-2.png]({{ "/assets/img/contents/ki-2.png"}})

#### Ingress Controller

![ki-1.png]({{ "/assets/img/contents/ki-1.png"}})

> 이미지: [Certified Kubernetes Administrator(CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/) 발췌

- `80:31280/TCP`, `443:32443/TCP`등의 서비스와 함께 NodePort 형식으로 외부와의 연결을 담당
- 보통 nginx, haproxy 등 proxy 서버로 구현되어 있다.
- Ingress Resource의 설정값(Ingress Rule)에 따라 클러스터 내의 Service로 연결한다.

#### Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
...
spec:
  rules:
  - host: service.yourdomain.com # 이 부분 추가 (하이픈은 맨 위에만 있어야함)
    http:
      paths:
      - pathType: Prefix
        path: "/service"
        backend:
          service:
            name: service-name
            port:
              number: 8080
```

- Ingress Controller에서 어떤 서비스로 라우팅할 것인지 규칙을 명세한 Resource이다.
- path등의 값으로 어떤 service에 연결할 것인지에 대한 명세이다.
- Kubernetes secret 리소스를 참고하여 tls 연결을 설정할 수 있다.

---
## Ingress 적용

### 인증서 파일 구성

PEM 포맷의 RSA crt, key가 필요하다.

본 문서에서는 Let's Encrypt의 DNS 포맷으로 발급된 *.yourdomain.com 도메인 인증서를 사용한다.

```sh
# yourdomain.com.crt
-----BEGIN CERTIFICATE-----
MII...
-----END CERTIFICATE-----

# yourdomain.com.crt
Private-Key: (2048 bit)
modulus:
...
-----BEGIN RSA PRIVATE KEY-----
MII...
-----END RSA PRIVATE KEY-----
```


### Kubernetes secret 생성

kubernetes에서는 여러가지 시크릿 타입을 제공하는데 

그 중 SSL 인증서에 대한 정보를 담고 있는 Secret 타입은 `kubernetes.io/tls` 이다.

secret 생성하는 두 가지 방법이 존재

#### 1. secret yaml 파일 생성

```sh
$ vim service-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  # the data is abbreviated in this example
  tls.crt: |
        MIIC2DCCAcCgAwIBAgIBATANBgkqh ...
  tls.key: |
        MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ...
```

이 경우 인증서 값은 -----BEGIN CERTIFICATE-----와 -----END CERTIFICATE----- 사이에 있는 인코딩값을 넣어야한다고 한다.

> 확인결과 .crt 안에 있는 인증서 값을 모두 포함하여야함

```sh
$ kubectl create -f service-secret.yaml
```

kubernetes secret 생성

#### 2. 직접 생성(권장) 

```sh
# kubectl create secret {type} {name} --cert {certPath} --key {keyPath}
$ kubectl create secret tls secret-tls --cert ssl/yourdomain.com.crt --key ssl/yourdomain.com.key
```

Imperative 방식으로 생성

인증서 파일을 설정값으로 물고갈 수 있어서 권장

```sh
# secret 생성 확인
$ kubectl get secret
$ kubectl describe secret secret-tls
```
---
## Ingress TLS 적용

```yaml
$ vim jenkins-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
...
spec:
  rules:
  - host: jenkins.yourdomain.com # 이 부분 추가 (하이픈은 맨 위에만 있어야함)
    http: 
      paths:
      - pathType: Prefix
        path: "/jenkins"
        backend:
          service:
            name: jenkins
            port:
              number: 8080
  # 아래 부분 추가
  tls:
  - hosts:
    - jenkins.yourdomain.com
    secretName: secret-tls
```

```sh
# 적용
$ kubectl apply -f jenkins-ingress.yaml
```
위에서 생성한 Secret을 이용하면 Ingress에 인증서를 적용할 수 있다.

### 인증서 적용 확인

#### 1. Ingress Controller 접속 확인

```sh
# ingress-nginx 서비스 포트매핑 확인
$ kubectl get svc -A

NAMESPACE       NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   ingress-nginx-controller             NodePort    10.104.33.237    <none>        80:31623/TCP,443:32452/TCP   29h

$ curl -k -v https://10.104.33.237:443
...
* Connected to 10.104.33.237 (10.104.33.237) port 443 (#0)
...
```

#### 2. 외부 도메인 접속 확인

```sh
$ curl -k -v https://jenkins.yourdomain.com:32452
...
* Connected to jenkins.yourdomain.com (...) port 32452 (#0)
...
```

#### 3. 인증서 검증

```sh
$ openssl s_client -debug -connect https://jenkins.yourdomain.com:32452

...
subject=CN = *.yourdomain.com

issuer=C = US, O = Let's Encrypt, CN = R3
...
---
```

#### 4. 웹 브라우저로 접속하여 인증서 확인

![ki-3.png]({{ "/assets/img/contents/ki-3.png"}})

---
### SSL Redirect 끄기

위처럼 Ingress 설정에 tls를 적용하면 https 접속이 강제되므로 기존에 http에 운영중인 서비스에 영향이 있다.

따라서 기존의 http 서비스는 https로 redirect 되지 않도록 설정하여야 한다.

#### 1. Ingress rule 변경

```yaml
$ vim jenkins-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  ...
  namespace: default
  # 아래 annotations 추가
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
```

> Ingress rule의 `ssl-redirect` 설정은 host 주소와 port는 그대로 둔채 http 요청을 https로 리다이렉트한다.(서버에서 https redirect 응답)
>
> 예시) http://jenkins.yourdomain.com:31623/jenkins → https://jenkins.yourdomain.com:31623/jenkins

#### 2. ConfigMap 설정 변경

```yaml
# vim ~/ingress-controller/ingress-nginx/deploy/static/provider/baremetal/deploy.yaml

# Source: ingress-nginx/templates/controller-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.23.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 0.44.0
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
# 아래 부분 추가
data:
  hsts: "false"
  ssl-redirect: "false"
```

> Ingress controller의 `ssl-redirect` 설정은 host 주소는 그대로지만 https의 default port인 443으로 redirect한다.
>
> 예시) http://jenkins.yourdomain.com:31623/jenkins → https://jenkins.yourdomain.com/jenkins

ingress-controller의 baremetal template에 있는 ConfigMap으로 띄워져 있으므로 해당 부분 수정

만약 다른 환경으로 인하여 ConfigMap이 없는 경우 새로 생성하여 적용할 것

```sh
$ kubectl apply -f deploy.yaml
```
설정 변경 적용

추가로 적용기간에는 http와 https의 포트를 서로 다르게하여 둘 다 접속 가능하도록 만드는 것이 옳다.

현재 http와 https를 동시 서비스 중이기 때문에 서로 다른 포트에서 접근을 한다.

따라서 수동으로 Host 주소와 port를 변경해주어야 한다.

#### 3. Ingress rule 변경

```yaml
$ vim jenkins-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  ...
  namespace: default
  # 아래 annotations 추가
  annotations:
  	kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/proxy-redirect-from: http://jenkins.yourdomain.com:31623
    nginx.ingress.kubernetes.io/proxy-redirect-to: https://jenkins.yourdomain.com:32452
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
```

```sh
$ curl -v http://jenkins.jenkins.yourdomain.com:32452
* About to connect() to jenkins.yourdomain.com port 32452 (#0)
*   Trying ...
* Connected to jenkins.yourdomain.com (...) port 32425 (#0)
> GET /jenkins HTTP/1.1
> User-Agent: curl/7.29.0
> Host: jenkins.yourdomain.com:32452
> Accept: */*
>
< HTTP/1.1 302 Found
< Date: Fri, 05 Mar 2021 04:51:59 GMT
< Content-Length: 0
< Connection: keep-alive
< Location: https://jenkins.yourdomain.com:32452/
```
테스트해보면 redirect가 정상적으로 되는 것을 확인할 수 있다.

### 크롬의 경우 HSTS 기능 끄기

위와 같이 설정해도 크롬과 같은 웹브라우저의 경우 HSTS 기능때문에 웹브라우저 레벨에서 

위에서 설정한 proxy가 아니라 호스트와 포트는 그대로인 상태로 http → https 리다이렉션이 강제된다.

> http 접속을 한 후에 서버에서 https 응답을 주는데 http 접속 이전에 브라우저에서 https로 변경하기 때문이다.
>

시크릿 모드로 들어가서 테스트하거나 이미 크롬에 설정된 HSTS 설정을 제거해야한다.

1. `chrome://net-internals/#hsts` 접속
2. Query HSTS/PKP domain에서 HSTS 설정되어 있는지 검색
![ki-4.png]({{ "/assets/img/contents/ki-4.png"}})
3. Delete domain security policies에서 해당 도메인 HSTS 설정 지우기

이후 크롬을 통해 재접속하면 https redirect가 성공적으로 되는 것을 확인할 수 있다.

---
## 참고사항
1. Ingress controller 기본 인증서를 가져간다?

```sh
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
```

위처럼 Ingress Controller에서 기본 설정 SSL Cert를 가져간다고 나오는데 Ingress Controller에서 SSL을 설정하는 것이 아닌 Ingress의 Secret 설정에 있는 인증서 파일을 적용하는 것이기 때문에 무시해도 된다.

---
## Troubleshooting

### Ingress controller 기본 인증서를 가져간다?

```sh
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
```

위처럼 Ingress Controller에서 기본 설정 SSL Cert를 가져간다고 나오는데 Ingress Controller에서 SSL을 설정하는 것이 아닌 Ingress의 Secret 설정에 있는 인증서 파일을 적용하는 것이기 때문에 무시해도 된다.

### Kubernetes Ingress Controller Fake Certificate 인증서가 적용될 때

```sh
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            0a:2a:7b:52:02:51:fe:7d:ff:ad:65:ea:41:8a:95:44
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: O = Acme Co, CN = Kubernetes Ingress Controller Fake Certificate
```

Ingress에 tls 설정을 추가할 경우 기본적으로 해당 서비스는 https를 사용하게 되며 인증서 설정을 별도로 해주지 않은 경우 Ingress Controller fake certificate를 기본적으로 사용하게 된다.

이 경우는 위에서 설정한 인증서 적용이 안 된 경우이다.

#### 1. kubernetes secret에서 cert와 key를 잘 물고 갔는지 확인

#### 2. Ingress yaml 파일에 tls 적용이 제대로 되었는지 확인

- ssl을 적용할 경우 연동되는 서비스와 tls 설정에 host 이름을 꼭 추가해주어야한다.


---

## Reference

1. [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
2. [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
3. [Secret](https://kubernetes.io/docs/concepts/configuration/secret/)
4. [Certified Kubernetes Administrator(CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)