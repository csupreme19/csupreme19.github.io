---
layout: post
title: CentOS 파티션 생성 및 마운트
feature-img: assets/img/titles/centos-logo.jpeg
thumbnail: assets/img/contents/cpm-1.png
author: csupreme19
categories: System Linux
tags: [Centos, RHEL, Linux, OS, Partition, Mount]

---

# CentOS 파티션 생성 및 마운트 방법

### 1. 디스크 정보 확인

```sh
fdisk -l
```

![cpm-1.png]({{ "/assets/img/contents/cpm-1.png"}})

<br>

### 2. 파티션 구성하기

```sh
fdisk /dev/sda
```

![cpm-2.png]({{ "/assets/img/contents/cpm-2.png"}})

별도의 파티션을 구성하는 것이 아니라면 기본값으로 사용해요.

Partition type: p

Partition number: 1 (시스템에 따라 다를 수 있음)

First sector: 2048 (시스템에 따라 다를 수 있음)

Last sector: 4294967294 (시스템에 따라 다를 수 있음)

p 명령어로 파티션 설정 확인 후 

w 명령어로 저장하고 fdisk 나와요.

<br>

### 3. 파티션 포맷하기

```sh
mkfs.ext4 /dev/sda
```

![cpm-3.png]({{ "/assets/img/contents/cpm-3.png"}})

위에서 구성한 파티션을 ext4 포맷을 사용하여 포맷해요.

<br>

### 4. 파티션 마운트하기

```sh
mount /dev/sda /data
```

/dev/sda 파티션을 /data에 마운트해요.

<br>

### 5. 마운트 확인

```sh
df -h
mount -a
```

![cpm-4.png]({{ "/assets/img/contents/cpm-4.png"}})

![cpm-5.png]({{ "/assets/img/contents/cpm-5.png"}})

마운트 디렉토리에 lost+found 디렉토리 여부 확인하여 정상인지 확인해요.

<br>

### 6. 재부팅시 자동 마운트를 위한 설정

```sh
blkid
```

![cpm-6.png]({{ "/assets/img/contents/cpm-6.png"}})

하드디스크 UUID를 확인해요.

```sh
vim /etc/fstab
```

![cpm-7.png]({{ "/assets/img/contents/cpm-7.png"}})

```sh
UUID={UUID} {마운트 디렉토리} {파티션 포맷} defaults 0 0 추가
```

