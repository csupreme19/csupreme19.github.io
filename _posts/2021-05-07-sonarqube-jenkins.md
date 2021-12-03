---
layout: post
title: SonarQube 설치 및 Jenkins pipeline 연동하기
feature-img: assets/img/titles/sonarqube.png
thumbnail: assets/img/contents/sj-1.png
author: csupreme19
tags: [Sonarqube, Jenkins, Helm, Docker, Docker-Compose]

---

# SonarQube 설치 및 Jenkins pipeline 연동하기

![sonarqube.png]({{ "/assets/img/titles/sonarqube.png"}})

[SonarQube Documentation](https://docs.sonarqube.org/latest/)

본 문서에서는 Sonarqube 설치 및 Jenkins 파이프라인 연동한 내용을 정리하였다.

---
## SonarQube란?

![sj-1.png]({{ "/assets/img/contents/sj-1.png"}})

소스 품질 관리를 위한 자동화된 정적 코드 검증/분석 툴
- 버그 탐색
- 취약점 탐색
- 코드 냄새(Code Smell) 탐색
- 보안 핫스팟 탐색

**안전(Safe)** 하고 **깨끗한(Clean)** 코드를 유지하게끔 도움을 줌(소스코드 품질 관리)

1. 커밋/머지(SCM)
2. 체크아웃, 빌드, 테스트(CI/CD)
3. 분석/검증(SonarQube)

> Sonarlint: 코딩/컴파일시 소스 품질 관리를 위한 IDE 플러그인

---
## SonarQube 설치

두 가지 방법으로 설치해 보았다.

- Using Helm Chart
    - k8s 위에서 관리하는 자원의 형태로 사용시
- Using Docker Compose
    - 호스트 위에서 컨테이너형으로 간단하게 사용시

### Using Helm Chart

> k8s helm을 이용하여 설치하는 방법

#### 1. Helm 설치

```sh
# Ubuntu
$ curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
$ sudo apt-get install apt-transport-https --yes
$ echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
$ sudo apt-get update
$ sudo apt-get install helm
```



#### 2. Helm Chart values.yaml 가져오기

   ```shell
   # helm 저장소 추가
   $ helm repo add oteemocharts https://oteemo.github.io/charts
   $ wget https://raw.githubusercontent.com/Oteemo/charts/master/charts/sonarqube/values.yaml
   ```

> 현시점 기준 8.5.1-community

#### 3. ingress 리소스 설정

   ```shell
   $ vim sonarqube-ingress.yaml
   ```
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
    metadata:
      name: sonarqube
      namespace: default
      annotations:
        kubernetes.io/ingress.class: "nginx"
        nginx.ingress.kubernetes.io/ssl-redirect: "false"
        nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
    spec:
      rules:
      - http:
          paths:
          - pathType: Prefix
            path: "/sonarqube"
            backend:
              service:
                name: sonarqube-sonarqube
                port:
                  number: 9000
   ```
   ```shell
   $ kubectl apply -f sonarqube-ingress.yaml
   ```
> /sonarqube context로 설정
>
> > Ingress HTTP(/sonarqube) -> sonarqube 서비스(9000)

#### 4. PersistenceVolume, PersistenceVolumeClaim 설정

   ```shell
   $ vim sonarqube-storage.yaml
   ```
   ```yaml
    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: sonarqube-pv-volume
    spec:
      storageClassName: manual
      capacity:
        storage: 25Gi
      accessModes:
        - ReadWriteMany
      hostPath:
        path: "/nfs_nas/volumes/sonarqube/data"
    ---
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: sonarqube-pv-claim
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 25Gi
    ---
    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: sonarqube-db-pv-volume
    spec:
      storageClassName: manual
      capacity:
        storage: 25Gi
      accessModes:
        - ReadWriteMany
      hostPath:
        path: "/nfs_nas/volumes/sonarqube/postgres"
    ---
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: sonarqube-db-pv-claim
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 25G
   ```
   ```shell
   $ kubectl apply -f sonarqube-storage.yaml
   ```

sonarqube에서 사용할 volume을 지정해주기 위하여 설정

용량과 hostPath는 사용 환경에 맞추어 변경할 것

#### 5. values.yaml 수정

아래 해당하는 부분 모두 수정

   ```yaml
    ...
    readinessProbe:
      initialDelaySeconds: 60
      periodSeconds: 30
      failureThreshold: 6
      sonarWebContext: /sonarqube/ # 변경(끝에 '/' 포함)
    livenessProbe:
      initialDelaySeconds: 60
      periodSeconds: 30
      sonarWebContext: /sonarqube/ # 변경(끝에 '/' 포함)
    ...
    # 사용할 플러그인 추가(아래 기본 플러그인은 이미 있으므로 충돌남 외부 플러그인만 적용할 것, 아래는 잘못된 예시)
    plugins:
      install: [
        "https://binaries.sonarsource.com/Distribution/sonar-java-plugin/sonar-java-plugin-6.9.0.23563.jar",
        "https://binaries.sonarsource.com/Distribution/sonar-javascript-plugin/sonar-javascript-plugin-7.4.3.15529.jar",
        "https://binaries.sonarsource.com/Distribution/sonar-xml-plugin/sonar-xml-plugin-2.2.0.2973.jar",
        "https://binaries.sonarsource.com/Distribution/sonar-css-plugin/sonar-css-plugin-1.4.2.2002.jar",
        "https://binaries.sonarsource.com/Distribution/sonar-html-plugin/sonar-html-plugin-3.4.0.2754.jar",
        "https://binaries.sonarsource.com/Distribution/sonar-typescript-plugin/sonar-typescript-plugin-2.1.0.4359.jar"
        ]
      lib: []
    ...
    env:
      - name: SONAR_WEB_CONTEXT # 추가
        value: /sonarqube
    ...
    persistence:
      enabled: true
      annotations: {}
      existingClaim: sonarqube-pv-claim # 추가
    ...
    postgresql:
      enabled: true
      ...
      persistence:
        enabled: true
        existingClaim: sonarqube-db-pv-claim # 추가
    ...
   ```

#### 6. helm install
   ```shell
   # helm v3
   # helm install [name] [chart] [-f:value]
   $ helm install sonarqube oteemocharts/sonarqube -f values.yaml
   
   # helm v2
   # helm install [--name:name] [chart] [-f:values]
   $ helm install --name sonarqube oteemocharts/sonarqube -f values.yaml
   ```



**참고**

**helm uninstall**

   ```shell
   # helm v3
   # helm uninstall [name]
   $ helm uninstall sonarqube
   
   # helm v2
   # helm del [--purge] [name]
   $ helm del --purge sonarqube
   ```

**helm upgrade**

   ```shell
   # helm upgrade [release] [chart] [-f:values]
   $ helm upgrade sonarqube oteemocharts/sonarqube -f values.yaml
   ```

#### 7. kubernetes pods 확인 및 대시보드 접속
   ```shell
   $ kubectl get pods -A | grep sonarqube
   $ kubectl logs [pod name]
   $ curl localhost:9000
   ```
> `http://external-ip:port/sonarqube` 접속 확인<br>초기 계정: admin/admin
>
> 접속 후 admin 비밀번호 변경 또는 비활성화 할 것

### Using Docker-Compose

#### 1. docker-compose.yaml 작성
   ```yaml
    version: "3.1"
    services:
      sonarqube:
        image: sonarqube:latest
        restart: always
        depends_on:
          - sonarqube-db
        container_name: sonarqube
        ports:
          - "9000:9000"
        networks:
          - sonarnet
        environment:
          TZ: Asia/Seoul
          SONAR_HOME: /opt/sonarqube
          SONAR_JDBC_USERNAME: sonar
          SONAR_JDBC_PASSWORD: sonar
          SONAR_JDBC_URL: jdbc:postgresql://sonarqube-db:5432/sonar
        ulimits:
          nofile:
            soft: 65536
            hard: 65536
          memlock:
            soft: -1
            hard: -1
        volumes:
          - /nfs_nas/volumes/sonarqube/data:/opt/sonarqube/data
          - /nfs_nas/volumes/sonarqube/extensions:/opt/sonarqube/extensions
          - /nfs_nas/volumes/sonarqube/logs:/opt/sonarqube/logs
          - /nfs_nas/volumes/sonarqube/temp:/opt/sonarqube/temp

      sonarqube-db:
        image: postgres
        container_name: sonarqube-db
        networks:
          - sonarnet
        environment:
          TZ: Asia/Seoul
          POSTGRES_USER: sonar
          POSTGRES_PASSWORD: sonar
        volumes:
          - /nfs_nas/volumes/sonarqube/postgres:/var/lib/postgresqli/data
   ```

#### 2. docker-compose 실행 및 접속 확인
   ```shell
   $ docker-compose up -d
   $ docker-compose ps
   $ curl localhost:9000
   ```

   > 외부 접속의 경우 포트포워딩 필요
   >
   > > 이 부분은 각 인프라의 환경별로 상이하기 때문에 별도로 기술하지는 않는다.

---

## Jenkins 연동

### 1. SonarQube 사이드

1. SonarQube Web 접속
2. Administration > Security > Users
3. (선택) admin 계정 deactivate 및 새 관리자 계정 설정
    1. admin 계정의 설정 > Deactivate
    2. Create User
    3. 생성된 계정의 Groups 리스트 > Unselected > sonar-administrators 체크
4. 위의 관리자 계정에서 Token 메뉴 진입
5. Generate Tokens > Token Name(`jenkins-token`) 입력 후 Generate
> SonarQube 입장에서는 Jenkins에서 사용할 토큰이므로 `jenkins-token`이라고 이름 지음


![sj-2.png]({{ "/assets/img/contents/sj-2.png"}})

6. 생성된 토큰 복사
> 토큰값은 생성시에만 볼 수 있으므로 필히 복사 및 다른 곳에 저장할 것


### 2. Jenkins 사이드

1. SonarQube 플러그인 설치
    1. Jenkins 관리 > 플러그인 관리
    2. `SonarQube` 검색 > `SonarQube Scanner for Jenkins` 설치
2. SonarQube 토큰 등록
    1. Jenkins 관리 > Manage Credentials
    2. 아래 Stores scoped to Jenkins의 Domains 항목 클릭
    3. Add Credentials
        - Kind: Secret text
        - Scope: Global
        - Secret: 위 SonarQube에서 생성했던 토큰 붙여넣기
        - ID: 원하는 ID(`sonarqube-token`) 입력
3. SonarQube 서버 연동
    1. Jenkins 관리 > 시스템 설정 > SonarQube servers
    2. Environment variables 체크
    > SonarQube servers 항목에 입력하는 config 정보들을 Jenkins에서 환경변수로 사용할 것인지 확인하는 항목
    
    
    
    ![sj-3.png]({{ "/assets/img/contents/sj-3.png"}})

3. SonarQube Installations 항목 입력
    - Name: 원하는 서버 이름 입력(ex: sonarqube server)
    - Server URL: SonarQube 서버 주소 입력
    - Server authentication token: 2번 항목에서 생성한 토큰 선택

### 3. Jenkins Pipeline

테스트용 메이븐 프로젝트 기준으로 작성

Gradle 프로젝트 또는 다른 설정을 보려면 [SonarQube for Gradle](#bkmrk-sonarqube-for-gradle)을 참고

SonarQube와 연동할 파이프라인 스크립트에 아래와 같이 Stage 추가

```java
    stage("SonarQube analysis") {
        withSonarQubeEnv('sonarqube server') {
            sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.0.2155:sonar'
        }
    }

    stage("Quality Gate"){
        timeout(time: 1, unit: 'HOURS') {
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
        }
    }
```

**Stage 설명**
- `"SonarQube analysis"`: mvn 빌드 파일 기준 SonarQube 검사 진행

- `"Quality Gate"`: 검사한 코드의 품질을 통해 통과/미통과 판별(Quality Gate 프로파일 설정은 SonarQube에서 진행)

> `withSonarEnv()`에는 위의 설정에서 Jenkins에서 설정한 SonarQube Server 이름으로 설정해야함<br><br>Qulaity Gate의 경우 SonarQube에서 Jenkins로 Webhook이 설정되어 있어야함 ([Webhook 설정](/2021/05/10/sonarqube-config.html) 참고)



![sj-4.png]({{ "/assets/img/contents/sj-4.png"}})

Jenkins 빌드시 각 Stage 정상 통과 및 SonarQube Webhook 동작 여부 확인

> 연동 성공시 Build History에 SonarQube 대시보드로 이동할 수 있는 아이콘이 나옴


---

## Reference

1. [SonarScanner for Jenkins](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/)
2. [SonarQube Scanner for Jenkins](https://www.jenkins.io/doc/pipeline/steps/sonar/#sonarqube-scanner-for-jenkins)