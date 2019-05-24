---
layout: post
title:  "Django Bookmark Project"
date:   2019-05-24
excerpt: "인터넷 즐겨찾기 관리 서비스"
tag:
- python
- django
- bookmark
- pythonanywhere
comments: true
---

> **책 '배프의 오지랖 파이썬 웹 프로그래밍'을 공부한 내용을 정리한 글**<br>
> **<a href="http://glowingedge.pythonanywhere.com/bookmark/">완성된 홈페이지</a>**

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

{% highlight python %}
from django.contrib import admin

url patterns = [
    path('bookmark/', include('bookmark.urls'))
]
{% endhighlight %}
