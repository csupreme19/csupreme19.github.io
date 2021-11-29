---
layout: post
title: CentOS 파티션 생성 및 마운트
feature-img: assets/img/titles/centos-logo.jpeg
thumbnail: assets/img/contents/cpm-1.png
author: csupreme19
tags: [Centos, RHEL, Linux, OS, Partition, Mount]

---

# CentOS 파티션 생성 및 마운트

### 1. 디스크 정보 확인

```sh
fdisk -l
```

![cpm-1.png]({{ "/assets/img/contents/cpm-1.png"}})

### 2. 파티션 구성하기

```sh
fdisk /dev/sda
```

![cpm-2.png]({{ "/assets/img/contents/cpm-2.png"}})

별도의 파티션을 구성하는 것이 아니라면 기본값 사용

Partition type: p

Partition number: 1 (시스템에 따라 다를 수 있음)

First sector: 2048 (시스템에 따라 다를 수 있음)

Last sector: 4294967294 (시스템에 따라 다를 수 있음)

p 명령어로 파티션 설정 확인

w 명령어로 저장하고 fdisk 나오기



### 3. 파티션 포맷하기

```sh
mkfs.ext4 /dev/sda
```

![cpm-3.png]({{ "/assets/img/contents/cpm-3.png"}})

위에서 구성한 파티션을 포맷한다

ext4 포맷 사용



### 4. 파티션 마운트하기

```sh
mount /dev/sda /data
```

/dev/sda 파티션을 /data에 마운트 



### 5. 마운트 확인

```sh
df -h
mount -a
```

![cpm-4.png]({{ "/assets/img/contents/cpm-4.png"}})

![cpm-5.png]({{ "/assets/img/contents/cpm-5.png"}})

마운트 디렉토리에 lost+found 디렉토리 확인



### 6. 재부팅시 자동 마운트를 위한 설정

```sh
blkid
```

![cpm-6.png]({{ "/assets/img/contents/cpm-6.png"}})

하드디스크 UUID 확인

```sh
vim /etc/fstab
```

![cpm-7.png]({{ "/assets/img/contents/cpm-7.png"}})

```sh
UUID={UUID} {마운트 디렉토리} {파티션 포맷} defaults 0 0 추가
```

