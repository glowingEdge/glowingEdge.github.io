---
layout: post
title:  "Django Bookmark Project 2"
date:   2019-05-25
excerpt: "북마크 목록, 추가 기능 구현"
tag:
- python
- django
- bookmark
- pythonanywhere
comments: true
---

> **책 '배프의 오지랖 파이썬 웹 프로그래밍'을 공부한 내용을 정리한 글**<br>
> **<a href="http://glowingedge.pythonanywhere.com/bookmark/">완성된 홈페이지</a>**

## Djnago MTV

Django는 MTV패턴을 사용하며, 개발도 각각의 부분을 연결하는 식으로 진행한다.(사실 이름만 다를뿐 각각 기존 웹의 MVC라 생각해도 될 듯 하다..)<br>
사용자가 특정 URL을 입력하면 해당 URL을 처리하는 View를 호출하고, View에서는 필요한 Model을 읽어와 정보를 처리하고, Template(HTML)을 호출하도록 한다.

## 북마크 목록 기능 구현
### 목록 View 만들기

Django에는 클래스형 뷰와 함수형 뷰가 있다. 클래스형 뷰는 자주 사용하는 기능을 장고가 미리 준비해뒀으며, 이들을 상속받아 사용하면 된다.
{% highlight python %}
// views.py
from django.views.generic import ListView
from .models import Bookmark

class BookmarkListView(ListView):
    model = Bookmark
{% endhighlight %}
상속받은 ListView 클래스의 소스를 따라 여행하다보면 model이라는 attribute를 사용하는 다양한 작업을 미리 정의해놨다.

### URL 만들기

URL 설정은 bookmark/urls.py를 생성하여 진행하며, config폴더의 urls.py는 루트에 해당하며, 서브 파일(각 앱의 urls.py)로 넘기게 된다.<br>
루트 urls.py에 모두 저장해도 기능 작동엔 상관 없지만, 앱 별로 다른 프로젝트에 재사용할 수 있기 위해선 분리하는 것이 좋다. (유지보수에도 이쪽이 좋아보인다.)<br>
 
{% highlight python %}
// config/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('bookmark/', include('bookmark.urls')),
    path('admin/', admin.site.urls),
]
{% endhighlight %}

{% highlight python %}
// bookmark/urls.py
from django.urls import path
from .views import BookmarkListView

urlpatterns = [
    path('', BookmarkListView.as_view(), name='list'),
]
{% endhighlight %}

### Temaplate 만들기
`bookmark` 폴더 하단에 `templates/bookmark/bookmark_list.html`와 같이 2개의 폴더와 HTML파일을 생성한다. `templates/bookmark`와 같이 앱과 같은 이름을 쓰는 이유는 각 앱에 해당하는 templates 폴더의 파일들을 찾기 위해 정해진 위치이다.
{% highlight html %}
{% raw %}
// bookmark/templates/bookmark/bookmark_list.html
  <div class="btn-group">
        <a href="#" class="btn btn-info">Add Bookmark</a>
    </div>
    <p></p>
    <table class="table">
        <thead>
            <tr>
                <th scope="col">#</th>
                <th scope="col">Site</th>
                <th scope="col">URL</th>
                <th scope="col">Modify</th>
                <th scope="col">Delete</th>
            </tr>
        </thead>
        <tbody>
            {% for bookmark in object_list %}
                <tr>
                    <td>{{bookmark.id}}</td>
                    <td><a href="#">{{bookmark.site_name}}</a></td>
                    <td><a href="{{bookmark.url}}" target="_blank">{{bookmark.url}}</a></td>
                    <td><a href="#" class="btn btn-success btn-sm">Modify</a></td>
                    <td><a href="#" class="btn btn-danger btn-sm">Delete</a></td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
{% endraw %}
{% endhighlight %}

제네릭뷰 사용시 모델의 오브젝트가 여러 개일 경우 object_list라는 변수로 HTML에 전달한다.

## 북마크 추가 기능 구현

### View 만들기

{% highlight python %}
// views.py
from django.views.generic import ListView, CreateView
from django.urls import reverse_lazy
from .models import Bookmark

#...
class BookmarkCreateView(CreateView):
    model = Bookmark
    fields = ['site_name', 'url']
    success_url = reverse_lazy('list')
    template_name_suffix = '_create'

{% endhighlight %}

CreateView라는 제네릭뷰를 이용하여 구현한다.<br>
fields : 모델의 어떤 fields를 입력받을 것인지 설정한다.<br>
success_url : default값으로는 상세 페이지로 이동한다.<br>
template_name_suffix : 사용할 템플릿의 접미사를 지정한다. html명을 `사용할 모델명_create.html`하겠다는 의미이다.

### URL 만들기
{% highlight python %}
// urls.py
from django.urls import path
from .views import BookmarkListView, BookmarkCreateView


urlpatterns = [
    path('', BookmarkListView.as_view(), name='list'),
    path('add/', BookmarkCreateView.as_view(), name='add'),
]
{% endhighlight %}

### HTML 만들기

{% highlight html %}
{% raw %}
// bookmark/templates/bookmark/bookmark_create.html
    <form action="" method="post">
        {% csrf_token %}
        {{form.as_p}}
        <input type="submit" value="Add" class="btn btn-info btn-sm">
    </form>
{% endraw %}
{% endhighlight %}

form은 HTML->서버로 자료를 전달하기 위해 사용되는 태그이며, action메서드는 전달할 대상 페이지를 지정하는 부분이다. 비워둘 경우 현재 페이지에 전달한다.<br>
csrf_token은 CSRF(Cross Site Request Forgery) 공격을 막는 용도이다. 해커가 로그인된 사용자 권한으로 사이트를 공격하는걸 막기 위한 용도이다.<br>
form.as_p로는 클래스형 뷰의 옵션 값으로 설정한 필드를 출력하고, p태그로 감싸는 역할을 한다.<br><br>

### Add bookmark 버튼 연결하기
북마크 목록 페이지의 'Add Bookmark'버튼에 북마크 추가 페이지를 연결한다.
{% highlight html %}
{% raw %}
// bookmark/templates/bookmark/bookmark_list.html
// ...
    <a href="{% url 'add' %}" class="btn btn-info">Add Bookmark</a>
// ...
{% endraw %}
{% endhighlight %}

`python manage.py runserver` 명령어로 서버를 실행시켜 `127.0.0.1:8000/bookmark/add` 주소로 접속하여 구현된 기능을 확인한다.