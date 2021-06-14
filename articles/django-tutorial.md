
## 最初に

本格的なWebアプリケーションを作成したいのでPythonのフレームワークDjango（読み方：ジャンゴらしい。ディーどこ行った。）についてチュートリアルをこなしながら学んで行こうと思う。実際に作成するアプリは質問に対して回答して投票を表示するアプリになる。

この記事は公式チュートリアルの1〜4までに気になった事躓いた事をまとめていく。全部で1〜7まであるが4までにアプリは完成する。

5からはテストコード等を書いていくのでボリュームが多くなるため、前編として今回の記事を投稿する。後編も必ず書こうと思う。

## 今回のチュートリアルで作成したもの

質問一覧のページがあって、そこから質問に対して投票を行う。その後今まで投票された数を表示するページリダイレクトされる。

![django-tutorial](https://user-images.githubusercontent.com/23703281/119465477-a8c5db00-bd7e-11eb-8245-df9008d26b42.gif)

## VSCodeのPython用の拡張機能をインストールする。

コードを書くにあたって構文エラーは事前に無くしたいので、拡張機能をインストールする。マイクロソフトがPython用に提供しているものがあるのでそちらをインストールする。

自分はanaconda環境での使用をしているので下記の通知が出てきた。

`terminal.integrated.inheritEnv` を `false` にした方が良いらしい。

```
We noticed you're using a conda environment. If you are experiencing issues with this environment in the integrated terminal, we recommend that you let the Python extension change "terminal.integrated.inheritEnv" to false in your user settings.
```

VSCodeの code > preference > settings に `terminal.integrated.inheritEnv` と入力して出てきたチェックを外す。

[【Mac/Python】VSCodeターミナル動作が通常ターミナルと違う時 | ゆうきのせかい](https://yuki.world/vscode-terminal-python/)

それだけだと `Import "django.contrib" could not be resolved from source` という警告がでたままなので、赤枠の箇所をクリックしてDjangoをインストールした環境を選択すると警告文が消えます。

![](https://storage.googleapis.com/zenn-user-upload/koqbmvs8nwqpy3bbt2tbjzlhutyx)

## インストール

好きなフォルダーを作成して、そこに開発していく。Pythonの環境構築は自分は下記のように行っている。

[https://techblog-pink.vercel.app/posts/cc111706c3167](https://techblog-pink.vercel.app/posts/cc111706c3167c)

anacondaで仮想環境を作成して、Djangoをインストールしていく。

```bash
pip install Django
```

開発を始めたいフォルダに移動して下記を実行する。

```bash
django-admin startproject mysite
```

mysiteというディレクトリが生成される。

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

フォルダはこのような構成になっている。 `mysite/mysite` となっているのが不思議だ。外側の `mysite` はDjangoのシステム的には何でも良いらしい。

`urls.py` はプロジェクトのURLを宣言する。目次のような機能を提供する。

サーバを起動する。

実行すると [http://127.0.0.1:8000/](http://127.0.0.1:8000/) でサイトにアクセス出来るようになる。

最初はデータベース等の設定をしていないので、ターミナルに警告が表示されるが他に問題がなければアクセス出来る。 `ctr + c` でサーバを終了することが出来る。

```bash
python manage.py runserver
```

  

## GitHubで管理する

これから開発していくので、Githubにコードをあげて進捗を管理したい。しかし、このままだと `settings.py` に書かれた `SECRET_KEY` も一緒にアップロードしてしまうので別ファイル `local_settings.py` に `SECRET_KEY` を書く。ちなみに私は気付かずに一度GitHubに、そのままアップしてしまった。そのため新たに `SECRET_KEY` を作成する手間が掛かる。

`local_settings.py`

```python
SECRET_KEY = 'settings.pyにあったシークレットキーまたは、新たに作成したもの'
```

この変数を `settings.py` で読み込んでいく。下記のコードを追加する。

```python
try:
    # 同じ階層のlocal_settingsファイルからSECRET_KEYをkeyとして読み込む。
    # 参考記事等では .local_settingsとなっているが、local_settingsはファイルなので必要ない
    # 将来的に複雑にファイルを分ける必要が出てフォルダにする場合は .が必要となる。
    # ダメだった .local_settingsが正しかった。Django環境だとパッケージとして見なされるのか...
    from .local_settings import SECRET_KEY as key
except ImportError:
    pass

# シークレットキーの部分を読み込んだ変数に置き換える。
SECRET_KEY = key
```

実際に `test.py` と `test2.py` を同じ階層に作成してどのように読み込まれるか調べてみた。

`test.py`

```python
hello = "hello"
```

`test2.py`

```python
# .testとするとImportError: attempted relative import with no known parent package
# と表示される。
# asを付けない場合は helloで読み込まれる。
from test import hello as A

print(A)
```

`settings.py` と同じ環境に出来ていると思ったが、全然違った。 `local_settings.py` は パッケージとして認識されるが、 `test.py` はパッケージとして認識されないので `.` を使用するとエラーになる。そもそも `python test2.py` と直接実行しているので `settings.py` とは違う実行状況になる。

詳しくは[ここで回答を頂いている。](https://ja.stackoverflow.com/questions/75892/python-django-%e3%81%aeimport%e3%81%ae%e9%9a%9b%e3%81%ab%e4%bd%bf%e7%94%a8%e3%81%99%e3%82%8b-%e3%83%89%e3%83%83%e3%83%88-%e3%81%8c%e3%81%a9%e3%81%ae%e3%82%88%e3%81%86%e3%81%ab%e4%bd%bf%e7%94%a8%e3%81%95%e3%82%8c%e3%82%8b%e3%81%ae%e3%81%8b%e5%88%86%e3%81%8b%e3%82%89%e3%81%aa%e3%81%84/75895?noredirect=1#comment85968_75895)

## SECRET_KEYの作成

自分は知らずにシークレットキーをGitHubにあげてしまったので新たに作り直す必要があるのでシークレットキーを生成してくれるプログラムを実行する。これを直接ターミナルで実行するとシークレットキーが生成されるのでそれを `local_settings.py` に貼り付ける。

`get_random_secret_key.py`

```python
from django.core.management.utils import get_random_secret_key

secret_key = get_random_secret_key()
text = 'SECRET_KEY = \'{0}\''.format(secret_key)
print(text)
```

これでようやくチュートリアルに専念してコードを書き進める事が出来る。

## プロジェクトとアプリ

Djangoではプロジェクトの中にアプリが含まれる。なので特定のDjangoで作成されたWebサイト全体をプロジェクトと呼び、その中に含まれる小規模な投票アプリ、ログシステムをアプリと呼ぶ。

## アプリを作成する。

`manage.py` と同じ階層に移動して

```bash
python manage.py startapp polls
```

を実行すると `polls` というフォルダが生成される。

```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

これでプロジェクトにpollsというアプリが作成された事になる。

## Viewを作成する。

URLからパスにアクセスがあって、その際に実行する関数がViewになるここでHTMLファイルを返したりと処理を決めることが出来る。

`views.py` に記述する。

```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

viewメソッドの第一引数には必ずHttpRequestクラスを受け取る。

引数 `request` には今ユーザがアクセスしているURLやIPアドレスなどの情報が入ってくる。そして戻り値としては HttpResponseクラスを返す必要がある。

実際にHttpResponseとHttpRequestの中身がどんな感じになっているのか気になる人は下記のページで確認できます。

ざっくりですが、 `views.py` があって実行されるとクラスの中身はこんな感じに格納されているみたいです。

```python

def sample(request):
    print(request)
    response = HttpResponse('')
    print(response)
    return response

# 実行結果
# <WSGIRequest: GET '/hello/'>
# <HttpResponse status_code=200, "text/html; charset=utf-8">
```

[Djangoはリクエストを受け取ってレスポンスを返しているだけです【詳しく解説】](https://codor.co.jp/django/request-and-response)

`views.py` だけではURLと紐づいていないので URLconfを作成する。

## URLconf `urls.py` を作成する

URLconfを作成するには `urls.py` というファイルを `views.py` と同じ階層に作成する。mysiteフォルダ内には `urls.py`  がすでにあるので pollsフォルダ内に作成する。中身はこんな感じになる。

### include()を使ってアプリのURLを結び付ける。

`polls/urls.py`

```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

 `<int:question_id>` が `views.py` 関数の引数として渡される。この `<>` を使用すると、URLの一部がキャプチャされ渡される。文字列 `:quesiton_id>` は一致するパターンを定義し、 `<int:` の部分はURLパスに当てはまる値の型を指定している。なので `<str:, <slug:` などもある。 

そして、これをmysiteフォルダ内の `urls.py` に結びつけてあげる必要がある。一応こっちが最初に読み込まれるのでここに後から追加したアプリのURLを `include()` を使って追加していくイメージになる。

`mysite/urls.py` 

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

これでpollsアプリのURLを結び付けることができた。サーバを起動して [http://localhost:8000/polls/](http://localhost:8000/polls/) にアクセスするとViewが返されるようになる。

### path()の引数

4つの引数を受け取ることが出来る。そのうち `route` と `view` の2つは必須で残り `kwargs` と `name` は省略出来る。

- route：URLパターンを含む文字列が入る。リクエストを処理する際に `urlpatterns` を順番にみて最初にマッチしたものを取り出す。このパターンはGET、POSTのパラメータに影響は受けないあくまでURLパスだけを見る。
- view：URLがマッチしたら、そこに付随するView関数を返す仕組みになっている。
- kwargs：任意のキーワード引数を辞書としてView関数に渡せる。
- name：URLに名前を付ける事で `reverse()` を使って呼び出せるようになり、htmlのテンプレートでformに指定するURLを変更する際に動的にURLが変更されるようになる。

`polls/urls.py` がこんな感じだとして

```python
from django.urls import path

from . import views
import re

urlpatterns = [
    path('', views.index, name='index'),
    path('<int:number>', views.subView, name='suburl'),
]
```

試しに下記の `views.py` を実行してみる。

```python
from django.http import HttpResponse
from django.urls import reverse

def index(request):
    urlName = reverse('index')
    print(urlName)
    return HttpResponse("Hello, world. You're at the polls index.{0}".format(urlName))

# 実行結果
# /polls/
```

こうすると例えば `pollllllls/urls.py` とURLを変更してもコードを変更する必要がないので、変更に強いコードになります。

※pathのnameについて

[python、djangoのurls.pyで設定するnameってなんやねん？？ - Qiita](https://qiita.com/sr2460/items/11a1129975913ed584d3)

## DjangoのURL

ユーザがDjangoで作られたサイトにアクセスした際にどのような処理が走るのか。

1. ROOT_URLCONFに設定されているURLを確認する。（HttpRequestオブジェクトにurlconfという属性が設定されている場合はその値をROOT_URLCONFとする。）
2. urlpatternsという名前の変数を探す。この変数値は `django.urls.path()` または `django.urls.re_path()` インスタンスのsequenceでなければならない。
3. urlpatternsから順番に要求されたURLパターンを探す。
4. マッチしたらViewを返す。
5. マッチしなかったらエラーハンドリングビューを返す。

URLconfのサンプル

pathの左側がマッチするURL（route）, 右側がマッチしたら呼び出される関数（view）

```python
from django.urls import path

from . import views

urlpatterns = [
    # /articles/2003/にアクセスした場合
    # Views.special_case_2003(request)を呼び出す。最後の/もしっかりないとマッチしない。
    # 引数としてrequestが関数に渡る。
    path('articles/2003/', views.special_case_2003),
    path('articles/<int:year>/', views.year_archive),
    path('articles/<int:year>/<int:month>/', views.month_archive),
    path('articles/<int:year>/<int:month>/<slug:slug>/', views.article_detail),
]
```

`/articles/2005/03/` というアクセスがあった場合上記のパターンの中から`views.month_archive(request, year=2005, month=3)` という `views.py` （上記で記したviews.pyとは別で例としてあげてるurls.pyに対応するviews.pyがあったらという話で見てもらいたい。）に書かれたviewメソッドに引数を渡して呼び出す事になる。引数 `request` にはHttpRequestクラスが入り 今ユーザがアクセスしているURLやIPアドレスなどの情報が入ってる。

`/articles/2003/03/building-a-django-site/` なら最後のパターンにマッチして、このように `views.article_detail(request, year=2003, month=3, slug="building-a-django-site")` 関数を呼び出す。

## タイムゾーンの設定

デフォルトではUTCと世界標準時間になっているので、ここで日本時間に変更しておきたいと思う。 `settings.py` の `TIME_ZONE = 'UTC'` を `TIME_ZONE = 'Asia/Tokyo'` に変更する。

## データベースの作成

ここでデータベース用語についてまとめて置く。

![](https://storage.googleapis.com/zenn-user-upload/ury5lr13z2uiguvdb3hajfo31gll)

- カラム：縦列の事を指し、別名では列と呼ばれる。
- カラム名：列全体に付けられた名前
- フィールド：データが入っている場所。
- フィールド名：そのデータが入っているカラム名を指す。
- 項目名：フィールド名を指す。
- テーブル：Excelでいうシートのようなもの。
- レコード：データそのものを指す言葉になる。もう一つは横列の事を指し、行と呼ばれる。そして行をロウと呼ぶこともある。

これからデータベースを設定していく、チュートリアルの段階なのでまずは複雑な設定の必要がないSQLiteを使用していく。

```bash
python manage.py migrate
```

実行すると `settings.py` に書かれた `INSTALLED_APPS` の設定を参照して `mysite/settings.py` ファイルのデータベース設定に従って必要な全てのデータベーステーブルを作成する。

## migrate（マイグレート）するとは

データベースを削除してから作り直すと、DBに保存されている情報が全て削除されてしまう。こういった事態を回避する方法として、データベースマイグレーションを行う方法が生まれた。マイグレーションとは、DBに保存されているデータを保持したまま、テーブルの作成やカラムの変更などを行うことが出来る。

## モデルの作成 `models.py`

`models.py` はデータを保存したり、取り出したりするときの設定を記録するファイルになる。データベースの取扱説明書と書かれることが多い。

### `models.py` からデータベースが作成される流れ

1. models.pyファイルでモデル（データベースの型）を作成する。
2. migrationファイルを作成する。
3. migrate（最終的にはおそらくSQL文に変換されて、データベースに命令を出してデータベースを作成する。）する。ここで上記のファイルを元にデータベースが作成される。

つまりモデルはデータベースのレイアウトとそれに付随するメタデータになる。

簡単な例を示す。（チュートリアルとは関係ないモデル）

```python
# modelsモジュールを読み込むこれはデータベースを作成するのに必要な機能が格納されている。
from django.db import models

# データベース作成機能を継承してモデルを書き込んでいく。
class BookModel(models.Model):
    # booknameという文字を入力することが出来るフィールドを作成する命令をだす。
    # bookname = models.CharField(max_length = 50)とすると50文字までと制限をかけれる。
    # CharFieldには必須の引数がありmax_lengthを設定しないといけない。
    bookname = models.CharField(max_length = 50)
    # CharFieldとほぼ同じで文字列を扱うがTextFieldの方がデータの読み出し等にコストがかかるらしい。
    summary = models.TextField()
    # 整数値を入れるフィールドを作成する。
    rating = models.IntegerField()
```

## ForeignKey（外部制約キー）

これを使ってこの後2つのモデルを双方向に参照できるようにするのだが、その前にデータベースにおいて外部制約キーまたは外部キーとも呼ばれるがどのような役割を果たしているのかみていこうと思う。

外部キーとは関係データベースにおいてデータの整合性を保つための制約（参照整合性制約）

外部キーに設定されている列（子テーブルのカラム）には、参照先となるテーブルの列内（親テーブルのカラム内）に存在している値しか設定できない。

そのため、外部キーに設定されている子テーブルの列内に親テーブルの列内に存在しない値を追加しようとするとエラーになる。

なので新しく値を追加したい場合は一度、親テーブルで追加する必要がある。

このように制約を結ぶ事でデータの整合性を保つ事ができる。

Djangoでは foreign keyが設定されている方が子テーブルになる。

今度は少し複雑なモデルを見ていく。

図のようなQuestionとChoiceという2つのモデルを作成する。

![](https://storage.googleapis.com/zenn-user-upload/2010557ea93110bd32304c66.png)

ChoiceにQuestionが `ForeignKey` を使って紐ずけられている。 `on_delete=models.CASCADE` は紐づけられたモデルが削除される際にどのような動作をするかを決める事が出来る。削除された後そのモデルだった部分をNullで埋めたり、そもそも削除できないようにしたりと出来る。 `CASCADE` は紐づけられた側のモデルで関連するオブジェクトも削除するという動きになる。なので Questionが削除されたら、Questionと関連のあるChoice側のオブジェクトも削除するような動作を取る。

詳しくはここの記事が分かりやすい。

[Django2.0から必須になったon_deleteの使い方 - DjangoBrothers](https://djangobrothers.com/blogs/on_delete/)

```python
from django.db import models

class Question(models.Model):
    # Question textというフィードを作成してそこに入る文字列は200文字までと制限している。
    question_text = models.CharField(max_length=200)
    # 基本的には変数名がフィールド名として使用されるが、引数で文字列を渡す事でフィールド名設定する事が出来る。
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    # Question ← → Choiseと双方向のやりとりが可能となる。
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    # Votesフィールドは整数値を受け付ける。最初は0が入る。
    votes = models.IntegerField(default=0)
```

モデル間の双方向やりとりについて

[【Django】1対多の関係（ related_name, _set.all() ）について - Qiita](https://qiita.com/taki_21/items/d461072e6ea65630171a)

モデルを作成したのでデータベースにマイグレート（モデルを元にデータベースのレイアウトを作成するデータを追加するわけではない。）していく

migrationファイルを作成する前に、新たに作成したアプリpollsを伝える必要がある。 `settings.py` の `INSTALLED_APPS` の配列に `'polls.apps.PollsConfig',` を追加する。

次にmigrationファイルを作成する。

新たにファイルを作成する必要はなくターミナルでコマンド実行する。

```bash
python manage.py makemigrations polls
```

実行すると `models.py` を元にmigrationファイルが生成される。

```bash
python manage.py migrate
```

実行するとmigrationファイルを元にデータベースが作成される。

試しにコマンドラインからDjango shellを通してデータベースを変更したりしてみる。

シェルに入るには下記のコマンドを実行する。

```bash
python manage.py shell
```

するとPythonコード `>>>` を書ける状態になるのでここからデータベースにアクセスするコードを書く。

```python
# 作成したモデルを読み込む
from polls.models import Choice, Question

# 格納されたデータを確認する。まだ追加していないので、空になっている。
Question.objects.all()

from django.utils import timezone
# モデルにデータを追加する。
q = Question(question_text="What's new?", pub_date=timezone.now())
# データベースに保存する。
q.save()

q.id
q.question_text
q.pub_date

# データの上書き
q.question_text = "What's up?"
q.save()

# データが格納されているのが確認できる。
Question.objects.all()
# 実行結果：<QuerySet [<Question: Question object (1)>]>
```

![](https://storage.googleapis.com/zenn-user-upload/d768fm3g728u653rzmzij3l9yhf9)

このままではadminページでオブジェクト名が `Question object (1)` と表示され分かりにくいので 特殊メソッド `__str__()` をモデルに追加する。

あと追加で `was_published_recently(self)` メソッドを書きました。

データが最近追加されたかどうかを判定するメソッドで `True or False` で返します。

```python
class Question(models.Model):
  # クラス変数を定義する。データベースフィールドを表現している。
  # Charフィールドは文字のフィールド
  question_text = models.CharField(max_length=200)

  # 日時のフィールド
  pub_date = models.DateTimeField('date published')

  def __str__(self):
    # インスタンスを生成して、printした際にここが実行される。
    # シェルで表示されるオブジェクトに質問名が使われるだけでなく
    # adminでオブジェクトを表現する際にも使用されるので追加する必要がある。
    return self.question_text

  def was_published_recently(self):
    now = timezone.now()
    # now - datetime.timedelta(days=1)は今の時間から一日引いた日付を出す。
    # 2021-05-19 23:29:56.216634こんな感じの値になる。
    # pub_dateが現在時刻より過去で現在時刻から一日以内の場合はTrueを返すメソッド
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
  
class Choice(models.Model):

  # これはChoiceがQuestionに関連付けられている事を伝えている。
  # データベースの多対一、多対多、一対一のようなデータベースリレーションシップに対応する。
  # Question ← → Choiseと双方向のやりとりが可能となる。
  question = models.ForeignKey(Question, on_delete=models.CASCADE)
  choice_text = models.CharField(max_length=200)
  votes = models.IntegerField(default=0)

  def __str__(self):
    return self.choice_text
```

すると下記のように表示されるので、

![](https://storage.googleapis.com/zenn-user-upload/ejo29u3azzc9mjkets12mobuxn42)

何のデータが入っているのか分かり易くなった。

Djangoのshell内でも下記のようにオブジェクトの中身が分かり易くなった。

```python
from polls.models import Choice, Question
Question.objects.all()
# 実行結果：<QuerySet [<Question: What's up?>]>

# 続けて色々な関数を試してデータベースから
# データを取得してみる。
# filterをかけてデータを取得する。
# idはデータベースにデータを追加した際に1から順番に自動で割り振られる。
Question.objects.filter(id=1)
# 実行結果：<QuerySet [<Question: What's up?>]>

# Questionオブジェクトのquestion_textフィールドで"What"から
# 始まるデータを取得する。
Question.objects.filter(question_text__startswith="What")
# 実行結果：<QuerySet [<Question: What's up?>]>

from django.utils import timezone

# 今年作成されたデータを取得する。
current_year = timezone.now().year
Question.objects.get(pub_date__year=current_year)

# id 2のデータを取得する。
# filterで指定しなくてもgetでも取得できる。
Question.objects.get(id=2)
# 実行結果はない場合はエラーになります。

# プライマリーキーと呼ばれるものでいまいちidとの違いが分からない。
# 取得するデータはidの場合と同じになる。
q = Question.objects.get(pk=1)
q.was_published_recently()
# 実行結果：True

# ChoiseはQuestionと関連付けられてるのでQuestionからも
# データにアクセスする事ができる。
# Choice側にはまだデータを入れてないので結果は何も表示されない。
# Choicecは質問に対する回答の選択肢をデータとして持つ。
# qには今What's upという質問が入っているので、それに対しての
# 選択肢を作成した。
q.choice_set.all()
q.choice_set.create(choice_text="Not much", votes=0)
q.choice_set.create(choice_text="The sky", votes=0)
c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# 選択肢が関連づけられている質問を返す。
c.question
# 実行結果：<Question: What's up?>
q.choice_set.all()

# 選択肢が何個あるか数える。
q.choice_set.count()

Choice.objects.filter(question__pub_date__year=current_year)
# 実行結果：<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# just hackingの選択肢だけ削除する。
c = q.choice_set.filter(choice_text__startswith="Just hacking")
c.delete()
```

## pkとidの違い

pkは `primary key` の略で、データベースでは `主キー` と呼ばれている。主キーはテーブルで一意の値を取る。

どのレコードを主キーにするかはフィールド名を定義する時に `primary_key=True` を付ければ設定できる。

codeというフィールドに入る値を主キーとする。

```python
code = models.CharField(max_length=10, primary_key=True)
```

djangoの場合はModel（=テーブル）には必ず1つの主キー用のフィールドが必要になる。ユーザが定義しない場合はidという名前のAutoFiekd（int型の連番1~n）が作成される。

そのため、pkキーを定義しない場合はpkはidのショートカットになる。

pkキーを上記の `code` のように定義した場合はidは作成されない。

## SQLに直接アクセスする。（おまけ）

sqllite3を使用してデータベースを作成している場合はmysiteフォルダ内にデータベースのファイルが生成されていると思うので、下記のコマンドからSQLで操作するシェルに入る事ができる。

```bash
sqlite3 db.sqlite3
# シェル内での操作
# 作成されたテーブル一覧を確認できる。
>>> .table
auth_group                  django_admin_log          
auth_group_permissions      django_content_type       
auth_permission             django_migrations         
auth_user                   django_session  
                            # 先ほどモデルから作成されたテーブル          
auth_user_groups            polls_choice              
auth_user_user_permissions  polls_question

# 構造を確認できるみたいだけど、見てもよく分からなかった。
>>> .schema

# シェルから抜ける。
>>> .quit
```

モデルからデータベースを操作する事ができたので、次に先ほど登場したadminページにアクセスしたいと思う。

## adminページアクセスする。

ログインが必要なのでユーザを下記のコマンドから作成する。

```bash
python manage.py createsuperuser
# 実行すると下記の入力画面が登場する。
Username：名前を入力
Email address：@と.comがあれば架空で良い
Password：
Password（again）：
```

ユーザを作成したら開発サーバを起動して[http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/) にアクセスするとログイン画面となるのでログインする。

![](https://storage.googleapis.com/zenn-user-upload/52f576fd75064ac41c38887f.png)

するとそこから作成したモデルを閲覧したりデータを追加したりできる。

![](https://storage.googleapis.com/zenn-user-upload/7bb1032881a75b5066c66cee.png)

ChoiceモデルからはQuestionを選択して使用することしか出来ないのが確認できる。 右側にある + ボタンを押すと Questionのページに飛びそこから新しい質問を追加することはできる。

## Views.pyからデータベースの値を取得する

先ほどDjango shellで使用したPythonコードを使って、 `views.py` にデータを取得していく。

```python
# 最後はHttpResponseを返す必要があるのでimportする。
from django.http import HttpResponse

# データベースを操作するためにモデルを読み込んでおく。
from .models import Question

def index(request):
    # データベースから最新5件を取得する。
    # こんな感じのデータになる。<QuerySet [<Question: test3>, <Question: hello>, <Question: what's up?>]>
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    # "test3, hello, what's up?"区切った文字列にしてHttpResponseに渡す。
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged
```

## Viewとページデザインを切り離す

Viewではデータの取得や操作を専門的に行ってもらい、そのデータをテンプレート（htmlにPythonの変数を入れられる。）に渡してページをレンダリングしてもらうようにする。

pollsディレクトリの中に、templatesディレクトリを作成する。システムでそのディレクトリを認識する。作成した templatesディレクトリにpollsフォルダを作成する。なので `polls/templates/polls` みたいなディレクトリが完成する。その中にテンプレート `index.html` を作成する。なぜ `templates/polls` とするのかというとDjangoは名前がマッチした最初のテンプレートを使用するので、もし異なるアプリケーションの中に同じ名前のテンプレートがあるとそちらを読み込む。それを回避するために名前空間（所属する領域）を与えている。

`views.py` をテンプレートにデータを渡せるように書き換える。 

```python
from django.http import HttpResponse
from django.template import loader

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    # テンプレートを読み込む
    template = loader.get_template('polls/index.html')
    # 辞書型に最新5件のデータを格納する。
    context = {
        'latest_question_list': latest_question_list,
    }
    # 辞書型のデータをテンプレートに渡してページを作成する。その結果をHttpResponseに返す。
    return HttpResponse(template.render(context, request))
```

### view.pyをさらに短くする。

`from django.template import loader` 

`from django.http import HttpResponse`

を使わない書き方

より簡素にする事が出来る。

その場合、 `render()` 関数は第一引数に `requestオブジェクト` , 第二引数に `テンプレート名` , 第三引数に `辞書型（テンプレートに渡したいデータ）` を記述する。

```python
from django.shortcuts import render
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

### 404エラーを出力する。

```python
from django.http import Http404
from django.shortcuts import render

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)

# 質問の詳細ページのビュー
def detail(request, question_id):
    try:
        # アクセスのあったURLでpkの値が変わる。/polls/1/なら1になる。
        # データベースでエラーになるとHttp404を出力する。
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

### 上記のdetail()を短くする。

`django.shortcuts` にはこうしたコードを省略する関数が多くあるので調べると面白いかもしれない。

```python
from django.shortcuts import get_object_or_404, render

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)。

# モデルにアクセスするobjects.get()とHttp404が一緒になっている。
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

### index.html

`index()` に対するテンプレートにはこのように記述する。

ビューで `latest_question_list` オブジェクトが辞書型に格納されて渡されているので、それを受け取ってテンプレート内でオブジェクトに格納された値を属性アクセス `.` して取得している。

```html
<!-- 受け取った変数にデータがあるか確認する。 -->
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

### index.htmlのハードコード（直接記述している）を削除する

このように直接URLを書き込むと変更に弱いコードになってしまうので、 `polls.urlsモジュールのpath()` 関数でname引数を定義したのでそれをURLに使用する。  `{%url%}` を使う。

```html
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

変更後

```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

そしてテンプレートを入れるディレクトリを作成した際のように名前空間を追加する。システムが別々のアプリ内で同じname引数を含んでいても区別が付けられるように `template/polls/detail.html` にアクセスしたい場合は  `polls:detail` と記述する。

```html
<!-- 受け取った変数にデータがあるか確認する。 -->
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

そして、URLconf（urls.py）に名前空間を追加 `app_name = 'polls'`

```python
from django.urls import path

from . import views

# ここを新たに追加した。
app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

こうするとモジュールに指定されたURLの定義を検索出来る。例えば `polls/specifics/12` のようにURLを変更した場合

`urls.py` に書かれたパスを変更する事でテンプレート側に変更を加える必要がない。

```python
path('specifics/<int:question_id>/', views.detail, name='detail'),
```

### detail.html

`detail()` に対するテンプレートはこのように記述する。

```html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

## フォームを使って質問に対して回答を送信する。

`detail.html` に `<form>` を追加してサーバにデータを送信して質問に対して、投票出来るようにする。

下記のように `detail.html` を変更する。

```html
<!-- 質問の内容 -->
<h1>{{ question.question_text }}</h1>

<!-- もしデータベースから質問が取得出来ない場合エラーが表示される。 -->
{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">

<!-- セキュリティのため -->
{% csrf_token %}

<!-- 質問に対する選択肢を並べる -->
{% for choice in question.choice_set.all %}
    <!-- forloop.counterはforタグのループが何度実行されたかを表す値です。 -->
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```

## views.pyにvote()関数を追加する

コードの流れとしては

1. ユーザがdetailページの質問に対する選択肢を選択する。
2. Voteボタンをクリックする。
3. データがサーバに送信される。
4. 選択された選択肢の投票数をインクリメントする。
5. results.htmlにリダイレクトする。Postデータが成功した後は基本的に `HttpResponse` ではなく `HttpResponseRedirect` を返す必要がある。

 

```python
# 追加した。HttpResponse, HttpResponseRedirect
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
# 追加した。reverse
from django.urls import reverse
# Choiceを追加した。
from .models import Choice, Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)。

# モデルにアクセスするobjects.get()とHttp404が一緒になっている。
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})

# 質問に対して選択して投票する。
def vote(request, question_id):
    # まず質問があるかどうか確認する。
    question = get_object_or_404(Question, pk=question_id)
    try:
        # ユーザが選択した値からpk値を取得して、それを元にモデルから選択肢のオブジェクトを取得する。
        # なければYou didn't...choiceと表示される。
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        # 選択肢オブジェクトから何回投票されたか表示するvotesオブジェクトをインクリメントする。
        selected_choice.votes += 1
        # データベースに保存する。
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        # データの保存に成功したら、results.htmlにリダイレクトする。
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

### reverse()関数とは

この関数を使うと、vote関数中でのURLのハードコードを防ぐ事が出来る。

引数としては `polls:results` リダイレクト先のビュー名とそのビューに与えるURLパターン `question.id` を渡せる。

```python
reverse('polls:results', args=(question.id,))

# 返り値
'/polls/3/results/'
```

## Post通信が成功した際のresults関数を作成する。

views.pyにresults関数を追加する。

```python
# 先ほどまで書いてきたviews.pyにresults関数を追加する。

def results(request, question_id):
    # 指定したpkキーにデータがあれば返す、なければエラーを返す。
    question = get_object_or_404(Question, pk=question_id)
    # 質問オブジェクトを引数で貰ってページを作成する。
    return render(request, 'polls/results.html', {'question': question})
```

results.htmlを作成する。

```html
<!-- 質問を表示する。 -->
<h1>{{ question.question_text }}</h1>

<!--質問の選択肢とそれに対する投票数を取得する。-->
<ul>
{% for choice in question.choice_set.all %}
    <!--choice.votes|pluralizeは投票数が2以上の場合vote s とsを追加してくれる。-->
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

[Built-in template tags and filters | Django documentation | Django](https://docs.djangoproject.com/en/dev/ref/templates/builtins/?from=olddocs#pluralize)

## 汎用ビューを使って今まで書いたコードをさらに短くする。

`views.py` に書かれた `index(), detail(), results()` 関数は3つとも似たような機能でURLを介して渡されたパラメータに従ってデータベースからデータを取り出しページを作成する。これらの一連の動作はよくある事なのでDjangoでは汎用ビューというショートカットを用意してより簡素に機能を実装出来るようにしている。

汎用ビューを適用するにはいくつかこれまでに書いたコードを修正する必要がある。

1. URLconfを変換する。
2. 古い不要なビューを削除する。
3. 新しいビューにDjango汎用ビューを設定する。

### URLconfの修正

変更前

```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

変更後

```python
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

`views.IndexView.as_view(), views.DetailView.as_view(), views.ResultsView.as_view()` と `as_view()` と書くようになった。

そして、 `question_id` が `pk` に変更された。ここは同じでもいいような気もする。結局同じ数値が返り値として入るから。

※後述する `DetailView` には `pk` キーを渡す必要があるので同じではダメなようだ。

### viewsの修正

`index(), detail(), results()` 関数を削除しクラスベースに書き換える。

indexでは `ListView` を継承している。

detail, resultsでは `DetailView` を継承している。

**ListView**

「オブジェクトのリストを表示する。」

メソッドのフローチャート継承したメソッドが下記の順番で自動で実行される。

1.[setup()](https://docs.djangoproject.com/ja/3.2/ref/class-based-views/base/#django.views.generic.base.View.setup)
2.[dispatch()](https://docs.djangoproject.com/ja/3.2/ref/class-based-views/base/#django.views.generic.base.View.dispatch)
3.[http_method_not_allowed()](https://docs.djangoproject.com/ja/3.2/ref/class-based-views/base/#django.views.generic.base.View.http_method_not_allowed)
4.[get_template_names()](https://docs.djangoproject.com/ja/3.2/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin.get_template_names)
5.[get_queryset()](https://docs.djangoproject.com/ja/3.2/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectMixin.get_queryset)
6.[get_context_object_name()](https://docs.djangoproject.com/ja/3.2/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectMixin.get_context_object_name)
7.[get_context_data()](https://docs.djangoproject.com/ja/3.2/ref/class-based-views/mixins-multiple-object/#django.views.generic.list.MultipleObjectMixin.get_context_data)
8.[get()](https://docs.djangoproject.com/ja/3.2/ref/class-based-views/generic-display/#django.views.generic.list.BaseListView.get)
9.[render_to_response()](https://docs.djangoproject.com/ja/3.2/ref/class-based-views/mixins-simple/#django.views.generic.base.TemplateResponseMixin.render_to_response)

このようにメソッドが実行されるので継承したクラスに `get_quesryset()` メソッドを追加して内容を上書きする事が出来る。

*template_name*

`ListView` ではデフォルトの場合 `<app name>/<model name>_list.html` を自動で生成して使用する。

その場合、テンプレート名は `polls/question_list.html` になる。

しかし元々作成してある `polls/index.html` を使用したい場合は `template_name` に `'polls/detail.html'` を代入する事でDjangoがそちらを使用するように認識してくれる。

**DetailView**

「あるタイプのオブジェクト詳細ページを表示する。」

なので ListViewの詳細ページをDetailViewで表示するみたいな使われ方をする。

*template_name*

そしてデフォルトでは `DetailView` は `<app name>/<model name>_detail.html` という名前のテンプレートを自動生成して使用する。

その場合、テンプレート名は `polls/question_detail.html` になるが、今回は自動生成されたものではなく元々作成してある `polls/detail.html` を使いたいので `template_name` を指定して元々のテンプレートを使用する。方法はListViewの時と同じで `template_name` に `polls/detail.html` を代入する。

**model**

このクラス変数はビューが使用するモデルを指定している。

`model = Question` の場合は裏側で `Question.objects.all()` を行ってくれる。なので `queryset = Question.objects.all()` としても良い。

```python
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question

class IndexView(generic.ListView):
    # デフォルトのビューを使用せず、元々作成してあったものを使用する。
    template_name = 'polls/index.html'
    # 自動で渡されるquestion_listというコンテキスト変数の変数名を独自のものに変更している。
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """最新の5件を取得する。"""
        return Question.objects.order_by('-pub_date')[:5]

class DetailView(generic.DetailView):
    # 自分がどのモデルに対して動作するかを伝えている。
    # おそらくget_object_or_404(Question, pk=question_id)のQuestion部分を担っている。
    # pkの部分はurls.pyで先に指定してある。
    model = Question
    template_name = 'polls/detail.html'

class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'

def vote(request, question_id):
    ... # 前回のまま変更しない。
```

これでサーバを起動して特にエラーもなく質問のリストページ（index.html）、詳細ページ（投票するページdetail.html）、投票後の今ままでの投票数を表示するページ（results.html）が表示されていれば汎用ビューでのアプリ構築ができたと思う。

## 最後に

過去にRuby on railsのフレームワークの中身がどう動作しているのかイメージ出来ないのが苦痛（フレームワークは面倒な中身を気にしなくてもアプリが作れるように設計してあるので仕方ないかもしれない。）で挫折しているので今回Djangoのチュートリアルまだ途中ですが挫折せずにアプリ作成まで出来てよかったです。普段Jsonしか触らなかったので少しですがデータベースを作成して操作する経験が出来たのでこれを気にSQL構文をもう少し勉強しようと思う。多対多、多対一の関係とかも自分で作成出来るまでになります。自分の作成したアプリのER図をかける書けるようになりたい。

### 参照

[SECRET_KEYを誤ってGitHubにプッシュしたときの対処法（Django編） - Qiita](https://qiita.com/frosty/items/bb5bc1553f452e5bb8ff)

[Pythonの相対インポートで上位ディレクトリ・サブディレクトリを指定 | note.nkmk.me](https://note.nkmk.me/python-relative-import/)

[ワイルドカードインポート(import *)は推奨されない](https://python.civic-apps.com/wildcard-import/)

[「カラム名」と「フィールド名」の違い｜「分かりそう」で「分からない」でも「分かった」気になれるIT用語辞典](https://wa3.i-3-i.info/diff529name.html)

[3. データベースマイグレーション | densan-labs.net](https://densan-labs.net/tech/codefirst/migration.html)

[モデル(データベース)の作成](https://codor.co.jp/django/making-model)

[プログラミングでよく見かける"コンテキスト(context)って何？ - Qiita](https://qiita.com/dojyorin/items/0bd3ef167991cfc703b1)

[Python Django チュートリアル(3) - Qiita](https://qiita.com/maisuto/items/eece9d880d94fd241a0d)

[DJangoのお勉強(1) - Qiita](https://qiita.com/ShimantoAkira/items/ec1130fbd2a65cd5b7f3)

[ドキュメント](https://docs.djangoproject.com/ja/3.2/intro/)

[Djangoの汎用ビュー入門（ListView）](https://hombre-nuevo.com/python/python0057/)

[FOREIGN KEY制約(外部キー制約を設定する)](https://www.dbonline.jp/mysql/table/index11.html)

[外部キー制約とは｜「分かりそう」で「分からない」でも「分かった」気になれるIT用語辞典](https://wa3.i-3-i.info/word17575.html)

[Django ForeignKeyで1対多のモデルを構築](https://noumenon-th.net/programming/2019/12/02/foreignkey/)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。