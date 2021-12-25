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

# はじめての Django アプリ作成、その2

> Database の設定
- powershell

デフォルトでmysite/settings.pyの中のINSTALLED_APPに入っているアプリケーションを使用するためにはマイグレーションを実行して白紙の状態から設定に沿ってデータベースを最低一つに更新する必要があります。ここの段階ではmakemigrationsコマンドを実行する必要はありません。

`py manage.py migrate`

> モデルの作成

これから開発する簡単な poll アプリケーションでは、Question と Choice の2つのモデルを作成します。Question　モデルには question と publication date という属性（カラム）が存在しがあり、Choice モデルには text of the choice と vote と　参照先を指定するquestion という3つのフィールドがあります。また　生成された各　Choice（選択肢） は Question（質問） に関連づけられています。

    
```
from django.db import models

# Create your models here.

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE) #CASCADE->関連するモデルもすべて削除する処理　質問(Question)が消されたら当然選択肢(Choice)も消える
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

また参照されたモデルを削除する際に紐ついている関連モデルをどうしたいかをon_deleteで指定するがon_deleteのパターンは以下のとおりである。

? :on_update他の場合は自分で調べてみよう。

CASCADE:参照している子モデルもすべて削除する
SET_DEFAULT:削除した際にデフォルトで設定した値が入る
SET_NULL:削除した際に参照している子モデルの値はNULLになる
PROTECT:参照している子モデルがあると親要素を削除できない
DO_NOTHING:何もしない
SET():独自の処理を設定できる

> モデルを有効にする

さて上記に書いたコードだけでdjangoは以下のことができるようになります。
- アプリケーションのデータベーススキーマを作成 (CREATE TABLE 文を実行) 
- Question や Choice に Python からアクセスするためのデータベース API を作成

ですがここで忘れてはいけないことがあります。ここではまだpollsアプリケーションを作成したことをプロジェクトに教えてあげていません。

まずはアプリケーションをプロジェクトに含めるためにmysite/settings.pyファイルのINSTALLED_APPS設定にpolls.apps.PollsConfigと追加してあげましょう。
(PollsConfigクラスはpollsディレクトリの下にあるappsファイルの中にあるPollsConfigクラスを指しています。）
```
INSTALLED_APPS = [
    'polls.apps.PollsConfig', #pollsとしても良いです
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
? :INSTALLED_APPS設定には、アプリケーション構成クラスを直接指定するか、 使用するパッケージを設定することになってます。なのでpollsとしても良いし、polls.apps.PollsConfigとしても良いです。

これで Django は、pollsアプリケーションが含まれていることを認識できます。

もうひとつコマンドを実行しましょう:
