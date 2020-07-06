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

#### 結果画面

これまでの流れから、どこに変更が必要かわかりますか？　`polls/views.py`、 `polls/templates/polls/results.html`、`polls/urls.py` に変更が必要だと思った方は、正解です。順を追っていきます。

`polls/views.py` では結果画面を出すための処理を書きます。

```diff
  class DetailView(generic.DetailView):
      model = Question
      template_name = 'polls/detail.html'

+ class ResultsView(generic.DetailView):
+     model = Question
+     template_name = 'polls/results.html'
+
  def vote(request, question_id):
```

次は、 `polls/templates/polls/result.html` です。

```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

結果画面の url を設定するため、 `polls/urls.py` を編集します。

```diff
  urlpatterns = [
      path('<int:pk>/', views.DetailView.as_view(), name='detail'),
+     path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
      path('<int:question_id>/vote/', views.vote, name='vote'),
  ]
```

ちょっと一息ついて考えてみましょう。現状では投票したあと、同じ詳細画面に戻っていました。ユーザーとしては投票したら結果が知りたいはずです。 `polls/views.py` を次のように変更して結果画面に移動するようにしましょう。

```diff
-         return HttpResponseRedirect(reverse('polls:detail', args=(question.id,)))
+         return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

これで投票の最低限の機能ができました。

##### 考えてみよう

- reverse は一体何をやっているんだろう？　自分の言葉で説明してみよう

#### 一覧画面

このままでは、質問の番号がわからない限り投票に参加ができません。直近で公開された質問を表示する一覧画面を作ります。これまで何度も画面を追加してきたので、なんとなく流れがイメージできていると思います。

まずは、 `polls/views.py` です

```diff
 from .models import Choice, Question

+class IndexView(generic.ListView):
+    template_name = 'polls/index.html'
+    context_object_name = 'latest_question_list'
+
+    def get_queryset(self):
+        """公開された順で5件取得"""
+        return Question.objects.order_by('-pub_date')[:5]

 class DetailView(generic.DetailView):
```

`get_queryset` というのはデータベースにどういう問い合わせをするか決めることができます。ここでは公開された順に 5 件取得しています。件数は簡単に変更できそうですね。

次に、 `polls/templates/polls/index.html` を新規作成します。

```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

このテンプレートでは、最新 5 件取得しようとしますが、もし何も質問がなければそれ専用のメッセージを出すようになっています。

`polls/urls.py` にも IndexView を追加します。

```diff
 app_name = 'polls'
 urlpatterns = [
+    path('', views.IndexView.as_view(), name='index'),
     path('<int:pk>/', views.DetailView.as_view(), name='detail'),
     path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
     path('<int:question_id>/vote/', views.vote, name='vote'),
```

これで一覧画面が完成します。Django Admin を使って質問を 5 件以上追加して、最大 5 件しか閲覧できないことを確認してください。

```diff
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

### 考えてみよう

- このウェブアプリケーションを改良するとしたらどう改良するといいでしょう（ヒント：あなたの作りたいものではなく、使う人の気持ちを考えましょう）
- `Question.objects.order_by('-pub_date')[:5]` とは、どういう意味でしょうか？　ここでは新しい順に表示しましたが、古い順に表示するにはどうしたらいいでしょうか？

## ウェブアプリケーションを公開しよう

ここまでとてもシンプルなウェブアプリケーションを作ってきました。お疲れさまでした。あなたのローカル環境には、あなたが作ったウェブアプリケーションが動いていますね。ウェブアプリケーションは、インターネットを通じて世界中の人々にサービスを使ってもらえるという魅力があります。これから実際にアプリケーションを公開してみます。

### Heroku

Heroku は無料から使えるウェブアプリケーションのホスティングサービスです。使い勝手も非常によく、 `git push` するだけでウェブアプリケーションがデプロイできます。

- アカウントがない人は、必ず作っておいてください
- Heroku CLI をインストールしておいてください

### Heroku をコマンドから使えるようにする

Heroku CLI をインストールしていれば、次のコマンドで heroku をコマンドラインから制御することができます。登録時の ID やパスワードが必要になります。

```
$ heroku login
```

### Heroku にデプロイする

Heroku にデプロイするためにいくつか設定が必要です。少々複雑ですが一度設定すれば、簡単にデプロイすることができます。

#### HTTP サーバーの追加

Heroku で Django を使うためにはアプリケーションサーバーの gunicorn を使います。あなたがこれまで利用していた `python manage.py runserver` でもウェブアプリケーションを利用できますが、これはあくまで開発用です。本番環境では、gunicorn という高速で軽量、そして安定している HTTP サーバーを利用します。

`requirements.txt` へ gunicorn を追加して、 gunicorn ライブラリをインストールしましょう。

```diff
Django
+ gunicorn
```

ライブラリをインストールするときは、　`pip install -r requirements.txt` とするんでしたよね。やってみましょう。

#### HTTP サーバー経由で起動してみる

gunicorn 経由でウェブアプリケーションを起動するためには、次のようにします。config ディレクトリの wsgi というモジュールを読み込ませています。

```
$ gunicorn config.wsgi
```

#### Heroku にウェブアプリケーションの起動方法を教える

Heroku は Procfile というファイルに起動方法を書いておくと、そのとおりに起動してくれます。ここでは、 gunicorn 経由でウェブアプリケーションを起動するため、 Procfile の中身を次のようにします。

`web:` は Heroku にウェブアプリケーションですよと伝えるための識別子です。後ろに続くコマンドが実際に実行されるコマンドです。

```Procfile
web: gunicorn config.wsgi --log-file -
```

#### Heroku に Python のバージョンを教える

Heroku はプログラミング言語の指定がない場合、自動的に推定して実行してくれます。たとえば Python アプリケーションを作ると自動的に Python 3.7.6 になります。今回は、 Python 3.8.3 を利用しているため、 `runtime.txt` に Python 3.8.3 を使う旨を記述します。

```runtime.txt
python-3.8.3
```

なお、 `runtime.txt` に書くテキストは、 https://devcenter.heroku.com/articles/python-support#specifying-a-python-version で決められています。なんでもバージョンが使えるとは限りません。

#### Heroku で URL を発行する

Heroku では一つ一つのアプリケーションに自動的に URL を発行してくれます。まず設定をするために、次のコマンドを打ちましょう。

```
$ heroku create
```

次のように URL が発行されているはずです。ここに出ている URL はあなたのウェブアプリケーションがデプロイされる URL です。

```
$ heroku create
Creating app... done, ⬢ protected-badlands-45530
https://protected-badlands-45530.herokuapp.com/ | https://git.heroku.com/protected-badlands-45530.git
```

#### Heroku にアクセスできるようにする

セキュリティのために、サーバーが受理するリクエストを制限することがあります。デフォルトでは開発環境(127.0.0.1)でのみアクセス可能となっています。そのまま Heroku へデプロイしてしまうとサーバーはリクエストを受け付けません(403 を返します)。

`config/settings.py` を次のように編集して、 heroku.com のサブドメインならリクエストを受理するようにしましょう。

```diff
-ALLOWED_HOSTS = []
+ALLOWED_HOSTS = [
+    "127.0.0.1",
+    ".herokuapp.com",
+]
```

#### 静的ファイルを配信できるようにする

Django は非常に高機能なフレームワークです。しかし、JavaScript や CSS、画像などのファイル配信は未対応です。このようなアプリケーション側でほとんど加工せずにファイルをそのまま配信することを静的配信と言います。

Django は静的配信はウェブアプリケーションの役割ではないというスタンスにいるわけですね。通常であれば、 Nginx や Apache のようなウェブサーバーが静的配信を行います（そしてそちらのほうが効率がいいです）。Heroku で動作させるためには、静的配信もウェブアプリケーションで実行する必要があります。

そこで whitenoise というライブラリをインストールします。 `requirements.txt` に追記しましょう。

```diff
 Django
 gunicorn
+whitenoise
```

```
$ pip install -r requirements.txt
```

whitenoise は、Django のミドルウェアとして動作します。ミドルウェアは、リクエストの最初や最後に自動実行される仕組みで必要に応じて追加したり削除したりできます。

whitenoise の公式ドキュメント http://whitenoise.evans.io/en/stable/#quickstart-for-django-apps によると、 `SecurityMiddleware` のすぐ下に入れてくださいと書いてあるため、そこに挿入します。

```diff
 MIDDLEWARE = [
     'django.middleware.security.SecurityMiddleware',
+    'whitenoise.middleware.WhiteNoiseMiddleware', # http://whitenoise.evans.io/en/stable/#quickstart-for-django-apps
     'django.contrib.sessions.middleware.SessionMiddleware',
     'django.middleware.common.CommonMiddleware',
     'django.middleware.csrf.CsrfViewMiddleware',
```

また、 http://whitenoise.evans.io/en/stable/django.html#make-sure-staticfiles-is-configured-correctly によれば、 `config/settings.py` の `STATIC_ROOT` を設定する必要があります。

```diff
 # https://docs.djangoproject.com/en/3.0/howto/static-files/

 STATIC_URL = '/static/'
+STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

#### いざデプロイ！

長い道のりでしたね。ここまでの設定で Heroku であなたのアプリケーションを公開する準備が整いました。変更したファイルをすべてコミットしてプッシュしましょう。

```
$ git add .
$ git commit -m"Heroku にデプロイするため"
$ git push heroku master
```

ずらずらとログが流れてくると思います。成功を祈りましょう。

#### Heroku の URL にアクセスしてみよう！

あなたの URL（https://protected-badlands-45530.herokuapp.com/ のような）にアクセスしてみてください。まだ polls のデータは入っていないので `/polls/1` にアクセスしてもエラーになります。

#### Heroku 上でマイグレーション、そして Django Admin でデータを作る

なぜデータがないのでしょうか？　そもそも以前やったようなデータマイグレーションを行っていないことに気づいた方もいるようですね。そうです、データマイグレーションとデータ作成が必要です。

`heroku run` というコマンドで heroku のサーバー上でコマンドを実行することができます。

```
$ heroku run python manage.py migrate
```

次に、Django Admin に入るためにスーパーユーザーを作成します。このアプリケーションはすでに世界中に公開されています。 作成するユーザーのパスワードは強固なものを利用してください。

```
$ heroku run python manage.py createsuperuser
```

あなたの URL に `/admin` を付け加えて Django Admin にアクセスしてみましょう。ユーザー名とパスワードは、いま入力したものです。ログインできたら、Question と Choice を作成してみてください。

入力できたら、再度アクセスしてアプリケーションで投票ができるか確認してください。

### ウェブアプリケーションを改良しよう

以上です。

#### スーパーユーザーを作る

データを作るためには、以前やったように Django Admin に入る必要があります

#### 静的ファイルの配信を Django

いんすとーる

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

```
