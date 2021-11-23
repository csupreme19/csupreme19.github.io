---
layout: post
title: Let's Encrypt 인증서 발급 및 갱신
feature-img: assets/img/titles/letsencrypt-horizontal.png
thumbnail: assets/img/titles/letsencrypt-horizontal.png
author: csupreme19
tags: [Certificate, Certbot, Domain, DNS]

---

# Let's Encrypt 인증서 발급 및 갱신

![letsencrypt-horizontal.png]({{ "/assets/img/titles/letsencrypt-horizontal.png"}})

[Let's Encrypt Docs](https://letsencrypt.org/docs/)

본 문서는 무료 SSL 인증서를 제공하는 Let's Encrypt를 사용하여 인증서 발급 및 갱신을 하는 방법을 정리하였다.

---
## 사전 준비 사항

1. SSL 인증서 적용을 위해서는 도메인 구매 필수
2. 구입한 도메인과 웹서버 연결: 가비아와 같은 도메인 구매 기관에서 설정 필요
3. root 유저 상태로 가정하고 진행

인증서 발급 방법은 수동으로 certbot 설치 후 발급 진행 하는 방법과 docker image로 발급하는 방법이 있다.

---
## docker image로 인증서 발급 방법
docker volume path의 mount 지점에 인증서가 생성된다.
```sh
$ docker run -it --rm --name certbot -v '/etc/letsencrypt:/etc/letsencrypt' -v '/var/lib/letsencrypt:/var/lib/letsencrypt' certbot/certbot certonly -d *.yourdomain.com --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

## certbot 설치 후 발급 방법

> Ubuntu 20.04 LTS 대상 certbot 설치 시 ppa repo는 deprecated 됨.
>
> > nginx + ubuntu 20.04 환경 시 certbot 설치 매뉴얼 참조하여 설치 진행 참고: https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx
>

Ubuntu 20.04 환경에서 certbot 설치
```sh
# 필요 시 snapd package 설치. 
# ubuntu 16.04 부터 기본 설치 되어 있음.(https://snapcraft.io/docs/installing-snap-on-ubuntu)

# snap 최신으로 업데이트
$ sudo snap install core; sudo snap refresh core

# 기존 certbot 설치되어 있으면 삭제
$ sudo apt-get remove certbot

# certbot 설치
$ sudo snap install --classic certbot

# certbot 실행파일 link 설정
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot

# certbot 버전 확인
$ certbot --version
```

Ubuntu
```sh
$ apt-get update
$ apt-get install software-properties-common
$ add-apt-repository ppa:certbot/certbot
$ apt-get update
$ apt-get install certbot
```

CentOS
```sh
$ sudo yum install epel-release -y
$ sudo yum install certbot -y
```

Option) certbot 웹서비스 플러그인 설치

```sh
# apach 인 경우
$ apt-get install python-certbot-apach

# nginx 인 경우
$ apt-get install python-certbot-nginx
```

---
## 인증서 발급

- webroot, standalone, dns의 3가지 방식 중 dns 방식으로 설치 진행
- DNS 방식 인증서는 와일드카드 url 지원하므로 *.yourdomain.com 형식으로 발급 요청
- certonly 옵션으로 인증서만 발급 → 웹 서비스의 설정 파일 자동으로 변경하지 않음
- standalone 방식으로 발급 시 web serivce를 잠시 중단해야 함
- 하루 동안 3번만 발급 가능하므로 발급 시 주의 필요

### manual 옵션으로 dns 방식 지정

```sh
# dns 방식 인증서 발급 
$ certbot certonly --manual --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -m csupreme19@gmail.com -d *.yourdomain.com
```

#### DNS TXT 값 표시되는 화면에서 엔터 눌러 진행하지 않고 대기

dns 방식 인증서 발급 command 입력 후 더 이상 진행하지 않고 **대기**한 후 아래 도메인 설정 등을 진행한다.

![lr-1.png]({{ "/assets/img/contents/lr-1.png"}})

#### **DNS TXT record 값 설정**

도메인 구입한 곳(가비아 등)의 DNS 관리메뉴로 진입하여 dns txt record 값을 설정.

예를 들어 가비아의 경우 아래 그림과 같이 'DNS 레코드 추가' 메뉴에서 TXT 레코드를 추가 가능

![lr-2.png]({{ "/assets/img/contents/lr-2.png"}})

호스트 컬럼에는 **_acme-challenge** 만 입력

가비아의 경우 설정한 값 예시는 아래 이미지 참조. record 값 앞/뒤로 쌍따옴표 추가. 

![lr-3.png]({{ "/assets/img/contents/lr-3.png"}})

해당 record 설정 후 저장하여 반영. 아래 command로 도메인 서버에 레코드가 반영 되었는지 확인 가능.

#### **도메인 서버 레코드 확인**

```sh
$ nslookup -type=txt _acme-challenge.yourdomain.com
# 또는
$ dig +noall +answer _acme-challenge.yourdomain.com txt
```

정상 반영 시 아래 이미지와 같이 확인 가능. 

![lr-4.png]({{ "/assets/img/contents/lr-4.png"}})

이후 앞 과정의 dns 방식 인증서 발급 대기 화면에서 enter를 눌러 진행

![lr-5.png]({{ "/assets/img/contents/lr-5.png"}})

#### 인증서 저장 위치

인증서 생성 후 저장되는 위치:

  - Certificate is saved at: /etc/letsencrypt/live/{domain name}/fullchain.pem
  - Key is saved at:         /etc/letsencrypt/live/{domain name}/privkey.pem

#### 인증서 확인

```sh
$ certbot-auto certificates
```

#### 인증서 삭제

```sh
$ certbot-auto delete
```

---
## 인증서 갱신

Let's Encrypt의 경우 무료이지만 인증서 유효 기간이 3개월로 짧다. 

인증서 만료 전 반드시 갱신을 해야만 site에 접속이 안되는 현상을 방지할 수 있다.

google calendar에 3개월 주기로 알람을 등록하여 인증서 갱신 관리는 하는 것을 추천.

또한 인증서를 manual 로 생성했을 경우 certbot 자동 갱신 명령어로는 갱신 되지 않는다.

이 경우 아래 명령어로 갱신가능

```sh
$ certbot --server https://acme-v02.api.letsencrypt.org/directory -d "*.ccpinfra.xyz" --manual --preferred-challenges dns-01 certonly
```

명령어 입력 후 나오는 DNS TXT record 값을 도메인 구입처(예를 들어 가비아 등)에서 DNS 설정에 들어가 업데이트 후 진행한다.

> 인증서를 갱신하여 새로운 인증서를 받아도 기존 인증서는 만료되지 않는다.

---
### 인증서 갱신 후 서비스별 적용 가이드

기본적으로 NAS에 인증서 정보를 저장하고 각 서비스에 가져다 사용한다.

본 예제에서는 인증서를 `/nas/volumes/ssl`에서 관리

- k8s의 경우 kubernetes secret 형태로 떠져 있음
- 그 외 docker로 직접 올라가있는 경우 각 서비스별 인증서 정보 적용

위에서 발급받은 인증서파일을 관리하는 인증서 폴더로 옮긴다.

```shell
# 인증서 복사
$ cp /etc/letsencrypt/live/yourdomain.com-0001/fullchain.pem /nas/volumes/ssl
$ cp /etc/letsencrypt/live/yourdomain.com-0001/privkey.pem

# 인증서 포맷 변환
$ openssl x509 -inform PEM -in fullchain.pem -out yourdomain.com.crt
$ openssl rsa -in privkey.pem -text -text > yourdomain.com.key
```

> 기존의 인증서는 백업할 것

---
## Troubleshooting

### 1. NET::ERR_CERT_COMMON_NAME_INVALID

인증서 갱신 및 발급 후에 인증서 적용이 성공적으로 되었으나 `NET::ERR_CERT_COMMON_NAME_INVALID`로 인증서가 유효하지 않음

#### 인증서의 주소와 인증서가 적용된 도메인의 주소가 일치하지 않을 때 발생하는 오류

1. 인증서의 도메인 주소가 제대로 적용되었는지 확인한다.
2. `*` wildcard가 포함되어 있는지 확인한다.
3. 도메인 주소가 정상인 경우 전세계의 DNS 서버에 적용이 바로 되지 않고 시간이 어느정도 소요되어서 발생하는 문제로 기다리면 해결된다.

#### 도메인 서버 레코드 확인

```sh
$ nslookup -type=txt _acme-challenge.subdomain.yourdomain.com
# 또는
$ dig +noall +answer _acme-challenge.sbubdomain.yourdomain.com txt
```

`Authoritative answers can be found from:`에 리스트로 도메인 서버 정보가 나오면 등록이 완료된 것


---

## Reference

1. [Let's Encrypt Docs](https://letsencrypt.org/docs/)
2. [Certbot](https://certbot.eff.org/lets-encrypt)
3. [Secret](https://kubernetes.io/docs/concepts/configuration/secret/)
4. [Certified Kubernetes Administrator(CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)