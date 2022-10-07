---
title: "Docker初心者がRailsのローカル環境を構築してみる"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [docker, ruby, rails]
published: true
---

# はじめに

Web アプリケーションの構築をする際は **Docker** がよく使用されています。
`Docker` を採用することで `Ruby` のようなプログラミング言語をローカル環境自身にインストールなしに開発することができます。
さらに本番環境への適用もコンテナを配置するだけになるので OS の違いによるコードを書く必要がなくなります。

Docker については日本語ドキュメントがありますのでこちらが参考になります。
https://docs.docker.jp/

今回は Docker 初心者である私が勉強を兼ねてコンテナで Rails 環境を構築していきたいと思います。

## 実行環境

- macOS：Big Sur
- CPU：Intel

## 手順

1. Docker インストール
2. Rails コンテナ作成に必要なファイルを準備
3. コンテナイメージのビルド
4. Rails コンテナで Rails プロジェクト作成
5. データベースの準備
6. コンテナ起動＆動作確認

# Docker インストール

まずは Mac に Docker をインストールする必要があります。
以下サイトから「**Docker Desktop for Mac**」をインストールします。
https://matsuand.github.io/docs.docker.jp.onthefly/desktop/mac/install/

Docker Desktop の利用料金については商用利用の場合、有償となりますが今回は個人利用ですので問題ありません。

> 大規模エンタープライズ (従業員 250 名以上、または年間収益 1 千万 US ドル以上) 向けの商用利用に対しては、有償サブスクリプションが必要です。
> この有償サブスクリプションには、2022 年 1 月 31 日までの猶予期間が設けられています。 詳しくは こちら を参照してください。

インストール完了後、Docker Desktop が起動されれば OK です。

# Rails コンテナ作成に必要なファイルを準備

今回はわかりやすくするために**rails-docker**という作業フォルダを作成します。

```shell
$ mkdir rails-docker
$ cd rails-docker
```

作成したフォルダに以下のファイルを用意します。

- Dockerfile
- docker-compose.yml
- Gemfile
- Gemfile.lock
- entrypoint.sh

以下のコマンドで一括作成できます。

```shell
$ touch {Dockerfile,docker-compose.yml,Gemfile,Gemfile.lock,entrypoint.sh}
```

## Dockerfile

Dockerfile というのはコンテナの元になっているイメージを定義するファイルです。
Ruby をコンテナで動かしたいので Ruby のイメージをもとに必要なパッケージをインストールするよう記述しています。
今回は、Ruby バージョン 3.1.2 を使用するようにしていきます。

```Dockerfile:Dockerfile
FROM ruby:3.1.2

# データベース用にPostgreSQLをインストール
RUN apt-get update -qq && apt-get install -y postgresql-client
RUN mkdir /myapp
WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
# bundlerバージョンを上げないとエラーになる対策
RUN gem update --system 3.3.20 && bundle install
COPY . /myapp

# コンテナ起動時に実行させるentrypoint.shを追加
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

# Rails サーバ起動
CMD ["rails", "server", "-b", "0.0.0.0"]
```

## docker-compose.yml

docker-compose.yml というのはコンテナ自身を定義するファイルです。
今回は Web サーバー(Rails)と DB サーバー(PostgreSQL)のコンテナを起動するために「**db**」「**web**」という名前のコンテナを起動するように設定しています。

```yml:docker-compose.yml
version: "3.9"
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password
  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db
```

## Gemfile & Gemfile.lock

Gemfile には Rails で開発する際に使用する予定のパッケージ等を記述します。
今回はテストのため Rails のみがインストールされるように記述します。

```Gemfile:Gemfile
source 'https://rubygems.org'
gem 'rails', '~> 7.0.3.1'
```

Gemfile.lock についてはこちらで手を加える必要はないので何もしなくて OK です。

## entrypoint.sh

entrypoint.sh はスクリプトファイルで Dockerfile で指定したタイミングで実行されます。
今回は `server.pid` が存在するとサーバーが起動できない対策のために `server.pid` を削除するように設定しています。

```sh:entrypoint.sh
#!/bin/bash
set -e

rm -f /myapp/tmp/pids/server.pid

exec "$@"
```

# コンテナイメージのビルド

関連ファイルの準備ができたら以下コマンドでコンテナイメージをビルドします。

```shell
$ docker-compose build
```

コンテナを起動するためにはイメージが必要なのでここで作成します。

# Rails コンテナで Rails プロジェクト作成

コンテナで Rails プロジェクトを作成するために**web**コンテナ内で Rails new を実行させます。
コンテナ内でコマンド実行させるには`docker-compose run`というコマンドを使用します。

以下のコマンドを実行します。

```shell
$ docker-compose run web rails new . --force --no-deps --database=postgresql
```

実行後に Rails 関連ファイルが作成されているはずです。

# データベースの準備

Rails コンテナから DB コンテナに通信してデータベースを作成させるため Rails の`database.yml`を編集します。

```yml:database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: db
  username: postgres
  password: password

development:
  <<: *default
  database: myapp_development

test:
  <<: *default
  database: myapp_test
```

host を DB コンテナ名の**db**を指定することで Rails からみたデータベースを DB コンテナにすることができます。

データベースを Rails コマンドで作成します。

```shell
$ docker-compose run web rails db:create
```

# コンテナ起動＆動作確認

ここまで準備できれば、いよいよコンテナを起動していきます。
以下コマンドを実行します。

```shell
$ docker-compose up -d
```

Dockerfile の最後に`rails server`を実行するように指定しているので、コンテナを起動すると自動的に Rails サーバーが起動するようになります。

コンテナ起動後に Rails サーバーにアクセスしてみましょう。
ブラウザで http://localhost:3000/ と入力し以下のおなじみの画面が表示されていれば無事完了です。
![RailsStart](/images/articles/Docker-RailsStart.png)

動作確認が済んだらコンテナを削除しておきましょう。

```shell
$ docker-compose down
```

# まとめ

Docker 初心者からですとざっくりしたものしかイメージしづらいので、なんとなくコンテナ起動できればいいかとなりがちです。
なのでコンテナを起動するまでの流れや Dockerfile、Docker-compose の意味などを理解することがいいと思います。
そうすることで開発言語等が変わったとしても比較的簡単対応できるようになるのかと考えています。
今回扱ったファイルを GitHub に上げています。
https://github.com/Takuty-a11y/Rails-Docker-Sample

# 参考資料

https://zenn.dev/tmasuyama1114/articles/4ed199ce0478e7
https://mseeeen.msen.jp/rails-docker/
https://qiita.com/P-man_Brown/items/32fdba14e88219f8d2f0
