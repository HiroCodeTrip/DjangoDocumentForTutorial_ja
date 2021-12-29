# はじめての Django アプリ作成、その 1 プロジェクトの作成とページ表示の基礎
このチュートリアルでは、簡単な投票 (poll) アプリケーションの作成に取り組ん でもらいます。

Poll アプリケーションは 2 つの部分からなります:

- ユーザが投票したり結果を表示したりできる公開用サイト
- 投票項目の追加、変更、削除を行うための管理 (admin) サイト

> プロジェクトを作成する

django-admin startproject config
ここではdjangoのベストプラクティスを見習いconfigというプロジェクトを作成しました。

> 開発用サーバー

`py manage.py runserver`

>　Polls アプリケーションをつくる

`py manage.py startapp polls`

>はじめてのビュー作成
- config/urls.py


http://127.0.0.1:8000/polls/
にアクセスしたときpolls/urlsファイルにいく。
この時点でpolls/urlsは作成しよう。

```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```


- polls/urls.py

http://127.0.0.1:8000/polls/
にアクセスしたときにindex関数を表示する。

```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

- polls/views.py


index関数の中身として
"Hello, world. You're at the polls index."と表示させる

```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
