---
layout: post
title: Tomcat session clustering 적용기
feature-img: assets/img/titles/tomcat-logo.svg
thumbnail: assets/img/contents/tsc-1.png
author: csupreme19
categories: Development
tags: [Tomcat, Session, Clustering, Multicast, Replication]
---

# Tomcat session clustering 적용기

![tomcat-logo.png]({{ "/assets/img/titles/tomcat-logo.svg"}})

[Clustering/Session Replication How-To](http://tomcat.apache.org/tomcat-9.0-doc/cluster-howto.html)

본 문서에서는 Kubernetes 환경에서의 Tomcat간 세션 공유를 위한 session clustering 적용기를 다룬다.

---

## 문제점

<div class="mermaid">
  flowchart LR
  subgraph k8s
  subgraph Node A
  A[Tomcat A]
  end
  subgraph Node B
  B[Tomcat B]
  end
  C[Service]
  end
  D[User]
  D-->C
  C-->A & B
  A x-.session.-x B
</div>



Spring boot 기반 Tomcat  서비스는 클라이언트에서 요청시 세션을 Tomcat이 올라가 있는 노드에만 local 저장된다.

이 경우 Pod replication, crash/shutdown 등으로 인하여 노드간 세션이 공유되지 않아 접속이 끊기는 문제가 발생할 수 있음

---

## 요구사항

Kubernetes 클러스터 내에서 Pod 간 세션을 유지할 수 있는 방법 수립

1. 유지보수가 쉬울 것(관리 포인트 적을 것)
2. 어플리케이션(서비스) 코드에 영향이 없을 것
3. 구성하는데 추가적인 리소스가 들어가지 않을 것

---

## 방법

### 1. Sticky session

![tsc-1.png]({{ "/assets/img/contents/tsc-1.png"}})

클라이언트가 세션을 맺은 서버랑만 통신하는 것

사용자 입장에서는 세션이 끊기지 않고 유지되는 장점이 있으나 

pod crash, shutdown 등의 문제로 pod 재시작, 중지시 세션이 끊기는 문제가 있다.

kubernetes session affinity 설정으로 비교적 간단히 설정이 가능하나

완벽한 session replication이 아니다.

### 2. Tomcat session in-memory replication(Multicast)

Tomcat에서 기본적으로 제공하는 클러스터링

서버 인메모리에 저장하고 SimpleTCPCluster와 같은 클래스를 이용하여 리플리케이션하는 방법

tomcat xml(server.xml, pom.xml) 수정이 필요하여 spring boot에서 기본으로 제공하는 embedded tomcat에서 사용이 안된다고 하나 java config를 이용하여 embedded tomcat도 설정 가능한 것으로 보인다.

multicast 기능을 지원해야하지만 multicast는 대부분의 cloud 환경에서는 제공하지 않음

Tomcat 설정 파일을 바꾸려면 Spring boot의 경우 `@Configuration` 설정을 통하여 Bean을 주입해야하여 소스코드 추가가 필요하다.

### 3. Tomcat session persistence replication

redis, dynamoDB와 같은 DB에 세션 정보를 저장하고 각 서버로 클러스터링 하는 방법

session이 별도의 저장소에 저장되어 사용되는 장점이 있음

---
## Tomcat session in-memory replication(Multicast) 테스트

[Clustering/Session Replication How-To](http://tomcat.apache.org/tomcat-9.0-doc/cluster-howto.html)

### EPC Multicast 지원 테스트

현재 사용중인 클라우드(EPC)에서 Multicast 기능을 지원하는지 테스트한다.

멀티캐스트 IP 대역은 사설 224.0.1.0 ~ 238.255.255.255의 대역 IP가 예약되어 있으며

멀티캐스트 사용을 위해서는 L2 장비가 IGMP 프로토콜을 지원해야한다고 한다.

>AWS와 같은 메이저 클라우드에서는 지원을 안한다고 하니 확인 필요

**iperf**라는 네트워크 테스트 툴을 이용하여 EPC에서 멀티캐스트 IP 대역 사용이 가능한지 테스트한다.



#### kubernetes worker 노드에서 진행

```shell
# iperf 설치 (root 진행)
$ wget -O /usr/bin/iperf https://iperf.fr/download/ubuntu/iperf_2.0.9
$ chmod +x /usr/bin/iperf

# Server side
$ iperf -s -u -B 228.0.0.4 -i 1 -p 45564
------------------------------------------------------------
Server listening on UDP port 45564
Binding to local address 228.0.0.4
Joining multicast group  228.0.0.4
Receiving 1470 byte datagrams
UDP buffer size:  208 KByte (default)
------------------------------------------------------------
[  3] local 228.0.0.4 port 45564 connected with 10.213.196.14 port 49293
[ ID] Interval       Transfer     Bandwidth        Jitter   Lost/Total Datagrams
[  3]  0.0- 1.0 sec   129 KBytes  1.06 Mbits/sec   0.006 ms    0/   90 (0%)
[  3]  1.0- 2.0 sec   128 KBytes  1.05 Mbits/sec   0.011 ms    0/   89 (0%)
[  3]  2.0- 3.0 sec   128 KBytes  1.05 Mbits/sec   0.007 ms    0/   89 (0%)
[  3]  0.0- 3.0 sec   386 KBytes  1.05 Mbits/sec   0.007 ms    0/  269 (0%)

# Client side (다른 노드에서 실행)
$ iperf -c 228.0.0.4 -u -T 32 -t 3 -i 1 -p 45564
------------------------------------------------------------
Client connecting to 228.0.0.4, UDP port 45564
Sending 1470 byte datagrams, IPG target: 11215.21 us (kalman adjust)
Setting multicast TTL to 32
UDP buffer size:  208 KByte (default)
------------------------------------------------------------
[  3] local 10.213.196.14 port 49293 connected with 228.0.0.5 port 45564
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 1.0 sec   131 KBytes  1.07 Mbits/sec
[  3]  1.0- 2.0 sec   128 KBytes  1.05 Mbits/sec
[  3]  2.0- 3.0 sec   128 KBytes  1.05 Mbits/sec
[  3]  0.0- 3.0 sec   386 KBytes  1.05 Mbits/sec
[  3] Sent 269 datagrams
```

> Tomcat의 경우 기본 multicast 주소는 `228.0.0.4`에 포트는 `45564` 사용

서버에서 클라이언트 각각 실행하여 클라이언트에서 송신하는 메시지가 서버에 멀티캐스트 되는지 확인

다른 노드에서 실행시 정상 송수신을 확인하였다. (Zone: Prd-private)

---

### Tomcat session clustering 샘플 프로젝트 작성

#### `build.gradle` 작성

```gradle
plugins {
	id 'org.springframework.boot' version '2.5.0'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.projectlombok:lombok:1.18.12'
	annotationProcessor 'org.projectlombok:lombok:1.18.12'
	implementation 'org.apache.tomcat:tomcat-catalina-ha:9.0.46'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}
```



#### 세션 정보 확인을 위한 간단한 `Controller` 작성

```java
@RestController
public class MainController {
	@GetMapping("/")
	public String getSession(HttpServletRequest request) {
		HttpSession session = request.getSession();
		String result = "session ID: " + session.getId() + "\n\nsession created: " + session.getCreationTime() + "\n\nsession accessed: " + session.getLastAccessedTime();
		return result;
	}
}
```



#### Tomcat config 설정

#### `TomcatClusterContextCustomizer`

```java
@Component
public class TomcatClusterContextCustomizer implements TomcatContextCustomizer {
	@Value("${env.multicast.receiver.port}")
	private Integer receiverPort;
	
	@Override
	public void customize(final Context context) {
		context.setDistributable(true);	// web.xml <distribute/>
		// Manager setting
		BackupManager manager = new BackupManager();
		manager.setNotifyListenersOnReplication(true);
		context.setManager(manager);
		configureCluster((Engine)context.getParent().getParent());
	}
	
	private void configureCluster(Engine engine) {
		// Cluster setting
		SimpleTcpCluster cluster = new SimpleTcpCluster();
		cluster.setChannelSendOptions(8);
		// Channel setting
		GroupChannel channel = new GroupChannel();
		// Membership setting
		McastService mcastService = new McastService();
		mcastService.setAddress("228.0.0.4");
		mcastService.setPort(45564);
		mcastService.setFrequency(500);
		mcastService.setDropTime(3000);
		channel.setMembershipService(mcastService);
		// Receiver setting
		NioReceiver receiver = new NioReceiver();
		receiver.setAddress("auto");	// tomcat의 LAN 주소를 가져온다. Ipv4로 설정해야함
		receiver.setPort(4000);
//		receiver.setPort(receiverPort);   톰캣이 같은 node에서 replication 구동시 리시버 포트는 서로 달라야함 이 경우 별도 포트 지정
		receiver.setAutoBind(100);
		receiver.setSelectorTimeout(5000);
		receiver.setMaxThreads(6);
		channel.setChannelReceiver(receiver);
		// Sender setting
		ReplicationTransmitter sender = new ReplicationTransmitter();
		sender.setTransport(new PooledParallelSender());
		channel.setChannelSender(sender);
		// Intercepter setting
		channel.addInterceptor(new TcpPingInterceptor());
		channel.addInterceptor(new TcpFailureDetector());
		channel.addInterceptor(new MessageDispatchInterceptor());
		cluster.addValve(new ReplicationValve());
		cluster.addValve(new JvmRouteBinderValve());
		cluster.setChannel(channel);
		cluster.addClusterListener(new ClusterSessionListener());
		engine.setCluster(cluster);
	}
}
```



#### `TomcatClusterUtil` 작성

```java
@Configuration
public class TomcatClusterUtil implements WebServerFactoryCustomizer<TomcatServletWebServerFactory>{
	@Autowired
	TomcatClusterContextCustomizer tomcatClusterContextCustomizer;
	
	@Override
	public void customize(final TomcatServletWebServerFactory factory) {
		factory.addContextCustomizers(tomcatClusterContextCustomizer);
	}
}
```

spring boot와 같이 설치형 tomcat이 아닌 embedded tomcat에서는 server.xml, web.xml과 같은 설정 파일의 직접 수정이 불가능하므로

`TomcatContextCustomizer`를 이용하여 Context에 설정한다.

> 항목별 자세한 설명은 [공식문서](http://tomcat.apache.org/tomcat-9.0-doc/cluster-howto.html) 참고



#### 인스턴스별 포트 설정을 위한 `application.yaml` 작성

```yaml
spring:
  profiles:
    active: instance1

---
spring:
  config:
    activate:
      on-profile: instance1
server:
  port: 8080
env:
  multicast.receiver.port: 4000

---
spring:
  config:
    activate:
      on-profile: instance2
server:
  port: 8081
env:
  multicast.receiver.port: 4001
```

> 로컬 테스트시 같은 포트로 톰캣 인스턴스 실행이 불가능하므로 포트 설정을 하기 위함



#### spring boot Run configuration 설정

1. ##### tomcat-session-replication-test1
   
   - Profile: instance1
   - JVM Arguments: -Djava.net.preferIPv4Stack=true
2. ##### tomcat-session-replication-test2
   
   - Profile: instance2
   - JVM Arguments: -Djava.net.preferIPv4Stack=true

> preferIPv4Stack `true`로 설정해야 위에서 Receiver주소 설정값 `receiver.setAddress("auto");`가 작동한다.

#### 로드밸런서 설정

Tomcat에서 지원하는 session clustering을 사용하려면 로드밸런서를 통해 같은 도메인이름을 사용하여야 세션이 공유된다.

서로 다른 도메인을 사용하거나 다른 서버인 경우 기본적으로 같은 session을 가지고 있다고 판단하지 않는 것 같다.

따라서 같은 도메인 설정을 위한 로드밸런서 역할을 할 nginx를 사용하여 리버스 프록시 설정을한다.



#### 맥 로컬 기준 작성

#### nginx 설치 및 설정

```shell
$ brew install nginx	# nginx 설치
$ vim /usr/local/etc/nginx/nginx.conf	# 로드밸런서 설정
 17   http {
 18     include       mime.types;
 19     default_type  application/octet-stream;
 ...
 35     upstream myproject {
 36       server 127.0.0.1:8080;
 37       server 127.0.0.1:8081;
 38     }
 39
 40     server {
 41         listen       80;
 42         server_name  localhost;
 ...
 48         location / {
 49           proxy_pass http://myproject;
 50         }
 $ nginx	# local nginx 구동
```

localhost:80 접속시 `127.0.0.1:8080`, `127.0.0.1:8081`로 각각 로드밸런싱



---

### 테스트(Local)

#### 테스트 순서
1. tomcat-session-replication-test1 구동
2. localhost 접속
3. session 확인
4. tomcat-session-replication-test2 구동
5. tomcat-session-replication-test1 중지
6. localhost 접속
7. session 확인



#### 테스트 진행 및 결과

1. tomcat-session-replication-test1 구동
![tsc-2.png]({{ "/assets/img/contents/tsc-2.png"}})
2. localhost 접속
3. session 확인
![tsc-3.png]({{ "/assets/img/contents/tsc-3.png"}})
4. tomcat-session-replication-test2 구동
5. tomcat-session-replication-test1 중지
![tsc-4.png]({{ "/assets/img/contents/tsc-4.png"}})
6. localhost 접속
7. session 확인
![tsc-5.png]({{ "/assets/img/contents/tsc-5.png"}})

session ID를 1번 인스턴스에서 생성했음에도 2번 tomcat으로 접속시 session ID가 서로 동일한 것을 확인할 수 있다.

---

### 테스트(Dev EPC)

현시점 기준 172.x 대역의 개발 클러스터에 설정하여 세션 공유가 kubernetes 환경에서도 적용되는지 테스트한다.

### kubernetes 배포
1. #### gitlab 프로젝트 생성 및 commit
2. #### 이미지 빌드를 위한 `Dockerfile`, k8s 배포를 위한 `yaml` 작성
   
   1. ##### `Dockerfile`
   
   
   
   ```Dockerfile
   FROM openjdk:8-jdk-alpine
   
    ADD build/libs/tomcat-session-replication-test-0.0.1-SNAPSHOT.jar TomcatSessionClusteringTest.jar
    RUN apk --no-cache add tzdata && \
            cp /usr/share/zoneinfo/Asia/Seoul /etc/localtime && \
            echo "Asia/Seoul" > /etc/timezone
   
    ENTRYPOINT ["java", "-Duser.timezone=Asia/Seoul", "-Djava.net.preferIPv4Stack=true", "-jar", "/TomcatSessionClusteringTest.jar", "--spring.profiles.active=instance1"]
    
   ```
   2. ##### `kubernetes.yaml`
   
   
   
   ```yaml
   ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      namespace: development
      name: tomcatsessionclustertest-deploy
      labels:
        app: tomcatsessionclustertest
    spec:
      replicas: 2	# pod를 2개 띄움
      selector:
        matchLabels:
          app: tomcatsessionclustertest
      template:
        metadata:
          labels:
            app: tomcatsessionclustertest
        spec:
          containers:
          - name: tomcatsessionclustertest
            image: gitlab.yourdomain.com/tomcatsessionclustertest:latest
            volumeMounts:
            - name: tz-seoul
              mountPath: /etc/localtime
            ports: 
            - containerPort: 8080
          imagePullSecrets:
          - name: reg
          volumes:
          - name: tz-seoul
            hostPath: 
              path: /usr/share/zoneinfo/Asia/Seoul
          affinity:
          # 같은 node에 뜨지 않게 하기 위한 Affinity 설정
            podAntiAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
              - labelSelector:
                  matchExpressions:
                  - key: app.kubernetes.io/name
                    operator: In
                    values:
                    - tomcatsessionclustertest
                 topologyKey: kubernetes.io/hostname
    ---
    apiVersion: v1
    kind: Service
    metadata:
      namespace: development
      name: tomcatsessionclustertest-svc
      labels:
        app: tomcatsessionclustertest
    spec:
      ports:
      - port: 8080
        nodePort: 32599
        targetPort: 8080
      selector:
        app: tomcatsessionclustertest
      type: NodePort
      
   ```
3. #### Jenkins 등록

파이프라인 아래와 같이 수정
```groovy
node ('jenkins-slave'){
    def gitBranch = 'master'
    def gitCredId = 'gitlab-ssh-key'
    def gitUrl = 'ssh://git@gitlab.yourdomain.com/group/'
    def projectUrl = gitUrl + 'tomcat-session-clustering-test.git'
    def dockerRepoUrl = 'http://nexus.yourdomain.com'
    def imageName = 'tomcatsessionclustertest'
    def defaultYaml = 'kubernetes.yaml'
    def deployYaml = 'kubernetes_deploy.yaml'

    stage('Checkout')    {
         git(
            url: projectUrl,
            credentialsId: gitCredId,
            branch: gitBranch
        )
    }
    
    stage('Build') {
        sh "chmod +x ./gradlew"
        sh "./gradlew clean build -x test"
    }

    stage('Analysis & Push') {
        parallel (
            'Docker Build': {
                def dockerfile = 'Dockerfile'
                app = docker.build("${imageName}:${env.BUILD_ID}","-f ${dockerfile} ./")
            }
        )
        
        parallel (
            'Registry Push': {
                docker.withRegistry("${dockerRepoUrl}", "dockerhub") {
                    app.push("$BUILD_ID")
                }
            }
        )
    }
    
    stage('Kube Deploy'){
        
        sh "cat ${defaultYaml} | sed 's/latest/${env.BUILD_ID}/g' > ${deployYaml}"

        withKubeConfig(credentialsId: 'jenkins-builder', serverUrl: 'https://kubernetes') {
            sh "kubectl apply -f ${deployYaml}"
        }
    }
    
}
```

> CI/CD 환경 및 Kubernetes 환경은 각각 다르므로 위 파이프라인은 참고만 할 것

4. #### kubernetes 배포 확인

  kubernetes 대시보드 접속 또는 명령어 확인

```shell
# alias k=kubectl
$ k get po -n development
$ k get svc -n development
```



### 외부 접속 설정 및 접속 테스트

1. 방화벽 오픈
   - 외부 -> kubernetes 32599 NodePort
2. 접속 확인
![tsc-6.png]({{ "/assets/img/contents/tsc-6.png"}})
3. 접속 로그 확인
![tsc-7.png]({{ "/assets/img/contents/tsc-7.png"}})
4. 접속된 파드 제거
```shell
$ k get pod -n development
$ k delete pod tomcatsessionclustertest-deploy-7d75d5548b-rknfh -n development
```
5. 재접속
![tsc-8.png]({{ "/assets/img/contents/tsc-8.png"}})



### 테스트 결과

![tsc-6.png]({{ "/assets/img/contents/tsc-6.png"}})

![tsc-8.png]({{ "/assets/img/contents/tsc-8.png"}})

세션 공유가 되지 않는것으로 확인되었다.

kubernetes 환경에서 host 서버간 multicast 통신은 되는 것으로 판단되나 파드간 multicast 통신을 위해서는 `multus-cni`를 사용하여 NIC로 호스트와 연결해야함

> 참고: [https://stackoverflow.com/a/61234801/15263734](https://stackoverflow.com/a/61234801/15263734)

Code dependency도 높고 kubernetes 환경에 별도의 CNI 설치를 요구하므로 사용이 어렵다.

---
## Tomcat session persistence replication 테스트

Redis, dynamoDB, hazelcast 등의 in-memory DB를 이용하여 세션을 저장하고 공유하는 방법 테스트

현재 EPC에서 Redis를 사용하기 때문에 redis를 이용하여 세션 저장하는 방법 테스트

### 테스트(LOCAL)

#### 맥 로컬 기준 작성

#### 1. Redis 설치

[https://hub.docker.com/\_/redis/](https://hub.docker.com/_/redis/)

```shell
$ docker pull redis
$ docker run --name redis -p 6379:6379 -d redis
$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                       NAMES
2e96b0e93c0a   redis     "docker-entrypoint.s…"   4 seconds ago   Up 2 seconds   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   redis
```

#### 2. 샘플 프로젝트 작성

##### `build.gradle`

```java
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	implementation 'org.springframework.boot:spring-boot-starter-data-redis'
	implementation 'org.springframework.session:spring-session-data-redis'
}
```

##### `application.yaml`

```yaml
spring:
  profiles:
    active: local

---
spring:
  config:
    activate:
      on-profile: local
  redis:
    host: 127.0.0.1
    port: 6379
server:
  port: 8080

---
spring:
  config:
    activate:
      on-profile: development
  redis:
    host: 172.25.0.163
    port: 6379
server:
  port: 8080
```

##### `MainController.java`

```java
@RestController
public class MainController {
	
	@GetMapping("/")
	public String getSession(HttpServletRequest request) {
		HttpSession session = request.getSession();
		String result = "session ID: " + session.getId() + "\n\nsession created: " + session.getCreationTime() + "\n\nsession accessed: " + session.getLastAccessedTime();
      	System.out.println(request.getLocalPort());
		return result;
	}

}
```

SpringBootApplication에 `@EnableRedisHttpSession` 추가
```java
@SpringBootApplication
@EnableRedisHttpSession
public class TomcatSessionReplicationTestApplication {
	public static void main(String[] args) {
		SpringApplication.run(TomcatSessionReplicationTestApplication.class, args);
	}
}
```



#### Spring boot Run configuration 설정

1. ##### tomcat-session-replication-test1
   
   - JVM Arguments: -Djava.net.preferIPv4Stack=true
   - Program Arguments: --server.port=8080
2. ##### tomcat-session-replication-test2
   
   - JVM Arguments: -Djava.net.preferIPv4Stack=true
   - Program Arguments: --server.port=8081



### 테스트(Local)

#### 테스트 순서

1. tomcat-session-replication-test1 구동
2. localhost 접속
3. session 확인
4. tomcat-session-replication-test2 구동
5. tomcat-session-replication-test1 중지
6. localhost 접속
7. session 확인
8. redis key 확인

#### 테스트 진행
1. tomcat-session-replication-test1 구동
![tsc-9.png]({{ "/assets/img/contents/tsc-9.png"}})
2. localhost 접속
3. session 확인
![tsc-10.png]({{ "/assets/img/contents/tsc-10.png"}})
4. tomcat-session-replication-test2 구동
5. tomcat-session-replication-test1 중지
![tsc-11.png]({{ "/assets/img/contents/tsc-11.png"}})
6. localhost 접속
7. session 확인
![tsc-12.png]({{ "/assets/img/contents/tsc-12.png"}})
8. redis key 확인
```shell
$ docker run -it redis bash
root@c5aeb4bf0493:/data# redis-cli -h 192.168.0.56		# host local ip
192.168.0.56:6379> keys *
1) "spring:session:expirations:1622602500000"
2) "spring:session:sessions:expires:063e4085-0cfc-4c49-b0c9-65a334327e13"
3) "spring:session:sessions:063e4085-0cfc-4c49-b0c9-65a334327e13"
```

redis에 session 정보가 저장이 되는 것을 확인 가능



![tsc-10.png]({{ "/assets/img/contents/tsc-10.png"}})

![tsc-12.png]({{ "/assets/img/contents/tsc-12.png"}})

session ID를 1번 인스턴스에서 생성했음에도 2번 tomcat으로 접속시 session ID가 서로 동일한 것을 확인할 수 있다.



### 테스트(Dev EPC)

### Kubernetes 배포

2번 테스트 항목의 Kubernetes 배포 과정과 똑같음

Dockerfile `"--spring.profiles.active=development"`로 수정

### 테스트 진행(Pod fail)
1. 접속 확인
![tsc-13.png]({{ "/assets/img/contents/tsc-13.png"}})
2. 접속 로그 확인
![tsc-14.png]({{ "/assets/img/contents/tsc-14.png"}})
3. 접속된 파드 중지
```shell
$ k get pod -n development
$ k delete pod tomcatsessionclustertest-deploy-79d5796f9c-dwxv2 -n development
```
4. 재접속
![tsc-15.png]({{ "/assets/img/contents/tsc-15.png"}})

톰캣 세션이 끊기지 않은 것을 확인할 수 있다.

> Redis가 죽으면 세션 접속 시점에 redis에 접속하므로 계속 reconnecting해 서비스가 죽는 문제가 존재하기는 한다.


---

## 결론

### 1. Sticky session

![tsc-1.png]({{ "/assets/img/contents/tsc-1.png"}})

클라이언트가 세션을 맺은 tomcat에 대해서만 통신을 하여 일반적인 상황에서는 세션이 유지되지만

정상적인 상황에서 로드밸런싱이 전혀 되지 않으며 해당 pod, node가 죽었을 때 세션이 끊기는 문제가 있다.



### 2. Tomcat session in-memory replication(Multicast)

Tomcat에서 기본적으로 가이드하고 있는 Session Replication 방법

![tsc-6.png]({{ "/assets/img/contents/tsc-6.png"}})

![tsc-8.png]({{ "/assets/img/contents/tsc-8.png"}})

로컬 환경에서는 세션이 유지되었으나 클라우드 환경에서는 다른 노드간 세션이 유지되지 않는 것으로 확인되었다.

Tomcat 서버 인메모리에 세션 정보를 저장하고 SimpleTCPCluster와 같은 클래스를 이용하여 리플리케이션 한다.

multicast 기능을 지원해야하지만 multicast는 대부분의 cloud 환경에서는 제공하지 않는다고 한다.

```
1. 유지보수가 쉬울 것(관리 포인트 적을 것)
2. 어플리케이션(서비스) 코드에 영향이 없을 것
3. 구성하는데 추가적인 리소스가 들어가지 않을 것
```

무엇보다도 세션 클러스터링 요구사항의 2, 3 번에 위배되므로 합리적인 방법이 아니다.



### 3. Tomcat session persistence replication

redis, dynamoDB와 같은 DB에 세션 정보를 저장하고 각 서버로 레플리케이션 하는 방법

![tsc-13.png]({{ "/assets/img/contents/tsc-13.png"}})

![tsc-13.png]({{ "/assets/img/contents/tsc-15.png"}})

다른 노드간 세션이 유지되는 것을 확인할 수 있었다.

session이 in-memory가 아닌 별도의 저장소에 저장되어 사용되는 장점이 있으며 코드 수정이 거의 없다는 장점이 있다.

> redis 라이브러리 때문에 의존성이 아예 없을 수는 없다.

따라서 가장 합리적인 Session clustering / Session replication이라고 할 수 있다.

---

## Reference

1. [Clustering/Session Replication How-To](http://tomcat.apache.org/tomcat-9.0-doc/cluster-howto.html)
2. [https://stackoverflow.com/a/61234801/15263734](https://stackoverflow.com/a/61234801/15263734)