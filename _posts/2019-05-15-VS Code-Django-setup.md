---
layout: post
title:  "VS Code에 Django 개발 환경 셋업하기"
date:   2019-05-15
excerpt: "VS Code에 Django 개발 환경 셋업하기"
tag:
- python
- django
- vscode
comments: true
---

## 들어가기

PyCharm이라는 편리한 IDE가 있음에도 VS Code의 가벼움이나 하다못해 작은 아이콘들의 직관성은 참 좋다.
때문에 '배프의 오지랖 파이썬 웹 프로그래밍'을 공부하기에 앞서, VS Code로 개발 환경을 세팅하기로 했다.

## 준비물

- 윈도우 환경
- VS Code
- Python 3.xx
{: .notice}

## 시작하기

1. Python을 설치하고 나서 VS Code를 관리자 권한으로 실행 (켜진 상태에서 셋업하려다 삽질..)
2. VS Code 상단의 Terminal - New Termianl
3. 그럼 Windows PowerShell 이 뜬다. (cmd랑은 다른 windows만의 cli인가보다 - <a href="https://m.blog.naver.com/PostView.nhn?blogId=detect1554&logNo=221145457681&proxyReferer=https%3A%2F%2Fwww.google.com%2F">Soyunviajero님 블로그 참고</a>)
4. 아래 커맨드를 입력한다.

    {% raw %}
    cmd> Set-ExecutionPolicy Unrestricted
    {% endraw %}

5. 가상환경을 설치하고 실행한다.

    {% raw %}
    user> pyhon -m venv '가상환경 이름'
    user> .\venv\Scripts\activate
    {% endraw %}

6. pip를 통해 Django를 설치한다.

    {% raw %}
    user> pip install django
    {% endraw %}