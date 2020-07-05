# チュートリアルに必要なもの

- Git
- Python 3.8.3
- Django 3.0.8
- Heroku のアカウント

# エディター

とくにこだわりがなければ、Visual Studio Code を推奨します。

# 事前準備

## Heroku CLI のインストール

https://devcenter.heroku.com/articles/getting-started-with-python?singlepage=true

### Python 環境構築

https://developer.mozilla.org/ja/docs/Learn/Server-side/Django/development_environment に従って Python と Python 仮想環境を用意しておいてください。ただし、Python のバージョンは 3.8.3 にしてください。

# チュートリアル

## requirements.txt に Django を追加しよう

requirements.txt は、ライブラリのリストを記載します。あなた以外の誰かがあなたのアプリケーションを再構築する上で重要になります。また、デプロイするときにも使われます。

次のように追加してください。

```requirements.txt
Django
```

requiremnets.txt をもとにライブラリをインストールします。これで Django がインストールされます。

```
$ pip install -r requirements.txt
```

## Django のプロジェクトを始める

さっそく始めましょう。本番でも使えるようなウェブアプリケーションを１から作るのはかなり大変な作業です。
Django は最初に必要最低限のファイルを自動的に用意するコマンドをもっています。フレームワークなどで、最初にファイル群を生成するコマンドのことをスキャフォールドと言います。

```
$ django-admin startproject config .
```

自動的にファイルやフォルダがたくさん作成されました。次のようになっているはずです。
`config` はプロジェクト全体の設定が書かれたディレクトリです。のちほど説明します。

```
.
├── README.md
├── config
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
└── requirements.txt
```

`manage.py` はこれからたくさんお世話になるツールです。

## 開発サーバーを立ち上げる

プログラムを書いていく前に、現在のアプリケーションを確認しましょう。 `You have 17 unapplied migration(s).` というような警告が出ますが、いまは無視して問題ありません。

```
$ python manage.py runserver
```

`http://127.0.0.1:8000/` へアクセスするとアプリケーションを確認できます。ロケットが飛んでいるようなら成功です。
確認できたら、コンソールへ戻り Ctrl+C でプログラムを停止します。

## 投票アプリケーションを作る準備

これからとてもシンプルなアプリケーションを作ります。再び manage.py の力を借ります。

```
$ python manage.py startapp polls
```

`polls` というディレクトリが自動的に作成されました。この中のファイルに手を入れることで、投票アプリケーションを作っていきます。

## もっともシンプルな画面を作る

いよいよ、あなたの力でウェブアプリケーションの画面を作ります。画面を作るためには、最低限 `polls/views.py` と `polls/urls.py`、 `config/urls.py` を変更する必要があります。

`polls/views.py` を開いて次のように書いてください。

```polls/views.py
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

これが、どのような表示になるのか想像してみてください。

次に、 `polls/urls.py` を作成します。注目してほしいのは、 path 関数の第 2 引数です。 views.py ファイルの index 関数を渡しています。これは先ほど書いたものです。

```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

あと一歩で設定が終わります。もうすぐあなたの最初の画面がお披露目になります。

`config/urls.py` を次のように書き換えます。ここでは、 `polls/urls.py` を `include` していますね。 `include` が何かはまだわかりませんが。

```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

疑問はたくさんあると思いますが、動作を確認してみましょう。 manage.py の出番です。

```
$ python manage.py runserver
```

起動できたら、 `http://127.0.0.1:8000/polls/` へアクセスしましょう。あなたが作った最初の画面が出ているはずです。

### 遊んで理解を試しましょう

変更をする前に、どうなるか予想してみるのがコツです。

- 表示されるテキストを変更できますか？
- `http://127.0.0.1:8000/polls/` ではなくて `http://127.0.0.1:8000/投票/` にしたい場合は、どう変更すればいいでしょうか
- なぜ `polls/urls.py` と `config/urls.py` の２つを変更しなければならないのでしょうか？　１つだけにすることはできるでしょうか？

## データベースを使った開発

Django はデフォルトでは SQLite を使用します。ウェブアプリケーションにおいて、本番環境ではスケーラビリティの観点で使われないデータベースです。しかし、とても手軽に扱えるという特徴があり、ローカル環境で使う分にはもってこいです。

### タイムゾーン設定

データベースに記録される時間は、UTC が標準になっています。日本だけに閉じたサービスなどは、JST（日本標準時）にしたほうがわかりやすいです。設定を変更するには、 `config/settings.py` を次のように変更します。

```diff
- TIME_ZONE = 'UTC'
+ TIME_ZONE = 'Asia/Tokyo'
```

`-` で始まる行は削除、 `+` で始まる行は追加、することを指します。

### データベースの準備

データベースのデータは長期間に渡り保存されます。その長い期間の中で、新しい情報を保存するようにしたり、不要な情報を丸々削除することが必ず起こります。テーブルを追加したり、削除したりすることも含まれます。このようなデータベースの構造変更（変更に関係のないデータを保持したまま）をデータベースマイグレーションと言います。

以前、次のような警告をあなたは目にしていました。

> You have 17 unapplied migration(s).

この警告を消すには、マイグレーションを実行します。

```
$ python manage.py migrate
```

OK が、ずらずらと出ることが正常です。

### 投票に必要なデータ

あなたは投票アプリケーションを作っています。投票に必要なデータはなんでしょうか？　ここでは次の情報だけ扱ってみましょう。

- 質問文
- 質問の公開日時
- 選択肢
- 選択肢が選ばれた数

この情報を扱うために、 `polls/models.py` を変更します。

```polls/models.py
from django.db import models


class Question(models.Model):
    """質問"""
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    """質問の選択肢"""
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Question はテキストと、公開日時(pub_date)を持っており、Choice はどの質問に紐づくか、選択肢のテキスト、投票数を持っています。各変数の右辺は、データの型が書かれています。たとえば、 models.CharField は文字列、 models.IntegerField は数値です。この書き方のおかげで、データベースのデータ型とアプリケーションのデータ型が紐づくようになっています。

勘が鋭い方は気づいていると思います。`polls/models.py` を編集したことでデータが増えました。つまり、マイグレーションをしなければなりません。

### polls のデータベースマイグレーション

`python manage.py startapp` でアプリケーションを作成し、 `models.py` を変更したあとはマイグレーションが必要です。その前に、 `config/settings.py` を編集して、 polls をプロジェクトに追加します。

```diff
 INSTALLED_APPS = [
+    'polls.apps.PollsConfig',
     'django.contrib.admin',
     'django.contrib.auth',
     'django.contrib.contenttypes',
     'django.contrib.sessions',
     'django.contrib.messages',
     'django.contrib.staticfiles',
 ]
```

polls ディレクトリにある、 `apps.py` の `PollsConfig` クラスを登録しました。これでプロジェクトから認識されるようになります。apps.py は各アプリケーションの設定を置いておく場所です。

いまは、 `polls/models.py` にデータ型などの情報が書かれているため、マイグレーションファイルを自動生成することができます。マイグレーションファイルは、どのようなマイグレーションを行うのか書かれたファイルです。

次の操作によって `polls/migrations/0001_initial.py` というマイグレーションファイルが生成されます。中身は数行のシンプルなファイルです。

```
$ python manage.py makemigrations polls
```

まだデータベースマイグレーションは行われていません。適用するには、 migrate コマンドを使います。

```
$ python manage.py migrate
```

次のような手順でデータマイグレーションは進行します。

- models.py を変更
- python manage.py makemigrations
- python manage.py migrate

#### 考えてみよう、調べてみよう

- なぜマイグレーションファイルを作る必要があるんだろうか？（ヒント：Django はさまざまなデータベースを扱えます）
- models.CharField はデータ型を表している。どんな種類があるんだろうか？
- データマイグレーションが必要かどうかチェックするコマンドはないだろうか？

### 高機能な管理画面 Django Admin で質問や回答をデータを用意する

Django の魅力の１つに Django Admin の存在があります。 Django Admin は高機能な管理画面で、最初から用意されています。簡単にデータを入力、編集できることはもちろん、管理ユーザーごとの権限管理もサポートしています。

#### Django Admin にログインできるユーザーを作成する

次のコマンドで特権ユーザーを作成します。特権ユーザーは、すべての権限をもったユーザーです。実務ではもっと細かく権限を采配します。

```
$ python manage.py createsuperuser
```

ユーザー名とメールアドレス、パスワードを問われるので適当に入力します。

#### Django Admin で入力する

開発サーバーを起動します。

```
$ python manage.py runserver
```

管理画面の URL（`http://127.0.0.1:8000/admin` ）にアクセスしてログインしてみましょう。
本来ならこの画面で、 poll アプリケーションのために質問や選択肢を編集するはずですが、それっぽいものが見当たりません。

編集できるようにするためには、 `polls/admin.py` を編集します。

```polls/admin.py
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

開発サーバーは立てたままで、Django Admin をリロードすると項目が現れます。これで質問を追加、編集できるようになりました。

#### 試してみよう

- 回答を Django Admin で追加、編集できるようにするにはどうしたらいいか？（ヒント admin.site.register を使う）

### 回答も Django Admin で登録

```polls/admin.py
from django.contrib import admin

from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```

### 質問の一覧画面、結果画面を作成する

あなたが最初に作った画面は、インタラクティブ要素が一切なくそっけないものでした。ここから、質問に対して一覧したり、結果を見たり、実際に投票できるようにしていきます。

#### views.py の大幅変更

`polls/views.py` を変更します。Django には一覧画面、詳細画面をとても簡単に作成できる generic.ListView、 generic.DetailView というクラスがあります。使ってみましょう。

少しずつ進めるため、一覧画面を削除して詳細画面のみに絞ります。詳細画面では、投票できるようにするため、 vote 関数は定義しておきます。

```polls/views.py
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:detail', args=(question.id,)))
```

`template_name` というのは、どういう見た目にするか書かれたテンプレートです。見た目と表示処理を分離することで変更しやすくなっています。

#### template を使った表示

`polls/template/polls` というディレクトリを作成します。 polls という名前が何度も出てきて気持ち悪いですが、 Django の仕様です。まずは、 `polls/template/polls/detail.html` を用意し、詳細画面を作ります。

```polls/template/polls/detail.html
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```

views.py の index 関数はなくなり、DetailView クラスや vote 関数が増えたので、 `polls/urls.py` も忘れずに編集します。generic の View は as_view()を呼び出す必要があります。

```diff
  from django.urls import path

  from . import views

+ app_name = 'polls'
  urlpatterns = [
-     path('', views.index, name='index'),
+     path('<int:pk>/', views.DetailView.as_view(), name='detail'),
+     path('<int:question_id>/vote/', views.vote, name='vote'),
  ]
```

開発サーバーを起動して、　`http://127.0.0.1:8000/polls/1/` へアクセスしてみてください。投票画面っぽいものができています。実際に投票も動作します。

しかし、結果画面がないためどれくらい投票されているのかわかりません。次は結果画面を作ります

```diff
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]

`polls.apps.PollsConfig` を追加しています。

`polls/views.py` は表示に関わる部分、 `polls/models.py` はデータに関わる部分を持っていることがなんとなくわかってきていると思います。

データベースのテーブルを作成することを

requirements.txt

```

\$ django-admin

```

## 必要ライブラリのインストール

### macOS の場合

```

\$ env LDFLAGS="-I/usr/local/opt/openssl/include -L/usr/local/opt/openssl/lib" pip install -r requirements.txt

```

### Windows の場合

```

pip install -r requirements.txt

```

## Django を始める

```

$ mkdir project
$ cd project
\$ django-admin startproject config .

```

```
