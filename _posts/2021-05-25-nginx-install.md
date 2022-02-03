---
layout: post
title: Nginx Proxy 설치하기
feature-img: assets/img/titles/nginx-logo.svg
thumbnail: assets/img/titles/nginx-logo.svg
author: csupreme19
categories: OSS Nginx
tags: [Nginx, Proxy]

---

# Nginx Proxy 설치하기

![nginx-logo.svg]({{ "/assets/img/titles/nginx-logo.svg"}})

[Installing NGINX Open Source](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/?_ga=2.177804005.1113104653.1621840207-1766246859.1621579363)

설치형 nginx 구축했던 내용을 정리해봤어요.

---
## nginx 설치

```shell
$ apt-get update 
$ apt-cache policy nginx # 설치 가능 버전 확인
$ apt-get install nginx -y
```

### 설치 확인

```shell
$ nginx -v # 버전 확인
nginx version: nginx/1.14.0 (Ubuntu)
$ curl -i -v localhost:80 # 접속 확인
* Rebuilt URL to: localhost:80/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 80 (#0)
...
<
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host localhost left intact

```

---
### nginx 명령어
```shell
$ nginx # nginx 구동
$ nginx -s quit # nginx 중지
$ nginx -s reload # 설정 리로드
$ nginx -s reopen # 로그 재시작
$ nginx -s stop # nginx 중지(fast shutdown)
```

위와 같이 binary 직접 실행하는 방법도 있으나 systemd로 관리하는것이 더 좋으므로 아래와 같이 실행을 권장해요.

```shell
$ systemctl status nginx.service	# 상태 확인
$ systemctl start nginx.service		# 구동
$ systemctl reload nginx.service	# 재설정
$ systemctl restart nginx.service	# 재시작
$ systemctl stop nginx.service		# 중지
```

실행 후 아래와 같이 config 파일 경로가 맞는지 확인했어요.

```shell
# -V는 configure 옵션 확인하는 플래그
$ /usr/sbin/nginx -V 2>&1 | grep --colour=auto conf
--conf-path=/etc/nginx/nginx.conf
```

---
## Troubleshooting

`/run/nginx.pid failed`가 나올때

`nginx /run/nginx.pid` 값에 심볼릭 링크가 제대로 걸리지 않은 경우

`nginx` 명령어 대신 `nginx -c /etc/nginx/nginx.conf` 사용하여 nginx 구동하면 해결할 수 있었어요.

```shell
$ nginx -c /etc/nginx/nginx.conf
$ nginx -s reload
```

> systemctl로 실행시 발생하는지는 확인이 필요해요.


---

## Reference

1. [Installing NGINX Open Source](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/?_ga=2.177804005.1113104653.1621840207-1766246859.1621579363)