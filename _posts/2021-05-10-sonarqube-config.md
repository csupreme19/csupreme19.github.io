---
layout: post
title: SonarQube 설정 및 트러블슈팅
feature-img: assets/img/titles/sonarqube.png
thumbnail: assets/img/contents/sc-1.png
author: csupreme19
tags: [Sonarqube, Jenkins, Webhook, Ingress, Kubernetes, GitLab]

---

# SonarQube 설정 및 트러블슈팅

![sonarqube.png]({{ "/assets/img/titles/sonarqube.png"}})

[SonarQube Documentation](https://docs.sonarqube.org/latest/)

본 문서에서는 Sonarqube Jenkins 연동, GitLab 연동, Ingress 인증서 설정, 웹훅 설정 등 내용을 모아 정리하였으며

설정시 오류를 해결했던 내역을(트러블슈팅) 정리하였다.

---

## SonarQube 설정

### SonarQube Ingress SSL 인증서 적용

#### 1. k8s Secret 생성

> k8s secret 생성은 [Kubernetes Nginx Ingress 적용](/2021/04/03/kubernetes-ingress.html) 참고

#### 2. Ingress 리소스 설정

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
     nginx.ingress.kubernetes.io/proxy-body-size: "20M"		// 20M 초과시 HTTP 413 방지하기 위하여
 spec:
   rules:
   - host: sonarqube.yourdomain.com # 추가
     http:	# -(하이픈) 제거
       paths:
       - pathType: Prefix
         path: "/sonarqube"
         backend:
           service:
             name: sonarqube-sonarqube
             port:
               number: 9000
   tls:	# tls 이하 부분 모두 추가
   - hosts:
     - sonarqube.yourdomain.com # 등록한 도메인명 입력
     secretName: secret-tls # k8s secret 리소스명 입력
```
```shell
$ kubectl apply -f sonarqube-ingress.yaml
```

> Nginx Ingress Controller 사용하였다.
> Ingress HTTP, HTTPS(/sonarqube) -> sonarqube 서비스(9000)
>
> > tls 설정에 도메인 입력시 IP 주소로는 Ingress 동작하지 않음

#### 3. 인증서 적용 확인

![sc-1.png]({{ "/assets/img/contents/sc-1.png"}})

도메인 접속 및 확인

> Jenkins의 SonarQube Server 설정 값에 기존의 IP주소로 되어있는 경우 도메인 주소로 변경 필요(http)

---
### Jenkins Webhook 설정

SonarQube가 검사를 끝낸 뒤 성공/실패 여부를 전달하기 위하여 SonarQube에 Jenkins로의 Webhook이 설정되어 있어야함

![sc-2.png]({{ "/assets/img/contents/sc-2.png"}})

#### 1. SonarQube 대시보드 로그인

#### 2. Administration > Configuration > Webhooks

#### 3. Create

- Name: 원하는 이름(`jenkins-webhook`)
- URL: jenkins 주소 (http 사용할 것)
- Secret: (선택)

#### 4. 빌드시 웹훅 동작 여부를 이 페이지에서 실시간으로 확인 가능

---
### Jenkins 파이프라인 - JavaScript, TypeScript 연동

JavaScript는 기본적으로 브라우저 위에서 실행되는 언어이기 때문에 이를 실행할 수 있는 프레임워크인 NodeJS가 젠킨스에 필요하다.

#### 1. NodeJS 플러그인 설치

1. Jenkins 관리 > 플러그인 관리 > 설치가능 > nodejs 검색
2. 재시작 없이 설치하기(업데이트, 디펜던시 추가 등으로 재시작 필요시 재시작 후 설치하기)

![sc-3.png]({{ "/assets/img/contents/sc-3.png"}})

#### 2. NodeJS Tools 설정

1. Jenkins 관리 > Global Tool Configuration
2. NodeJS > NodeJS installations...
3. nodejs installer 정보 입력
    - Name: 원하는 이름(nodejs)
    - Install automatically: 체크
    - Install from nodejs.org
    - Version: NodeJS 14.17.0 (원하는 버전 but latest LTS 버전인 14버전 권장)
4. 저장

#### 3. 적용할 pipeline에 nodejs tool 환경 추가

```groovy
node ('jenkins-slave'){
    ...
    env.NODEJS_HOME = "${tool 'nodejs'}"	// 2-3 항목에서 입력한 nodejs installer 이름
    env.PATH="${env.NODEJS_HOME}/bin:${env.PATH}"
    sh 'npm --version'
    ...
```
위와같이 설정해주면 jenkins-slave에서 `node`, `npm` 명령어 사용 가능

#### 4. sonar property 추가

```
sonarqube {
       properties {
                property "sonar.sources", "src/main"
                property "sonar.tests", "src/test"
       }
}
```
> 위 코드는 gradle 프로젝트 build.gradle 파일 기준

소나큐브 검증시 기본적으로 java 경로만 잡는 경우

`sonar.sources = src/main`, `sonar.tests = src/test` 등으로 경로를 지정해주어야 JS 코드도 검사 가능

---
### SonarQube for Maven 설정

Maven의 경우 설치형이기 때문에 별도로 dependency 설정이 필요하지 않다.

sonar-maven-plugin을 저장소에서 가져와서 바로 실행한다.

```groovy
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

---
### SonarQube for Gradle 설정

Gradle 프로젝트의 경우 보통 gradlew의 wrapper 형태로 제공되는 경우가 대부분이다.

따라서 sonarqube 플러그인을 프로젝트의 build.gradle dependency에 추가해주어야한다.

아래 부분 프로젝트의 build.gradle에 추가

```groovy
# plugin 부분 추가
	plugin {
    	id 'org.sonarqube' version '3.0'
    }
# 또는
    buildscript {
        repositories {
            jcenter()
        }
        dependencies {
            classpath("org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:3.0")
        }
    }
    
	apply plugin: 'org.sonarqube'


# repository 추가
	repositories {   	
		jcenter()
	}
    
# sonarqube 프로퍼티 추가
    sonarqube {
		properties {
			property "sonar.sources", "src/main"
		}
	}
    
# multi module project의 경우
# 아래와 같이 검사할 모듈명 명시 후 안에 적용
project(":project-web") {
	apply plugin: 'org.sonarqube'

	sonarqube {
		properties {
			property "sonar.sources", "src/main"
		}
	}
}
# 또는 전체 적용시
subprojects {
	apply plugin: 'org.sonarqube'

	sonarqube {
		properties {
			property "sonar.sources", "src/main"
		}
	}
}

```

빌드시 sonarqube gradle plugin 다운로드 및 검사 확인

---
### SonarQube Quality Profiles 적용

기본적으로 각 언어별 Sonar way라는 Default Profile이 적용되어 있음

![sc-4.png]({{ "/assets/img/contents/sc-4.png"}})

1. sonarqube 대시보드 로그인
2. (선택) Quality Profiles > Create 또는 Java Sonar way 프로필 설정 > Copy
    - Name: 원하는 이름
    - Language: 원하는 언어
    - Parent: none
3. 새로 생성한 Profile 설정 > Set as Default
4. 설정 > Activate More Rules에서 추가로 적용할 Rule 탐색
5. Activate 클릭하여 적용

---
### SonarQube GitLab 계정/그룹 연동

SonarQube 로그인시 GitLab OAuth2를 사용하여 gitlab 계정 연동을 할 수 있다.


#### 1. GitLab 사이드
1. GitLab OAuth2 Application 등록
    1. Admin Area > Applications > New application
    2. New application 입력
        - Name: 원하는 어플리케이션 이름 입력(GitLab SonarQube)
        - Redirect URI: `${sonarqube-uri}/oauth2/callback/gitlab` 입력 (public URL이어야한다.)
        - Trusted: 현재 등록하는 어플리케이션을 신뢰할 것인지? 체크
        - Confidential: Client Secret을 암호화통신할 것인지? 체크
        - Scopes: api 체크(gitlab oauth2 api 사용권한)
    3. Submit 후 앱 정보 확인
    4. Application ID, Secret 복사

#### 2. SonarQube 사이드

![sc-5.png]({{ "/assets/img/contents/sc-5.png"}})

1. Server Base URL 설정
    1. Administration > Configuration > General Settings
    2. Server base URL 입력
        - 외부 연동시 기본적으로 public 통신이기 때문에 public url 입력
            - https://sonarqube.yourdomain.com/sonarqube
    3. Save
2. ALM Integration 설정
    1. Administration > Configuration > ALM Integrations > GitLab
    2. GitLab Authentication 항목 입력
        - Enabled: true
        - GitLab URL: public gitlab URL 입력(`https://gitlab.yourdomain.com`)
        - Application ID, Secret: 1-4에서 복사한 항목 각각 입력
        - Allow users to sign-up: gitlab oauth2로 처음 로그인하는 사용자를 sonarqube에 등록할 것인지? 체크
        - Synchronize user groups: 소나큐브에 gitlab 그룹명과 일치하는 그룹이 생성되어 있다면 유저를 자동으로 등록한다. 체크
3. #### Java caCerts 인증서 설정(Kubernetes TLS Secret)
   
    > Java에서는 https 통신시 기본적으로 java keystore에 인증서를 요구한다.<br />위에서 public 경로를 http로 설정하였으면 상관 없으나 https로 설정한 경우 진행
    >
    > 현재 Java 기반 서비스들은 Kubernetes 환경에 떠있으므로 해당 환경으로 진행
    1. 기존 k8s에 `kubernetes.io/tls` 타입으로 떠있는 tls 타입 시크릿은 사용불가하므로 opaque 타입(기본타입) secret을 생성한다.
    2. sonarqube-secret.yaml 생성
       ```shell
    	$ vim sonarqube-secret.yaml
       ```
       ```yaml
       apiVersion: v1
       kind: Secret
       metadata:
         name: sonarqube-secret
       data:
         cert-1.crt: MIIFIzCC... # .crt(X.509 포맷)의 인증서값
       ```
       ```shell
       $ kubectl apply -f sonarqube-secret.yaml
       ```
    3. helm values.yaml 수정
       ```shell
       $ vim values.yaml
       ```
       ```yaml
       ...
        caCerts:
          image: adoptopenjdk/openjdk11:alpine
          enabled: true
          secret: sonarqube-secret
       ...
       ```
       ```shell
       # helm upgrade
       $ helm upgrade sonarqube oteemocharts/sonarqube -f values.yaml
       ```
4. #### 테스트
   
    1. 로그아웃 후 메인페이지에 들어가면 아래 이미지와 같이 Log in with GitLab 버튼 생성 확인
    ![sc-6.png]({{ "/assets/img/contents/sc-6.png"}})
    2. 버튼 누르면 GitLab에 로그인되어 있는 경우 자동으로 소나큐브 사용자 생성 및 연동이 완료된다.
    3. gitlab 그룹명과 일치하는 그룹을 미리 생성해 놓으면 로그인시 해당 유저 그룹 자동 연동

---
### SonarQube 배지

![sc-7.png]({{ "/assets/img/contents/sc-7.png"}})

각 프로젝트 대시보드 > 우상단 Project Information

markdown 형태로 실시간 정보 배지를 embed 가능

대신 해당 배지를 사용하려면 SonarQube 프로젝트가 Public으로 설정되어야하며 이는 소스 취약점을 외부에서 누구나 볼 수 있다는 것을 의미한다.

---
## Troubleshooting

### 1. 분석 권한 없음

```shell
[ERROR] Error during SonarScanner execution
[ERROR] You're not authorized to run analysis. Please contact the project administrator.
```

빌드 후 검증 수행시 위와 같은 오류가 날 때

SonarQube 플러그인 설치 및 실행은 완료되었으나 Jenkins - SonarQube 연동 계정이 Analysis 실행 권한이 없는 경우이다.

![sc-8.png]({{ "/assets/img/contents/sc-8.png"}})

1. SonarQube 대시보드 로그인(관리자 계정 필요)
2. Administration > Security > Global Permissions
3. 연동된 유저가 속한 그룹 혹은 유저 자체에 Execute Analysis 권한 체크

### 2. class 파일 없음 

```shell
[ERROR] Failed to execute goal org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar (default-cli) on project sample-project: Your project contains .java files, please provide compiled classes with sonar.java.binaries property, or exclude them from the analysis with sonar.exclusions property. -> [Help 1]
```

pipeline 또는 job 실행 순서에 빌드 이전에 분석을 시도하고 있는지 체크할 것.

Java의 경우 sonarqube는 신뢰도를 높이기 위하여 .java 파일만으로 코드 분석을 하지 않고 .java와 .class 파일을 함께 분석한다고 한다.

> [https://docs.sonarqube.org/latest/analysis/languages/java/#header-2](https://docs.sonarqube.org/latest/analysis/languages/java/#header-2) 참조

따라서 컴파일된 .class 파일이 있어야하므로 maven 또는 gradle 플러그인을 사용하여야함.

만약 없을 경우, 수동으로 컴파일하여 .class 파일을 넣어줘야함.

### 3. 소스 분석시 HTTP 413 REQUEST ENTITY TOO LARGE

소스 분석시 `HTTP 413 REQUEST ENTITY TOO LARGE`가 나오는 경우

검증할 소스가 sonarqube의 max body size(기본값: 20m)를 초과하여 나오는 경우로 

아래와 같이 sonarqube-ingress.yaml에 proxy body size 설정

```yaml
...
   apiVersion: networking.k8s.io/v1
   kind: Ingress
    metadata:
      name: sonarqube
      namespace: default
      annotations:
        kubernetes.io/ingress.class: "nginx"
        nginx.ingress.kubernetes.io/ssl-redirect: "false"
        nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
        nginx.ingress.kubernetes.io/proxy-body-size: "20M"		// 20M 초과시 HTTP 413 방지하기 위하여
...
```


---

## Reference

1. [SonarScanner for Jenkins](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/)
2. [SonarQube Scanner for Jenkins](https://www.jenkins.io/doc/pipeline/steps/sonar/#sonarqube-scanner-for-jenkins)
3. [SonarQube GitLab Integration](https://docs.sonarqube.org/latest/analysis/gitlab-integration/)