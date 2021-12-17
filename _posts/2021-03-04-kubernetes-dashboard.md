---
layout: post
title: Kubernetes Dashboard 설치 및 외부접속 설정
feature-img: assets/img/titles/kubernetes-logo.png
thumbnail: assets/img/contents/kd-1.png
author: csupreme19
categories: DevOps Kubernetes
tags: [Kubernetes, K8S, Dashboard, Ingress, RBAC, Security]

---

# Kubernetes Dashboard 설치 및 외부접속 설정

![kubernetes-logo.png]({{ "/assets/img/titles/kubernetes-logo.png"}})

[kubernetes dashboard](https://github.com/kubernetes/dashboard)

Kubernetes dashboard를 구성 후 외부접속 설정

---
## Kubernetes dashboard 설치

### Using K8S Deployment

#### 1. kubernetes dashboard template 가져오기

```sh
$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
$ mv recommended.yaml kubernetes-dashboard.yaml # 이름 변경(편의상)
```

#### 2. ServiceAccount 생성 및 RBAC 설정

```sh
$ vim kubernetes-dashboard-rbac.yaml
```

```yaml

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

위 설정은 현재 k8s에 미리 정의되어 있는 cluster-admin 역할을 바인딩하므로 해당 SA의 토큰으로 접속시 cluster의 모든 권한을 획득한다.

모니터링 SA의 경우 아래와 같이 신규 Role을 생성하여 권한을 지정하여 바인딩한다.

```sh
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: monitoring-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-monitoring
rules:
- apiGroups: ["*"]
  resources: ["namespaces", "deployments", "pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-monitoring
subjects:
- kind: ServiceAccount
  name: monitoring-user
  namespace: kubernetes-dashboard
```

> 네임스페이스, 디플로이먼트, 파드 3가지 리소스에 대해서 읽기 권한만 부여

#### 3. 로컬 접속 테스트

```sh
# 로컬 k8s 프록시 실행
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
# 접속 확인
$ curl -v http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

> 참고: `kubernetes-dashboard.yaml`의 모든 리소스는 모두 kubernetes-dashboard라는 namespace안에 존재한다.

#### 4. 베어러 토큰 발급

```sh
# 토큰 발급
{% raw %}
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/{위에서 생성한 SA명} -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}" >> token
{% endraw %}

# 토큰 확인
$ cat token
```

Kubernetes dashboard는 기본적으로 API형태로의 로컬 접근만 허용하고 있어 외부 접속 설정이 따로 필요하다.

---
## Kubernetes Dashboard 외부 접속 설정

>  참고: Kubernetes Dashboard는 기본적으로 https 연결을 하도록 설정되어 있으며 로컬에서의 접근만 허용한다

Kubernetes를 외부에 노출시키는 방법은 크게 4가지가 있다.

#### 1. Proxy 방식

#### 2. NodePort 방식

#### 3. API Server 방식

#### 4. Ingress 방식

본 문서에서는 4번 Ingress 방식을 사용한다.

### 1. Proxy 방식

> kubectl proxy를 이용하여 포트와 주소를 지정한 후 Host 머신에 띄우는 방식

```sh
$ kubectl proxy --port=9090 --address=10.109.190.106 --accept-hosts='^*$'

$ curl -k -v http://10.109.190.106:9090/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

Host가 외부에 노출되어 있는 상황이라면 해당 IP를 이용하여 외부에서도 접근이 가능하다.

#### 단점

- Host가 외부와 직접 연결되는 환경에서만 사용할 수 있다.
- 대시보드가 localhost에서 띄워져야만 로그인 기능을 사용할 수 있다.
- 외부에서 무차별 접속이 가능하며 모든 권한을 갖는다.
- 방화벽 등 보안 설정이 추가로 필요하다.

### 2. NodePort 방식

> NodePort 방식의 서비스를 사용하여 포트를 직접 외부에 노출시키는 방식

```sh
$ vim kubernetes-dashboard.yaml
```

```yaml

---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 9090
      targetPort: 9090
  selector:
    k8s-app: kubernetes-dashboard

---
```

Service type을 NodePort로 변경

```sh
$ kubectl get svc -n kubernetes-dashboard
kubernetes-dashboard	NodePort	10.109.190.106	<none>	443:31384/TCP	28s

# master ip 확인
$ kubectl cluster-info
```

NodePort 설정 확인

31384 포트로 외부에 오픈되어 있는 것을 확인할 수 있다.

`https://master-ip:31384`로 접속 확인

#### 단점

- Host가 외부와 직접 연결되는 환경에서만 사용할 수 있다.
- 대시보드는 https 연결을 기본으로 하므로 인증서가 없어 접속 불가

### 3. API Server 방식

> 위의 Proxy 방식과 비슷하나 API 서버가 외부에 노출되어 있어 API 서버에 직접 접속하는 방식

```sh
$ curl -k -v https://<master-ip>:<apiserver-port>/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

이 경우 kubernetes dashboard 인증서를 브라우저 자체에 따로 추가해주어야 하는 추가 작업이 존재한다. (본 문서에서는 다루지 않는다.)

게다가 kube-apiserver를 외부에 직접 노출하는것은 권장하지 않는다.

#### 단점

- Host가 외부와 직접 연결되는 환경에서만 사용할 수 있다.
- Kubernetes 관리자가 아닌 사용자가 직접 브라우저에 인증서를 설치하는 과정이 필요하다.
- Kubernetes API Server를 외부와 직접 노출하는 것은 바람직하지 않다.

### 4. Ingress 방식

> Kubernetes service를 외부와 연결시켜주는 Ingress를 사용하는 방식

#### 선행사항

- Ingress Controller 설치
- 도메인 연결 및 SSL 인증서 발급

kubernetes-dashboard는 https 요청을 강제하지만 외부에서 Ingress Controller로 들어오는 부분이 

TLS 인증 처리가 되어 있으므로 kubernetes 내부에서 동작하는 dashboard 서비스 및 pod는 http를 사용 가능하다.

#### 1. `kubernetes-dashboard.yaml` 수정

```sh
$ vim kubernetes-dashboard.yaml
```

```yaml

...
---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 9090			# 포트 수정
      targetPort: 9090		# 포트 수정
  selector:
    k8s-app: kubernetes-dashboard
---
...
---

kind: Deployment
apiVersion: apps/v1
metadata:
  ...
  spec:
      containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.2.0
          imagePullPolicy: Always
          ports:
            - containerPort: 9090 # 포트 수정
              protocol: TCP
          args:
            #- --auto-generate-certificates 주석 처리
            - --namespace=kubernetes-dashboard
            - --insecure-bind-address=0.0.0.0 # 추가
            - --enable-insecure-login # 추가
            - --token-ttl=10800	# 토큰 세션 유지시간(선택)
            # Uncomment the following line to manually specify Kubernetes API server Host
            # If not specified, Dashboard will attempt to auto discover the API server and connect
            # to it. Uncomment only if the default does not work.
            # - --apiserver-host=http://my-address:port
            ...
          livenessProbe:
            httpGet:
              scheme: HTTP # HTTP로 변경
              path: /
              port: 9090 # 포트 수정
            initialDelaySeconds: 30
            timeoutSeconds: 30
...

```

```sh
$ kubectl apply -f kubernetes-dashboard.yaml
```

##### 참고

- `--namespace=kubernetes-dashboard`: namespace 설정
- `--insecure-bind-address=0.0.0.0`: http 모든 IP 접속 허용
- `--enbale-insecure-login`: http 접속 허용

#### 2. `kubernetes-dashboard-ingress.yaml` 생성

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/ingress.class: "nginx" # kong ingress의 경우 kong
spec:
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 9090

```

```sh
$ kubectl apply -f kubernetes-dashboard-ingress.yaml
```



#### 3. 접속 확인

```sh
# Ingress Controller 외부 노출 포트 확인
$ kubectl get svc -n ingress-ngninx
ingress-nginx-controller             NodePort    10.104.33.237   <none>        80:31623/TCP,443:32452/TCP   5h
```

`http://external-ip:31623/` 접속
(방화벽 포트포워딩 된 경우 해당 포트로 접속)

![kd-1.png]({{ "/assets/img/contents/kd-1.png"}})

http를 이용하여 접속할시 로그인이 비활성화 된다.

`https://external-ip:32452/` 접속
(방화벽 포트포워딩 된 경우 해당 포트로 접속)

![kd-2.png]({{ "/assets/img/contents/kd-2.png"}})

![kd-3.png]({{ "/assets/img/contents/kd-3.png"}})

https를 이용하여 접속시 로그인은 활성화 되지만 인증서 정보가 유효하지 않다고 나온다.

이제 SSL 인증서를 서버 Ingress에 적용할 차례이다.

#### 4. Kubernetes secret 생성

```sh
# kubectl create secret {secretType} {secretName} --namespace={namespace} --cert={certPath} --key={keyPath}
$ kubectl create secret tls secret-tls --namespace=kubernetes-dashboard --cert=ssl/yourdomain.com --key=ssl/yourdomain.com

# 생성 확인
$ kubectl get secret -n kubernetes-dashboard
$ kubectl describe secret secret-tls -n kubernetes-dashboard
```

ssl 인증서 정보를 담고 있는 kubernetes secret을 생성한다.

이 때, namespace는 Pod의 namespace와 동일해야하므로 설정해준다.

> 인증서 경로는 ssl 하위에 있다고 가정

#### 5. `kubernetes-dashboard-ingress.yaml` 수정

```sh
$ vim kubernetes-dashboard-ingress.yaml
```

```yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  # 아래 tls 부분 추가
  tls:
  - hosts:
    - k8s.yourdomain.com
    secretName: secret-tls
  rules:
  - host: k8s.yourdomain.com # host 도메인 추가
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 9090
              
```

```sh
$ kubectl apply -f kubernetes-dashboard-ingress.yaml
```



#### 6. HTTPS 접속 및 인증서 확인

```sh
# Ingress Controller 외부 노출 포트 확인
$ kubectl get svc -n ingress-ngninx
ingress-nginx-controller             NodePort    10.104.33.237   <none>        80:31623/TCP,443:32452/TCP   5h
```

`https://external-ip:32452/` 접속
(방화벽 포트포워딩 된 경우 해당 포트로 접속)

![kd-4.png]({{ "/assets/img/contents/kd-4.png"}})

![kd-5.png]({{ "/assets/img/contents/kd-5.png"}})

인증서 적용 확인

#### 7. 베어러 토큰 발급

```sh
# 토큰 발급
{% raw %}
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}" >> token
{% endraw %}

# 토큰 확인
$ cat token
```

![kd-3.png]({{ "/assets/img/contents/kd-3.png"}})

로그인 화면에서 Token 입력 후 로그인

![kd-6.png]({{ "/assets/img/contents/kd-6.png"}})

로그인 확인

---

## Metric-server 설치
각 서버에서 cpu, memory metric을 가져와 dashboard에서 그래프로 보여주려면 설치해야한다.
```sh
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

metrics-server error because it doesn't contain any IP SANs 에러 발생할 경우

deployment에 아래를 추가한다.

```yaml
 - args:
        - --kubelet-insecure-tls
```


---
## Troubleshooting

### 1. 404 page not found
![kd-7.png]({{ "/assets/img/contents/kd-7.png"}})

위와 같이 404 page not found가 나오는 경우

Ingress에서 서비스를 찾지 못한 경우로 경로 설정을 잘 확인해본다.

나는 ingress Path 설정을 `"/"`가 아니라 `"/dashboard"`로 해놓았었는데 root path가 아니면 적용이 되지 않는 버그가 존재하였다.

### 2. kubernetes-dashboard pod가 자꾸 죽어서 재시작되는 경우

위에서 대시보드가 http 프로토콜로 작동하도록 설정한 부분에 누락되거나 잘못 설정한 값이 있는지 확인

나는 deployment쪽의 Liveness probe 설정에 http가 아닌 https로 되어 있어서 문제가 발생하였다.


---

## Reference

1. [kubernetes dashboard](https://github.com/kubernetes/dashboard)