---
layout: post
title: Elastic APM Java Agent 적용하기
feature-img: assets/img/titles/elastic-logo.png
thumbnail: assets/img/contents/eaja-1.png
author: csupreme19
categories: DevOps Elastic
tags: [Elasticsearch, Elastic, ELK, APM, ArgoCD, GitOps, Helm, Java, JVM]

---

# Elastic APM Java Agent 적용하기

![elastic-logo.png]({{ "/assets/img/titles/elastic-logo.png"}})

Elastic APM Java Agent 적용기

---

### APM Agent 설치
[APM Agents](https://www.elastic.co/guide/en/apm/agent/index.html)

apm-agent를 사용하기 위해서는 apm-server가 필요하다.

apm-server가 구축되어 있지 않으면 [Elastic APM Server](/2021/07/22/elastic-apm-server.html) 문서 참고

APM Java Agent 공식 지원 설치 방법은 3가지가 있다.

#### 1. Manual setup with -javaagent flag

#### 2. Automatic setup with apm-agent-attach-cli.jar

#### 3. Programmatic API setup to self-attach

본 문서에서는 최종적으로 아래 `-javaagent`와 k8s의 InitContainer를 사용하는 방법을 사용

#### 4. -javaagent flag with Kubernetes InitContainer

---

### Manual setup with -javaagent flag
[APM Java Agent Reference](https://www.elastic.co/guide/en/apm/agent/java/current/index.html)

별도 코드 수정이 필요없는 -javaagent 플래그를 이용한 에이전트 설치 방법

#### 1. Dockerfile 수정

```Dockerfile
FROM openjdk:8-jre-alpine

ADD your-project/build/libs/*.jar your-project.jar
ADD https://search.maven.org/remotecontent?filepath=co/elastic/apm/elastic-apm-agent/1.25.0/elastic-apm-agent-1.25.0.jar elastic-apm-agent.jar

ENTRYPOINT ["java", "-javaagent:/elastic-apm-agent.jar", "-Delastic.apm.service_name=your-project", "-Delastic.apm.application_packages=com.company.app", "-Delastic.apm.server_url=http://{엘라스틱 APM 서버 주소}:{포트}", "-Delastic.apm.environment=local", "-jar", "your-project.jar"]
```

> maven repo에서 APM Agent jar를 받아서 -javaagent 플래그로 JVM 위에 같이 실행한다.

##### 플래그
- -Delastic.apm.service_name=your-project
  - APM 모니터링시 확인할 서비스명(필수)
   - APM 메뉴에 표출될 서비스명을 적기
- -Delastic.apm.application_packages=com.company.app
  - APM 모니터링할 root package(필수)
   - APM 모니터링할 Java root package명 적기
- -Delastic.apm.server_url=http://{엘라스틱  APM 서버 주소}:{포트}
  - APM Server 주소(필수)
   - APM Server 엔드포인트
- -Delastic.apm.environment=local
  - APM 모니터링 환경(옵션)

그 외 자세한 configuration 값은 [APM Java Agent Configuration](https://www.elastic.co/guide/en/apm/agent/java/current/configuration.html) 참고

#### 2. Jenkins 빌드

#### 3. 로그 확인

```shell
2021-07-27 14:21:04,144 [main] INFO  co.elastic.apm.agent.util.JmxUtils - Found JVM-specific OperatingSystemMXBean interface: com.sun.management.OperatingSystemMXBean
2021-07-27 14:21:04,535 [main] INFO  co.elastic.apm.agent.configuration.StartupInfo - Starting Elastic APM 1.25.0 as your-project on Java 1.8.0_212 Runtime version: 1.8.0_212-b04 VM version: 25.212-b04 (IcedTea) Linux 4.15.0-144-generic
2021-07-27 14:21:04,535 [main] INFO  co.elastic.apm.agent.configuration.StartupInfo - environment: 'local' (source: Java System Properties)
2021-07-27 14:21:04,535 [main] INFO  co.elastic.apm.agent.configuration.StartupInfo - server_url: 'http://{엘라스틱 APM 서버 주소}:{포트}' (source: Java System Properties)
2021-07-27 14:21:04,536 [main] INFO  co.elastic.apm.agent.configuration.StartupInfo - application_packages: 'com.company.app' (source: Java System Properties)
2021-07-27 14:21:11,952 [main] INFO  co.elastic.apm.agent.impl.ElasticApmTracer - Tracer switched to RUNNING state
2021-07-27 14:21:12,340 [elastic-apm-server-healthcheck] INFO  co.elastic.apm.agent.report.ApmServerHealthChecker - Elastic APM server is available: {  "build_date": "2021-04-20T19:55:39Z",  "build_sha": "32f34ed4298d648bf9476790f2a8a54d72805bb6",  "version": "7.12.1"}
```

Application 시작 전에 위와 같은 로그가 나오면 정상 동작 확인

---

### Automatic setup with apm-agent-attach-cli.jar

[Automatic setup with `apm-agent-attach-cli.jar`](https://www.elastic.co/guide/en/apm/agent/java/current/setup-attach-cli.html#setup-attach-cli-supported-environments)

해당 방법은 현재 Host에서 실행되는 모든 JVM에 적용하므로 원하는 서비스만 선택 적용이 어렵다는 단점이 있음

또한 서비스별 config를 적용할 수 있는 가이드가 제공되지 않는다.

APM 서버에서 수집하는 서비스명이 동일하게 나오는 문제가 있음

아래는 모든 k8s worker 노드에 수행되어야함

#### 1. apm-agent-attach-cli.jar 다운로드

```shell
$ wget -v -O apm-agent-attach-cli-1.25.0.jar https://search.maven.org/remotecontent?filepath=co/elastic/apm/apm-agent-attach-cli/1.25.0/apm-agent-attach-cli-1.25.0.jar
```
#### 2. `attach.sh` 작성

```shell
#!/usr/bin/env bash
set -ex

attach () {
    # only attempt attachment if this looks like a java container
    if [[ $(docker inspect ${container_id} | jq --raw-output .[0].Config.Cmd[0]) == java ]]
    then
        echo attaching to $(docker ps --no-trunc | grep ${container_id})
        docker cp ./apm-agent-attach-*-cli.jar ${container_id}:/apm-agent-attach-cli.jar
        docker exec ${container_id} java -jar /apm-agent-attach-cli.jar --config
    fi
}

# attach to running containers
for container_id in $(docker ps --quiet --no-trunc) ; do
    attach
done

# listen for starting containers and attach to those
{% raw %}
docker events --filter 'event=start' --format '{{.ID}}' |
while IFS= read -r container_id
do
    attach
done
{% endraw %}
```

현재 구동중인 JVM 컨테이너와 앞으로 구동될 JVM 컨테이너에 java agent attach하는 방법

#### 3. jq 설치

```shell
$ apt install jq
$ jq --version
jq-1.5-1-a5b5cbe
```

위 attach 스크립트에서 java를 검사하는 방법은 jq 사용하여 json 검사하는 방법이므로 jq 설치 필요

#### 4. 스크립트 실행

```shell
$ chmod +x ./attach.sh
$ ./attach.sh &
```

---

### Programmatic API setup to self-attach

[Programmatic API setup to self-attach](https://www.elastic.co/guide/en/apm/agent/java/current/setup-attach-api.html)

소스코드 dependency 추가하여 직접 설정하는 방법

어플리케이션 코드에 직접적인 코드 및 수정이 필요하므로 별로 추천하지는 않는다.

마이크로서비스에서의 많은 서비스들 일일이 추가하기도 어렵고 중복되기 때문

#### 1. `pom.xml` 

```xml
<dependency>
    <groupId>co.elastic.apm</groupId>
    <artifactId>apm-agent-attach</artifactId>
    <version>${elastic-apm.version}</version>
</dependency>
```

#### 2. `Application.java`

```java
import co.elastic.apm.attach.ElasticApmAttacher;
import org.springframework.boot.SpringApplication;

@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        ElasticApmAttacher.attach();
        SpringApplication.run(MyApplication.class, args);
    }
}
```

각 어플리케이션의 java main class에 위 코드를 추가한다.

---

### Kubernetes InitContainer(사용)

참고: [엘라스틱서치 APM을 이용해서 쿠버네티스 환경의 자바 어플리케이션 모니터링하기(1)](https://m.blog.naver.com/olpaemi/221788820388)

`-javaagent` 와 Kubernetes에서 제공하는 InitContainer 기능을 사용하는 방법

#### InitContainer

<div class="mermaid">
flowchart LR
  subgraph Pod
	  A[InitContainer]
    B[Container]
    subgraph Volume
    C[EmptyDir]
  	end
  end
  A--Create-->C
  C-.Delete.->A
  C--Mount-->B
</div>

[Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

Deployment 파드 안에 컨테이너 앱 실행 이전에 실행된다.

InitContainer는 항상 실행이 보장되어 있으므로 서비스와 모니터링을 별개의 레벨로 분리할 수 있음



공유 볼륨을 이용하여 apm-agent jar를 공유 볼륨에 넣어놓고

컨테이너 실행시 InitContainer에서 apm-agent jar를 복사하는 방식

#### `kubernetes.yaml` 수정(추가된 부분만 작성)

```shell
apiVersion: apps/v1
kind: Deployment
...
spec:
  template:
    metadata:
    ...
    spec:
      initContainers:                        # apm-agent 적용을 위한 initContainer
      - name: elastic-java-agent
        image: docker.elastic.co/observability/apm-agent-java:1.25.0
        volumeMounts:
        - mountPath: /elastic/apm-agent
          name: elastic-apm-agent
        command: ['cp', '-v', '/usr/agent/elastic-apm-agent.jar', '/elastic/apm-agent']
      containers:
      - name: your-project
        image: abcde
        volumeMounts:
        ...
        - name: elastic-apm-agent
          mountPath: /elastic/apm-agent
        env:                                # apm-agent 필수 환경변수 추가
        - name: ELASTIC_APM_SERVER_URL 
          value: "http://{엘라스틱 APM 서버 주소}:{포트}"
        - name: ELASTIC_APM_SERVICE_NAME 
          value: "your-project"  
        - name: ELASTIC_APM_ENVIRONMENT 
          value: local
        - name: JAVA_TOOL_OPTIONS
          value: -javaagent:/elastic/apm-agent/elastic-apm-agent.jar
      ...
      volumes:
      ...
      - name: elastic-apm-agent
        emptyDir: {}
...
```

적용 이후 Jenkins 배포

#### pod 로그 확인

```shell
Picked up JAVA_TOOL_OPTIONS: -javaagent:/elastic/apm-agent/elastic-apm-agent.jar
2021-08-04 14:13:53,838 [main] INFO  co.elastic.apm.agent.util.JmxUtils - Found JVM-specific OperatingSystemMXBean interface: com.sun.management.OperatingSystemMXBean
2021-08-04 14:13:54,174 [main] INFO  co.elastic.apm.agent.configuration.StartupInfo - Starting Elastic APM 1.25.0 as your-project on Java 1.8.0_212 Runtime version: 1.8.0_212-b04 VM version: 25.212-b04 (IcedTea) Linux 4.15.0-144-generic
2021-08-04 14:13:54,226 [main] INFO  co.elastic.apm.agent.configuration.StartupInfo - service_name: 'your-project' (source: Environment Variables)
2021-08-04 14:13:54,227 [main] INFO  co.elastic.apm.agent.configuration.StartupInfo - environment: 'local' (source: Environment Variables)
2021-08-04 14:13:54,227 [main] INFO  co.elastic.apm.agent.configuration.StartupInfo - server_url: 'http://{엘라스틱 APM 서버 주소}:{포트}' (source: Environment Variables)
2021-08-04 14:13:54,227 [main] WARN  co.elastic.apm.agent.configuration.StartupInfo - To enable all features and decrease startup time, please configure application_packages
2021-08-04 14:14:01,639 [main] INFO  co.elastic.apm.agent.impl.ElasticApmTracer - Tracer switched to RUNNING state
2021-08-04 14:14:02,030 [elastic-apm-server-healthcheck] INFO  co.elastic.apm.agent.report.ApmServerHealthChecker - Elastic APM server is available: {  "build_date": "2021-04-20T19:55:39Z",  "build_sha": "32f34ed4298d648bf9476790f2a8a54d72805bb6",  "version": "7.12.1"}
```

apm-server 접속 및 available 확인

### Kibana 확인

![eaja-1.png]({{ "/assets/img/contents/eaja-1.png"}})

![eaja-2.png]({{ "/assets/img/contents/eaja-2.png"}})

![eaja-3.png]({{ "/assets/img/contents/eaja-3.png"}})

![eaja-4.png]({{ "/assets/img/contents/eaja-4.png"}})

인덱스 정상 수집 확인

---

## Reference

1. [APM Agents](https://www.elastic.co/guide/en/apm/agent/index.html)
2. [APM Java Agent Reference](https://www.elastic.co/guide/en/apm/agent/java/current/index.html)
3. [APM Java Agent Configuration](https://www.elastic.co/guide/en/apm/agent/java/current/configuration.html)
4. [Automatic setup with `apm-agent-attach-cli.jar`](https://www.elastic.co/guide/en/apm/agent/java/current/setup-attach-cli.html#setup-attach-cli-supported-environments)
5. [Programmatic API setup to self-attach](https://www.elastic.co/guide/en/apm/agent/java/current/setup-attach-api.html)
6. [엘라스틱서치 APM을 이용해서 쿠버네티스 환경의 자바 어플리케이션 모니터링하기(1)](https://m.blog.naver.com/olpaemi/221788820388)
7. [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

