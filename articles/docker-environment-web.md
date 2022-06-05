---
title: "無料でdocker, travisCI, Herokuを使ってwebアプリの開発環境を作成してみた" # 記事のタイトル
emoji: "✅" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["docker", "heroku", "travisci", "初心者"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.07.19'
qiitaId: '74da09ad83db087722b0' #記事のslug
---

## 最初に

Docker Composeを使用してwebアプリを開発する環境を作成していこうと思う。

## Rubyが動作するDockerfileを作成する

```dockerfile
FROM ruby:2.5
RUN apt-get update && apt-get install -y \
  build-essential \
  libpq-dev \
  nodejs \
  postgresql-client \
  yarn

WORKDIR /product-register
COPY Gemfile Gemfile.lock /product-register/
RUN bundle install
```

`touch Gemfile Gemfile.lock` を作成する。

Gemfileのみ更新する。

```gemfile
source 'https://rubygems.org'
gem 'rails', '~>5.2'
```

## Docker Composeを使ってコンテナを実行する

上記のコンテナを実行する場合、 `docker run -V ~/Desktop/Product_register:/product-register -p 3000:3000 -it 152898587708 bash` このような長いコマンドになる。

毎回これを入力するのは大変なので `YAML = YAML ain't markup language` に記述して `Docker compose` からコンテナを実行する。

### docker-compose.yml を記述する

```yaml
version: '3'

services:
  web:
    build: .
    # portsは -pの役割
    ports:
      - '3000:3000'
    # volumesは -vの役割
    volumes:
      - '.:/product-register'
    # ttyは -itのtを意味してる
    tty: true
    # -iを意味してる
    stdin_open: true
```

### docker composeのコマンド

```bash
# docker build . と同じ
docker-compose build

# docker run imageID と同じイメージがなければbuildも一緒にしてくれる。
docker-compose up

# docker psと同じ
docker-compose ps

# docker exec コンテナIDと同じ
docker-compose exec サービス

# buildしてrun
docker-compose up --build

# stopしてrm
docker-compose down

# コンテナ停止とそのイメージを削除する際に使用する。
docker-compose down --rmi local -v

# コンテナの停止
docker-compose stop
```

docker composeからコンテナを実行してみる。 `-d` は `detach` でコンテナを実行したままコンテナから出るという意味になる。なので `docker-compose ps` でバックグラウンドでコンテナが起動しているのが確認できると思う。

```bash
docker-compose up -d
```

これで実行したコンテナに入る。

```bash
docker-compose exec web bash

ls

# 実行結果
Dockerfile  Gemfile  Gemfile.lock  docker-compose.yml
```

## railsの環境を構築する

dockerfileに `RUN bundle install` というコマンドが書かれているので `--skip-bundle` でバンドルインストールをスキップする。  `docker-compose exec web bash` でコンテナ内に入った後に下記のコマンドを実行する。

```bash
rails new . --force --database=postgresql --skip-bundle
```

そして `exit` でコンテナを出た後に再びビルドする。

`--build` オプションを付ける事でイメージがあっても強制的に再ビルドする。

```bash
docker-compose up --build -d
```

そして再びコンテナ内に入って `rails s -b 0.0.0.0` でrailsサーバを起動する。

[`http://localhost:3000/`](http://localhost:3000/) にアクセスするとデータベースの接続エラーが表示される画面が出力される。

`rails new` で作成された `database.yml` の設定ファイルに追加する。

```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  # 下記のパスワードまでを追加する
  # dbがdocker-compose.ymlのdbコンテナと一致してアクセスする。rubyのコンテナからpostgresのコンテナへ
  host: db
  user: postgres
  port: 5432
  password: <%= ENV.fetch("DATABASE_PASSWORD")%> 
  # For details on connection pooling, see Rails configuration guide
  # http://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

# ...この下にテスト・本番環境の設定がある。
```

そして `docker-compose.yml` にもdbコンテナの 設定を追加する。もしデータベースのコンテナが起動 `docker-compose up` 出来ない場合は環境変数の箇所にあるコメントアウトを外すと上手くいく。どうしてそうなるかはpostgresについて深掘りする必要がある。dbが起動するとタブが使用出来なくなるので別タブを開いてコンテナに入る `docker-compose exec web bash` そして `rails g scaffold product name:string price:integer vender:string` でアプリを作成する。そしてアプリの要求するデータベーステーブルを作成する `rails db:migrate` ここまで終了したら `rails s -b 0.0.0.0` でサーバを起動して [http://localhost:3000/products](http://localhost:3000/products) にアクセスするとアプリのページにいける。

```yaml
version: '3'

volumes:
  db-data:

services:
  web:
    build: .
    # portsは -pの役割
    ports:
      - '3000:3000'
    # volumesは -vの役割
    volumes:
      - '.:/product-register'
    
    # 環境変数に設定される
    environment:
      - 'DATABASE_PASSWORD=postgres'

    # ttyは -itのtを意味してる
    tty: true
    # -iを意味してる
    stdin_open: true
    # 先にdbを作成して下さいと指定する。
    depends_on:
      - db
    # webからdbにアクセスできるようになる。
    links:
      - db

  db:
    image: postgres
    # environment:
    #   - 'POSTGRES_USER=postgres'
    #   - 'POSTGRES_PASSWORD=postgres'
    
    # コンテナが削除されてもデータは残るようにローカル側のフォルダに保存するように設定する。
    # 下記のようなフォルダパスはなくdocker側でデータが管理される。実際のローカル環境からはアクセス出来ない。
    volumes:
      - 'db-data:/var/lib/postgresql/data'
```

### rails テスト

テスト機能が元々あり `test/controllers/products_controller_test.rb` ここにテストコードが書かれている。下記のコマンドでtestを実行できる。

```bash
rails test
```

## TravisCIを使ってテスト環境を構築してみる

最初にGitHubで `product-register` というリポジトリを作成して、そこに作成したrailsアプリのコードをあげる。

[Travis CI - Test and Deploy with Confidence](https://www.travis-ci.com/)

のページに行ってそこで GitHubでサインインした後に作成したリポジトリを連携する。

最近よく聞くGithub actionsも同じ類のCIツールだとは知らなかった。

`product-register/.travis.yml` を作成する。

```yaml
sudo: required

services: docker

before_install:
  - docker-compose up --build -d

script: 
  - docker-compose exec --env 'RAILS_ENV=test' web rails db:create
  - docker-compose exec --env 'RAILS_ENV=test' web rails db:migrate
  - docker-compose exec --env 'RAILS_ENV=test' web rails test
```

そして、サーバ上でPostgresが起動出来る用に `docker-compose.yml` のdbに環境変数 `'POSTGRES_HOST_AUTH_METHOD=trust'` を追加する。

```yaml
version: '3'

volumes:
  db-data:

services:
  web:
    build: .
    # portsは -pの役割
    ports:
      - '3000:3000'
    # volumesは -vの役割
    volumes:
      - '.:/product-register'
    
    # 環境変数に設定される
    environment:
      - 'DATABASE_PASSWORD=postgres'

    # ttyは -itのtを意味してる
    tty: true
    # -iを意味してる
    stdin_open: true
    depends_on:
      - db
    links:
      - db

  db:
    image: postgres
    environment:
      - 'POSTGRES_HOST_AUTH_METHOD=trust'
    #   - 'POSTGRES_USER=postgres'
    #   - 'POSTGRES_PASSWORD=postgres'
    volumes:
      - 'db-data:/var/lib/postgresql/data'
```

そして GitHubに変更をpushすると travisCIにデプロイされテストが実行される。問題がなければ下記のように `Done. Your build exited with 0` と表示される。

![https://user-images.githubusercontent.com/23703281/126106189-d22223be-2611-4aa3-8774-f892d13e7c98.png](https://user-images.githubusercontent.com/23703281/126106189-d22223be-2611-4aa3-8774-f892d13e7c98.png)

## Herokuに本番環境をあげる

herokuのアカウントを作成する。（クレジットカードがいらない。）

ログインしたら

create new app > Create  App 作成したら Resources > Heroku Postgres 選択して追加する。

次にrailsにある `database.yml` の productionの設定を下記のように変更する。

```yaml

... テスト環境用の設定

production:
  url: <%= ENV['DATABASE_URL'] %>
#
# production:
#   <<: *default
#   database: product-register_production
#   username: product-register
#   password: <%= ENV['PRODUCT-REGISTER_DATABASE_PASSWORD'] %>
```

Dockerfile.prod とすることで本番環境で使用するDockerfileを作成することが出来る。

```dockerfile
FROM ruby:2.5
RUN apt-get update && apt-get install -y \
  build-essential \
  libpq-dev \
  nodejs \
  postgresql-client \
  yarn

WORKDIR /product-register
COPY Gemfile Gemfile.lock ./
RUN bundle install
COPy . .
CMD ["rails", "s"]
```

そして Herokuに rails特有の環境変数を追加する。 `master.key` にある値を https://dashboard.heroku.com/apps/アプリ名/settings > Config Vars に `SECRET_KEY_BASE` と入力した valueの項目に貼り付けて addをクリックする。

上記の `DATABASE_URL` は Herokuでpostgresを追加した際に Config Varsに追加されるので、それをHerokuにデプロイされたrails側で読み込めるようになっている。そして docker-composeに記述された環境変数は herokuの方で読み込まれる `Dockerfile.prod` には記述されてないので `'DATABASE_PASSWORD=postgres'`

### HerokuとGitHub レポジトリを連携する

https://dashboard.heroku.com/apps/アプリ名/deploy/github > Deploy から GitHub Connectを選択する。そうするとGitHubのアカウントが出るのでそこから連携したいレポジトリを選択する。そして `wait for CI to pass deploy` にチェックを入れる。そして `Enable Automatic Deploys` をクリックする。

次に `.travis.yml` に下記の用コードを追加する。

```yaml
sudo: required

services: docker

before_install:
  - docker-compose up --build -d
  - docker login -u "$HEROKU_USERNAME" -p "$HEROKU_API_KEY" registry.heroku.com

script: 
  - docker-compose exec --env 'RAILS_ENV=test' web rails db:create
  - docker-compose exec --env 'RAILS_ENV=test' web rails db:migrate
  - docker-compose exec --env 'RAILS_ENV=test' web rails test

deploy:
  provider: script
  script: 
    docker build -t registry.heroku.com/$HEROKU_APP_NAME/web -f Dockerfile.prod .;
    docker push registry.heroku.com/$HEROKU_APP_NAME/web;
    heroku run --app $HEROKU_APP_NAME rails db:migrate;
  on:
    # masterにマージされた時だけデプロイする。
    branch: master
```

そして、travisCIで環境変数を設定する。 [https://travis-ci.com/github/GitHubユーザ名/アプリ名/settings](https://travis-ci.com/github/wimpykid719/product-register/settings) > Environment Variables に `HEROKU_USERNAME=_` と `HEROKU_API_KEY=heroku authorization token` と `HEROKU_APP_NAME=herokuでcreateしたapp名` を入力する。

`heroku authorization token` は [https://dashboard.heroku.com/account/applications](https://dashboard.heroku.com/account/applications) で作成することが出来る。

最後に `config/routes.rb` に `root 'products#index'` を追加することで herokuがアプリをデプロイしたページにアクセスした際に productsのページがルートページとして表示される。

これで GitHubにコードをpushするとtravisCIでテスト環境（docker-composeでビルドされるrailsのコンテナとpostgresのコンテナ）が走ってエラーがなければheroku（dockerfile.prod railsのみの環境 postgresはherokuのサイトから追加したのでコンテナとして必要ない）に本番環境にビルドされる。

## 今回の環境での開発フロー

実際の開発フローは Githubで featureブランチを切ってpushするとtravisCIのみが走る、その後プルリクエストを作成してコードがマージされる際にherokuにデプロイされる。

GitHubでの操作は下記の記事にまとめてある。

[すぐ業務に就けるようにGitHubで一人チーム開発を体験してみた。](https://techblog-pink.vercel.app/posts/git-github-team)

### 参照

とても分かりやすくおすすめの講座です。
[米国AI開発者がゼロから教えるDocker講座](https://www.udemy.com/share/103aTR2@FG5jV2FjcFEOc05BBXdzfj1HSldLYA==/)

[DockerComposeで生成したコンテナ・イメージ・ボリューム・ネットワークを一括削除 | TOMILOG](https://blog.ryou103.com/post/docker-compose-down/)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。