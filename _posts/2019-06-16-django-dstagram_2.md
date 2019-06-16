---
layout: post
title:  "Django Dstagram Project 2"
date:   2019-06-16
excerpt: "뷰 생성 및 URL 연결"
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

## View 만들기

사진 목록, 업로드, 확인, 수정, 삭제 기능을 위한 뷰를 만든다.

### 목록 뷰

목록 뷰는 함수형 뷰를 사용한다. 함수형 뷰는 request를 기본 매개변수로 사용하며, 모든 기능을 직접 만들어야 한다.
{% highlight python %}
# phots/views.py
from django.shortcuts import render
from .models import Photo

def photo_list(request):
    photos = Photo.objects.all()
    return render(request, 'photo/list.html', {'photos': photos})
{% endhighlight %}

### 업로드, 확인, 수정, 삭제 뷰

나머지 뷰는 제네릭 뷰를 이용하여 만든다.

{% highlight python %}
# phots/views.py
from django.shortcuts import render, redirect
from .models import Photo
from django.views.generic import CreateView, DeleteView, UpdateView

class PhotoUploadView(CreateView):
    model = Photo
    fields = ['photo', 'text']
    template_name = 'photo/upload.html'

    def form_valid(self, form):
        form.instance.author_id = self.request.user.id  # authro_id: foreign key의 경우 _id suffix가 자동으로 붙는다.
        if form.is_valid():
            form.instance.save()
            return redirect('/')
        else:
            return self.render_to_response({'form': form})

class PhotoDeleteView(DeleteView):
    model = Photo
    success_url = '/'
    template_name = 'photo/delete.html'

class PhotoUpdateView(UpdateView):
    model = Photo
    fields = ['photo', 'text']
    template_name = 'photo/update.html'
{% endhighlight %}

### 업로드 뷰

CreateView를 상속받아 만든다. template_name으로 사용할 html 파일명을 지정해준다.<br>
form_valid 메서드는 오버라이드한 메서드로, 업로드를 끝낸 후 이동할 페이지를 호출하기 위해 사용하는 메서드이다. <br>
입력값을 검증해 이상이 있다면 작성된 내용을 그대로(form) 호출한 페이지로 표시한다.

## URL 연결과 인라인 뷰

제네릭뷰를 이용해 간단한 뷰를 만들 때는 아래 DetailView처럼 URL에서 간단한 인자만 넘겨주어 구현할 수 있다.<br>
이러한 방식을 인라인 뷰라고 한다.

{% highlight python %}
# phots/urls.py
from django.urls import path
from django.views.generic.detail import DetailView
from .views import *
from .models import Photo

app_name = 'photo'

urlpatterns = [
    path('', photo_list, name='photo_list'),
    path('detail/<int:pk>/', DetailView.as_view(model=Photo, template_name='photo/detail.html'), name='photo_detail'),
    path('upload/', PhotoUploadView.as_view(), name='photo_upload'),
    path('delete/<int:pk>/', PhotoDeleteView.as_view(), name='photo_delete'),
    path('update/<int:pk>/', PhotoUpdateView.as_view(), name='photo_update'),
]
{% endhighlight %}

app_name은 네임스페이스라고 하며, 템플릿 url템플릿 태그를 사용할 때 `photo:photo_detail`과 같은 형태로 사용할 수 있다.<br>
마지막으로 config/urls.py에 photo/urls.py로 연결하는 코드를 추가한다. 사이트URL에 접속시 메인페이지를 photo app으로 한다.

{% highlight python %}
# config/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('', include('photo.urls')),
    path('admin/', admin.site.urls),
{% endhighlight %}