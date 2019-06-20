---
layout: post
title:  "Django Dstagram Project 5"
date:   2019-06-20
excerpt: "Heroku에 배포 시 Secret key 감추기"
tag:
- python
- django
- dstagram
- aws s3
- heroku
- disqus
comments: true
---

> **책 '배프의 오지랖 파이썬 웹 프로그래밍'을 공부한 내용 정리 및 추가**<br>
> **<a href="https://dstagram-glowingedge.herokuapp.com/">완성된 홈페이지</a>**<br>
> **<a href="https://github.com/glowingEdge/dstagram">완성된 소스코드</a>**

책을 따라 기본적인 AWS S3(이미지 서버) 구성법과 Heroku에 배포하는 법을 배워볼 수 있었고, 관련 내용이 인터넷에 정리가 된 글이 참 많은 듯 하다.<br>
배포를 마치고 뿌듯해한지 단 하루만에, AWS쪽에서 AWS_SECRET_ACCESS_KEY가 Github repo에 유출되었다는 안내와 대처법을 보내주었다. 대체 어떻게 하루만에 알고 자동으로 메일까지 보냈을까 참 신기하다.<br>
신기함을 뒤로하고 요금폭탄을 피하는 방법을 찾아 이곳저곳, 이 방법 저 방법을 기웃거렸다.

## 환경 변수를 이용해 Heroku 에 배포하기

비밀키들을 숨기는 방법은 여러가지가 있겠지만, 이번에 공부한 방법은 환경변수를 이용한 방법이다.<br>
각종 언어를 설치할 때 잡아주던 그 환경변수다.<br>

### Django 환경변수 설정

Django에서는 `os.environ.get('KEY_NAME')`과 같은 방법으로 환경변수의 value값을 불러올 수가 있다.<br>
따라서 settings.py에서 아래와 같은 방법으로 소스코드에서 감출 수가 있다.

{% highlight python %}
# settings.py
...
SECRET_KEY = os.environ.get('SECRET_KEY')
AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
...
{% endhighlight %}

이렇게 설정한 뒤, Heroku에는 어떻게 배포할 수 있을까?<br>
처음엔 방법을 몰라 local git에서 github로 올리는 커밋과, heroku로 올리는 커밋을 분리해서 heroku에는 secret key가 포함된 걸 보내려 했었다.<br>
그러기엔 git은 너무 어려웠다..

### Heroku 환경변수 설정

내가 쓰고 있는 윈도우 cmd에서 `set` 커맨드로 환경 변수를 설정할 수 있듯이,<br>
Heroku에서는 `config`커맨드가 그 역할을 한다.<br>
이를 활용해 아래처럼 heroku에 환경변수를 설정해준다.

{% highlight python %}
heroku config   // 환경변수 목록 확인
heroku config:set SECRET_KEY=A1B2C3D4F4E5
heroku config:set AWS_ACCESS_KEY_ID=A1B2C3D4F4E5
...
{% endhighlight %}

**Heroku 공식사이트 참고:**: https://devcenter.heroku.com/articles/config-vars

## 마치며..

Heroku에 배포하는 방법을 AWS 유출사고(?)로 인해 공부할 기회가 생겨서 다행이다.<br>
AWS측에서 secret key에 유출에 대한 대처 메일을 상당히 친절하게 보내준 덕분에(비록 영어였지만) AWS 설정에 대해서도 많이 배우는 기회가 됐다.<br>