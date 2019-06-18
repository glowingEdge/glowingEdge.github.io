---
layout: post
title:  "Django Dstagram Project 4"
date:   2019-06-17
excerpt: "회원 관리 구현"
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

## 회원 관리

회원 가입, 로그인, 로그아웃 기능을 만든다.<br>
회원 관리를 위한 기능은 Account라는 이름의 앱으로 사진 관리 기능을 하는 Photo 앱과 구분하여 구현한다.

## Account 앱 생성
    
    $python manage.py startapp accounts

앱 생성 후 config/settings.py에 accounts 앱도 추가해준다.

## 로그인, 로그아웃 기능 추가

로그인, 로그아웃을 관리하는 기능이 Django에 이미 구현되어 있다.(그것도 굉장히 잘)<br>
전체 과정은 제네릭뷰를 이용하는 과정과 비슷하다. <br>

### URL과 View 추가

먼저 accounts/urls.py에서 뷰를 호출하도록 하고, config/urls.py에서도 연결해준다.<br>
LoginView, LogoutView의 소스를 살펴보면 제네릭 뷰에서 여러 설정(class member)들과 비슷하게 이미 로그인된 유저와 로그아웃 후에 대한 redict 옵션같은게 보인다.

{% highlight python %}
# accounts/urls.py
from django.urls import path
from django.contrib.auth import views as auth_view

urlpatterns = [
    path('login/', auth_view.LoginView.as_view(), name='login'),
    path('logout/', auth_view.LogoutView.as_view(template_name='registration/logout.html'), name='logout'),
]
{% endhighlight %}

{% highlight python %}
# config/urls.py
...
urlpatterns = [
    path('', include('photo.urls')),
    path('accounts/', include('accounts.urls')),
    path('admin/', admin.site.urls),
]
...
{% endhighlight %}

### 템플릿 생성

로그인, 로그아웃 템플릿을 작성한다.

{% highlight html %}
{% raw %}
// accounts/templates/registration/login.html
{% extends 'base.html' %}
{% block title %}- Login{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-2"></div>
    <div class="col-md-8 panel panel-default">
        <div class="alert alert-info">Please enter your login informations.</div>
        <form action="" method="post">
            {{form.as_p}}
            {% csrf_token %}
            <input class="btn btn-primary" type="submit" value="Login">
        </form>
    </div>
    <div class="col-md-2"></div>
</div>
{% endblock %}
{% endraw %}
{% endhighlight %}

{% highlight html %}
{% raw %}
// accounts/templates/registration/logout.html
{% extends 'base.html' %}
{% block title %}- Logout{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-2"></div>
    <div class="col-md-8 panel panel-default">
        <div class="alert alert-info">You have been successfully logged out.</div>
        <a class="btn btn-primary" href="{% url 'login' %}">Click to Logint</a>
    </div>
    <div class="col-md-2"></div>
</div>
{% endblock %}
{% endraw %}
{% endhighlight %}

앞서 작성한 base.html에서 상단 네비게이션바의 로그인. 로그아웃의 링크도 연결해준다.

{% highlight html %}
{% raw %}
// templates/base.html
...
<li class="nav-item"><a href="{% url 'logout' %}" class="nav-link">Logout</a></li>
{% else %}
<li class="nav-item"><a href="{% url 'login' %}" class="nav-link">Login</a></li>
...
{% endraw %}
{% endhighlight %}

로그인 후 목록페이지로 안내하기 위해 settings.py를 수정해준다.<br>
`LOGIN_REDIRECT_URL`의 default 값은 /profile 이다.

{% highlight python %}
# config/settings.py
...
LOGIN_REDIRECT_URL = '/'
{% endhighlight %}

## 회원가입 기능 만들기

회원가입 기능을 위해 뷰를 만들고, 폼(Form)을 구현하는 방법을 이용한다.

{% highlight python %}
# accounts/forms.py
from django.contrib.auth.models import User
from django import forms

class RegisterForm(forms.ModelForm):
    password = forms.CharField(label='Password', widget=forms.PasswordInput)
    password2 = forms.CharField(label='Repeat Password', widget=forms.PasswordInput)

    class Meta:
        model = User
        fields = ['username', 'first_name', 'last_name', 'email']

    def clean_password2(self):
        cd = self.cleaned_data
        if cd['password'] != cd['password2']:
            raise forms.ValidationError('Passwords not matched')
        return cd['password2']
{% endhighlight %}

forms.ModelForm을 이용해 User 모델의 자료를 입력받도록 하며, password의 경우 widget 옵션으로 사용자가 입력시 `*****`형태로 보이도록 한다.<br>
form에 전달된 데이터의 유효성을 검사하는 각 필드의 clean_ 메서드들이 수행되고 나면, 사용자가 정의한 clean_password2(self) 메서드가 호출된다.<br>
cd[]로 전달된 값은 View에서 Form.IsValid() 이후 cleaned_data['field']의 형태로 사용 가능하다.<br>
<br>
값을 전달해주는 Form을 생성하였으니 가입 기능을 처리하는 View를 만들어준다.<br>
**Django docs 참고:** https://docs.djangoproject.com/en/2.2/ref/forms/validation/ <br>
**초보몽키님 블로그 참고:** https://wayhome25.github.io/django/2017/05/06/django-model-form/

{% highlight python %}
# accounts/views.py
from django.shortcuts import render
from .forms import RegisterForm

def register(request):
    if request.method == 'POST':
        user_form = RegisterForm(request.POST)
        if user_form.is_valid():
            new_user = user_form.save(commit=False)
            new_user.set_password(user_form.cleaned_data['password'])      # request.POST['password']를 쓴다면 유효성 검사가 안된, 전달받은 값을 그대로 사용하는 셈이다.
            new_user.save()
            return render(request, 'registration/register_done.html', {'new_user': new_user})
    else:
        user_form = RegisterForm()

    return render(request, 'registration/register.html', {'form': user_form})
{% endhighlight %}

new_user = user_form.save(commit=False) : user모델 객체를 생성하고, commit=False 옵션으로 DB에 저장하지않고 메모리상에만 생성한다.<br>
new_user.set_password 메서드를 통해 암호를 저장해야 암호화된 상태로 저정된다. DB를 열어보면 모두 hash값처럼 생긴 string으로 저정되어 있다. password field가 charField인 이유는 이것때문인 것 같다.<br>
회원가입이 완료되면 register_done페이지로 안내한다.<br>
<br>
뷰를 연결하기 위해 url을 수정하고, 템플릿도 작성해준다.

{% highlight python %}
# accounts/urls.py
from django.urls import path
from django.contrib.auth import views as auth_view
from .views import register

urlpatterns = [
    path('login/', auth_view.LoginView.as_view(), name='login'),
    path('logout/', auth_view.LogoutView.as_view(template_name='registration/logout.html'), name='logout'),
    path('register/', register, name='register')
]
{% endhighlight %}

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

{% highlight html %}
{% raw %}
// accounts/templates/registration/register_done.html
{% extends 'base.html' %}

{% block title %}- Registration Done{% endblock %}

{% block content %}
<div class="row">
        <div class="col-md-2"></div>
        <div class="col-md-8 panel panel-default">
            <div class="alert alert-info">Registration Success. Welcome, {{new_user.username}}</div>
            <a class="btn btn-info" href="/">Move to main</a>
        </div>
        <div class="col-md-2"></div>
</div>
{% endblock %}
{% endraw %}
{% endhighlight %}

마지막으로 base.html에서 네비게이션바의 회원가입 링크도 연결해준다.
{% highlight html %}
{% raw %}
// templates/base.html
...
<li class="nav-item"><a href="{% url 'register' %}" class="nav-link">Signup</a></li>
...
{% endraw %}
{% endhighlight %}