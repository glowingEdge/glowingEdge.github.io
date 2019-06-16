---
layout: post
title:  "Django Bookmark Project 4"
date:   2019-05-27
excerpt: "템플릿 확장과 정적 파일"
tag:
- python
- django
- bookmark
- pythonanywhere
comments: true
---

> **책 '배프의 오지랖 파이썬 웹 프로그래밍'을 공부한 내용을 정리 및 추가**<br>
> **<a href="http://glowingedge.pythonanywhere.com/bookmark/">완성된 홈페이지</a>**<br>
> **<a href="https://github.com/glowingEdge/bookmark">완성된 소스코드</a>**

## 디자인 입히기

홈페이지 어딜가나 떠있는 GNB(Global Navigation Bar)를 구현하려면, 가장 단순무식하게는 페이지 수만큼 같은 내용을 복사+붙여넣기 하는 식일 것이다. 이를 줄이기위해 Django에서는 **템플릿 확장**이라는 방법을 제공한다.<br>
기준이 되는 템플릿을 만들고, 이를 상속받아 사용하는 방식이다.

### base.html

어느 페이지에나 공통으로 들어가는 템플릿 페이지는 다음처럼 사용한다.<br>
{% raw %}`{% block 'XXX' %}`{% endraw %} 와 같은 형식임에 주의하자.

{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    {% block content %}

    {% endblock %}
</body>
</html>
{% endraw %}
{% endhighlight %}

### 템플릿 확장

확장할 템플릿의 사용법 또한 간단하다. {% raw %}`{% extends 'base.html' %}`{% endraw %}로 상속을 받고,
{% raw %}`{% block XXX %}`{% endraw %} 과 {% raw %}`{% endblock %}`{% endraw %} 사이에 내용을 넣어주면 된다.

{% highlight html %}
{% raw %}
{% extends 'base.html' %}

{% block title %}Detail{% endblock %}

{% block content %}
    {{object.site_name}}<br/>
    {{object.url}}
{% endblock %}
{% endraw %}
{% endhighlight %}

## 정적 파일(Static Files) 사용하기

정적 파일이란 로컬 서버에 있는 파일을 의미한다. css, js, 사진 파일 등이 될 것이다.<br>
프로젝트 루트에 static 폴더를 만들어 관리하는 법을 배워보자.<br>

### 프로젝트 루트에 static 폴더 생성

### settings.py 수정

{% highlight python %}
// settings.py
#...
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
#...
{% endhighlight %}

### style.css 생성

### base.html 에서 정적파일 불러오기

사용 방식(문법)을 기억하자. 아래 코드를 넣은 후 서버를 실행하여, 크롬의 개발자 도구(`F12`)에서 `Sources`탭에서 statc-style.css가 보인다면 성공이다.

{% highlight html %}
{% raw %}
<head>
    {% load static %}
    <link rel="stylesheet" href="{% static 'style.css' %}">
    // ...
</head>
{% endraw %}
{% endhighlight %}

