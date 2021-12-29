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

# さてではここで忘れてはいけないのがオブジェクトをデータベースに保存することです。
忘れないようにね！ 
>>> q.save()

# 今qには自動付与されたidがある状態ですね。確認しましょう。
>>> q.id
1

# 属性して指定を用いて値にモデルフィールドの値にアクセスして確認してみましょう。
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# 値を変えてみて保存してみよう！
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all()コマンドを使用してQuestionsモデルにあるデータをすべて表示してみましょう。
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```
ここでちょっと待ってください！ 最後の<Question: Question object (1)> は、全く何の意味か分かりませんね。なので(polls/models.py ファイル内にある) Question モデルを編集してこれを修正しましょう。 __str__() メソッドを Question と Choice の両方に追加します。このメソッドは文字列として処理したい呼び出しの時に自動で呼び出され文字列に変換してくれます。

またinteractive shellでの利便性のためだけでなくadminサイトでの表現にも関わってくるので__str__()メソッドは忘れないようにしてくださいね！

- polls/models.py
```
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

モデルクラスにクラスメソッドを追加してみましょう。
ここではさっきやったtimezoneの操作機能を搭載するためにpythonの標準モジュールからdatetime と　django.utils からtimezone をimportしています。

- polls/models.py
```
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
        #現時刻から一日以内にpubishしたか否か
```


さてでは変更を保存して、もう一度 python manage.py shell を実行して新しい Python 対話シェルを始めましょう:

- powershell
```
>>> from polls.models import Choice, Question

# __str__()が動くことを確認してみましょう。 
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# djangoは豊富なデータベース操作APIを提供してくれているので使ってみましょう。

>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# 今年publishした質問を表示させてみましょう。

>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# 存在しないIDをリクエストしてみてエラーが出ることを確認してみよう。

>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# プライマリーキーを入手するのが一般的なのでdjangoではプライマリーキーの操作にも対応しています。
# 次の操作はQuestion.objects.get(id=1).と同一の操作とみなしても今のところは問題ありません。
>>> Question.objects.get(pk=1)
<Question: What's up?>

# 前の操作で追加したクラスメソッドが動くことを確認してくださいね。

>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# では質問(Question)にいくつか選択肢(Choice)を追加していきましょう。
# クリエイト命令は新しい選択肢（Choice）オブジェクトを生成します。SQL分のようなinsert文ではありませんよ。 
# 使用可能なChoiceオブジェクトに質問（Choice)を追加し出来た質問オブジェクトを返すのです。
# djangoはForeignKey relationの反対側を保持するためにsetを作ります。
# 例) a question's choice #APIでアクセスできます。
# まあこういってもわかりませんね。とりあえずやってみましょう。

>>> q = Question.objects.get(pk=1)

# ｑの関係オブジェクトセットからすべての質問を取り出します。まあ何もないけどね。

>>> q.choice_set.all()
<QuerySet []>

# 質問を作ってみましょう。
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# 逆に選択肢(Choice)オブジェクトも関係ある質問（Question)にアクセスできるAPIを持っていますよ！

>>> c.question
<Question: What's up?>

# もちろん当然質問からも選択肢に行けます。

>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# APIは自動的に質問と選択肢の関係を追跡します。
# アンダースコア二個で関係を指定します。
# ”関係する質問の中から一年いないにpublishされた全ての選択肢を見つけてこい”をやってみよう。
# (前に作ったcurrent_yearを再利用してやってみよう）

>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# delete()を使って一つ選択肢を消してみよう。
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

モデルの関係などについては詳細としてこれを読んでね↓
For more information on model relations, see [Accessing related objects][https://docs.djangoproject.com/en/4.0/ref/models/relations/]. For more on how to use double underscores to perform field lookups via the API, see [Field lookups][https://docs.djangoproject.com/en/4.0/topics/db/queries/#field-lookups-intro]. For full details on the database API, see our [Database API reference][https://docs.djangoproject.com/en/4.0/topics/db/queries/].

> Django Adminの紹介

djangoはあらかじめサイト管理者向けの一元化されたコンテンツ編集インタフェースを提供しています。その名もずばりadminサイトです！ハハハいい名前ですねではどう使うのか見てみましょう。

1. 管理ユーザーを作成する
まず最初に私達はadminサイトにログインできるユーザーを作成する必要があります。下記のコマンドを実行しましょう。

- powershell
```
py manage.py createsuperuser
```
好きなユーザー名を入力しEnterを押してください。

```
Username: admin(好きな名前)
```
希望するemailアドレスを入力するよう促されます:

```
Email address: admin@example.com
```
最後のステップはパスワードの入力です。2回目のパスワードが1回目と同じことを確認するため、パスワードの入力を2回求められます。パスワードは正常に入力出来ていても表示はされないのであしからず。

Password: **********
Password (again): *********
Superuser created successfully.


2. 開発サーバーの起動

Django adminサイトはデフォルトで使えるようになっています。開発サーバーを起動して探索を始めましょう。

```
py manage.py runserver
```
![画像](https://docs.djangoproject.com/ja/4.0/_images/admin01.png)

デフォルトでは translation機能がオンになっているので、setting.pyにあるLANGUAGE_CODE を設定すると、与えられた言語でログイン画面が表示されるようになります(Django に適切な翻訳があれば)。

3. admin サイトに入る
今回は、前のステップで作成したスーパーユーザーのアカウントでログインしてみましょう。
![画像](https://docs.djangoproject.com/ja/4.0/_images/admin02.png)

既にいくつかのタイプの編集可能なコンテンツがあるはずです（groups と users）。
これらは Django に含まれる認証フレームワーク ![django.contrib.auth][https://docs.djangoproject.com/ja/4.0/topics/auth/#module-django.contrib.auth] のおかげで使えるようになっています。

4. Poll アプリを admin 上で編集できるようにする¶
ところで、 polls アプリはどこにあるんでしょう？ admin のインデックスページを見ても表示されていませんね。やるべきことは1つです: Question オブジェクトがadmin インタフェースを持つということを、adminに伝える必要があります。これを行うために、ファイル polls/admin.py を開いてこのように編集しましょう

- polls/admin.py
```
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

5. admin の機能を探究してみる

さてQuestion を登録したので、 DjangoはadminインデックスページにQuestionを表示すべきという事がわかるようになりました。
![画像](https://docs.djangoproject.com/ja/4.0/_images/admin03t.png)

"Questions" をクリックしましょう。質問（questions） のための "change list" ページが表示されます。このページにはデータベース中のすべての 質問（question） が表示され、自由に選んで変更することができます。ここには以前作成した "What's UP?" question もありますね
![画像](https://docs.djangoproject.com/ja/4.0/_images/admin04t.png)
"What's up?" questionを編集するためにクリックしましょう
![](https://docs.djangoproject.com/ja/4.0/_images/admin05t.png)

以下の点に注意してください:

- フォームは Question モデルから自動的に生成されます。
- モデルのフィールドの型 (DateTimeField 、 CharField など) はそれぞれ異なる HTML 入力ウィジェットと対応しています。各種のフィールドは、自分自身を Django admin サイトでどう表示するか知っています。
- 各 DateTimeField は JavaScript ショートカットがついています。日付 (dates) のカラムには「今日 (today)」 へのショートカットとカレンダーポップアップボタンがあります。 時刻 (times) には「現在 (now)」へのショートカットと、よく入力される時刻のリストを表示するポップアップボタンがあります。

ページの末尾の部分には操作ボタンがいくつか表示されています:

- 保存 (Save) – 変更を保存して、このモデルのチェンジリストのページに戻ります。
- 保存して編集を続ける (Save and continue editing) – 変更を保存して、このオブジェクトの編集ページをリロードします。
- 保存してもう一つ追加 (Save and add another) – 変更を保存して、このモデルのオブジェクトを新規追加するための空の編集ページをロードします。
- 削除 (Delete) – 削除確認ページを表示します。

もし「Date published」の値があなたが以前 チュートリアルその1 で作成した questionと一致しないのであれば、それはおそらくあなたが TIME_ZONE で正しい値を設定することを忘れていたことを意味します。これを変更して、ページをリロードし、正しい値が表示されるか確認してください。

「今日」や「現在」ショートカットをクリックして、「Date published」を変更してみましょう。変更したら、「保存して編集を続ける」を押します。次に、右上に ある「履歴 (History)」をクリックしてみましょう。ユーザが管理サイト上でオブジェクトに対して行った変更履歴の全てを、変更時刻と変更を行ったユーザ名付きでリストにしたページが表示されます

![](https://docs.djangoproject.com/ja/4.0/_images/admin06t.png)

モデルの API や admin サイトに慣れてきたら、 チュートリアルその3 を読んで、 polls アプリにビューをさらに追加する方法を学習しましょう。
