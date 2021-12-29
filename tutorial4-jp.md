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

```
  path('<int:question_id>/vote/', views.vote, name='vote'),
```
このとき、 vote() 関数のダミー実装も作成しました。今度は本物を実装しましょう。以下を polls/views.py に追加してください:
 
```
```
  
```
```
  
```
```
  
```
```
  
```
```
  
