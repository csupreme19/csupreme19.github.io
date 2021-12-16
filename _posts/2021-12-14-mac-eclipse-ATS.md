---
layout: post
title: Mac Eclipse ATS 오류 해결방법
feature-img: assets/img/titles/eclipse-logo.png
thumbnail: assets/img/contents/mea-1.png
author: csupreme19
tags: [Mac, ATS, iOS, Eclipse]

---

# Mac Eclipse ATS 오류 원인 및 해결방법

![eclipse-logo.png]({{ "/assets/img/titles/eclipse-logo.png"}})

[https://bugs.eclipse.org/bugs/show_bug.cgi?id=574611](https://bugs.eclipse.org/bugs/show_bug.cgi?id=574611)

맥에서 이클립스 Application Transport Security 오류 해결 방법에 대하여 정리하였다.

---

## 문제점

![mea-1.png]({{ "/assets/img/contents/mea-1.png"}})

이클립스에서 Spring MVC를 이용해 개발중 위와 같은 문제가 발생하였다.

Eclipse 내부 브라우저 사용시 위와 같은 에러 메시지가 발생하며 접근이 거부 되었다.

---

## 원인 파악

조금의 구글링을 해보니 iOS, Mac 환경에서 XCode 앱의 경우 ATS(Application Transport Security) 기능이 활성화 되어 있으며

해당 정책으로 인하여 insecure한 리소스로의 접근을 기본적으로 차단하도록 설정되어 있다.

http로의 접근을 차단하고 https로의 접근만 허용한다.

>[Preventing Insecure Network Connections](https://developer.apple.com/documentation/security/preventing_insecure_network_connections)
>
>iOS 9.0, macOS 10.11 SDK 이상 부터 활성화 된 기능이며 Eclipse 2021-06 부터 해당 SDK를 사용한 것으로 보임

![mea-2.png]({{ "/assets/img/contents/mea-2.png"}})

현재 사용중인 Eclipse 2021-09는 macOS 10.14 SDK를 이용하여 개발된 애플리케이션으로 해당 기능이 활성화 되어 있다.

취지는 좋으나 로컬 개발 환경 및 테스트 용도로는 http를 사용하는 경우가 많으므로 https 사용을 강제하는 것은 개발자로서 매우 거슬리는 일이다.

해당 기능을 비활성화하거나 우회 방법을 찾아보자.

---

## 해결 방법

### 1. Eclipse 패키지 info.plist 수정

Eclipse 앱 패키지 안에 있는 패키지 정보인 info.plist를 수정하여 ATS 기능을 비활성화 할 수 있다.

`/Applications/Eclipse.app/Contents/info.plist`

#### HTTP 모두 허용

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<plist version="1.0">
  
  <dict>
    <!-- 추가 -->
    <key>NSAppTransportSecurity</key>
    <dict>
      <key>NSAllowsArbitraryLoads</key>
      <true/>
    </dict>
  
    <key>NSRequiresAquaSystemAppearance</key>
    ...
  </dict>
  
</plist>

```

해당 앱에서 모든 Http 요청을 허용한다. 현재 로컬호스트로의 접근만 허용하면 되므로 추천하는 방법은 아니다.

<br>

#### HTTP 도메인 허용

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<plist version="1.0">
  
  <dict>
    <!-- 추가 -->
    <key>NSAppTransportSecurity</key>
    <dict>
      <key>NSExceptionDomains</key> 
      <dict>
        <key>localhost</key> 
        <dict> 
          <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key> 
          <true/>
          <key>NSIncludesSubdomains</key>
          <true/>
        </dict>
      </dict>
    </dict>
  
    <key>NSRequiresAquaSystemAppearance</key>
    ...
  </dict>
  
</plist>
```

localhost 도메인과 서브도메인에 대하여 HTTP 요청을 일시적으로 허용한다.



![mea-3.png]({{ "/assets/img/contents/mea-3.png"}})

설정 이후 Eclipse 앱을 완전히 종료 후 재실행하면 해결이 되는 모습을 볼 수 있다.



참고) ATS 설정 관련 파라미터는 다음과 같다.

```json
NSAppTransportSecurity : Dictionary {
    NSAllowsArbitraryLoads : Boolean
    NSAllowsArbitraryLoadsForMedia : Boolean
    NSAllowsArbitraryLoadsInWebContent : Boolean
    NSAllowsLocalNetworking : Boolean
    NSExceptionDomains : Dictionary {
        <domain-name-string> : Dictionary {
            NSIncludesSubdomains : Boolean
            NSExceptionAllowsInsecureHTTPLoads : Boolean
            NSExceptionMinimumTLSVersion : String
            NSExceptionRequiresForwardSecrecy : Boolean
            NSRequiresCertificateTransparency : Boolean
        }
    }
}
```

> 항목 별 자세한 내용은 [Preventing Insecure Network Connections](https://developer.apple.com/documentation/security/preventing_insecure_network_connections) 참고

<br>

### 2. 외부 웹 브라우저 사용

![mea-4.png]({{ "/assets/img/contents/mea-4.png"}})

Eclipse 자체 웹 브라우저를 사용할 이유가 없다.

이클립스 설정에서 외부 웹 브라우저(크롬 등)을 사용하도록 설정해준다.



![mea-5.png]({{ "/assets/img/contents/mea-5.png"}})

설정 이후 외부 브라우저에서 http 요청이 정상적인 것을 확인할 수 있다.



둘 중 편한 방법으로 하면 되지만 개인적으로는 2번을 추천한다.

1번 방법은 배포 패키지를 직접 수정한다는 거부감이 있으며 보안상으로도 best practice는 절대 될 수 없다.

2번 방법은 외부 웹 브라우저를 사용하면 개발자 모드나 다른 확장 프로그램을 사용해 디버그하기 쉬우며 반응성 웹이나 유저 에이전트를 테스트하기에도 용이하다

---

## Reference

1. [https://bugs.eclipse.org/bugs/show_bug.cgi?id=574611](https://bugs.eclipse.org/bugs/show_bug.cgi?id=574611)
2. [Preventing Insecure Network Connections](https://developer.apple.com/documentation/security/preventing_insecure_network_connections)

