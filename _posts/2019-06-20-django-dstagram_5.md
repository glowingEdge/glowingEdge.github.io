---
layout: post
title:  "Django Dstagram Project 5"
date:   2019-06-20
excerpt: "댓글 서비스와 권한 관리"
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


Instagram 카피서비스라면 댓글 기능은 빠질 수가 없을텐데, 댓글 기능은 JavaScript를 통한 제어를 많이 필요로 한다.<br>
하지만 Disqus라는 상당히 편리한 댓글 서비스가 있어, 이를 활용해보려 한다.

## Disqus Script 가입

Disqus(https://disqus.com)에 접속하여 회원가입 후, `I want to install Disqus on my site`탭으로 들어가 사이트의 이름을 설정한다.<br>
그 다음 요금제를 선택하고, 플랫폼 선택의 경우 선택하지 않고 가장 밑의 `I don't see my platform listed, install manually with Universal Code`를 선택한다.<br>
그 다음 설정들도 특별히 설정해줄 필요가 없다. 

## Disqus 앱 설치

Django에서 Disqus를 앱의 형태로 바로 사용할 수가 있다.

    $pip install django-disqus

settings.py에는 `disqis`, `django.contrib.sites` 두 개의 앱을 추가하고, 다음 두 가지 설정도 추가해준다.<br>
DISQUS_WEBSITE_SHORTNAME = 'Disqus가입시 설정한 이름' <br>
SITE_ID = 1    // Disqus에서 관리되는 사이트 번호<br>

## Detail페이지에 Disqus 추가

Detail페이지 하단에 댓글 기능을 추가한다.

{% highlight html %}
{% raw %}
// accounts/templates/registration/register.html
{% extends 'base.html' %}

{% block title %}- Registration{% endblock %}

{% block content %}
<div class="row">
        <div class="col-md-2"></div>
        <div class="col-md-8 panel panel-default">
            <div class="alert alert-info">Please enter your account informations.</div>
            <form action="" method="post">
                {{form.as_p}}
                {% csrf_token %}
                <input class="btn btn-primary" type="submit" value="Register">
            </form>
        </div>
        <div class="col-md-2"></div>
</div>
{% endblock %}
{% endraw %}
{% endhighlight %}

## 사이트 권한 관리

Django에서 제공하는 기능 중에, 사이트에서 로그인된 유저만 기능을 사용할 수 있도록 관리하는 기능이 있다.<br>
데코레이터(Decorator)를 쓰는 방법과 믹스인(Mixin)을 쓰는 방법이 있다. <br>
함수형 뷰의 경우에는 데코레이터를 쓰고, <br>
클래스형 뷰의 경우에는 믹스인을 상속하면 된다.

{% highlight python %}
# photo/views.py
...
from django.contrib.auth.decorators import login_required
from django.contrib.auth.mixins import LoginRequiredMixin


@login_required
def photo_list(request):
    photos = Photo.objects.all()
    return render(request, 'photo/list.html', {'photos': photos})

class PhotoUploadView(LoginRequiredMixin, CreateView):
...
{% endhighlight %}