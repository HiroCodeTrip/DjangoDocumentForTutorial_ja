# はじめての Django アプリ作成、その 4

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

```
```
  
