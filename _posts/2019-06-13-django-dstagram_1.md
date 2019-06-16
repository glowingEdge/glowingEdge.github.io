---
layout: post
title:  "Django Dstagram Project 1"
date:   2019-06-13
excerpt: "모델 생성 및 관리자 페이지 커스터마이징"
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

## 만들 기능

인스타그램처럼 사진을 올리고 사진에 댓글을 다는 카피 서비스를 만든다. Django에 대해서 뿐만 아니라 `Disqu`를 통한 댓글달기 서비스 연동, `AWS S3`를 이용하는 이미지 서버, `Heroku`를 통한 배포를 배운다.

### 글 목록

글 목록을 보여주는 페이지. 사진별로 작성자, 설명, 댓글달기 버튼을 출력한다.

### 글 생성, 수정 

사진을 업로드하고 설명을 입력하여 글을 생성하고 수정한다.

### 사진 상세

사진의 상세 정보를 확인한다.  목록과 동일한 내용 + 수정, 삭제, 댓글 달기.

### 사진 삭제

삭제를 하기 전에 확인 메세지를 출력하고, 확인 버튼을 눌러 삭제한다.

### 로그인, 로그아웃

로그인 된 유저만 사진들을 볼 수 있다.<br>
로그아웃을 하면 성공 메세지와 함께 로그인 페이지로 이동하기 버튼을  출력한다

### 회원가입

ModelForm을 이용해 회원가입에 필요한 폼을 출력하고, 회원가입을 할 수 있다.

## Project 구성

역할에 따라 사진을 관리하는 기능을 하는 Photo App, 계정을 관리하는 

## 프로젝트 생성 및 환경 설정

    // myvenv라는 이름의 가상환경 생성 및 실행
    $ python -m venv myvenv
    $ ./myvenv/Scripts/activate

    $ pip install django

    // 현재폴더(.)에 config 라는 이름으로 설정 폴더 생성
    $ django-admin startproject config .

    // DB 초기화, 관리자 계정 생성
    $ python manage.py migrate
    $ python manage.py createsuperuser

    // photo 앱 생성
    $ python manage.py startapp photo

## 모델 생성 및 관리자 페이지 연결

### 모델 생성

settings.py에 Photo App 등록
{% highlight python %}
//settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'photo',            // 끝에 ','를 붙여주면 좋다. 자주 까먹고 추가할 때 오류를 내기 쉽다.
]
{% endhighlight %}

사진 관리에 필요한 기본 모델을 구성한다.
{% highlight python %}
// photo/models.py
from django.db import models
from django.contrib.auth.models import User

class Photo(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='user_photos')
    photo = models.ImageField(upload_to='photos/%Y/%m/%d', default='photos/no_image.png')
    text = models.TextField()
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-updated']

    def __str__(self):
        return self.author.username + " " + self.created.strftime("%Y-%m-%d %H:%M:%S")

    def get_absolute_url(self):
        return reverse('photo:photo_detail', args=[str(self.id)])
{% endhighlight %}

#### author

ForeignKey를 사용하여 User Table과 관계를 만든다. (User는 Django의 기본 모델)<br>
on_delete는 연결된 모델이 삭제될 경우의 행동을 지시해주고, 다음 6가지 옵션이 있다.<br>
CASCAED: 연결된 객체가 지워지면 해당 하위 객체도 같이 삭제<br>
PROTECT: 하위 객체가 남아 있다면 연결된 객체가 지워지지 않음<br>
SET_NULL: 연결된 객체만 삭제하고 필드 값을 null로 설정<br>
SET_DEFAULT: 연결된 객체만 삭제하고 필드 값을 설정된 기본 값으로 변경\n
SET(): 연결된 객체만 삭제하고 지정된 값으로 변경<br>
DO_NOTHING: 아무일도 하지 않음<br>
<br>
related_name의 경우, 연결된 객체에서 하위 객체의 목록을 부를 때 사용하는 이름이다. Photo의 경우 특정 유저의 글을 볼때 유저 객체에서 user_phots 속성을 참조하면 된다.

#### photo

이미지를 저장하는 필드. upload_to로 사진이 업로드 될 경로를 설정한다. 만약 업로드가 되지 않는다면 default 값으로 대체한다.

#### created / updated

auto_now_add=True 로 설정하면 객체가 추가될 때 자동으로 값을 설정한다.<br>
auto_now=True 로 설정하면 객체가 수정될 때마다 자동으로 값을 설정한다.

#### Class meta

ordering 클래스 변수로 정렬 기준을 지정한다. String값에 '-'를 붙이면 내림차순을 의미한다.

#### get_absolute_url(self)

객체 추가나 수정 시에 상세 페이지 주소를 반환하는 메서드이다. 이 대신에 reverse 메서드를 사용할 수도 있다.<br>
return값의 args는 여러값들을 리스트로 전달할 수 있다.

### 데이터베이스 적용

DB migration을 하려하면 이미지를 다루는 모듈이 없어 에러가 난다. 설치해준다.

    $ python manage.py makemigrations bookmark
    ERROR
    ...
    $ pip install pillow
    $ phython manage.py migrate photo 001   //가장 최근 변경사항 적용시 001 또는 photo까지 생략 가능

### 관리자 페이지 연결 및 커스마이징

모델을 이용해 작업할 때, 뷰를 만들어 확인하려면 시간이 걸리고, 데이터 입력이 필요하기에 관리자 페이지에서 모델을 관리하면 편하다.

{% highlight python %}
// photo/admin.py
from django.contrib import admin
from .models import Photo

class PhotoAdmin(admin.ModelAdmin):
    list_display = ['id', 'author', 'created', 'updated']
    raw_id_fields = ['author']
    list_filter = ['created', 'updated', 'author']
    search_fields = ['text', 'created']
    ordering = ['-updated', '-created']

admin.site.register(Photo, PhotoAdmin)
{% endhighlight %}

list_display: 목록에 보여줄 필드. 함수를 만들어 등록하는 방법도 있다.<br>
raw_id_fields: 연결된 모델의 객체 목록을 출력해야 하는데, 드랍다운으로 표시되기엔 너무 길 때 사용한다.(ex. 사용자가 100만명일 때 드랍다운이라면?) 드랍다운 대신 검색 팝업을 띄운다.<br>
search_fields: 검색기능으로 검색할 필드를 설정한다. ForeignKey 필드는 사용불가.

## 업로드 폴더 관리

프로젝트가 커지면서 여러 앱에서 필요한 사진이나 파일들을 저정해야 하는데, 이를 관리하기 위한 Django 옵션을 설정한다.

{% highlight python %}
// photo/settings.py
...
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
...
{% endhighlight %}

MEDIA_URL: 브라우저에서 파일을 서비할 때 사용하는 가상URL이다. 이러한 가상URL로 대체함으로써 서버 내부 폴더 구조를 숨겨 정보를 적게 유출하여 보안을 향상한다.<br>
MEDIA_ROOT: 이 옵션에 지정한 경로 밑으로 앱들이 폴더를 만들고 파일을 업로드하게 된다.