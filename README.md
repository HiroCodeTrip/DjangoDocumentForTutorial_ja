# django_tutorial
Build simple vote-webapp with django

Djangoの公式チュートリアルに沿って投票アプリを作成していきます。


# 事前準備

端末->windows11

pythonはインストールしている前提とします。

していない人はhttps://www.python.org/downloads/
からインストールしましょう。

とりあえずpythonに標準でついている仮想化環境を作成してそこで作業します。

1. ファイル名 `django_tutorial`
2. python仮想環境作成`python -m venv venv`
3. 仮想環境をアクティベイト化します。`.\venv\Scripts\activate`
4. 仮想環境にdjangoをインストール`pip install django` 
5. 一応確認`pip freeze`

# はじめての Django アプリ作成、その 1
このチュートリアルでは、簡単な投票 (poll) アプリケーションの作成に取り組ん でもらいます。

Poll アプリケーションは 2 つの部分からなります:

- ユーザが投票したり結果を表示したりできる公開用サイト
- 投票項目の追加、変更、削除を行うための管理 (admin) サイト

> プロジェクトを作成する

django-admin startproject config
ここではdjangoのベストプラクティスを見習いconfigというプロジェクトを作成しました。

> 開発用サーバー

`py manage.py runserver`

>Polls アプリケーションをつくる

`py manage.py startapp polls`

>はじめてのビュー作成
- config/urls.py

```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
- polls/urls.py

```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

- polls/views.py

```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```





