---
layout: post
title:  "Django Bookmark Project 5"
date:   2019-05-27
excerpt: "Pythonanywhere에 배포하기"
tag:
- python
- django
- bookmark
- pythonanywhere
comments: true
---

> **책 '배프의 오지랖 파이썬 웹 프로그래밍'을 공부한 내용을 정리한 글**<br>
> **<a href="http://glowingedge.pythonanywhere.com/bookmark/">완성된 홈페이지</a>**<br>
> **<a href="https://github.com/glowingEdge/bookmark">완성된 소스코드</a>**

## 배포

웹 서비스를 여러 사람이 같이 사용하려면 인터넷 어딘가에 있는 서버에 업로드 해야만 한다. 업로드하고 사용할 수 있도록 하는 과정을 `배포`라고 부른다.<br>
로컬 컴퓨터를 배포하는 서버로 만들 수도 있지만, 개념을 익히기 위해 깃허브 pythonanywhere라는 편리한 서비스를 이용해본다.

### 깃허브에 소스코드 올리기

Git의 기본 사용법은 꽤나 익숙하기에 짧게 쓴다.<br>
bookmark 라는 이름으로 repository를 생성한다. 이 때 언어를 python으로 설정하고 .gitignore를 자동생성한다.<br>
가상 환경, db.sqlite3, cahce, 기타 등등을 자동으로 작성해준다.<br>
마지막으로 setting.py 내용을 수정한다.

{% highlight python %}
// settings.py
DEBUG = False
#...
ALLOWED_HOSTS = ['*']
#...
STATIC_ROOT = os.path.join(BASE_DIR, 'static_files')
{% endhighlight %}

DEBUG = False : 템플릿이 없을 때 뜨는 페이지같은 디버깅 페이지를 띄우지 않게 되며, static파일을 자동으로 잡아주지 않고 서버에서 연결해주는 파일을 사용하게 된다.<br>
ALLOWED_HOSTS = ['*'] : 모든 사용자에게 접속을 개방한다. ip주소를 넣어서 사용자를 제한할 수도 있다.
STATIC_ROOT : 배포 상태에서, 서버가 사용자에게 정적파일을 전달하기 위해서 매번 Django 폴더트리를 찾아 전달하는 것은 비효율적이기 때문에, 한 폴더에 복사본을 만들어 서버가 사용하도록 한다. 이를 위한 설정이 `STATIC_ROOT`이다. 

### Pythonanywhere 사용하기

#### 가입 및 초기 설정
https://www.pythonanywhere.com 로 접속해 가입하며, 무료 버전인 `Create a Beginner account`를 선택한다.<br>
가입 후 대시보드가 나타나는데, New consle -> $Bash를 이용한다.<br>
기본적인 세팅은 다음의 순서로 진행한다.<br>
-git clone<br>
-가상환경 생성 : `virtualenv venv --python==python3.7`<br>
-가상환경 실행 : `source venv/bin/activate`<br>
-Django 설치 : `pip install django`<br>
-DB 초기화 : `python manage.py migrate`<br>
-관리자 생성 : `python manage.py createsuperuser`<br>
-정적파일 모으기 : `python manage.py collectstatic` (STATIC_ROOT로 설정한 'staticfiles'에 정적파일이 모두 복사된다)

#### 웹 앱 설정

오른쪽 설정의 Web을 눌러 웹 앱 설정 페이지로 이동한다.<br>
Add a new web app을 통해 생성한 뒤에, Manual configuration으로 파이썬 버전을 선택한다.<br>
`WSGI configuration file:`을 클릭해 다음 코드를 입력하고 `SAVE`를 누른다.

    import os
    import sys

    # assuming your django settings file is at '/home/glowingEdge/mysite/mysite/settings.py'
    # and your manage.py is is at '/home/glowingEdge/mysite/manage.py'
    path = '/home/glowingEdge/bookmark'
    if path not in sys.path:
        sys.path.append(path)

    os.environ['DJANGO_SETTINGS_MODULE'] = 'config.settings'

    # then:
    from django.core.wsgi import get_wsgi_application
    application = get_wsgi_application()

`Virtualenv:`부분도 $BASH에서 설정한 가상환경 경로를 잡아준다.<br>
`Static files:` 마찬가지로 $BASH에서 설정한 경로(ex. /home/[user]/bookmark/static_files)를 잡아준다.<br>
`Reload:` 아래 버튼으로 실행하면 모두 끝.
