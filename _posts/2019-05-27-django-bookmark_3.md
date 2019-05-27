---
layout: post
title:  "Django Bookmark Project 3"
date:   2019-05-27
excerpt: "북마크 확인, 수정, 삭제"
tag:
- python
- django
- bookmark
- pythonanywhere
comments: true
---

> **책 '배프의 오지랖 파이썬 웹 프로그래밍'을 공부한 내용을 정리한 글**<br>
> **<a href="http://glowingedge.pythonanywhere.com/bookmark/">완성된 홈페이지</a>**
>**<a href="https://github.com/glowingEdge/bookmark">완성된 소스코드</a>**

## 북마크 확인 기능 구현

### 목록 View 만들기

클래스형 뷰 중 DetailView를 이용하여 구현한다.
{% highlight python %}
// views.py
from django.views.generic import ListView, CreateView, DetailView
from django.urls import reverse_lazy
from .models import Bookmark

#...
class BookmarkDetailView(DetailView):
    model = Bookmark
{% endhighlight %}

### URL 만들기

{% highlight python %}
// urls.py
from django.urls import path
from .views import BookmarkListView, BookmarkCreateView, BookmarkDetailView

urlpatterns = [
    path('', BookmarkListView.as_view(), name='list'),
    path('add/', BookmarkCreateView.as_view(), name='add'),
    path('detail/<int:pk>/', BookmarkDetailView.as_view(), name='detail'),
]
{% endhighlight %}

<int:pk> : 컨버터라고 하는 기능으로, 뒤쪽(pk)은 컨버터를 통해 반환받는 패턴에 일치하는 값의 변수명이다.<br>
기본 제공되는 컨버터의 종류는 다음과 같다.<br>
str : 비어있지 않은 모든 문자와 매칭. 단 '/'는 제외. 디폴트 컨버터<br>
int : 0을 포함한 양의 정수와 매칭<br>
slug : 아스키 문자나 숫자, 하이픈, 언더스코어를 포함한 슬러그 문자열<br>
uuid : UUID와 매칭. 같은 페이지에 여러 URL이 연결되는 것을 막으려고 사용<br>
path : '/'을 포함하는 str. URL의 일부가 아닌 전체 매칭이 필요할 때 사용


### Temaplate 만들기

{% highlight html %}
{% raw %}
// bookmark/templates/bookmark/bookmark_detail.html
    {{object.site_name}}<br/>
    {{object.url}}
{% endraw %}
{% endhighlight %}

간단한 정보 확인 페이지 용도이며, 목록 페이지에서 URL 연결로 마무리 짓는다.

{% highlight html %}
{% raw %}
// bookmark/templates/bookmark/bookmark_list.html
// ...
    <td><a href="{% url 'detail' pk=bookmark.id %}">{{bookmark.site_name}}</a></td>
// ...
{% endraw %}
{% endhighlight %}

## 북마크 수정과 삭제 기능 구현

수정과 삭제는 앞에서 몇 번 따라했던 흐름과 크게 다르지 않으며, 클래스형 뷰의 이름, 수정/삭제 작업 후의 흐름에 대한 처리만 정리하도록 한다.

### 북마크 수정

북마크 수정은 UpdateView를 사용한다.<br>
기존과 동일하게 진행할 경우, 수정을 마친 후 'No URL to redirect to. Either provide aurl or define a get_absolute_url method onthe Model' 이라는 오류 메세지를 보게 된다. View에서 success_url도 없고, Model에서 get_absolute_url 메소드도 없다는 뜻이다.<br>
get_absolute_url 메소드는 다음처럼 사용할 수 있다.

{% highlight python %}
// models.py
from django.db import models
from django.urls import reverse

class Bookmark(models.Model):
    #...
    def get_absolute_url(self):
        return reverse('detail', args=[str(self.id)])
{% endhighlight %}

### 북마크 삭제

북마크 삭제는 DeleteView를 사용한다.<br>
생성 기능과 마찬가지로 View에서 success_url을 목록 페이지로 가도록 reverse_lazy를 사용해 설정한다.<br><br>

나머지는 비슷하게 구현 후 실행하면, 이번에는 템플릿이 없다는 메세지가 나온다. 템플릿 이름을 보면 'bookmark_confirm_delete.html'로 되어있다는 걸 알 수 있다. 다음처럼 템플릿을 추가해준다.

{% highlight html %}
{% raw %}
    <form action="" method="post">
        {% csrf_token %}
        <div class="alert alert-danger">Do you want to delete Bookmark "{{object}}"?</div>
        <input type="submit" value="Delete" class="btn btn-danger">
    </form>
{% endraw %}
{% endhighlight %}