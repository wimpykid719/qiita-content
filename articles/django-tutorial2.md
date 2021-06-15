---
title: "Django公式チュートリアル（5~7）で分からない所、徹底的に調べた。" # 記事のタイトル
emoji: "🍟" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["django", "python"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.05.26'
qiitaId: '' #記事のslug
---


## 最初に

前回のチュートリアルで作成した投票アプリに対してテストコードを書く所から始める。内容はチュートリアル5〜7をカバーする予定になる。

## Djangoでテストコードを書く

チュートリアルが用意してくれたバグに対してテストコードを書いていく。

まずはバグを確認する。 `Qustion.was_published_recently()` のメソッドはQuestionが昨日以降に作成された場合に `True` を返すが 未来の日付になっている場合にもTrueを返す。これがバグになる。自分は前編の記事で登場した `models.py` に書かれた `was_published_recently()` はバグに対応済みなので下記のコードに入れ替えてバグを作り出す必要がある。

これだと `pub_date` が未来の場合も `True` を返す。

```python
def was_published_recently(self):
  return self.pub_date >= timezone.now() - datetime.timedelta(days=1) 
```

### バグの確認

コードに故意にバグを生み出した所でバグを確認したいと思う。

```bash
python manage.py shell
```

データベース APIを叩いていく。

```python
import datetime
from django.utils import timezone
from polls.models import Question

# 投稿日を今から30日後に設定した Questionオブジェクトを作成する。
# この状態ではQuestionクラスからインスタンスを生成しただけでデータベースに保存はされていない。
future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))

# 結果
True
```

### テストを作成する。

pollsアプリのディレクトリに `tests.py` というファイルがあると思うのでそこにテストコードを書いていく。

```python
import datetime
from django.test import TestCase
from django.utils import timezone
from .models import Question

class QuestionModelTests(TestCase):

  def test_was_published_recently_with_future_question(self):
    """
      was_published_recently()はpubdateが現在時刻より未来に設定された
      場合はFalseを返さないといけない。
    """
    time = timezone.now() + detetime.timedelta(days=30)
    future_question = Question(pub_date=time)
    # ここで返却値がFalse出ない場合はテストに通らない事を設定している。
    self.assertIs(future_question.was_published_recently(), False)
```

### テストを実行する。

```bash
python manage.py test polls

# 実行結果
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionModelTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/path/to/mysite/polls/tests.py", line 16, in test_was_published_recently_with_future_question
    self.assertIs(future_question.was_published_recently(), False)
AssertionError: True is not False

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

テストは失敗したと出力されると思う。

### バグを修正する

`models.py` の記述された関数 `was_published_recently()` を元に戻してバグがない状態にしたいと思う。

```python
def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

もう一度実行してみる。

```bash
python manage.py test polls

# 実行結果

Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
```

今度はテストはOKと出力され、テストに通った。

## 複数のテストを実行する

先ほど作成した `QuestionModelTests` クラスに別のテストも追加してみましょう。

```python
import datetime
from django.test import TestCase
from django.utils import timezone
from .models import Question

class QuestionModelTests(TestCase):

  def test_was_published_recently_with_future_question(self):
    """
      was_published_recently()はpubdateが現在時刻より未来に設定された
      場合はFalseを返さないといけない。
    """
    time = timezone.now() + detetime.timedelta(days=30)
    future_question = Question(pub_date=time)
    # ここで返却値がFalse出ない場合はテストに通らない事を設定している。
    self.assertIs(future_question.was_published_recently(), False)

  # 新しくテストを追加していく。
  def test_was_published_recently_with_old_question(self):
    """
      was_published_recently()はpub_dateが1日より過去の場合
      Falseを返す
    """
    # 現在時刻より一日1秒前の質問のインスタンスを作成する。
    time = timezone.now() - timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    # 返り値がFalseならテストに通る。
    self.assertIs(old_question.was_published_recently(), False)

  def test_was_published_recently_with_recent_question(self):
    """
      was_published_recently()はpub_dateが1日以内ならTrueを返す
    """
    # 一日以内の質問インスタンスを作成する。
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True) 
```

これで過去、現在、未来に対してのテストが揃った。これで期待通りに動作する事を保証できるようになった。

## Djangoのviewをテストする。

ビューレベルでのユーザ動作をシュミレートする事ができるClientを用意しているので、 `tests.py` や shellで使用する事ができる。

最初はshellから使用してみる。

```bash
python manage.py shell
```

下記のコードを一行ずつshellで実行する。

```python
from django.test.utils import setup_test_environment

# テンプレートのレンダラーをインストールする
# response.context等の属性を調査できるようになる。
setup_test_environment()

from django.test import Client
# クライアントインスタンスを作成してページアクセスしたように操作する。
client = Client()

response = client.get('/')
# 実行結果
Not Found: /

response.status_code
# 実行結果
404

from django.urls import reverse
response = client.get(reverse('polls:index'))
response.status_code

# 実行結果
200

# ページのhtmlが返ってくる。
response.content

# 実行結果
b'\n  <ul>\n    \n      <li><a href="/polls/3/">test3</a></li>\n    \n      <li><a href="/polls/2/">hello</a></li>\n    \n      <li><a href="/polls/1/">what&#x27;s up?</a></li>\n    \n  </ul>\n\n'

response.context['latest_question_list']

# 実行結果
<QuerySet [<Question: test3>, <Question: hello>, <Question: what's up?>]>
```

現在の投票一覧は最新5件を取得しているため、未来の投稿日の質問も表示している。これを

`views.py` の `get_queryset()` に変更を加えていく。

```python
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

# 新しく追加した
from django.utils import timezone

from .models import Choice, Question

class IndexView(generic.ListView):
    # デフォルトのビューを使用せず、元々作成してあったものを使用する。
    template_name = 'polls/index.html'
    # 自動で渡されるquestion_listというコンテキスト変数の変数名を独自のものに変更している。
    context_object_name = 'latest_question_list'
    
    # 変更する箇所
    def get_queryset(self):
        """
         最新の5件を取得する。ただし投稿日が現在時刻より前にある投稿のみ表示。
         filter(pub_date__lte=timezone.now()) = 
         if Question.pub_date <= timezone.now():
           return Question.pub_date
        """
        return Question.objects.filter(
          pub_date__lte=timezone.now()
        ).order_by('-pub_date')[:5]

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

Djangoでは querySetの `filter()` を使用する際に変数を比較したい場合下記のような記述を取る事が出来る。

下記は `Product.weight` の値が2以下の場合 Trueになり返却されるオブジェクトになる。

```python
# weight <= 2
products = Product.objects.filter(weight__lte=2)
```

他にも比較したり出来る、下記のサイトで説明されている。

[Django逆引きチートシート（QuerySet編） - Qiita](https://qiita.com/uenosy/items/54136aff0f6373957d22)

## viewのテストを追加する

```python
import datetime
from django.test import TestCase
from django.utils import timezone
from .models import Question
# 新しく追加した。
from django.urls import reverse

class QuestionModelTests(TestCase):

  def test_was_published_recently_with_future_question(self):
    """
      was_published_recently()はpubdateが現在時刻より未来に設定された
      場合はFalseを返さないといけない。
    """
    time = timezone.now() + detetime.timedelta(days=30)
    future_question = Question(pub_date=time)
    # ここで返却値がFalse出ない場合はテストに通らない事を設定している。
    self.assertIs(future_question.was_published_recently(), False)

  # 新しくテストを追加していく。
  def test_was_published_recently_with_old_question(self):
    """
      was_published_recently()はpub_dateが1日より過去の場合
      Falseを返す
    """
    # 現在時刻より一日1秒前の質問のインスタンスを作成する。
    time = timezone.now() - timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    # 返り値がFalseならテストに通る。
    self.assertIs(old_question.was_published_recently(), False)

  def test_was_published_recently_with_recent_question(self):
    """
      was_published_recently()はpub_dateが1日以内ならTrueを返す
    """
    # 一日以内の質問インスタンスを作成する。
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
　
　# 新しく追加した
  def create_question(question_text, days):
    """
      引数から質問を作成する。過去に投稿された質問を作りたいなら-1~nの値を第二引数に取る、
      まだ公開されてない質問を作成したいなら+1~nの値を第二引数に取る。
      現在から10日後の投稿日の質問を作成したいなら
      例： create_question('今日は何食べる?' , 10)
      
　　　　15日前の質問を作成したいなら
          create_question('今日は何食べる?' , -15)
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)

class QestionIndexViewTests(TestCase):
  def test_no_question(self):
    """
      質問がデータベースにない際に適切なメッセージを
      表示出来てるか確認する。
    """
    response = self.client.get(reverse('polls:index'))
    # テスト合格条件
    # ステータスコードが200である事
    self.assertEqual(response.status_code, 200)
    # コンテンツに No polls are availableが含まれる事
    self.assertContains(response, "No polls are available.")
    # データベースが空である事
    self.assertQuerysetEqual(response.context['latest_question_list'], [])

  def test_past_question(self):
    """
      過去の投稿日の質問一覧が表示されるか確認する。
    """
    # 投稿日が30日前の質問を作成する。ダミーデータなので実際のデータベースにデータ作成されることはない。
    # そしてメソッドが終了すればダミーデータは破棄される。
    # なので新しくテストする際は質問は空の状態から始まる。
    question = create_question(question_text="過去の質問", days=-30)
    response = self.client.get(reverse('polls:index'))
    # テスト合格条件
    # 先ほど作成した質問が表示されているか表示する。
    self.assertQuerysetEqual(response.context['latest_quesiton_list'], [question],)

  def test_future_question(self):
    """
      投稿日が未来の質問が表示されていないか確認する。
    """
    create_question(question_text="未来の質問", days=30)
    response = self.client.get(reverse('polls:index'))
    # テスト合格条件
    # コンテンツに No polls are awailableが含まれる事
    self.assertContains(response, "No polls are available.")
    # 最新の質問5件が質問が空な事
    self.assertQuerysetEqual(response.context['latest_question_list'], [])

  def test_future_question_and_past_question(self):
    """
      過去・未来の質問の両方ある時に過去の質問だけ表示される。
    """
    # 片方だけ変数に入れるのはテストの合格条件を判別する際に過去質問が表示されているのを確認するため
    question = create_question(question_text="Past question.", days=-30)
    create_question(question_text="Future question.", days=30)
    response = self.client.get(reverse('polls:index'))
    self.assertQuerysetEqual(
      response.context['latest_question_list'],
      [question],
     )

    def test_two_past_questions(self):
      """
        過去の質問2つが表示されているか確認する。
      """
      question1 = create_question(question_text="Past question 1.", days=-30)
      question2 = create_question(question_text="Past question 2.", days=-5)
      response = self.client.get(reverse('polls:index'))
      self.assertQuerysetEqual(
        response.context['latest_question_list'],
        [question2, question1],
      )
```

システムに問題がなければテストに全て合格する。作成された質問はデータベースに保存される事なく各テストが実行されて終わるたびに破棄される。

## DetailViewのテスト

上記のテストは上手く動作して未来の質問はindexに表示されないが、 `detail.html` への正しいURLを知っていたり推測したユーザは、まだページに到達する事が出来る。そのため同じように未来の投稿日の場合はページを表示しないように `polls/views.py` コードを書き換える必要がある。

```python
class DetailView(generic.DetailView):
    # テンプレートで変数にアクセスする際はquestionになる。
    model = Question
    template_name = 'polls/detail.html'
    
    # 新しく追加した
    def get_queryset(self):
      """
        まだ公開されていない質問は除外する。
      """
      return Question.objects.filter(pub_date__lte=timezone.now())
```

そして新たに追加した機能が動作するか確認するテストを書く。

`tests.py` に下記のコードを追加する。

```python
class QuestionDataViewTests(TestCase):
  def test_future_question(self):
    """
      detail.htmlの未来の日付のページにアクセスする場合は404を表示する、
    """
    # 現在から5日後の質問を作成する
    future_question = create_question(question_text = '未来の質問', days=5)
    url = reverse('polls:detail', args=(future_question.id,))
    response = self.client.get(url)
    # 合格条件
    # ページにアクセスした際のステータスコードが404
    self.assertEqual(response.status_code, 404)

  def test_past_question(self):
    """
      過去の質問の場合はページを表示する。
    """
    past_question = create_question(question_text='過去の質問', days=-5)
    url = reverse('polls:detail', args=(past_question.id,))
    response = self.client.get(url)
    # ページに過去の質問が含まれている。
    self.assertContains(response, past_question.question_text)
```

detailビューのテストも書いてきましたが、同様にresultsビューが必要になるが似たようなコードになるのでチュートリアルでは紹介されていない。

別の問題として現在の状態ではChoice（質問に対する選択肢）を持たない質問が公開されている。それを `views.py` で処理する事が出来るので機能を追加して、ChoicesがないQuestionを作成し、それが公開されないことをテスト、同じようにChoiceがあるQuestionを作成し、それが公開されることをテストをする。

## get_queryset()に選択肢がない質問を表示しないようにfilterを追加する。

```python
class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    # テンプレート側でQuestion.objects.order_by('-pub_date')[:5]を呼び出す際の名前を設定している。
    context_object_name = 'latest_question_list'

    def get_queryset(self):
　　　　　# filter内の条件は現在より過去の質問かつ選択肢がある場合に質問オブジェクトを返すようになっている。
        return Question.objects.filter(pub_date__lte=timezone.now(), choice__isnull=False).distinct().order_by('-pub_date')[:5]

class DetailView(generic.DetailView):
    # テンプレートで変数にアクセスする際はquestionになる。
    model = Question
    template_name = 'polls/detail.html'
    def get_queryset(self):
        print(Question.objects)
        return Question.objects.filter(pub_date__lte=timezone.now(), choice__isnull=False).distinct()
```

`choice__isnull=False` で逆参照を行い 各質問にぶら下がる選択肢を確認する。 選択肢がある場合はおそらく内部でこんな感じに取得できると考えている。 Djangoのシェルに移動して直接データベースAPIを操作して選択肢があり参照関係になっている質問を確認する事ができる。

![](https://storage.googleapis.com/zenn-user-upload/2010557ea93110bd32304c66.png)

```python

# 登録された選択肢を全て取り出して、それぞれがどこの質問に結びつけらているか表示している。
# 図でいう1, 1, 2を取り出しているのでそれに結びついたQuestionオブジェクトが表示されている。
[obj.question for obj in Choice.objects.all()]

# 実行結果
[<Question: what's up?>, <Question: what's up?>, <Question: hello>]
```

そして参照関係にない質問は `obj.questtion` しても空なので `false` になりその質問には選択肢がないと判断する事ができる。

### distinct()で重複する結果を表示しないようにする。

![Join 001](https://user-images.githubusercontent.com/23703281/120579717-f7503500-c462-11eb-828d-7cd32685cc2a.png)

親テーブルと子テーブルをJoinして作成された新しいテーブルになる。

そして同じフィールドに別の値を入れる事が出来ないので選択肢に対してどの質問が参照されているのかという表示方法になる。

そのため、1つの質問で複数の選択肢を参照している質問は参照する選択肢の数だけ表示されることになる。

![polls_filter_join](https://user-images.githubusercontent.com/23703281/120579889-34b4c280-c463-11eb-8eeb-527e5a8dece5.png)


[http://127.0.0.1:8000/polls/](http://127.0.0.1:8000/polls/)

アクセスすると質問が重複して表示される。

この重複項目をなくすために `distict()` を使用する。

![distinct](https://user-images.githubusercontent.com/23703281/120580005-5dd55300-c463-11eb-8b34-2399662f8223.png)

そうすると重複項目がなくなり、選択肢がない質問だけを表示する事ができる。

## 新しく追加した機能のテストコードを書いていく。

まず `tests.py` の `create_question()` で選択肢を含む質問を作成できるようにする。

`choice_texts` に値がある場合は、それを元に選択肢を作成する。複数作成することもできる。

```python
def create_question(question_text, days, choice_texts=[]):
  """
    質問を `question_text` と投稿された日から作成する。現在より過去の時間で投稿したい場合は days= -days、
    未来の時間で投稿したい場合は対してはdays= +daysとする。
  """
  time = timezone.now() + datetime.timedelta(days=days)
  q = Question.objects.create(question_text=question_text, pub_date=time)
  # 選択肢がある場合とない場合で変数に格納した際返ってくるモデルが変わるから注意が必要
  # 選択肢があるとChoiceオブジェクトが変える。ないとQuestionオブジェクトになる。
  if choice_texts:
    for choice_text in choice_texts:
      return q.choice_set.create(choice_text=choice_text, votes=0)
  else:
    return q
```

これを使って、先ほど追加した indexページ、detailページで選択肢がない質問が表示されていないか確認するテストコードを書いていく、そして前回作成したテストも選択肢がない質問の場合ページが表示されなくなっているので、作成する質問に選択肢を付けてあげないとテストが通らなくなっている。

その修正も行う。このように一部変更を加えたために今まで通ってたテストを含めて、全体を修正しなくてはならないコードはとても修正が大変なので良いコードとは言えないかもしれない。もしもっと良いテストコードの書き方があったら教えて下さい。

`tests.py` これがテストの全体コードになる。

```python
import datetime
from django.http import response
from django.test import TestCase
from django.urls import reverse
from django.utils import timezone
from .models import Choice, Question

# テストコードの書き方はTestCaseを継承する事
# メソッド名をtestから始める事でDjango側で実行してくれるようになる。

class QuestionModelTests(TestCase):
  def test_was_published_recently_with_future_question(self):
    """
      was_published_recently()はpub_dateが未来の場合Falseを返す。
    """

    time = timezone.now() + datetime.timedelta(days=30)
    future_question = Question(pub_date=time)

    self.assertIs(future_question.was_published_recently(), False)

  def test_was_published_recently_with_recent_question(self):
    """
      was_published_recently()はpub_dateが昨日までに投稿されたものなら　Trueを返す。
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)

    self.assertIs(recent_question.was_published_recently(), True)
  
def create_question(question_text, days, choice_texts=[]):
  """
    質問を `question_text` と投稿された日から作成する。現在より過去の時間で投稿したい場合は days= -days、
    未来の時間で投稿したい場合は対してはdays= +daysとする。
  """
  time = timezone.now() + datetime.timedelta(days=days)
  q = Question.objects.create(question_text=question_text, pub_date=time)
  # 選択肢がある場合とない場合で変数に格納した際返ってくるモデルが変わるから注意が必要
  # 選択肢があるとChoiceオブジェクトが変える。ないとQuestionオブジェクトになる。
  if choice_texts:
    for choice_text in choice_texts:
      return q.choice_set.create(choice_text=choice_text, votes=0)
  else:
    return q

class QuestionIndexViewTests(TestCase):
  def test_no_questions(self):
    # reverse('polls:index')でpollsのindexページURLを返している。それを利用してアクセスしている。
    response = self.client.get(reverse('polls:index'))
    self.assertEqual(response.status_code, 200)
    self.assertContains(response, "No polls are available")

    self.assertQuerysetEqual(response.context['latest_question_list'], [])

  def test_past_question(self):
    question = create_question(question_text="Past question.", days=-30, choice_texts=['game set'])
    response = self.client.get(reverse('polls:index'))
    self.assertQuerysetEqual(
      response.context['latest_question_list'],
      [question.question],
    )
  
  def test_future_question(self):
    create_question(question_text="Future question.", days=30, choice_texts=['game set'])
    response = self.client.get(reverse('polls:index'))
    self.assertContains(response, "No polls are available")
    self.assertQuerysetEqual(response.context['latest_question_list'], [])

  def test_future_question_and_past_question(self):
    # 片方だけ変数に入れるのはテストの合格条件を判別する際に過去質問が表示されているのを確認するため
    question = create_question(question_text="Past question.", days=-30, choice_texts=['game set'])
    create_question(question_text="Future question.", days=30, choice_texts=['game set'])
    response = self.client.get(reverse('polls:index'))
    self.assertQuerysetEqual(
      response.context['latest_question_list'],
      [question.question],
    )
  
  def test_two_past_question(self):
    question1 = create_question(question_text="Past question 1.", days=-30, choice_texts=['game set'])
    question2 = create_question(question_text="Past qustion 2.", days=-5, choice_texts=['game set'])
    response = self.client.get(reverse('polls:index'))
    self.assertQuerysetEqual(
      response.context['latest_question_list'], [question2.question, question1.question],
    )

  def test_choice_question(self):
    """
      Indexページで
      選択肢のある質問を表示する。
    """
    choice_question = create_question(question_text='Indexページでの選択肢のある質問', days=-1, choice_texts=['game set'])
    url = reverse('polls:index')
    response = self.client.get(url)
    self.assertContains(response, choice_question.question)
  
  def test_no_choice_question(self):
    """
      Indexページで
      選択肢がない質問は表示しない。
    """
    no_choice_question = create_question(question_text='Indexページでの選択肢のない質問', days=-1)
    url = reverse('polls:index')
    response = self.client.get(url)
    self.assertNotContains(response, no_choice_question)

class QuestionDataViewTests(TestCase):

  def test_future_question(self):
    """
      detail.htmlの未来の日付のページにアクセスする場合は404を表示する、
    """
    # 現在から5日後の質問を作成する
    future_question = create_question(question_text = '未来の質問', days=5)
    url = reverse('polls:detail', args=(future_question.id,))
    response = self.client.get(url)
    # 合格条件
    # ページにアクセスした際のステータスコードが404
    self.assertEqual(response.status_code, 404)

  def test_past_question(self):
    """
      Detailページ
      過去の質問の場合はページを表示する。
    """
    past_question = create_question(question_text='過去の質問', days=-5, choice_texts=['geme set'])
    url = reverse('polls:detail', args=(past_question.id,))
    response = self.client.get(url)
    # ページに過去の質問が含まれている。
    self.assertContains(response, past_question.question)
  
  def test_choice_question(self):
    """
      Detailページ
      選択肢のある質問を表示する。
    """
    choice_question = create_question(question_text='detailページでの選択肢がある質問', days=-1, choice_texts=['game set'])
    url = reverse('polls:detail', args=(choice_question.id,))
    response = self.client.get(url)
    self.assertContains(response, choice_question.choice_text)

  def test_no_choice_question(self):
    """
      選択肢がない質問は表示しない。
    """
    no_choice_question = create_question(question_text='detailページでの選択肢のない質問', days=-1)
    url = reverse('polls:detail', args=(no_choice_question.id,))
    response = self.client.get(url)
    self.assertEqual(response.status_code, 404)
```

まずは今まで動作していたテストが選択肢がない質問だったので、選択肢 `['game set']` を追加して再び動作するように変更する。

その際に `choice_set.create` で選択肢を追加した場合、返り値が Questionオブジェクトではなく Choiceオブジェクトになるので質問を取り出す際は `返り値.question` とする必要がある。

問題なければ、テストが13個実行され OK と表示される。

## スタイルシート・静的ファイルを追加する。

### スタイルシートを追加

pollsディレクトリにstaticディレクトリを作成する。そうするとDjangoはそこから静的ファイルを探してくれる。 `polls/static/polls` と templateディレクトリを作成した時みたいになる。

先ほど追加したディレクトリに `style.css` を追加する。 `polls/static/polls/style.css` のようになる。

style.css

```css
li a {
    color: green;
}
```

polls/templates/polls/index.html の上部に下記のコードを追加する。

```css
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

### 画像を追加する

`polls/static/polls/images/` とディレクトリを作成する。その中に 好きな画像をおく。

スタイルシートで背景画像として読み込む

```css
body {
    background: white url("images/background.gif") no-repeat;
}

li a {
    color: green;
}
```

## adminのフォームをカスタマイズする

### 編集フォームでのフィールドの並び順を替える

質問の詳細ページでのフィールドの並び順を変更する。

polls/admin.py

```python
from django.contrib import admin

from .models import Question

class QuestionAdmin(admin.ModelAdmin):
    # この順番で表示されるようになる。
    fields = ['pub_date', 'question_text']

# 第二引数で作成したclassを渡す
admin.site.register(Question, QuestionAdmin)
admin.site.register(Choice)
```

変更前

![QuestionAdmin変更前](https://user-images.githubusercontent.com/23703281/120580070-83625c80-c463-11eb-8a4b-1eb945075646.png)


変更後

pub_dateとquestion_textの位置が入れ替わってる。

![QuestionAdmin変更後](https://user-images.githubusercontent.com/23703281/120580139-9b39e080-c463-11eb-8426-11248fac4d00.png)

### フィールドを分割する。

polls/admin.py

```python
from django.contrib import admin

from .models import Question

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]

admin.site.register(Question, QuestionAdmin)
```

### ChoiceオブジェクトをQuestionフォームから追加・編集する。

現在Choiceフォームから質問に選択肢を追加・編集可能ですが、これだとページを移動したりと効率が悪いので Questionフォームから追加・編集できるようにする。

polls/admin.py

```python
from django.contrib import admin

from .models import Choice, Question

class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 3

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)
```

コードを追加するとQuestionフォームに3つ（extraで数の調整ができる）の `choice_text, votes` を設定できる項目が追加される。

今のままだと多くの画面スペースを必要とするのでこれを小さくする。

`class ChoiceInline` の引数を `TabularInline`  に変更する。

```python
class ChoiceInline(admin.TabularInline):
    #...
```

これでコンパクトになったと思う。

## pollsの質問一覧ページをカスタマイズする。

チェンジリストページと呼ばれるページで（[http://127.0.0.1:8000/admin/polls/question/](http://127.0.0.1:8000/admin/polls/question/)）質問の一覧が表示されている。

現在は オブジェクトの名前（どんな質問が格納されているのがわかる）だけが表示されていますが、各フィールドの値を表示してより多くの情報をここで確認できるようにする。

polls/admin.py

```python
class QuestionAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question_text', 'pub_date', 'was_published_recently')
```

各カラムのヘッダーをクリックすると並び替えを行えるが、 `was_published_recently` だけは並び替えをサポート出来ていないので、 `@` デコレータを使用して並び替えの対応させていく。

デコレータなのでクラスメソッドの直前に追加する。

polls/models.py

```python
from django.contrib import admin

class Question(models.Model):
    # ...

    # ここを新しく追加した。
    @admin.display(
        boolean=True,
        ordering='pub_date',
        description='Published recently?',
    )
    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

### 質問を日付でフィルター掛けれるようにする。

`pub_date` の日付を元に質問を絞れるようにする。フィルタは対象のフィールドの種類によって変化する。 `pub_date` は `DateTimeField` なので、Django はこのフィールドにふさわしいフィルタオプションが、「すべての期間 ("Any date")」「今日 ("Today")」「今週 ("Past 7 days")」「今月 ("This month")」

を用意してくれる。

polls/admin.py

```python
from django.contrib import admin

from .models import Choice, Question

class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 3

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    inlines = [ChoiceInline]
    # 新しく追加した。
    list_filter = ['pub_date']

admin.site.register(Question, QuestionAdmin)
```

### 質問の検索機能を追加する。

先ほどのコードにさらに変数を追加する。

`question_text` フィールドをユーザが入力した文字列を元に Likeクエリで検索するのでデータベースに割と負荷がかかるみたいで常識の範囲で使用しましょうとチュートリアルに記述されている。

```python
# ...
list_filter = ['pub_date']
# 新しく追加した
search_fields = ['question_text']
```

### 管理サイトの見た目をカスタマイズする。

管理サイトの上部に Django administration と書かれているのでこれを Polls administration と変更してみたいと思う。

`manage.py` が置かれているディレクトリに templates ディレクトリを作成する。その中に adminフォルダを作成する。 `templates/admin` みたいな構成になる。

その中にデフォルトのDjango adminのテンプレートをコピーして貼り付ける。

場所は 下記のコマンドから確認できる。anacondaの環境の場合は仮想環境内で実行する必要がある。

```bash
python -c "import django; print(django.__path__)"
```

そして開いたファイルを下記のように編集する。

変更前

```html
{% extends "admin/base.html" %}

{% block title %}{% if subtitle %}{{ subtitle }} | {% endif %}{{ title }} | {{ site_title|default:_('Django site admin') }}{% endblock %}

{% block branding %}
<h1 id="site-name"><a href="{% url 'admin:index' %}">{{ site_header|default:_('Django administration') }}</a></h1>
{% endblock %}

{% block nav-global %}{% endblock %}
```

変更後

```html
{% extends "admin/base.html" %}

{% block title %}{% if subtitle %}{{ subtitle }} | {% endif %}{{ title }} | {{ site_title|default:_('Django site admin') }}{% endblock %}

{% block branding %}
<h1 id="site-name"><a href="{% url 'admin:index' %}">Polls Administration</a></h1>
{% endblock %}

{% block nav-global %}{% endblock %}
```

次に `mysite/settings.py` を開いて `TEMPLATES` 設定オプションの中にある `DIRS` オプションを下記のように変更する。

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # ここを新しく追加した。
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

これでデフォルトのテンプレートをオーバライドすることが出来た。

これでチュートリアル5~7の内容は終了した。

## 最後に
チュートリアル5でテストコードを初めて書く経験が出来てよかったです。途中チュートリアルから外れた事をしようとした際に逆参照でモデルからデータを取得する方法がわからなくてかなり時間が掛かりました。SQLデータベースの理解がまだ乏しいのでもう少しデータベースに慣れてからDjangoでアウトプットとして、Webアプリを作成したいと思います。


### 参照

[DjangoのModelからデータを取り出す方法をまとめとく - やる気がストロングZERO](https://yaruki-strong-zero.hatenablog.jp/entry/django_model_lookup)

[LEFT JOIN / INNER JOIN を実行すると同じ内容のレコードが複数含まれる - SQLの構文](https://www.ipentec.com/document/sql-duplicate-records-with-join)

[](https://sleepless-se.net/2018/07/06/django%E9%96%A2%E9%80%A3%E3%83%A2%E3%83%87%E3%83%AB%E3%81%AE%E3%83%95%E3%82%A3%E3%83%BC%E3%83%AB%E3%83%89%E3%81%A7%E6%9D%A1%E4%BB%B6%E6%8C%87%E5%AE%9A%EF%BC%88filter%E3%81%99%E3%82%8B%E6%96%B9/)

[ドキュメント](https://docs.djangoproject.com/ja/3.2/intro/tutorial07/)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。