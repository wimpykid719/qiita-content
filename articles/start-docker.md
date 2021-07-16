---
title: "全くの0からDockerを学んでAWSにデータサイエンス可能なコンテナを立てる" # 記事のタイトル
emoji: "🐳" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["docker", "aws", "データサイエンス", "linux", "python"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
date: '2021.07.16'
qiitaId: 'e9df4f3a8273f255f183' #記事のslug
---

## 最初に

今回は全くの0からDockerを学んでいこうと思う。そこで何を学んでいったか書いていく。

## Linuxについて

Dockerに入る前にLinuxのコマンドについて学んでいく。

### シェル

shell（シェル）を通してKernelに命令を出している。自分のMacのシェルはbashを使用している。他にはzsh, sh等がある。シェルを使うのに必要なアプリがターミナルになる。ターミナル=シェルではない。

### 環境変数

自分が使用するシェルが何か環境変数の中身を表示させて確認する。

```bash
echo $SHELL

# 実行結果
/bin/bash
```

環境変数を作成する。

```bash
export AGE=20
echo $AGE

# 実行結果
20
```

## Linux コマンド

### cd （current directory）

パスを移動する際に使用する。

### pwd（print working directory）

今いるディレクトリを表示する。

### mkdir（make directory）

新しいフォルダを作成する。

### touch

新しいファイルを作成する。

### ls（list）

カレントディレクトリのファイル、フォルダを一覧表示

### rm

ファイル削除する。

### rm -r

フォルダを削除する。

## Dockerのインストール

Docker Hubに登録して、そこからインテルMac用のdmgをダウンロードしてインストールした。

### docker コマンド

これでDocker Hubにログインする。

```bash
docker login
```

Docker Hubから `hello-world` というリポジトリのイメージを持ってくる。

※下記のページのリポジトリをダウンロードした事になる。

[hello-world](https://hub.docker.com/_/hello-world)

```bash
docker pull hello-world
```

これで持ってきたイメージを確認する。

```bash
docker images
```

## イメージからコンテナを作成する

前回pullした `hello-world` からコンテナを作成する。

これでコンテナを作成する。

```bash
docker run hello-world
```

```bash
docker ps # これはアクティブなコンテナだけ表示するのでdocker runの後にすぐ終了するコンテナのため実行しても何も表示されない。
docker ps -a #オプションを付けて全てのコンテナを確認することができる。

# 実行結果
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
17180e0cffc8   hello-world   "/hello"   53 seconds ago   Exited (0) 52 seconds ago             tender_archimedes
```

## ubuntuのイメージでコンテナを作成する

ubuntuのイメージをプルしてないが、dockerが勝手にHostにイメージがなければDocker Hubからダウンロードしてきてくれる。 そしてbashを起動する。

dockerはimageをダウンロードする際にimage レイヤーに分けてダウンロードして組み立てる。

例えばubutnuを構築するのに必要なレイヤーが4つあったらそれをダウンロードして使用する。

こうすると何が便利なのかというとubutnuのレイヤーにアプリを追加したい際はubuntuレイヤー + アプリのレイヤーとすることでubuntuのレイヤーを再びダウンロードする必要はなくデータ量を節約することができる。

```bash
docker run -it ubuntu bash
```

業務ではこのubutnuレイヤーに自分たちが使用したいPythonだったり、Rubyのレイヤーを追加してただのubuntuイメージを更新して使用する。

dockerイメージを更新する方法は2種類ある。

1. コンテナから更新する。
2. Docker fileから更新する。

### コンテナからイメージを更新する。

dockerを終了する。

```bash
exit
```

終了したdockerを再び起動する。dockerのIDは `docker ps -a` で確認できます。

```bash
docker restart dockerのIDを指定する。
```

先ほどのようにdockerに対してコマンドを実行するには `exec` を使用する。

```bash
docker exec -it dockerID bash
```

`exit` 以外にも detachの `ctrl+p+q` キーを押すことでdockerからできることができる。

detachとexitの違いはexitはプロセスを切って出るがdetachの場合はプロセスを切らずにdockerから出る。なので `docker ps` でアクティブなコマンドとして表示される。

下記のコマンドで終了してないdockerに再び入ることができる。

```bash
docker attach dockerID
```

ここまでdockerを起動してbashを実行する手順

1. docker restart dockerID
2. docker exec -it dockerID bash （restartを実行してないとコンテナが起動してないのでエラーになる。）

restartで起動して exec でコンテナに入って `exit` で出ても restartでのプロセスは起動したままなので `docker ps` で起動した状態が確認できる。なんでかはわからないがdockerfileが関係している見たい。

testファイルが作成されたubuntuのdockerイメージを作成する。

```bash
docker commit 更新したいdockerID dockerName:tag
```

これでテストファイルが作成されたdockerイメージが出来上がる。

## 先ほど作成したイメージをリポジトリにpushして共有できるようにする

リポジトリ一つに対して1つのイメージで管理される。タグを使用するとイメージは同じだがバージョン別でタグを付け管理することができる。Ubuntu14.04やUbuntu18.04のように同じubuntuイメージだがバージョンが異なる。

Docker Hubにpushするためにイメージの名前をリポジトリに合わせる。dockerにイメージをpushする時にdockerはそのイメージ名でpush先を探すためpushしたいリポジトリに合わせて名前を変更する必要がある。

Docker Hub 以外にもレポジトリをpush出来る場所がある。

### dockerNameを変更する。

`docker tag 変更したいイメージ 変更したい名前` でコマンドを実行する。下記はdockerHubIdというユーザネームのpracticeというリポジトリにイメージをpushしたいので名前を変更する。

```bash
docker tag ubuntu:updated dockerHubId/practice
```

先ほどあげたDocker Hub以外にもpush出来るリポジトリがあると言ったが、どうやって指定するのか。

それは今まであげたイメージネームが実は下記のような構成になっている。

`<hostname>:<port>/<username>/<repository>:<tag>` デフォルトだと `hostname = registry-1.docker.io` とdocker hubのホスト名がデフォルト指定になっている。ここを変更すればdocker hub以外のリポジトリにpush、pullすることが出来る。 `username = library` になっている。 `tag = latest` になる。

イメージを変更したら `docker images` で確認すると変更されていることがわかる。

```bash
docker images

# 実行結果
REPOSITORY             TAG       IMAGE ID       CREATED             SIZE
ubuntu                 updated   c2309bbe5266   About an hour ago   72.7MB
wimpykid719/practice   latest    c2309bbe5266   About an hour ago   72.7MB
ubuntu                 latest    9873176a8ff5   3 weeks ago         72.7MB
hello-world            latest    d1165f221234   4 months ago        13.3kB

```

これを確認すると `IMAGES ID` が ubuntuとwimpykid719/practiceで同じことが確認できる。それは同じイメージ複数のタグが付いているイメージなる。

リポジトリに追加する。この際ubuntuのイメージレイヤーはdocker hubにあるのでローカルからpushされることはない。pushされるのは `test` ファイルを含んだレイヤーだけpushしている。

```bash
docker push イメージ名（自分の場合はwimpykid719/practice）
```

## docker hubにpushしたリポジトリをpullする

pullする前にローカルにある `wimpykid719/practice` を削除しておく。リポジトリと同じ名前のイメージがあるとエラーが起きる。

```bash
docker rmi イメージ名
```

リポジトリからpullする。 `docker images` とするとローカルにpullされたのが確認できる。

```bash
docker pull wimpykid719/practice:latest
```

## docker runを詳しく見る

docker runはコンテナをイメージから `create` + `start` を行なっている。

イメージからコンテナを作成する。

```bash
docker create イメージ名

# 実行結果
b28a66c32ab16c5a7837df2402b9bae0f9de62aad7ac247ed7bba0286125b8d8
```

この状態ではまだコンテナが作られただけで実行はされていない。ないのでSTATUSは `Created` になっている。

```bash
docker ps -a

# 実行結果
CONTAINER ID   IMAGE                  COMMAND    CREATED          STATUS                       PORTS     NAMES
b28a66c32ab1   hello-world            "/hello"   28 seconds ago   Created                                thirsty_chatelet
a7b5f9fffb7b   hello-world            "/hello"   2 minutes ago    Exited (0) 2 minutes ago               inspiring_shannon
a21104a79b6d   wimpykid719/practice   "bash"     3 hours ago      Exited (127) 2 minutes ago             angry_feistel
cdc77a94ae22   ubuntu                 "bash"     17 hours ago     Up 15 hours                            cool_ganguly
17180e0cffc8   hello-world            "/hello"   18 hours ago     Exited (0) 18 hours ago                tender_archimedes
```

コンテナを実行する。

```bash
docker start b28a66c32ab1 # コンテナ名またはコンテナを指定する。

# 実行結果
b28a66c32ab1 コンテナIDが返る。
```

`docker ps -a` で確認すると `exited` されたのが確認できる。なぜスタートしたのにすぐに終了するのか? これはdockerがコンテナを実行してすぐに終了するためである。なぜ `run` では出力されたテキストが出力されないのか? それは `start` ではコマンドの出力結果は表示されないためである。

下記のコマンドで出力結果が表示される。

```bash
docker start -a コンテナID
```

## コンテナ実行時にコマンドを上書きする

下記のコマンドはエラーになるなぜなら `hello-world` のコンテナに `bash` コマンドはないからである。

```bash
docker run -it hello-world bash（ここに上書きしたいコマンドを記述する）
```

これは実はデフォルトで `/bin/bash` を実行している。しかしこれだけだと `bash` の状態は保持されずコンテナがスタートして終了してしまう。そしてターミナルには何も表示されない。それは `-it` がないからである。

```bash
docker run ubuntu
```

## -it が何をしているのか?

`-i` はインプットを可能にする。

`-t` は出力を綺麗にする。

インプットなしでやるとどうなるのか。 Linuxにある `STDIN` チャネルをホストからコンテナに繋げているのが `-i` のためこれがないとコマンドをホスト側から入力してもコンテナに送ることが出来ない。

```bash
docker run -t ubuntu

# ubuntuに入った画面になるけどコマンドを入力しても何も起きない
```

これやると自分の環境だと別のターミナルタブを開いて`docker stop コンテナID` となるので注意が必要です。

`-t` がないと表示結果が綺麗でなくなります。

```bash
docker run -i ubuntu
# コマンド入力
ls

# 出力結果
bin
boot
dev
etc
home
lib
lib32
lib64
libx32
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

## Dockerオブジェクトの削除

終了した状態のコンテナしか削除出来ない。なので起動している場合は `docker stop コンテナID （スペースで複数のdockerIDを指定する事で終了できる。）`

```bash
docker rm コンテナIDまたはコンテナ名
```

これで終了したコンテナ全て削除することができる。そしてイメージも消える。

```bash
docker system prune
```

## コンテナはそれぞれの独立してファイルシステムがある

同じubuntuのイメージから作成してもコンテナ毎にファイルシステムは異なる。

ターミナルタブ1（コンテナ1）

```bash
docker run -it ubuntu bash
touch test1
```

ターミナルタブ2（コンテナ2）

```bash
docker run -it ubuntu bash
touch test2
```

## コンテナに名前を付ける

オプションnameを使用する事で好きな名前を付けれる。これはコンテナを常に起動しながら使う場合は名前があると良い。

```bash
docker run --name sample_container ubuntu
```

## コンテナを起動したまま使用すると起動してコマンド実行後削除

```bash
docker run -it -d ubuntu bash

# 実行後ターミナルに戻るコンテナは起動したまま
```

上記の対比としてコマンドを実行してコンテナを削除する。一回きりしか使用しないコンテナに対して行う。コンテナをExit後に削除する。

```bash
docker run --rm hello-world （イメージ名）
```

## Dockerfile

コンテナの設計図を記してある。これを元にイメージを作成できるので業務ではこちらを使用される。

Dockerfile

```
FROM ubuntu:latest
# コンテナ作成後に実行されるコマンド
RUN touch test
```

このDockerfileを使ってビルドする。下記だけだとダンブリングイメージと呼ばれる `<none>` のdocker イメージが作成される。

```
docker build .
```

なので名前とタグを指定してビルドする。

```
docker build -t new-ubuntu:latest .
docker run -it new-ubuntu bash
```

## Dockerfileのインストラクション

### FROM

ベースとなるイメージを決める。このイメージの上に独自でコマンドを実行したりする事でレイヤーが積み重なっていくイメージほとんどubuntuが使用される。

### RUN

右側のコマンドをLinuxコマンドとして実行できる。

### CMD

コンテナのデフォルトのコマンドを指定する。 ubuntuだったら `bash` になる。

書き方は

```
CMD ["コマンド", "コマンドの引数1", "コマンドの引数2"]
```

## RUN と CMDの違い

RUNはレイヤーを構成するが、CMDは構成しない。

## Dockerのレイヤーを少なくする。

実際の現場ではubuntuのイメージにパッケージをインストールして使用することが多い。

ubuntuでパッケージをインストールするコマンド

```bash
apt-get update #古いパッケージをインストールしないように最新にしておく。
apt-get install <package>
```

Dockerfile

```
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install xxx
RUN apt-get install xxx
RUN apt-get install xxx
RUN apt-get install xxx
```

こんな感じにインストールするとレイヤーが大きくなるので `&&` でコマンドを繋げる。順番は左から実行される。

`&&` でコマンドを繋げて `\` で改行して見やすくする。パッケージ名はアルファベット順がおすすめ。

```
FROM ubuntu:latest
RUN apt-get update && apt-get install \
xxx \
yyy \
zzz \
```

実際に作成したDockerfile

```
FROM ubuntu:latest
RUN apt-get update && apt-get install -y \
  curl \
  nginx
```

## Dokerfileでキャッシュを使う

これを使用すると前回ビルド時にインストールしたパッケージ等を再び行わなくて済む。

新しくパッケージを追加したくなったら

下記のようにコマンドを分けてあげるとキャッシュを使用する事ができる。ビルド時間の短縮に繋がる。

Dockerfileが完成したら最後にコマンドを繋げてあげれば良い。

```
FROM ubuntu:latest
RUN apt-get update && apt-get install -y \
  curl \
  nginx
RUN apt-get install -y \
  cvs
```

## Dockerビルドするにはディレクトリも必要

`docker build .` とビルドする際にカレントのディレクトリを指定する。そのディレクトリ内あるファイル等をビルドする際に使用することが出来るようにするためである。そのため `Dockerfile` だけを指定するのではなく `Dockerfile` のあるフォルダを指定する。これらをまとめてDocker Contextと呼ぶ。この際のコンテキスは環境、状況を意味する。基本的にこのDocker Context内に使用しないファイルは置かない。ビルド時にDocker demonにdocker contextが送信される。その際に使わないファイルでも転送する処理が走るため無駄に時間がかかってしまう。そして `Dockerfile` にdocker context内のファイルを使用する処理を書かなければイメージに追加されることはない。

※docker daemonはdockerをCLI操作するためのオブジェクト

## Dockerイメージにファイルを追加する

Dockerfileに `COPY` を追加してビルドする。

これでdockerイメージに sample.txt が 追加される。

```
FROM ubuntu:latest
RUN mkdir /new_dir
COPY sample.txt /new_dir/
```

もう一つファイルを追加する `ADD` というインストラクションがある。

これはtarの圧縮ファイルをコピーして解凍したいときにADDを使う。

## Dockerfileを切り替えて使用する

一つのビルドコンテキストで2種類のDockerfileを使っていきたい時、ビルドコンテキストにDockerfileを含めず、その上にある階層に複数のDockerfileを用意することで環境を変える事が出来る。

```bash
docker -f ../Dockerfile.dev build .
```

## CMDとENTRYPOINTの使い方

`CMD` はデフォルトで実行するコマンドを上書き出来るとした。そして `ENTRYPOINT` もデフォルトで実行するコマンドを設定出来る。

2つの何が違うのか

`ENTRYPOINT` を使用した場合、 `CMD` の動作は変わってくる。

`CMD` はコマンドのオプション・引数を受け取るものに変わる。

```
FROM ubuntu:latest
RUN touch test
ENTRYPOINT ["ls"]
CMD ["--help"]
```

そして `docker run image コマンド` でターミナルからコマンドを引数として受け取れなくなり、変わりに `ENTRYPOINT` に設定されたコマンドの引数のみを受け取るようになる。

例 `docker run image --help` 

`CMD` のみ使用した場合は `docker run image コマンド` と `CMD` をさらにターミナルから上書き出来る。

## コンテナの環境変数をDockerfileに記述する

コンテナ実行時に環境変数を使用したい場合は、 `ENV` インストラクションを使用する。

```
FROM ubuntu:latest
ENV key1 value
ENV key2=value
ENV key3="v a l u e" key4=v\ a\ u\ e
ENV key5 v a l u e

# 実行したコンテナのbashでenvと入力すると設定された環境変数が表示される
key4=v a u e
key5=v a l u e
key2=value
key3=v a l u e
key1=value
```

## 実行するコマンドのパスを設定する

パスを移動してコマンドを実行したい際 `RUN cd パス && そのパスで実行したいコマンド` を使用する事で設定出来るが `WORKDIR` インストラクションを使用してもパスを移動してコマンドを実行できる。

```
FROM ubuntu:latest
RUN mkdir sample_folder
WORKDIR /sample_folder
RUN touch sample_file
```

こうすると bashを開くのは `/sample_folder` のパスがスタートになる。

`WORKDIR` を使用する場合は `mkdir` コマンドを使用しなくてもパスがなければ作成される。

なのでこんな感じに短く記述できる。

```
FROM ubuntu:latest
WORKDIR /sample_folder
RUN touch sample_file
```

## コンテナとホスト側でファイルシステムを共有する

ホスト側で用意したファイルをイメージに含めずにDockerのコンテナで使用したい場合は下記のコマンドを使用してホスト側のファイルシステムをコンテナにマウントする。こうするとコンテナのルートディレクトリに `new_dir` が作成され、そのフォルダ内に `~/Desktop/mounted_folder` の内容が展開される。

```
docker run -it -v ~/Desktop/mounted_folder:/new_dir new-ubuntu2:latest bash 
```

## アクセス権限を制限してファイルシステムを共有する

ユーザIDとグループIDを指定してコンテナを実行する。

### 自身のユーザIDとグループIDを確認してみる

```bash
id -u
# 実行結果
501

id -g
# 実行結果
20
```

dockerfile

```bash
FROM ubuntu:latest
RUN mkdir created_in_Dockerfile
```

ビルドする。

```bash
docker build -t new-ubuntu2:latest .
```

ユーザを指定してコンテナを実行する。実行すると `root` ユーザではなく `I have no name` となる。

```bash
docker run -it -u $(id -u):$(id -g) -v ~/Desktop/mounted_folder:/created_in_run new-ubuntu2 bash

# 実行結果
I have no name!@adadc9522b1f:/$

ls -la

# 実行結果
drwxr-xr-x   1 root root    4096 Jul 12 04:25 .
drwxr-xr-x   1 root root    4096 Jul 12 04:25 ..
-rwxr-xr-x   1 root root       0 Jul 12 04:25 .dockerenv
lrwxrwxrwx   1 root root       7 Jun  9 07:27 bin -> usr/bin
drwxr-xr-x   2 root root    4096 Apr 15  2020 boot
drwxr-xr-x   2 root root    4096 Jul 12 02:21 created_in_Dockerfile
drwxr-xr-x   3  501 dialout   96 Jul 12 01:57 created_in_run
drwxr-xr-x   5 root root     360 Jul 12 04:25 dev
drwxr-xr-x   1 root root    4096 Jul 12 04:25 etc
drwxr-xr-x   2 root root    4096 Apr 15  2020 home
lrwxrwxrwx   1 root root       7 Jun  9 07:27 lib -> usr/lib
lrwxrwxrwx   1 root root       9 Jun  9 07:27 lib32 -> usr/lib32
lrwxrwxrwx   1 root root       9 Jun  9 07:27 lib64 -> usr/lib64
lrwxrwxrwx   1 root root      10 Jun  9 07:27 libx32 -> usr/libx32
drwxr-xr-x   2 root root    4096 Jun  9 07:27 media
drwxr-xr-x   2 root root    4096 Jun  9 07:27 mnt
drwxr-xr-x   2 root root    4096 Jun  9 07:27 opt
dr-xr-xr-x 167 root root       0 Jul 12 04:25 proc
drwx------   2 root root    4096 Jun  9 07:31 root
drwxr-xr-x   5 root root    4096 Jun  9 07:31 run
lrwxrwxrwx   1 root root       8 Jun  9 07:27 sbin -> usr/sbin
drwxr-xr-x   2 root root    4096 Jun  9 07:27 srv
dr-xr-xr-x  13 root root       0 Jul 12 04:25 sys
drwxrwxrwt   2 root root    4096 Jun  9 07:31 tmp
drwxr-xr-x  13 root root    4096 Jun  9 07:27 usr
drwxr-xr-x  11 root root    4096 Jun  9 07:31 var
```

その後、コンテナ内で `ls -la` で権限を確認する。

もしdockerのバグで表示が rootのままになっている場合は `ls -la created_in_run` と実行した後に再び `ls -la` と実行すると `drwxr-xr-x 3 501 dialout 96 Jul 12 01:57 created_in_run` と表示される。

### パーミッションの見方

```
d rwx rwx rwx
  所有者 所有グループ その他
```

一番左側はファイルの種類を表している。

d：ディレクトリ

_：ファイル

I：シムリンク

所有者・所有グループ・その他

r：読み取り

w：書き込み

x：実行

これで `created_in_Dockerfile` の権限を確認してみると下記のようになっているので

```
drwxr-xr-x   2 root root    4096 Jul 12 02:21 created_in_Dockerfile
```

`created_in_Dockerfile` は ディレクトリで 所有者は `読み書き実行可能` で所有グループは `読み実行可能` でその他も同様に `読み実行可能` 所有者は `root` 所有グループは `root` そのため先コンテナを実行した `id -u, id -g` では権限がないため `created_in_Dockerfile` に書き込みしようとすると `touch: cannot touch 'sample': Permission denied` と表示される。

なので共有サーバでコンテナを実行している際はアクセス権限を決めて実行したりする。そうすれば他のユーザはコンテナ内のファイルを確認したりはできるが書き込みは出来なくなる。

## ポートを使ってホストとコンテナ間を繋げる

ホスト側のブラウザからコンテナにアクセスして使用する `jupyter notebook` の環境を構築する。

```bash
docker run -it -p 8888:8888 --rm jupyter/datascience-notebook bash

# 実行結果
Unable to find image 'jupyter/datascience-notebook:latest' locally
latest: Pulling from jupyter/datascience-notebook
c549ccf8d472: Already exists 
6c65c50510f3: Pull complete 
a89eb75169a1: Pull complete 
4f4fb700ef54: Pull complete 
c18636dc46c6: Pull complete 
4f71d15c3a44: Pull complete 
7c5f5d979663: Pull complete 
a63f5c102e3f: Pull complete 
ec8b60697d02: Pull complete 
4ef8cbbd0ce3: Pull complete 
4e24d4e20fa9: Pull complete 
e888fd7dc15b: Pull complete 
014b3810d91b: Pull complete 
fd1d80db8223: Pull complete 
fb0bd36a1363: Pull complete 
08ea6858b237: Extracting [==================================================>]  244.1MB/244.1MB
6cfa3aad7e9a: Download complete 
487a0a7d48a3: Download complete 
b96652c05056: Download complete 
3475806cdab0: Download complete 
412bb6bca7e9: Download complete 
ca97efa92216: Download complete 
5c77367376d7: Download complete 
docker: failed to register layer: Error processing tar file(exit status 1): archive/tar: invalid tar header.
See 'docker run --help'.
```

実行するとエラーが発生して上手くコンテナを作成できなかったので、 とりあえず使ってないコンテナを全て削除して、いくつかイメージを削除して再びコマンドを実行したら作成できた。ここの詳細の原因については調べきれていない。

### コンフリクトでdocker イメージが削除出来ない場合

下記の状態で `docker rmi c2309bbe5266` を実行すると同じdocker イメージIDがあってコンフリクトが起こる。この場合は `docker rmi ubuntu:updated` とコンテナ名:タグで指定して削除すると上手くいく。

```bash
docker images

# 実行結果
ubuntu                         updated   c2309bbe5266   3 days ago    72.7MB
wimpykid719/practice           latest    c2309bbe5266   3 days ago    72.7MB
jupyter/datascience-notebook   latest    19a4681e1ba3   3 days ago    3.99GB
ubuntu                         latest    9873176a8ff5   3 weeks ago   72.7MB
```

bashに入れたら `jupyter notebook` とコマンドを実行するとコンテナでサーバが起動する。表示されたURLにブラウザからアクセスすると jupyter nootebook の画面が表示される。終了は `ctr + c` になる。

## コンテナに使用するCPUリソース、メモリを制限する

制限をかけないと複雑な処理をコンテナが行う際リソースを大量に消費してリソースが枯渇する。それを避けるためにリソースに制限をかける。

### 実際に使用するパソコン（Mac）のリソースを確認する

物理コアを確認する。

```bash
sysctl -n hw.physicalcpu_max

# 実行結果
2
```

論理cpuを確認する。Hyper Threadingを使用して物理コア1つの中でcpuが2つあるように見せている。

```bash
sysctl -n hw.logicalcpu_max

# 実行結果
4
```

メモリ数の確認する。

```bash
sysctl -n hw.memsize

# 実行結果 バイト数を表示する。
17179869184

#      B          KB    MB   GB
# 17179869184 / (1024*1024*1024)すると16となり16GB積んでるのが分かる。
```

### コンテナを起動する際のリソースを指定する

これは物理CPUとメモリを指定する。1CPUと最大2GBのメモリでコンテナを起動する。

```bash
docker run -it --rm --cpus 1 --memory 2g ubuntu bash
```

起動後に `docker inspect コンテナID` を使用してコンテナの詳細情報を確認してみる。起動してるコンテナのIDを調べるのは `docker ps` と入力する。

```bash
docker inspect コンテナID

# 実行結果
Jsonファイルみたいなのがたくさん表示される。
```

そのままでは見辛いので `grep` コマンドを使用して表示するデータを絞る。 `cpu` と記述された箇所だけ取り出す。 `-i` は大文字小文字を無視する。

```bash
docker inspect コンテナID | grep -i cpu

# 実行結果
"CpuShares": 0,
"NanoCpus": 1000000000,
"CpuPeriod": 0,
"CpuQuota": 0,
"CpuRealtimePeriod": 0,
"CpuRealtimeRuntime": 0,
"CpusetCpus": "",
"CpusetMems": "",
"CpuCount": 0,
"CpuPercent": 0,

docker inspect コンテナID | grep -i memory

# 実行結果
"Memory": 2147483648,
"KernelMemory": 0,
"KernelMemoryTCP": 0,
"MemoryReservation": 0,
"MemorySwap": 4294967296,
"MemorySwappiness": null,
```

`NanoCpus` がcpuのリソースを指している。これはナノスケールなので `10^9` で割ると `1` になり割り当てリソースは物理cpuを1つ割り当ててるということが分かる。

## データ解析用のDockerfileを書いてみる

ここからは上記で使用したコマンド類を使ってanacondaをインストールするDockerfileを書いていく。

Dockerfile

```
FROM ubuntu:latest
RUN apt-get update && apt-get install -y \
  sudo \
  wget \
  vim

WORKDIR /opt
RUN wget https://repo.continuum.io/archive/Anaconda3-2021.05-Linux-x86_64.sh
```

これでdockerビルドして、コンテナを実行してbashに入る。

```bash
docker run -it イメージID bash
```

すると `/opt` に ancondaのshファイルがダウンロードされているのでそれを実行する。

### anacondaをインストールする

```bash
sh Anaconda3 ダウンロードされたファイル.sh
```

Enterキーを押し続ける。途中でanacondaをインストールする場所を聞かれるので、その際は `/opt/anaconda3` とインストール先を変更する。 optディレクトリはLinuxでは追加アプリケーション用フォルダを意味する。

### anacondaのパスを通す

環境変数 `$PATH` にパスを追加する事で、コンピュータがプログラムをそのパスから探してきてくれるようになる。

現在の追加されているパスを確認する。6個のパスが追加されていることが確認できる。ここに新たにanacondaのパスを追加してコマンドが使用できるようにする。

```bash
echo $PATH

# 実行結果
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

パスを追加する。

どこにパスを追加すれば良いのか、大体はbinというフォルダに通す。

```bash
export PATH=/opt/anaconda3/bin:$PATH

echo $PATH

# 実行結果 PATHが追加された事が確認できる。
/opt/anaconda3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

### 上記の操作をdockerファイルに書き込んで自動で環境を作成できるようにする

anacondaのインストールする際の操作をオプションで行う。

```bash
sh -x Anaconda3-2021.05-Linux-x86_64.sh

# 実行結果
+ export OLD_LD_LIBRARY_PATH=
+ unset LD_LIBRARY_PATH
+ echo Anaconda3-2021.05-Linux-x86_64.sh
+ grep \.sh$
+ [ -n  ]
+ uname
+ [ Linux = Darwin ]
+ [ -d /proc ]
+ [ -r /proc ]
+ [ -d /proc/77 ]
+ [ -r /proc/77 ]
+ [ -L /proc/77/exe ]
+ [ -r /proc/77/exe ]
+ readlink /proc/77/exe
+ RUNNING_SHELL=/usr/bin/dash
+ [ -z /usr/bin/dash ]
+ [ ! -f /usr/bin/dash ]
+ [ -z /usr/bin/dash ]
+ [ ! -f /usr/bin/dash ]
+ [ -z /usr/bin/dash ]
+ [ ! -f /usr/bin/dash ]
+ dirname Anaconda3-2021.05-Linux-x86_64.sh
+ DIRNAME=.
+ cd .
+ pwd
+ THIS_DIR=/opt
+ basename Anaconda3-2021.05-Linux-x86_64.sh
+ THIS_FILE=Anaconda3-2021.05-Linux-x86_64.sh
+ THIS_PATH=/opt/Anaconda3-2021.05-Linux-x86_64.sh
+ PREFIX=/root/anaconda3
+ BATCH=0
+ FORCE=0
+ SKIP_SCRIPTS=0
+ TEST=0
+ REINSTALL=0
+ USAGE=
usage: Anaconda3-2021.05-Linux-x86_64.sh [options]

Installs Anaconda3 2021.05

-b           run install in batch mode (without manual intervention),
             it is expected the license terms are agreed upon
-f           no error if install prefix already exists
-h           print this help message and exit
-p PREFIX    install prefix, defaults to /root/anaconda3, must not contain spaces.
-s           skip running pre/post-link/install scripts
-u           update an existing installation
-t           run package tests after installation (may install conda-build)

+ which getopt
+ getopt bfhp:sut 
+ OPTS= -- 
+ [ ! 0 ]
+ eval set --  -- 
+ set -- --
+ true
+ shift
+ break
+ [ 0 = 0 ]
+ uname -m
+ [ x86_64 != x86_64 ]
+ uname
+ [ Linux != Linux ]
+ printf \n

+ printf Welcome to Anaconda3 2021.05\n
Welcome to Anaconda3 2021.05
+ printf \n

+ printf In order to continue the installation process, please review the license\n
In order to continue the installation process, please review the license
+ printf agreement.\n
agreement.
+ printf Please, press ENTER to continue\n
Please, press ENTER to continue
+ printf >>> 
>>> + read -r dummy
```

これを実行すると sh で取れるオプションが表示される。ここで `-b` と `-p` を使用する。

`-b` ：バッチモードでインストールする。

`-p` ：インストール先のフォルダを指定する。

### Dockerfileに追加してみる

anacondaのインストールをバッチモードで実行して、インストーラを削除して、インストール先は `/opt/anaconda3` にして最後に環境変数を追加するという事を行った。

次に追加で `pip` もアップデートしておく。

そして 作業ディレクトリを `/` に戻す。

そしてデフォルトのコマンドで `jupyter lab` を起動する。

```
FROM ubuntu:latest
RUN apt-get update && apt-get install -y \
  sudo \
  wget \
  vim

WORKDIR /opt
RUN wget https://repo.continuum.io/archive/Anaconda3-2021.05-Linux-x86_64.sh && \
  sh Anaconda3-2021.05-Linux-x86_64.sh -b -p /opt/anaconda3 && \
  rm -f Anaconda3-2021.05-Linux-x86_64.sh
  
ENV PATH /opt/anaconda3/bin:$PATH

RUN pip install --upgrade pip

WORKDIR /
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--allow-root", "--LabApp.token=''"]
```

これをビルドして `docker run -p 8888:8888 909c224ca062` で実行すると `[localhost:8888](http://localhost:8888)` で `jupyter lab` を開く事ができる。

## ファイルシステムを共有する。

前回行ったように、ファイルをマウントしてコンテナを実行する事でコンテナとホスト間のファイルシステムを共有できる。そのためコンテナにある `/work` ディレクトリ内にファイルを生成すると `/ds_python` フォルダ内にファイルが生成される。

その生成されたファイル自体はコンテナには存在しない、あくまで `/ds_python` をコンテナがマウントしているだけであるため。

```
docker run -p 8888:8888 -v ~/Desktop/docker_for_dsenv/ds_python/:/work --name my-lab 909c224ca062
```

## 上記で作成したdockerをAWSのインスタンス上で建てる

1. AWSのアカウントをAlways freeプランで作成する。（それでもインスタンスの起動時間が月750時間越えると課金されるらしい。）
2. ES2インスタンスをubuntuサーバで建てる。
3. インスタンス設定、ストレージ追加はデフォルトの選択のまま進む。
4. タグは `key=mydocker`, `value=dsenv` とする。
5. セキュリティは練習なので弱めに `全てのトラフィック` で設定する。
6. 作成ボタンをクリックすると `キーペア` を `mydocker` と名前を付けダウンロードするモーダルウィンドウが出るのでダウンロードする。のちにsshでawsのインスタンスに接続する際に使用する。

これで[インスタンス一覧のページ](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Instances:)でインスタンスが起動した事が伺える。このインスタンスは使わないときはインスタンスの状態をクリックして停止しておく、そうしないと月750時間（31.25日）の時間が進む。

## ダウンロードした鍵の権限をきつくする

`chmod` コマンドを使用して変更する。これを使用するとパーミッションが変更できる。

パーミッションの読み方の箇所で登場した所有者等の各項目に数字を指定する事で権限を設定できる。

数字は `4: read, 2:write, 1:execute, 0:no permission` の合計値で設定する。なのでread, writeの権限を所有者に設定したい場合は一番左の7に `4+2 = 6` を設定する。

```bash
chmod 777 fileパス

#     所有者、所有グループ、その他
```

今回の鍵は所有者（自分）だけがreadできるようにしたいので、 `400` をパーミッションとして設定する。確認すると ファイルで所有者のみ読み込める事が確認できる。

```bash
chmod 400 mydocker.pem
ls -la | grep -i docker.pem

# 実行結果
-r--------@  1 username  staff     1700  7 13 16:36 mydocker.pem
```

## 鍵を使ってsshでAWSのインスタンスにログインする

インスタンス一覧のページ下部にあるネットワーキングとパブリックDNSのコピーしてDNSアドレスの箇所に貼り付ける。アドレスはインスタンスを起動するたびに変わるので注意が必要。

```bash
ssh -i mydocker.pem ubuntu@DNSのアドレス
```

## AWSにDockerをインストールする

ubuntuインスタンスにログインした後はパッケージのアップデートする。ubuntuユーザは権限を持っていないので、 `sudo` をつけないと権限エラーになる。

```bash
sudo apt-get update
```

その次にdockerをインストールしていく。

```bash
sudo apt-get install docker.io

sudo docker --version

# 実行結果
Docker version 20.10.2, build 20.10.2-0ubuntu1~20.04.2
```

この毎回 `sudo` を付けるのは大変なのでdockerというユーザグループを作成してそこにubuntuを追加する。 `-a` はグループにユーザを追加するオプション、なので下記のコマンドはubuntuユーザをdockerグループに追加するコマンドになる。これで `sudo` なしで実行できるのはdockeコマンドだけという事を注意してほしい。 `apt-get` 等は以前 `sudo` が必要である。 これはdockerの内部機能でdockerというグループに所属しているユーザはdockerコマンドの権限を貰える。

```bash
sudo gpasswd -a ubuntu docker

cat /etc/group | grep -i docker

# 実行結果
docker:x:119:ubuntu

読み方としては グループ名：シャドウパスワード：グループID(GID)：所属するユーザ
```

GIDが`119` は `admin` らしい。

## 作成したDockerイメージをAWSのインスタンスに立てる

3種類の方法がある。

1. Dockerレジストリを使う（docker hubからイメージをpullする）
2. Dockerfileを送る
3. Docker imageをtarにして送る

### Docker imageをtarにして送る

```
FROM alpine
RUN touch test
```

Dockerfileを作成してビルドする。出来たイメージを `docker save` コマンドを使用してtarファイルにする。実行すると作業ディレクトリにtarファイルが作成される。

```bash
docker save イメージID > ファイル名.tar
```

### SFTPを使ってAWSにtarファイルをアップロードする

ファイルをアップする際は `ssh` でインスタンスに接続するのではなく、 `sftp` コマンドを使用する。

```bash
sftp -i mydocker.pem ubuntu@DNSアドレス
```

ログイン出来たら `put` を使用して tarファイルをアップロードする。ここでローカル側のtarファイルのパスを指定する。一番右は ubuntuインスタンスのパスを指定する何もしなければルートにtarファイルがアップロードされる。 

```bash
put temp_docker/myimage.tar
```

インスタンスからファイルをダウンロードすることもできる。

```bash
get 取得したいファイル名
```

### tarファイルからDockerイメージを作成する

先ほどアップロードした tarファイルを `docker load` コマンドを使ってDockerイメージを作成する。

sshでubuntuインスタンスにログインする。作成したコンテナはalpineのためshで実行する。これでコンテナをAWSで実行する事が出来た。

```bash
docker load < myimage.tar

docker images

# 実行結果
REPOSITORY   TAG       IMAGE ID       CREATED             SIZE
<none>       <none>    1c4791a4d285   About an hour ago   5.6MB

docker run -it  1c4791a4d285 sh
```

## AWSに解析用のDockerコンテナを建てる。

今のままだとボリュームが8GBしかなく容量が少ないので、20GBまで増量する。無料枠では30GBまでは使えるらしい。ボリュームの確認は `df -h` コマンドで確認できる。

[Elastic Block Store](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Volumes:sort=desc:createTime) のページにあるアクション > ボリュームの変更をクリックしてサイズを8GBから20GBに変更する。

その後 `exit` してから再びインスタンスにログインする。

それで `df -h` しても容量が変わってない事がある。

その場合、追加でコードを実行する必要がある。 `lsblk` でブロックデバイス（HDD, SSDストレージデバイスを指す）をツリー上で表示する。

```bash
lsblk

# 実行結果
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0     7:0    0 33.3M  1 loop /snap/amazon-ssm-agent/3552
loop1     7:1    0 55.5M  1 loop /snap/core18/1997
loop2     7:2    0 55.5M  1 loop /snap/core18/2074
loop3     7:3    0 70.4M  1 loop /snap/lxd/19647
loop4     7:4    0 67.6M  1 loop /snap/lxd/20326
loop5     7:5    0 32.3M  1 loop /snap/snapd/11588
loop6     7:6    0 32.3M  1 loop /snap/snapd/12398
xvda    202:0    0   20G  0 disk 
└─xvda1 202:1    0    8G  0 part /

sudo growpart /dev/xvda 1
lsblk

# 実行結果
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0     7:0    0 33.3M  1 loop /snap/amazon-ssm-agent/3552
loop1     7:1    0 55.5M  1 loop /snap/core18/1997
loop2     7:2    0 55.5M  1 loop /snap/core18/2074
loop3     7:3    0 70.4M  1 loop /snap/lxd/19647
loop4     7:4    0 67.6M  1 loop /snap/lxd/20326
loop5     7:5    0 32.3M  1 loop /snap/snapd/11588
loop6     7:6    0 32.3M  1 loop /snap/snapd/12398
xvda    202:0    0   20G  0 disk 
└─xvda1 202:1    0   20G  0 part /
```

`sudo growpart /dev/xvda 1` コマンドを使用するのは最初AWSから拡張したボリュームは `xvda` の最大値をあげるもので、その下にある `xvda1` のパーティションは変わらなかったが、最大値をあげる事でさらに大きくできるようになり、 `growpart` を使用して `xvda1` のボリュームを最大まで大きくしたイメージだと思う。

これでancondaをインストールできるボリュームが確保出来たので、 `docker build .` してコンテナを実行する。

```bash
docker run -v ~:/work -p 8888:8888 d0ce8fe5aaa0
```

実行したらブラウザで `パブリックDND:8888` でアクセスするとjupyter labにアクセスできる。

## AWSのubuntuに複数のユーザを作成してコンテナ内のアクセス権限をユーザ毎に制限する

まずubuntuのユーザを複数作成する。AWSのubuntuインスタンスにsshでログインして下記のコマンドを実行する。パスワード等聞かれますが使わないので適当に入力する。ユーザが `/home` ディレクトリに作成されたのが確認できる。

```bash
sudo adduser --uid 1111 aaa

sudo adduser --uid 2222 bbb

cd /home
ls

# 実行結果
aaa  bbb  ubuntu

# 1111ユーザでログインする。コンテナにはaaaホームディレクトリとbbbホームディレクトリをマウントする。
docker run -u 1111 -v /home/aaa:/home/aaa -v /home/bbb:/home/bbb -it ubuntu bash

id -u

# 実行結果
1111
# 1111で権限を持っているのがわかる。

# この状態で
cd /home/bbb

touch sample

# 実行結果 権限がないので拒否される。
touch: cannot touch 'sample': Permission denied

cd ../
ls -la

# 実行結果

drwxr-xr-x 1 root root 4096 Jul 14 06:34 .
drwxr-xr-x 1 root root 4096 Jul 14 06:34 ..
drwxr-xr-x 2 1111 1111 4096 Jul 14 06:20 aaa
drwxr-xr-x 2 2222 2222 4096 Jul 14 06:21 bbb

```

パーミッションを確認すると bbbフォルダは 所有者 2222 は読み書き実行可能 所有グループ 2222 ユーザが読み実行可能になっている。ためユーザ 1111は bbbフォルダ内にファイルを書き込む事が出来ない事が確認できる。

## 最後に

今回は全くの0からDockerを学んでdockerのコンセプト仕組み等を学んだ。Dockerfileに記述しておけばすぐに環境を壊しても作成できるのでとても便利なツールだと思った。以前はvagrantを使用して仮想環境を作成していたのですが、それよりも軽くアプリケーション毎にコンテナを小さく作っていくコンセプトが個人的に良いなと思った。そしてKubernetesというやつはそれぞれのコンテナを操作する際に使うツールなんだろうなって気がしてきた。

### 参照

とても分かりやすくおすすめの講座です。
[米国AI開発者がゼロから教えるDocker講座](https://www.udemy.com/share/103aTR2@FG5jV2FjcFEOc05BBXdzfj1HSldLYA==/)

[【Linux 】ファイル名を変更、ファイルを移動する「mv」コマンドの使い方](http://kawatama.net/web/1372)

[How to remove multiple docker images with the same imageID?](https://stackoverflow.com/questions/32944391/how-to-remove-multiple-docker-images-with-the-same-imageid)

[gpasswd【コマンド】とは｜「分かりそう」で「分からない」でも「分かった」気になれるIT用語辞典](https://wa3.i-3-i.info/word13795.html)

[Linux グループ一覧の確認と/etc/group ファイル](https://kazmax.zpp.jp/linux_beginner/etc_group.html)

[【 lsblk 】コマンド――ブロックデバイスを一覧表示する](https://www.atmarkit.co.jp/ait/articles/1802/02/news021.html)

[Dockerで未使用オブジェクトを消す「prune」オプションの整理 - Qiita](https://qiita.com/zembutsu/items/f577ea8dad6dc64d70b6)

[docker images を全削除する - Qiita](https://qiita.com/fist0/items/2fb1c7f894b5bdff79f4)

記事に関するコメント等は

🕊：[Twitter](https://twitter.com/Unemployed_jp)
📺：[Youtube](https://www.youtube.com/channel/UCT3wLdiZS3Gos87f9fu4EOQ/featured?view_as=subscriber)
📸：[Instagram](https://www.instagram.com/unemployed_jp/)
👨🏻‍💻：[Github](https://github.com/wimpykid719?tab=repositories)
😥：[Stackoverflow](https://ja.stackoverflow.com/users/edit/22565)

でも受け付けています。どこかにはいます。