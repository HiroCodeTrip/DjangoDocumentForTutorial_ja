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
    pub_date = models.DateTimeField("date published")　#verbose_nameを設定している。

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE) #CASCADE->関連するモデルもすべて削除する処理　質問(Question)が消されたら当然選択肢(Choice)も消える
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
上記のコードにおいてpub_date = models.DateTimeField("date published")とありますが、
これを指定することにより管理画面におけるモデル名の表示内容が変わります。pub_dateなど開発者以外に分かりにくい名前をこうすることで分かりやすい名前に整形することができます。またverbose_nameはClass Metaでも設定できます。

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

もうひとつコマンドを実行しましょう

- powershell
`py manage.py makemigrations polls`

これを実行することでDjangoにモデルの変更（新しいモデルを作成した）があったことを伝え、migrationsディレクトリに新たなデータベーススキーマの定義を保存することができました。

さてでは具体的にmigrationsディレクトリに生成されたファイルはどのようなsql文を実行するのでしょうか？一旦覗いてみましょう。
ここではsqlmigrateコマンドを使用し中身をどんなsql文が実行されているのかを表示してみます。sqlコマンドはマイグレーションファイル(migrationsディレクトリ配下にあるファイル群）の名前を引数に取りsql文を返します。

- powershell
 
`py manage.py sqlmigrate polls 0001`


- 表示された内容
```
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" 
    ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "question_text" varchar(200) NOT NULL, 
    "pub_date" datetime NOT NULL);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" 
    ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL, 
    "question_id" bigint NOT NULL REFERENCES "polls_question" ("id") DEFERRABLE        INITIALLY DEFERRED);
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");

COMMIT;
```
さて上記のコードですがいくつか気になるところは見つかりましたか？
以下に幾つかの気を付けるべきところを書き記したいと思います。
1. テーブル名はアプリケーションの名前（Polls)とモデルの名前を結合して生成されます。(ex.polls_question)
2. 主キー(primary key,ID)は自動に追加されます。#idというフィールドが自動的に作られる。
3. 便宜上djangoは外部キーフィールド名に`_id`と追加します。(ex.question_id)
4. DEFERABLEの部分は外部キーの処理がトランザクション終了までに実行されないことを定義しています。
5. sqlmigrateコマンドは生成されたsqlをスクリーンに表示するだけのコマンドです。djangoが何をしようとしているか確認したり、何かしらの形でsql文が必要になったときに役立ちます。

もし興味があれば `py manage.py check`を実行してみることも出来ます。これはマイグレーションを作成したりデータベースにふれることなくプロジェクトに問題があるかを確認する際に役立ちます。問題がない場合は以下のように表示されます。

- 表示される内容
`System check identified no issues (0 silenced).`

さてではもう一度migrateコマンドを実行してモデルのテーブルをデータベースに更新しましょう。
- powershell
`py manage.py migrate`

-　表示された内容
```Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
 ```
どうでしょうか無事にpolls.0001_initial.pyをデータベースに更新することはできましたか？
makemigrationsコマンドとmigrateコマンドの理解は難しいと思いますがここでは

- makemigrationsコマンド
models.py に新たに追加した内容をmigrationsディレクトリ下にファイルとしてあらたにスキーマを定義する。
- migrateコマンド
migrationsディレクトリ下にあるファイルを参照しデータベース(sqliteデータベース、この場合は .\db.sqlite3ファイル)に更新する。

という風に考えましょう。

つまり以下の三ステップをおぼえれば大丈夫という事です！
- モデルを変更する (models.py の中の)
- マイグレーションを作成するために python manage.py makemigrations を実行する
- データベースにこれらの変更を適用するために python manage.py migrate を実行する

> APIで遊んでみる
さてこの章では
> データベースを有効にする
にあった
- Question や Choice に Python からアクセスするためのデータベース API を作成
の部分を使用し、Django が提供する API で遊んでみましょう。

さてではPython対話シェルを起動し始めていきましょう。
- powershell
`py manage.py shell`
なぜ`python`をタイプして起動するのではなく`py manage.py shell`を使用しているのかについてはmanage.py が DJANGO_SETTINGS_MODULE 環境変数を設定してくれるからです。これにより、 Django に mysite/settings.py ファイルへの import パスが与えられます。

まあともかくこれによりデータベースAPIを使用することができるようになりましたね

ではどんどんコマンドを入力してみてDjangoデータベースAPIに慣れていきましょう。

-　powershell

```
>>> from polls.models import Choice, Question  # 先ほどmodels.pyで作成したモデルをimportしています。

# まだ(Question)質問はないのでモデルにレコードは存在していませんね。
>>> Question.objects.all()
<QuerySet []>

# では新たな質問（Question)を作成してみましょう！
# タイムゾーン設定はデフォルトでセッティングファイルで有効になっているので
# Djangoではdatetimeであるpub_dateはtzinfoオブジェクトとして取得されます。timezone.now()を使いましょう。
# datetime.datetime.now()と同じ動きをするのでこれを使ってくださいね。
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```



