---
layout: post
title: 시스템 Graceful shutdown
feature-img: assets/img/titles/ubuntu-logo.svg
thumbnail: assets/img/titles/ubuntu-logo.svg
author: csupreme19
categories: System Linux
tags: [System, OS, Ubuntu, Linux, Debian]

---

# 시스템 Graceful shutdown

![postgresql-logo.svg]({{ "/assets/img/titles/ubuntu-logo.svg"}})

---
## 시스템 재부팅 방법

Ubuntu 기준으로 시스템을 장애 없이 Graceful 하게 shutdown하는 방법을 정리해봤어요.

---

## 서버별 재부팅

### Kubernetes 서버
> [Gracefully shutdown kubernetes nodes](https://stackoverflow.com/a/59576322/15263734)

#### 1. 노드 drain

```shell
$ kubectl drain $NODENAME --ignore-daemonsets=true

# local storage 때문에 지울 수 없는 경우
$ kubectl drain $NODENAME --ignore-daemonsets=true --delete-emptydir-data
```
노드 drain을 통해 해당 노드에서 실행되는 파드를 모두 제거하고 해당 노드의 파드 스케줄링을 비활성화해요.
> Replica가 1개인 Deployment들은 다운타임이 발생할 수 있어요.

#### 2. 노드 drain 확인

```shell
$ kubectl get nodes $NODENAME
NAME               STATUS   					ROLES                  AGE    VERSION
$NODENAME          Ready SchedulingDisabled     control-plane,master   118d   v1.20.6
$ kubectl get pod -A -o wide | grep $NODENAME
argocd                 argocd-application-controller-0                  1/1     Succeeded   1          92d     10.244.2.96     $NODENAME    <none>           <none>
kube-system            filebeat-2svwt                                   1/1     Succeeded   0          21d     10.222.222.22   $NODENAME    <none>           <none>
kube-system            kube-flannel-ds-4whmt                            1/1     Succeeded   5          118d    10.222.222.22   $NODENAME    <none>           <none>
kube-system            kube-proxy-qjsc8                                 1/1     Succeeded   4          118d    10.222.222.22   $NODENAME    <none>           <none>
kube-system            metricbeat-2ggh2                                 1/1     Succeeded   1          69d     10.222.222.22   $NODENAME    <none>           <none>
kubernetes-dashboard   kubernetes-dashboard-64b46974f-56h9x             1/1     Succeeded   2          81d     10.244.2.98     $NODENAME    <none>           <none>
production               yourapp-prd-deploy-57f9947968-zklkq             1/1     Succeeded   0          4d15h   10.244.2.130    $NODENAME    <none>           <none>
```
- 노드 상태 `SchedulingDisabled` 확인
- 파드 상태 `Succeeded` 정상종료 확인
  - 파드 상태가 `Terminating`인 경우 파드가 중지되는 중이니 대기
  
  > `drain` 명령어는 파드의 정상 종료(Graceful shutdown)를 보장해요.

#### 3. 서버 재부팅

```shell
# 아래 두 명령어중 아무거나 수행
$ shutdown -r now
$ systemctl reboot
```

#### 4. 노드 파드 스케줄링 재활성화

```shell
$ kubectl uncordon $NODENAME
```

#### 5. 노드 상태 확인

```shell
$ kubectl get nodes $NODENAME
NAME               STATUS   ROLES                  AGE    VERSION
$NODENAME          Ready    control-plane,master   118d   v1.20.6
```

노드 상태 `Ready`만 있는 것 확인

> 기존 떠있는 Pod들은 Reschedule 시점까지 새로 생긴 Node로 스케줄 되지 않아요.

<br>

### RDBMS 서버 재부팅

#### 1. RDBMS 캐시 flush

```shell
$ sync;sync
```

#### 2. rdbms 서비스 중지

```shell
$ systemctl stop postgresql # postgresql의 경우
$ systemctl stop mysql # mysql의 경우
```

#### 3. 재부팅

```shell
# 아래 두 명령어중 아무거나 수행
$ shutdown -r now
$ systemctl reboot
```

#### 4. 서비스 정상 실행 확인

```shell
$ systemctl status postgresql@11-main.service # postgresql의 경우
$ systemctl status mysql.service # mysql의 경우

$ systemctl status metricbeat.service # metricbeat
$ systemctl status filebeat.service # filebeat
```

<br>

### 오픈소스 서버 재부팅

#### 1. 재부팅

```shell
# 아래 두 명령어중 아무거나 수행
$ shutdown -r now
$ systemctl reboot
```

#### 2. 서비스 정상 실행 확인

```shell
# 각 서버의 오픈소스에 해당되는 서비스 점검
$ systemctl status mongos.service # mongos
$ systemctl status mongod.service # mongod
$ systemctl status zookeeper.service # zookeeper
$ systemctl status kafka.service # kafka
$ systemctl status elasticsearch.service # elasticsearch
$ systemctl status logstash.service # logstash
$ systemctl status kibana.service # kibana
$ systemctl status nginx.service # nginx
$ systemctl status metricbeat.service # metricbeat
$ systemctl status filebeat.service # filebeat
```

정상 실행 안되었을시 하단 수동 구동 명령어 참고 바라요.

---

## 참고사항

### 재부팅 정보 확인 

#### 최근 재부팅 시간 확인

```shell
$ who -b
         system boot  2021-06-15 19:00

$ last reboot
reboot   system boot  3.10.0-1160.31.1 Tue Jun 15 19:00 - 10:06 (55+15:06)
reboot   system boot  3.10.0-1160.25.1 Wed Jun  9 20:25 - 10:06 (61+13:40)
reboot   system boot  3.10.0-957.el7.x Tue Jun  8 17:43 - 10:06 (62+16:23)
```

#### 업타임 확인

```shell
$ uptime
10:05:54 up 55 days, 15:05,  1 user,  load average: 0.00, 0.01, 0.05
```

<br>

### systemd 정보 확인

#### systemd 등록 서비스 리스트 확인

```shell
$ systemctl list-unit-files | grep enabled
$ systemctl status 오픈소스
$ ps -ef
```
systemd 시스템 데몬에 등록된 프로세스들은 리부트시 자동으로 실행되는데

systemd를 사용하지 않는 프로세스는 OS 재시작시 재기동 여부 확인 후 재기동되지 않으면 수동으로 시작이 필요해요.

systemd에 등록이 되어 자동 실행된다고 하더라도 실제 작동 여부 확인 필요해요.

<br>

### 서비스별 systemd 사용 여부

아래 테이블은 오픈소스 리스트와 systemd 등록 여부 및 수동 시작 명령어를 작성했어요.

> 설치형을 기준으로 작성되었으며 버전 및 환경별로 다를 수 있으니 참고 용도로만 사용하는 것을 권장드려요.

| 서비스         | systemd | 수동 시작 명령어(systemctl 불가시)                           |
| -------------- | ------- | ------------------------------------------------------------ |
| metricbeat     | O       | /usr/share/metricbeat/bin/metricbeat --environment systemd -c /etc/metricbeat/metricbeat.yml --path.home /usr/share/metricbeat --path.config /etc/metricbeat --path.data /var/lib/metricbeat --path.logs /var/log/metricbeat |
| filebeat       | O       | /usr/share/filebeat/bin/filebeat --environment systemd -c /etc/filebeat/filebeat.yml --path.home /usr/share/filebeat --path.config /etc/filebeat --path.data /var/lib/filebeat --path.logs /var/log/filebeat |
| mongos         | O       | /usr/bin/mongos --config /etc/mongos.conf                    |
| mongod         | O       | /usr/bin/mongod --config /etc/mongod.conf                    |
| postgresql     | O       | /usr/bin/pg_ctlcluster --skip-systemctl-redirect %i start    |
| mysql          | O       | /usr/sbin/mysqld                                             |
| zookeeper      | O       | /usr/local/zookeeper/bin/zkServer.sh start                   |
| kafka          | O       | /usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties |
| elasticseaerch | O       | /usr/share/elasticsearch/bin/systemd-entrypoint -p /var/run/elasticsearch/elasticsearch.pid --quiet |
| logstash       | O       | /usr/share/logstash/bin/logstash "--path.settings" "/etc/logstash" |
| kibana         | O       | /usr/share/kibana/bin/kibana "-c /etc/kibana/kibana.yml" --logging.dest="/var/log/kibana/kibana.log |
| nginx          | O       | nginx -c /etc/config/nginx.conf                              |

해당하는 서버 재부팅후 `systemctl status 서비스` 명령어로 각 서비스의 구동상태 확인 후 

구동되어 있지 않으면 `systemctl start 서비스`로 정상 실행여부 확인

그래도 실행되지 않으면 위 표에 있는 수동 시작 명령어 실행해요.


---

## Reference

1. [https://stackoverflow.com/a/59576322/15263734](https://stackoverflow.com/a/59576322/15263734)