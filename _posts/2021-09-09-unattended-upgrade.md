---
layout: post
title: Apt 자동 업그레이드 비활성화(Unattended upgrade)
feature-img: assets/img/titles/ubuntu-logo.svg
thumbnail: assets/img/titles/ubuntu-logo.svg
author: csupreme19
categories: System Linux
tags: [Linux, OS, Ubuntu, Debian, Package Manager, APT, Upgrade]

---

# Apt 자동 업그레이드 비활성화(Unattended upgrade)

![ubuntu-logo.svg]({{ "/assets/img/titles/ubuntu-logo.svg"}})

---
## 개요

Ubuntu 16, 18, 20에서는 기본적으로 Apt 패키지의 Unattended upgrade라는 자동 업데이트 타이머 크론잡 형태로 활성화 되어 있어

리눅스 커널, SSH, OpenSSL 등의 보안 패치를 위하여 자동으로 업데이트를 진행하도록 기본 설정되어 있어요.

보안 취약점을 패치한다는 취지에서는 좋지만 Install, Upgrade된 버전을 사용하려면 서버 재시작이 필수이고

커널 업데이트로 인하여 OS레벨에서의 문제점이나 장애가 발생할 수 있어

이런 장애는 파악하기 매우 어려우므로 운영 관점에서는 해당 기능을 비활성화 하는 것이 좋아요.

---

## Unattended upgrade 비활성화

### apt 로그 확인

```shell
$ cd /var/log/apt
$ tail history.log
```

### apt-daily 비활성화
```shell
$ systemctl stop apt-daily.timer
$ systemctl stop apt-daily-upgrade.timer
$ systemctl disable apt-daily.timer
$ systemctl disable apt-daily.service
$ systemctl disable apt-daily-upgrade.timer
$ systemctl disable apt-daily-upgrade.service
```

### unattended-upgrade 비활성화
```shell
$ vim /etc/apt/apt.conf.d/20auto-upgrades
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Download-Upgradeable-Packages "0";
APT::Periodic::AutocleanInterval "0";
APT::Periodic::Unattended-Upgrade "0";
```

### 비활성화 확인
```shell
$ systemctl status apt-daily.timer
$ systemctl status apt-daily.service
$ systemctl status apt-daily-upgrade.timer
$ systemctl status apt-daily-upgrade.service
$ systemctl list-unit-files | grep apt-daily

$ apt-config dump | grep Update-Package-Lists
```

