---
layout: post
title: Kubernetes RBAC Authorization 개요 및 적용
feature-img: assets/img/titles/kubernetes-logo.png
thumbnail: assets/img/contents/authorization-authentication.jpeg
author: csupreme19
tags: [Kubernetes, Authorization, Authentication, RBAC, ABAC, Security]

---

# Kubernetes RBAC Authorization 개요 및 적용

![kubernetes-logo.png]({{ "/assets/img/titles/kubernetes-logo.png"}})

[Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

본 문서에서는 쿠버네티스 API 서버에 접근하기 위한 4가지 인증 방식을 살펴보고 그 중 RBAC 인증 적용방법에 대하여 정리하였다.

---

## 인가(Authentication) vs 인증(Authorization)

![authorization-authentication.jpeg]({{ "/assets/img/contents/authorization-authentication.jpeg"}})

**인가(Authentication)**

해당 사용자가 누구인지 확인하는 것(회원가입, 로그인)

**인증(Authorization)**

해당 사용자에 대한 권한을 허락하는 것(호, 자원 접근)

---

## Kubernetes 인증 방식

Kubernetes API 서버에 접근하기 위해서는 인증 단계가 필요하다.

쿠버네티스에서는 인증 방식이 크게 4가지가 존재한다.

1. Node Authorization
2. ABAC Authorization
3. RBAC Authorization
4. Webhook Authorization

API Server의 `--authorization-mode`  flag를 확인하여 현재 활성화된 인증 모드를 확인할 수 있다.



```sh
$ cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -i authorization
    - --authorization-mode=Node,RBAC
```



기본적으로 Node, RBAC 인증 방식이 활성화되어 있는 것을 확인할 수 있다.

---

### Node Authorization

<div class="mermaid">
flowchart LR
  A[User]
  B[Kube API]
  C["Kubelet<br>Node"]
  subgraph Kubernetes Cluster
    C--Node Authorization-->B
  end
    A-->B
</div>




Kubernetes Clsuter에 속하는 Node들은 Kubelet에서 API 서버에 요청할 때 TLS 인증을 이용한다.

이 때 Kubelet의 Group은 `system:node`에 속해 있으며 해당 그룹에 속해있는 인증 요청은 Node Authorizer에 의하여 인증된다.

이 방식은 보통 Kubernetes TLS 부트스트랩 과정에서 자동으로 설정되므로 더 자세한 내용은 [공식 문서](https://kubernetes.io/docs/reference/access-authn-authz/node/) 참고

---

### ABAC(Attribute Based Access Control)

JSON 형식의 Policy 정의를 사용하여 해당 사용자를 인증하는 방식

##### Examples

<div class="mermaid">
flowchart LR
  A[User: admin]
  B[Group: system:authenticated]
  C[Kube API]
  subgraph Kuberntes Cluster
  A--ABAC-->C
  B--ABAC-->C
  end
</div>


```json
{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"user":"admin",     "namespace": "*",              "resource": "*",         "apiGroup": "*"                   }}
{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"group":"system:authenticated",  "nonResourcePath": "*", "readonly": true}}
```

기본적으로 비활성화 되어 있으며 kube-apiserver에 `--authorization-mode=ABAC`, `--authorization-policy-file=파일명` 설정을 추가하여야 한다.

Poilicy 정의 후 API 서버를 재시작해야하고 접근권한을 파일로 정의하기 때문에 후술할 RBAC에 비하여 관리하기가 어렵다는 단점이 있다.

---

### RBAC(Role Based Access Control)

사용자, 서비스의 접근 권한(인증)을 Role(ClusterRole)과 RoleBinding(ClusterRoleBinding) 자원에 기반하여 처리하는 방식으로 일반적으로 가장 많이 사용하고 관리하기 쉬운 인증 방식이다.



해당 User, SA(Service Account), Group등이 RoleBinding에 의하여 어떤 접근권한을 가지고 있는지 인증된다.



<div class="mermaid">
flowchart LR
  A[User]
  B[Group]
    subgraph Kubernetes Cluster
    C[Service Account]
  D[Role: Developer]
  E[Role: Production]
  end
  A--RBAC-->D & E
  B--RBAC-->D
  C--RBAC-->E
</div>

---
### Webhook Authorization

Kubernetes 내부에서 제공하는 인증이 아닌 외부의 인증 정책을 사용하기 위한 방식

Open Policy Agent와 같은 외부 오픈소스, Admission Controller를 사용할 때 사용한다.

<div class="mermaid">
flowchart LR
  A[User]
    subgraph Kubernetes Cluster
  B[Kube API]
    end
  C[Open Policy Agent]
  A-->B
  B--Authorization-->C
  C-.Authorization.->B
</div>

---

## RBAC 리소스

### 1. Role / ClusterRole

<div class="mermaid">
flowchart LR
  A[Role<br>Can view Pods<br>Can watch Pods<br>Can list Pods]
  A
</div>

어떤 리소스에 어떤 호출이 가능한지 권한/역할을 정의한 리소스이다.

Role과 ClusterRole이 있으며 ClusterRole은 Role과 달리 클러스터 레벨로 가지고 있어 네임스페이스가 존재하지 않는다.

#### Role 예제

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

Resources에 해당하는 자원의 verb에 해당하는 요청이 가능하다.

pod-reader를 예로 들면 pods를 get, watch, list 요청이 가능하다.



kubernetes의 자원 종류는 아래 명령어로 확인 가능하다.

```sh
$ kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumeclaims            pvc          v1                                     true         PersistentVolumeClaim
persistentvolumes                 pv           v1                                     false        PersistentVolume
...
```



#### Role 확인

```sh
$ kubectl get role -n kube-system
$ kubectl get clusterrole -n
$ kubectl describe role/kube-proxy -n kube-system
$ kubectl describe clusterrole/cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]
```



### 2. ServiceAccount

파드로 올라가있는 서비스에서 kubernetes 클러스터에 접근하기 위한 계정을 나타내는 리소스다. 



![kra-1.png]({{ "/assets/img/contents/kra-1.png" }})

> 이미지: [Certified Kubernetes Administrator(CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/) 발췌

쉽게 말해 사용자 계정이 아닌 서비스의 계정이다.

후술할 추상적인 User, Group과 달리 Kubernetes 자원 형태로 실존한다.

<div class="mermaid">
flowchart LR
  A[Service Account]
  B[Secret]
  A-.token.-B
</div>

#### SA 예제

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: monitoring-user
  namespace: kubernetes-dashboard
---
```
#### SA 확인

```sh
$ kubectl get sa -n kube-system
```

service account 생성시 해당 서비스 어카운트의 토큰을 가지고 있는 secret이 자동 생성된다.

#### 서비스 어카운트 사용방법

```sh
$ kubectl describe sa/monitoring-user -n kubernetes-dashboard
Name:                monitoring-user
Namespace:           kubernetes-dashboard
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   monitoring-user-token-r6nss
Tokens:              monitoring-user-token-r6nss
Events:              <none>
```

service account 생성시 해당 서비스 어카운트의 토큰을 가지고 있는 secret이 자동 생성된다.

#### 토큰 확인

```sh
$ kubectl describe secret monitoring-user-token-r6nss -n kubernetes-dashboard
Name:         monitoring-user-token-r6nss
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: monitoring-user
              kubernetes.io/service-account.uid: cca1bad1-fe4f-430c-978b-52d7f7ea53c5

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlVmM2...

# 참고) 한번에 토큰 조회
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/monitoring-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
eyJhbGciOiJSUzI1NiIsImtpZCI6IlVmM2...
```

API 호출시 Authorization 헤더에 해당 토큰을 넣어서 요청하면 인증을 할 수 있다.

#### 자원에 적용

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
...
spec:
  ...
  template:
    ...
    spec:
      containers:
        - name: dashboard-metrics-scraper
          image: kubernetesui/metrics-scraper:v1.0.6
          ...
      serviceAccountName: kubernetes-dashboard	# 해당 부분 추가
      ...
```

Pod 명세 `serviceAccountName` 에 위에서 생성한 SA 추가하면 해당 Pod는 API Server에 정의된 권한을 가지고 접근할 수 있다.

#### 요청 예시

```sh
# 직접 호출시 토큰 명시
$ curl -H "Authorization: Bearer eyJhbGciOiJSUzI1..." -ivk https://10.213.196.211:6443 
```


### 3. RoleBinding

위에서 생성한 ServiceAccount가 실질적으로 권한을 가지려면 해당 어카운트가 어느 자원에 접근할 수 있는지 허락하는 단계가 필요하다.

이것을 인증(Authorization)이라고 한다.

kubernetes에서는 RBAC, 즉 Role 역할을 기반으로하여 인증을 하기 때문에 Role을 ServiceAccount / User / Group에 바인딩하는 방법을 사용하며 이를 나타내기 위한 리소스가 RoleBinding이다.

RoleBinding과 ClusterRoleBinding이 있으며 ClusterRole의 경우 ClusterRoleBinding을 이용한다.

각각의 Role은 ServiceAccount, User, Group에 바인딩 될 수 있다.

#### **User / Group**

ServiceAccount와 달리 Kubernetes API server의 User와 Group은 별도로 정의된 Kubernetes Resource가 아니다.

API server에 접근하기 위한 아래와 같은 보안 설정 파일에 정의되어 있는 User와 Group이라는 추상적인 개념이다.

- Static Password file
- Static Token file
- Certificates

보통은 인증서 설정에 User와 Group 정보가 정의되어 있으며 `system:`  으로 시작하는 그룹은 미리 정의된 그룹이다.

User 정보는 kubeconfig에서 확인할 수 있다. 

```sh
$ kubectl config view
```

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.213.196.211:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```



<div class="mermaid">
flowchart LR
  A[User]
  B[Group]
  C[Service Account]
  D[Role]
  E[Secret]
  subgraph RoleBinding
  A & B & C---D
  end
  C -.token.- E
</div>

#### RoleBinding 예제

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
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
---
```
> ClusterRoleBinding 역시 클러스터 레벨이므로 네임스페이스가 따로 없음

#### RoleBinding 확인

```sh
$ kubectl get rolebinding
$ kubectl get clusterrolebinding
$ kubectl get pods --as jane	# User 권한 확인
```

---
### RBAC 사용 예시

>  시나리오: k8s dashboard에 접근하기 위한 모니터링 인증을 생성한다.



#### 1. `kubernetes-dashboard-rbac.yaml` 작성

ServiceAccount, ClusterRole, ClusterRoleBinding을 설정한다.
```yaml
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
  resources: ["namespaces", "deployments", "replicasets", "pods", "pods/log"]
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

Cluster의 자원이 아닌 헬스체크등 URI로 API를 호출해야하는 경우 아래와 같이 `resources` 대신 `nonResourceURLs`를 사용한다.

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-monitoring
rules:
- nonResourceURLs: ["/healthz*", "/livez*", "/readyz*", "/version*"]
  verbs: ["get"]
```

해당 ClusterRole을 가진 SA의 시크릿 토큰을 이용하여 위 URL에 해당하는 API를 호출할 수 있다.

#### 2. Token 확인

```sh
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/monitoring-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```



#### 3. Kubernetes 클러스터 접근 테스트

```sh
$ curl -H "Authorization: Bearer eyJhbGciOiJSUzI1..." -k https://10.213.196.211:6443/livez\?verbose
[+]ping ok
[+]log ok
[+]etcd ok
[+]poststarthook/start-kube-apiserver-admission-initializer ok
[+]poststarthook/generic-apiserver-start-informers ok
[+]poststarthook/priority-and-fairness-config-consumer ok
[+]poststarthook/priority-and-fairness-filter ok
[+]poststarthook/start-apiextensions-informers ok
[+]poststarthook/start-apiextensions-controllers ok
[+]poststarthook/crd-informer-synced ok
[+]poststarthook/bootstrap-controller ok
[+]poststarthook/rbac/bootstrap-roles ok
[+]poststarthook/scheduling/bootstrap-system-priority-classes ok
[+]poststarthook/priority-and-fairness-config-producer ok
[+]poststarthook/start-cluster-authentication-info-controller ok
[+]poststarthook/aggregator-reload-proxy-client-cert ok
[+]poststarthook/start-kube-aggregator-informers ok
[+]poststarthook/apiservice-registration-controller ok
[+]poststarthook/apiservice-status-available-controller ok
[+]poststarthook/kube-apiserver-autoregistration ok
[+]autoregister-completion ok
[+]poststarthook/apiservice-openapi-controller ok
livez check passed

# 참고) 실패시
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
  },
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}
```

---

## 요약

Kubernetes API 서버에 접근하기 위한 인증방식은 크게 4가지가 있다.

- Node
- ABAC
- RBAC
- Webhook

일반적으로 사용하는 인증 방식으로는 RBAC가 있으며 Role에 기반하여 인증하는 방식이다.

전체적인 구조를 보자면 아래와 같다. (ClusterRole도 동일)

<div class="mermaid">
flowchart LR
  A[User]
  B[Group]
  C[Service Account]
  D[Role]
  subgraph Kubernetes Cluster
  E[Secret]
  J[Pod]
  F[Kube API]
  RoleBinding
  end
  subgraph RoleBinding
  A & B & C---D
  end
  C -.token.- E
  C --> J
  J --Request--> F
  F -.Response.-> J
</div>



---

## Reference

1. [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
2. [Using Node Authorization](https://kubernetes.io/docs/reference/access-authn-authz/node/)
3. [Using ABAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/node/)
4. [Certified Kubernetes Administrator(CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)