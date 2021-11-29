---
layout: post
title: Jekyll Liquid 문법 빠져나오기(중괄호 무시)
feature-img: assets/img/contents/jekyll.png
thumbnail: assets/img/contents/jel-1.png
author: csupreme19
tags: [Blog, Jekyll, Liquid, Markdown, Template]

---

# Jekyll Liquid 문법 빠져나오기(중괄호 무시)

![jekyll.png]({{ "/assets/img/contents/jekyll.png"}})

[Jekyll Liquid](https://jekyllrb.com/docs/liquid/)

Jekyll 작성시 Liquid template `{{`, `}}` 이중 중괄호 및 중괄호 빠져나오기 (무시하기)

---

## Escape Liquid

Jekyll은 내부 템플릿 언어로 [Liquid](https://shopify.github.io/liquid/)를 사용한다.

Liquid는 Ruby로 작성된 템플릿 언어로

```sh
{% raw %}
{{ variable }}
{{ if sentence }}
{% endraw %}
```

등의 중괄호를 이용한 문법을 통하여 템플릿 변수나 로직을 실행한다.

문서를 작성하다 보니 아래와 같은 에러를 발견하게 되었다.

![jel-1.png]({{ "/assets/img/contents/jel-1.png"}})

이는 kubernetes에서 사용하는 go-template이 똑같은 이중괄호를 사용해서 나타난 문제였다.

이를 해결하기 위해서는 아래와 같이 `{{ "{% raw" }} %}`, `{{ "{% endraw" }} %}`  사이에 문법을 적어주면 된다.

#### Raw

```text
{{ "{% raw" }} %}
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/monitoring-user -o jsonpath="{.secrets[0].name}") -o {% raw %}go-template="{{.data.token | base64decode}}"{% endraw %}
eyJhbGciOiJSUzI1NiIsImtpZCI6IlVmM2...
{{ "{% endraw" }} %}
```

#### Template processed

```sh
{% raw %}
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/monitoring-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
eyJhbGciOiJSUzI1NiIsImtpZCI6IlVmM2...
{% endraw %}
```

---

## Reference

1. [Jekyll Liquid](https://jekyllrb.com/docs/liquid/)
2. [Liquid Template](https://shopify.github.io/liquid/tags/template/)

