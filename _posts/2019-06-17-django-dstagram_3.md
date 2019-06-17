---
layout: post
title:  "Django Dstagram Project 3"
date:   2019-06-17
excerpt: "템플릿 작성"
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

## 템플릿 분리와 확장

각 템플릿을 작성하면서 `runserver`기능을 통해 동작하는 결과물을 확인하면서 진행한다.

### base.html

템플릿 확장을 위해 부트스트랩을 적용한 base.html을 만든다. 메뉴바와 content block, footer로 구성하고,<br>
템플릿 태그와 함께 user.is_authenticated 를 이용해 메뉴바에서 로그인 여부를 판단할 수 있도록 한다. <br>
템플릿 작성 후에는 템플릿이 검색되도록 settings.py에 경로를 추가한다.

{% highlight html %}
{% raw %}
// templates/base.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
    <title>Dstagram {% block title %}{% endblock %}</title>
</head>
<body>

<div class="container">
    <header class="header clearfix">
        <nav class="navbar navbar-expand-lg navbar-light bg-light">
            <a class="navbar-brand" href="/">Dstagram</a>
            <ul class="nav">
                <li class="nav-item"><a href="/" class="active nav-link ">Home</a></li>
                {% if user.is_authenticated %}
                <li class="nav-item"><a href="#" class="nav-link">Welcome, {{user.get_username}}</a></li>
                <li class="nav-item"><a href="{% url 'photo:photo_upload' %}" class="nav-link">Upload</a></li>
                <li class="nav-item"><a href="{% url 'logout' %}" class="nav-link">Logout</a></li>
                {% else %}
                <li class="nav-item"><a href="{% url 'login' %}" class="nav-link">Login</a></li>
                <li class="nav-item"><a href="{% url 'register' %}" class="nav-link">Signup</a></li>
                {% endif %}
            </ul>
        </nav>
    </header>
    {% block content %}
    {% endblock %}

    <footer class="footer">
        <p>&copy; 2019 glowingEdge. Powered By Django 2</p>
    </footer>
</div>


</body>
</html>
{% endraw %}

{% endhighlight %}

{% highlight python %}
# config/settings.py
...
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
...
{% endhighlight %}

### 목록 페이지 및 정적 파일 경로 추가

{% highlight html %}
{% raw %}
// templates/photo/list.html
{% extends 'base.html' %}

{% block title %}
- List
{% endblock %}

{% block content %}
    {% for post in photos %}
        <div class="row">
            <div class="col-md-2"></div>
            <div class="col-md-8 panel panel-default">
                <p><img src="{{post.photo.url}}" style="width:100%;"></p>
                <button type="button" class="btn btn-xs btn-info">
                    {{post.author.username}}</button>
                <p>{{post.text|linebreaksbr}}</p>
                <p class="text-right">
                    <a href="{% url 'photo:photo_detail' pk=post.id %}" class="btn btn-xs btn-success">댓글달기</a>
                </p>
            </div>
            <div class="col-md-2"></div>
        </div>

    {% endfor %}
{% endblock %}
{% endraw %}
{% endhighlight %}

for문을 이용해 뷰에서 전달되는 photos변수에서 각 항목을 처리한다. <br>
photo.url로 사진의 경로를 가져오고, author.username으로 작성자명을 가져온다.<br>
사진의 경로를 찾지못해 사진이 표시되지 않으므로, 이를 위한 url을 추가한다.

{% highlight python %}
# config/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path('', include('photo.urls')),
    path('accounts/', include('accounts.urls')),
    path('admin/', admin.site.urls),
]

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)    # shell에서 확인해보면 ^media/(?P<path>.*)$ 처럼 추가된다.
{% endhighlight %}

### 업로드 페이지와 form태그의 method속성

{% highlight html %}
{% raw %}
// templates/photo/upload.html
{% extends 'base.html' %}
{% block title %}
- Upload
{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-2"></div>
    <div class="col-md-8 panel panel-default">
        <form action="" method="post" enctype="multipart/form-data">
            {{form.as_p}}
            {% csrf_token %}
            <input type="submit" class="btn btn-primary" value="Upload">
        </form>
    </div>
    <div class="col-md-2"></div>
</div>
{% endblock %}
{% endraw %}
{% endhighlight %}

form태그의 enctype을 통해 서버에 어떤 인코딩으로 전달할지를 정할 수 있다. method가 post일 때만 사용 가능하다.<br><br>
application/x-www-form-urlencoded: 기본 옵션으로, 모든 문자열을 ASCII HEX값으로 인코딩해 전달하며 띄어쓰기는 +로 변환한다.<br>
multipart/form-data: 파일 업로드 때 사용하는 옵션으로 데이터를 문자열로 인코딩 하지않고 전달한다.<br>
text/plain: 띄어쓰기만 +로 인코딩하여 전달한다.<br>

### 디테일 페이지

상세 정보 페이지는 목록 화면의 형태에서 댓글달기 대신 수정과 삭제버튼으로 구성

### 업데이트 페이지

업데이트 페이지는 일부 텍스트를 제외하고 업로드 페이지와 동일하게 구성

### 삭제 페이지

 객체 정보와 함께 확인 메세지를 출력하고, 삭제 확인을 한다.

{% highlight html %}
{% raw %}
// templates/photo/delete.html
{% extends 'base.html' %}
{% block title %}- Delete{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-2"></div>
    <div class="col-md-8 panel panel-default">
        <div class="alert alert-info">
            Do you want to delete {{object}}?
        </div>
        <form action="" method="post">
            {{form.as_p}}
            {% csrf_token %}
            <input type="submit" class="btn btn-danger" value="Confirm">
        </form>
    </div>
    <div class="col-md-2"></div>
</div>
{% endblock %}
{% endraw %}
{% endhighlight %}