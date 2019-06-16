---
layout: post
title:  "Django Bookmark Project 1"
date:   2019-05-24
excerpt: "환경 설정 및 모델 생성"
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

## 만들 기능

### 목록 페이지

저장한 북마크의 목록을 출력한다. 번호, 사이트 이름, 주소, 수정하기와 삭제하기 버튼이 테이블 형식으로 출력되며, 테이블 위에는 북마크 추가하기 버튼이 있다. 페이징 기능도 존재한다.

### 북마크 추가

사이트 이름과 주소를 넣어 북마크를 추가한다.

### 북마크 상세

저장된 사이트 이름과 주소를 띄워주는 페이지. 기본적인 CRUD를 위해 만들어 본다.

### 북마크 수정

상세 페이지와 비슷하지만, 내용을 수정하고 저장할 수 있다.

### 북마크 삭제

삭제 확인 메세지를 출력해 재확인 후 삭제한다.

### 관리자 페이지

내용을 파악할 수 있을 정도의 목록을 출력하도록 한다.


## 프로젝트 생성 및 환경 설정

    // myvenv라는 이름의 가상환경 생성 및 실행
    $ python -m venv myvenv
    $ ./myvenv/Scripts/activate

    $ pip install django

    // 현재폴더(.)에 config 라는 이름으로 설정 폴더 생성
    $ django-admin startproject config .

    // DB 초기화, 관리자 계정 생성 후 서버 실행
    $ python manage.py migrate
    $ python manage.py createsuperuser
    $ python manage.py runserver    // 127.0.0.1:8000

    // bookmark 앱 생성
    $ python manage.py startapp bookmark

## 모델 생성 및 관리자 페이지 연결

### 모델 생성

{% highlight python %}
// models.py
from django.db import models


class Bookmark(models.Model):
    site_name = models.CharField(max_length=100)
    url = models.URLField('Site URL')

{% endhighlight %}

모델을 이용해 데이터베이스에 테이블을 생성하려면 `makemigrateions` `migrate`명령어를 사용하며, 그 전에 bookmark앱을 `setting.py`에 추가해야 한다.

{% highlight python %}
//settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'bookmark',                     // 끝에 ','를 붙여주면 좋다. 자주 까먹고 추가할 때 오류를 내기 쉽다.
]
{% endhighlight %}

    // 데이터베이스 변경사항을 기록하는 파일 생성
    $ python manage.py makemigrations bookmark
    //변경사항 적용
    $ python manage.py migrate bookmark

### 관리자 페이지 연결

모델을 이용해 작업할 때, 뷰를 만들어 확인하려면 시간이 걸리고, 데이터 입력이 필요하기에 관리자 페이지에서 모델을 관리하면 편하다.

{% highlight python %}
//admin.py
from django.contrib import admin
from .models import Bookmark

admin.site.register(Bookmark)
{% endhighlight %}

`admin.py`는 관리자 페이지에서 보이는 내용의 변경, 기능 추가를 하는 곳이다.<br>
모델을 몇 개 등록해보면 'Bookmark object (1)'과 같은 식으로 목록이 나오므로, 해당 기능을 추가한다.

{% highlight python %}
from django.contrib import admin
from .models import Bookmark

class Bookmark(models.Model):
// models.py
#...
    def __str__(self):
        # 객체를 출력할 때 나타날 값
        return "이름: " + self.site_name + ", 주소: "+ self.url
{% endhighlight %}