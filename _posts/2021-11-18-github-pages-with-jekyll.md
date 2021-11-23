---
layout: post
title: Jekyll과 Github pages를 사용하여 블로그 만들기
feature-img: assets/img/titles/github-pages.png
thumbnail: assets/img/titles/github-pages.png
author: csupreme19
tags: [Blog, Github, Github pages, Jekyll]

---

# Jekyll과 Github pages를 사용하여 간단하게 블로그를 만들어보자



![github-pages.png]({{ "/assets/img/titles/github-pages.png"}})

[GitHub Pages](https://pages.github.com/)

본 문서는 현재 보고 있는 블로그를 구현하였던 내용을 담고 있다.

---

## 시작하게된 계기

문득 개발을 하고 있는데 **왜 남는게 없지?** 라는 생각이 들기 시작하였다.

요즘엔 양질의 개발자료를 손가락 몇번 놀리면 쉽게 찾을 수 있다보니 정보를 검증도 하지 않고 여과없이 받아들일 수도 있겠다는 생각이 들었고 정보를 내것으로 만들어야겠다는 필요성이 느껴졌다.

문서를 작성하면서 헷갈리는 부분도 정리할 수 있고 무엇보다 남들에게 설명하도록 나중에 내가 봐도 이해할 수 있도록 작성하려면 확실한 이해가 바탕이 되어야 하기 때문에 시작하게 되었다.

---

## 어떤걸 써야할까

웹개발이라는 것은 익숙하지만 결코 간단한 것이 아니다. (자세히 말하면 public하게 오픈하는 것)

1. JS, CSS 등의 Frontend 스택을 활용하여 프론트 웹을 개발한다.
2. 웹에서 사용할 REST API를 정의 및 개발하고 경우에 따라 DB를 구축한다.
3. Cloud 환경이든 자체 구축 서버든 웹을 빌드하여 올릴 서버가 필요하다.
4. 도메인, 인증서 구매 및 적용을 통해 외부에 오픈이 필요하다.
5. 추가적으로) 최소한의 보안과 무중단 배포를 위한 proxy 서버가 필요

##### 나는

1. 간단한 기술 블로그를 위하여 웹개발을 하기는 싫었다.
2. 자체 서버를 구축하기 싫었다. (비용적으로든 노력으로든)
3. 추가적인 비용을 들이기 싫었다.



따라서 자체 호스팅을 하는 사이트를 이용하는 방법을 선택하였다.

---

## 블로그 플랫폼

조사 결과 여러가지 블로그 플랫폼 서비스들이 존재하였다.

1. [https://velog.io](https://velog.io/)

   - 개발자를 위한 블로그 서비스(플랫폼)

   - 마크다운 작성 가능

2. [https://wordpress.org](https://wordpress.org/)

   - 전세계적으로 가장 유명한 블로그 서비스

   - 커스터마이징 기능이 강력

   - 방대한 생태계

3. [https://www.tistory.com](https://www.tistory.com/)

   - 유명한 블로그 플랫폼

   - WYSIWYG 방식의 에디터를 제공

   - 특정 환경에서 접속이 매우 느림

4. [https://blog.naver.com](https://blog.naver.com)

   - 유명한 블로그 플랫폼2

   - WYSIWYG 방식의 에디터를 제공

   - 구글 검색이 안되는 치명적인 단점이 있음

   - 개발 한정으로 매우 기능이 부실

5. [https://pages.github.com](https://pages.github.com/)

   - github과 연동됨

   - markdown 지원

   - 퍼블릭 도메인 간편하게 설정 가능

### 결론은?

웹 개발에 공을 들이기는 싫었고 현재 마크다운으로 작성되어 있는 문서들을 쉽게 올릴 수 있어야 하였고 github과 연동되는 Github pages를 사용하기로 하였다.

---

## Jekyll

![jekyll.png]({{ "/assets/img/contents/jekyll.png" }})

[https://jekyllrb.com/](https://jekyllrb.com/)

GitHub 공동 설립자 Tom Preston-Werner에 의해 개발된 Ruby 기반의 정적 사이트 생성기(Static site generator)이다.

여기서 SSG(Static Site Generator)란 DB없이 static file 즉 html만으로 돌아가는 웹을 의미한다.

마크다운 형태로 작성이 가능하여 개발자 freindly하며 구현도 매우 간편하게 할 수 있다.

Github pages와 궁합이 매우 좋으며 Github pages 공식 문서에서도 Jekyll을 이용하도록 안내하고 있다.

---

## Github pages

### 1. 저장소 생성

![gpwj-1.png]({{ "/assets/img/contents/gpwj-1.png" }})

Github pages는 계정의 github repository를 기반으로 웹을 제공한다.

`{사용자명}.github.io` 의 형태로 저장소를 생성하면 해당 저장소는 자동으로 github pages 저장소로 설정이 된다.

(올바른 branch에 커밋시 자동으로 github pages를 빌드하도록 설정이 되어있다.)

> 사진은 이미 저장소를 만들었기 때문에 중복되었다고 나옴

### 2. git 설정

```sh
# 유저명은 csupreme19로 가정
$ mkdir -p ~/git/csupreme19.github.io
$ cd ~/git/csupreme19.github.io
# git 사용자 설정
$ git config --local user.name csupreme19
$ git config --local user.email csupreme19@gmail.com

# git 로컬 저장소 초기화
$ git init

# git remote 저장소 연결
$ git remote add origin git@github.com:csupreme19/csupreme19.github.io.git
$ git remote -v
origin	git@github.com:csupreme19/csupreme19.github.io.git (fetch)
origin	git@github.com:csupreme19/csupreme19.github.io.git (push)
```

> 나는 사내 gitlab의 계정이 global로 설정되어 있어 해당 git 저장소에만 config를 적용하도록 `--local` flag를 사용하였다.

### 3. 테마 설정

[http://jekyllthemes.org/](http://jekyllthemes.org/)

위 사이트에서 마음에 드는 테마를 선택 후 github에서 fork 또는 clone한다.

> jekyll을 설치 하는 것이 원래 해야 할 일이지만 보통 테마에서 jekyll gemspec 명세를 제공하므로 jekyll 설치를 뒤로하고 테마 설정을 진행한다.

![type-on-strap.png]({{ "/assets/img/contents/type-on-strap.png" }})



본인은 [Type on Strap](http://jekyllthemes.org/themes/Type-on-Strap/) 테마를 선택하였다.



#### 선택 이유?

1. 현재까지도 유지보수가 되고 있는 점(선택 시점에 10시간 전에 릴리즈된 것을 확인)
2. 컨텐츠 중심의 가독성 좋은 테마
3. 반응형 웹
4. 다크 테마 지원
5. mermaid, katex 등 다이어그램, 수식 툴 지원 



단점이라면 한글 폰트가 생각보다 크게 보인다는 것인데... 이후 폰트 및 사이즈 변경 예정이다.

---

### Jekyll 설치 및 설정  

#### 선행사항

- Ruby 2.5.0 이상
- RubyGems
- GCC
- Make



#### Ruby 설치(macOS)

```sh
# Homebrew 설치
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Ruby 설치
$ brew install ruby
# zsh 사용시
echo 'export PATH="/usr/local/opt/ruby/bin:/usr/local/lib/ruby/gems/3.0.0/bin:$PATH"' >> ~/.zshrc
# bash 사용시
echo 'export PATH="/usr/local/opt/ruby/bin:/usr/local/lib/ruby/gems/3.0.0/bin:$PATH"' >> ~/.bash_profile

# 버전 확인
$ ruby -v
$ gem -v
$ gcc -v
```

다른 OS 설치 방법은 [Jekyll 설치 문서](https://jekyllrb.com/docs/installation/) 참조 



#### Jekyll 및 테마 설치

```sh
$ cd ~/git/csupreme19.github.io
$ git clone https://github.com/Sylhare/Type-on-Strap.git

# install
$ gem install jekyll
$ bundle add webrick
$ bundle install

# jekyll 로컬 서버 실행
$ bundle exec jekyll serve
Configuration file: /Users/csupreme19/git/csupreme19.github.io/_config.yml
            Source: /Users/csupreme19/git/csupreme19.github.io
       Destination: /Users/csupreme19/git/csupreme19.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 1.461 seconds.
 Auto-regeneration: enabled for '/Users/csupreme19/git/csupreme19.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```



#### _config.yml 수정

```yaml
# SITE CONFIGURATION
baseurl: ""		# 설정한 url이 subdomain이 된다. 즉 {url}/{baseurl} 도메인으로 접속해야함
url: "https://csupreme19.github.io"		# 도메인 주소 설정

# THEME-SPECIFIC CONFIGURATION
title: 내 블로그                             # 타이틀
description: "한글테스트"      # 구글 검색 엔진에서 사용하는 정보
avatar: assets/img/triangle.png                         # 상단 navbar 이미지
favicon: assets/favicon.ico                             # 웹 favicon

# Header and footer text
header_text: 헤더 텍스트  # 블로그 헤더 텍스트
header_feature_image: assets/img/pexels/triangular.jpeg # 헤더 이미지
# 푸터 텍스트
footer_text: >
  Powered by <a href="https://jekyllrb.com/">Jekyll</a> with <a href="https://github.com/sylhare/Type-on-Strap">Type on Strap</a>
  
...
# 나머지 설정은 github 문서 참조
```



### 1. 로컬 접속 테스트

![gpwj-2.png]({{ "/assets/img/contents/gpwj-2.png" }})

http://localhost:4000 접속 확인



### 2. git push 및 github pages 설정

```sh
$ git commit -a -m "initial commit"
$ git push -u origin main
```



![gpwj-3.png]({{ "/assets/img/contents/gpwj-3.png" }})

github 저장소 > Settings > Pages



##### 빌드 소스 변경

Source 부분 main 브랜치, / (root) 폴더로 변경

변경 후 `Your site is published at https://csupreme19.github.io/` 빌드 성공 메시지 확인



##### 접속

[https://csupreme19.github.io/](https://csupreme19.github.io/) 접속 확인

---

### 댓글창 활성화

Type-on-Strap 테마는 3가지의 Comment 오픈소스를 지원한다.



#### 1. Disqus

[disqus.com](https://disqus.com/)

장점

- 로그인 안해도 댓글 달 수 있음

단점

- 무겁다, 무료버전은 광고가 존재
- 개인적인 의견이지만 댓글창이 너저분해보인다.

#### 2. Cusdis

[cusdis.com](https://cusdis.com/)

장점

- Disqus에 비해 매우 깔끔한 레이아웃

단점

- 중국발이라 왠지 모를 거부감
- disqus와 마찬가지로 무겁다.

#### 3. Utterance

[utterance.es](https://utteranc.es/)

장점

- 매우 깔끔한 레이아웃
- 성능이 위 2 오픈소스에 비해 좋다.
- 완전 무료 오픈소스로 광고 없음

단점

- 댓글이 repo에 GitHub 이슈로 등록되는 구조라 GitHub 계정이 있어야만 댓글 가능



Utterance를 사용하기로 하였다.

일반적인 블로그가 아니라 GitHub Pages로 운영되는 GitHub 기반 블로그이며 주로 개발 내용을 다루기 때문에 GitHub 계정이 필요한 것은 큰 단점으로 다가오지 않았다.

또한 GitHub Issue로 등록되므로 Webhook을 등록하여 Slack 알람을 받는등 Alert 기능도 활성화 가능하다고 생각하였다.

---

### Utterance 적용하기

![utterance.png]({{ "/assets/img/contents/utterance.png" }})

#### 1. Public Repo 생성

Github pages repo가 public이므로 해당 repo 사용

#### 2. Utterance App 설치

[https://github.com/apps/utterances](https://github.com/apps/utterances)에서 설치

#### 3. _config.yml 설정

```yaml
# Comments
comments:
  utterances:
    repo: csupreme19/csupreme19.github.io
    issue-term: comment
```

위 두가지 설정만 하면 설정은 끝난다.

#### 4. 코멘트 적용 확인

```sh
$ bundle exec jekyll serve
```



![gpwj-4.png]({{ "/assets/img/contents/gpwj-4.png" }})

---

### 폰트 변경

맥 환경에서 봤을땐 폰트가 사이즈 말고는 괜찮았는데 윈도우 환경에서 보니 계단 현상이 존재하고 가독성이 떨어져 보이는 문제가 발생하였다.

테마(템플릿)를 사용하는 이유가 UI 개발에 힘을 쏟기 싫었기 때문인데 어쩔 수 없이 입맛에 맞는 커스텀은 필요한 것 같다.



#### 1. 폰트 .scss 파일 생성

일반 폰트는 케이티의 [Y 너만을 비춤체](https://noonnu.cc/font_page/500), 소스 코드 폰트는 네이버의 [D2 coding ligature](https://github.com/naver/d2codingfont)를 사용하였다.

```sh
$ cd _sass/external
$ vim _y-spotlight.scss
$ vim _d2-coding-ligature.scss
```

폰트 배포시 별도의 `@font-face` 소스를 제공한다.

없다면 기본 구성되어있는 `_source-sans-pro.scss` 파일을 복사하여 사용하자.

```scss
// _y-spotlight.scss
@font-face {
    font-family: 'Y_Spotlight';
    src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts-20-12@1.0/Y_Spotlight.woff') format('woff');
    font-weight: normal;
    font-style: normal;
}

// _d2-coding-ligature.scss
@font-face {
    font-family: 'D2 coding Ligature';
    src: url('https://cdn.jsdelivr.net/gh/everydayminder/assets/subset-D2Codingligature.woff') format('woff');
    font-weight: 400;
    font-style: normal;
}
```

폰트 소스는 cdn을 사용하였는데 D2 coding 폰트의 경우 공식으로 CDN을 제공하고 있지 않는 것 같다.

따라서 [everyminder](https://luran.me/515) 에서 제공한 woff CDN을 사용하였다.



#### 2. `_variables.scss` 수정

```sh
$ vim _sass/base/_variables.scss
```

`_variables.scss` 파일에 테마 변수 정보들이 담겨있다.

아래 부분처럼 위에서 추가했던 font-family를 추가한다.

```scss
/* TYPOGRAPHY */
$font-family-main: 'Y_Spotlight', 'Source Sans Pro', Helvetica, Arial, sans-serif;
$font-family-headings: 'Y_Spotlight', 'Source Sans Pro', Helvetica, Arial, sans-serif;
$font-family-logo: 'Y_Spotlight', 'Source Sans Pro', Helvetica, Arial, sans-serif;
$font-size: 0.875em;

$monospace: 'D2 coding ligature', Consolas, Menlo, Monaco, Lucida Console, Liberation Mono, DejaVu Sans Mono, Bitstream Vera Sans Mono, Courier New, monospace, sans-serif !default;
$font-size-code: 0.85em !default;
$font-height-code: 1.3em !default;
$border-radius: 4px !default;
```

하는 김에 글씨 크기가 커서 폰트 사이즈도 조정해 주었다.



#### 3. 확인

![gpwj-5.png]({{ "/assets/img/contents/gpwj-5.png" }})

폰트 적용이 된 것을 확인할 수 있다.

---

### Reference

1. [Jekyll Installation](https://jekyllrb.com/docs/installation/)
2. [GitHub Pages](https://pages.github.com/)
3. [Jekyll Themes](http://jekyllthemes.org/)
4. [Type-on-Strap](https://github.com/Sylhare/Type-on-Strap)

