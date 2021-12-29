# はじめての Django アプリ作成、その 4 フォームとテンプレートビュー(generic view)

> 簡単なフォームを書く

それでは、前回のチュートリアルで作成した投票詳細テンプレート ("polls/detail.html") を更新して、HTML の <form> 要素を入れましょう。

- polls/templates/polls/detail.html
```
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend><h1>{{ question.question_text }}</h1></legend>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="Vote">
</form>
```
簡単に説明:

上のテンプレートは、各質問の選択肢のラジオボタンが表示するものです。
  
各ラジオボタンの value は、関連する質問の選択肢のIDです。

各ラジオボタンの name は "choice" です。
  
つまり、投票者がラジオボタンの1つを選択し、フォームを送信すると、POSTデータ choice=# （＃は選んだ選択肢のID）が送信されます。
  
これは、HTMLフォームの基本的な概念です。
  
We set the form's action to {% url 'polls:vote' question.id %}, and we set method="post". Using method="post" (as opposed to method="get") is very important, because the act of submitting this form will alter data server-side. Whenever you create a form that alters data server-side, use method="post". This tip isn't specific to Django; it's good web development practice in general.
  
forloop.counter は、 for タグのループが何度実行されたかを表す値です。
  
(データを改ざんされる恐れのある) POST フォームを作成しているので、クロスサイトリクエストフォージェリを気にする必要があります。
  
ありがたいことに、 Django がこれに対応するとても使いやすい仕組みを提供してくれているので、あまり心配する必要はありません。
  
手短に言うと、自サイト内を URL に指定した POST フォームには全て、 {% csrf_token %} テンプレートタグを使うべきです。
	

  

送信されたデータを処理するための Django のビューを作成しましょう。 
チュートリアルその 3 で、以下のような投票アプリケーションの URLconf を作成したことを思い出しましょう:
- polls/urls.py
```
  path('<int:question_id>/vote/', views.vote, name='vote'),
```
このとき、 vote() 関数のダミー実装も作成しました。今度は本物を実装しましょう。以下を polls/views.py に追加してください
- polls/views.py
```
    from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # 投票フォーム画面を再掲示する
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # 成功したら常時HttpResposeRedirect を返す
        # post dataによりユーザーが戻るボタンを押したときにデータが２回投稿されるのを防ぐ
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```
このコードには、これまでのチュートリアルで扱っていなかったことがいくつか入っています:

- request.POST は辞書のようなオブジェクトです。キーを指定すると、送信したデータにアクセスできます。この場合、 request.POST['choice'] は、選択された選択肢の ID を文字列として返します。 request.POST の値は常に文字列です。

- Django では、同じ方法で GET データにアクセスするために request.GET も提供しています。ただし、このコードでは、POST 呼び出し以外でデータが更新されないようにするために、request.POST を明示的に使っています。

- POST データに choice がなければ、 request.POST['choice'] は KeyError を送出します。上のコードでは KeyError をチェックし、 choice がない場合にはエラーメッセージ付きの質問フォームを再表示します。

- choice のカウントをインクリメントした後、このコードは、 通常の HttpResponse ではなく HttpResponseRedirect を返します。 HttpResponseRedirect はひとつの引数（リダイレクト先のURL）をとります (この場合にURLをどう構築するかについては、以下のポイントを参照してください)。

As the Python comment above points out, you should always return an HttpResponseRedirect after successfully dealing with POST data. This tip isn't specific to Django; it's good web development practice in general.

- この例では、 HttpResponseRedirect コンストラクタの中で reverse() 関数を使用しています。この関数を使うと、ビュー関数中での URL のハードコードを防げます。関数には、制御を渡したいビューの名前と、そのビューに与える URL パターンの位置引数を与えます。この例では、 チュートリアルその 3 で設定した URLconf を使っているので、 reverse() を呼ぶと、次のような文字列が返ってきます。
```
    '/polls/3/results/'
```
この 3 は question.id の値です。
リダイレクト先の URL は 'results' ビューを呼び出し、最終的なページを表示します。

チュートリアルその 3 で触れたように、 request は HttpRequest オブジェクトです。 HttpRequest オブジェクトの詳細は [リクエスト・レスポンスオブジェクトのドキュメント](https://docs.djangoproject.com/ja/4.0/ref/request-response/) を参照してください。

誰かが質問の投票すると、 vote() ビューは質問の結果ページにリダイレクトします。このビューを書きましょう:

-polls/views.py
```
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```
チュートリアルその 3 の detail() とほぼ同じです。テンプレートの名前のみ違います。この冗長さは後で修正することにします。

それでは polls/results.html テンプレートを作成します:

- polls/templates/polls/results.html
```
    <h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```
ブラウザで /polls/1/ を表示して投票してみましょう。
票を入れるたびに、結果のページが更新されていることがわかるはずです。選択肢を選ばずにフォームを送信すると、エラーメッセージを表示されるはずです。
**競合状態について**
これまで作ってきた vote() ビューのコードは、小さな問題を抱えています。最初にデータベースから selected_choice オブジェクトを取得し、そこで votes の新しい値を計算し、データベースにそれを戻して保存します。もしウェブサイトのユーザー 2 人が まったく同時に 投票しようとすると、誤りが発生します。votes の元の値が 42 だったとしましょう。その時、両方のユーザーに対して新しい値として 43 が計算され保存されます、しかし 44 が本来想定される値です。

この問題は、「競合状態」と呼ばれています。この問題に興味がある人は、[Avoiding race conditions using F()](https://docs.djangoproject.com/ja/4.0/ref/models/expressions/#avoiding-race-conditions-using-f) を読むと、この問題の解決方法がわかります。

> 汎用ビューを使う: コードが少ないのはいいことだ

detail() ( チュートリアルその 3 ) と results() ビューはとても簡単で、先程も述べたように冗長です。
投票の一覧を表示する index() ビューも同様です。

These views represent a common case of basic web development: getting data from the database according to a parameter passed in the URL, loading a template and returning the rendered template. Because this is so common, Django provides a shortcut, called the "generic views" system.

汎用ビュー(generic view)とは、よくあるパターンを抽象化して、 Python コードすら書かずにアプリケーションを書き上げられる状態にしたものです。

これまで作成してきた poll アプリを汎用ビューシステムに変換して、 コードをばっさり捨てられるようにしましょう。変換にはほんの数ステップしかか かりません。そのステップは:

URLconf を変換する。
古い不要なビューを削除する。
新しいビューに Djangoの汎用ビューを設定する。
詳しく見てゆきましょう。

**なぜコードを入れ換えるの？**
一般に Django アプリケーションを書く場合は、まず自分の問題を解決するために汎用ビューが適しているか考えた上で、最初から汎用ビューを使い、途中まで書き上げたコードをリファクタすることはありません。ただ、このチュートリアルでは中核となるコンセプトに焦点を合わせるために、わざと「大変な」ビューの作成に集中してもらったのです。

電卓を使う前に、算数の基本を知っておかねばならないのと同じです。
    
> URLconf の修正

まず、 URLconf の polls/urls.py を開き、次のように変更します:

- polls/urls.py
```
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
2つ目と3つ目のパス文字列に一致するパターンの名前が <question_id> から <pk> に変更されたことに注意してください。

> views の修正

次に、古い index 、 detail 、と results のビューを削除し、代わりに Django の汎用ビューを使用します。
これを行うには、 polls/views.py ファイルを開き、次のように変更します:
- polls/views.py
```
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.
```
ここでは、ListView と DetailView を使用しています。
これらのビューはそれぞれ、「オブジェクトのリストを表示する」および「あるタイプのオブジェクトの詳細ページを表示する」という二つの概念を抽象化しています。

- 各汎用ビューは自分がどのモデルに対して動作するのか知っておく必要があります。これは、 model 属性を使用して提供されます。
- DetailView 汎用ビューには、 "pk" という名前で URL からプライマリキーをキャプチャして渡すことになっているので、 汎用ビュー向けに question_id を pk に変更しています。


デフォルトでは、 DetailView 汎用ビューは <app name>/<model name>_detail.html という名前のテンプレートを使います。

この場合、テンプレートの名前は "polls/question_detail.html" です。

template_name 属性を指定すると、自動生成されたデフォルトのテンプレート名ではなく、指定したテンプレート名を使うように Django に伝えることができます。

また、 results リストビューにも template_name を指定します。

これによって、 結果ビューと詳細ビューをレンダリングしたとき、（裏側ではどちらも DetailView ですが）それぞれ違った見た目になります。

同様に、 ListView 汎用ビューは <app name>/<model name>_list.html というデフォルトのテンプレートを使うので、 template_name を使って ListView に既存の "polls/index.html" テンプレートを使用するように伝えます。

このチュートリアルの前の部分では、 question や latest_question_list といったコンテキスト変数が含まれるコンテキストをテンプレートに渡していました。

DetailView には question という変数が自動的に渡されます。

なぜなら、 Django モデル (Question) を使用していて、 Django はコンテキスト変数にふさわしい名前を決めることができるからです。

一方で、 ListView では、自動的に生成されるコンテキスト変数は question_list になります。

これを上書きするには、 context_object_name 属性を与え、 latest_question_list を代わりに使用すると指定します。

この代替アプローチとして、テンプレートのほうを変えて、新しいデフォルトのコンテキスト変数の名前と一致させることもできます。

しかし、使用したい変数名を Django に伝えるだけのほうが簡単でしょう。

サーバを実行して、新しく汎用ビューベースにした投票アプリケーションを使ってみましょう。

汎用ビューの詳細は、[汎用ビューのドキュメント](https://docs.djangoproject.com/ja/4.0/topics/class-based-views/) を参照してください。

フォームや汎用ビューを使いこなせるようになったら、 チュートリアルその5 に進んで、投票アプリのテストについて学びましょう。
