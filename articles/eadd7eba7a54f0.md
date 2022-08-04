---
title: "RailsチュートリアルでHerokuデプロイ時に発生したエラーを解消する"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ruby, rails, railsチュートリアル]
published: true
---

# はじめに

Ruby on Rails をこれから学ぶ方は Rails チュートリアルから始めるという方が多いと思います。
私も Rails チュートリアルは始めて記載通りに進めて行ったのですが、Heroku デプロイ時に２つのエラーが出てきてしまいました。
これのエラーを解決するのにかなり時間がかかってしまったので同じようなエラーに陥ってしまう方も多いと思いここに解決策を記載していこうと思います。

# エラー ①

Rails チュートリアルで第 1 章を終えて Heroku へデプロイする以下のコマンドを実行

```shell
$ git push heroku master
```

実行後の以下のエラーが出てきました。

```shell
# 出力
[省略]
remote:        rake aborted!
remote:        Psych::BadAlias: Unknown alias: default
[省略]
remote:  !
remote:  !     Precompiling assets failed.
remote:  !
remote:  !     Push rejected, failed to compile Ruby app.
remote:
remote:  !     Push failed
```

上記のようなログが出て Heroku にデプロイが失敗してしまいます。
エラー内容みると`Psych::BadAlias: Unknown alias: default`というようなエラーコードになっています。
これは Rails6.0 系を使用する際は `Psych` はバージョン 3 系を使用するようにすれば解決します
ということで`Gemfile`に以下を追加します。

```Ruby
gem 'psych', '~> 3.1'
```

変更後`bundle install`を実行、Heroku デプロイで成功しました。
こちらのサイトを参考にさせていただきました。
https://qiita.com/kandalog/items/8fd20f79ecf73034795a

# エラー ②

デプロイが成功したのでサイトを見てみると
以下のようなエラーでサイトが表示されませんでした。
![RailsApplicationerror](/images/articles/Rails_Application_error.png)

これだけでは何のエラーかわからないので以下コマンドでログを出力してみます。

```shell
$ heroku logs
# 出力
[省略]
2022-08-04T13:13:26.371490+00:00 app[web.1]: /app/vendor/bundle/ruby/3.1.0/gems/bootsnap-1.10.3/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:27:in `require': cannot load such file -- net/smtp (LoadError)
[省略]
2022-08-04T13:13:26.516972+00:00 heroku[web.1]: Process exited with status 1
2022-08-04T13:13:26.630333+00:00 heroku[web.1]: State changed from starting to crashed
2022-08-04T13:14:42.492165+00:00 heroku[router]: at=error code=H10 desc="App crashed" method=GET path="/" host=rails-tutorial-hello-app-12345.herokuapp.com request_id=779c2923-961d-479e-9153-afafade5a212 fwd="14.13.33.129" dyno= connect= service= status=503 bytes= protocol=https
2022-08-04T13:14:43.396388+00:00 heroku[router]: at=error code=H10 desc="App crashed" method=GET path="/favicon.ico" host=rails-tutorial-hello-app-12345.herokuapp.com request_id=b529b4b1-ffc0-449b-9860-b0ea0263b9a4 fwd="14.13.33.129" dyno= connect= service= status=503 bytes= protocol=https
```

エラー内容みると`cannot load such file -- net/smtp (LoadError)`というようなエラーコードになっています。
`net-smpt` というファイルが読み込みできていない、つまり`net-smpt`ライブラリがインストールできていないことになります。
調べてみると、元々標準ライブラリであったが Ruby3.1 から標準ライブラリでなくなったものが存在していないため今回のエラーが出てしまったようです。

以下のライブラリをインストールしデプロイするとエラーなく表示されました。

```Ruby
gem 'net-imap'
gem 'net-pop'
gem 'net-smtp'
```

デプロイ後画面
![RailsApplicationhelloWorld](/images/articles/Rails_Application_helloWorld.png)

無事 Rails チュートリアルと同じ画面が表示されました。

こちらのサイトを参考にさせていただきました。
https://qiita.com/iloveomelette/items/4f8304cba9601e48db38

# まとめ

環境構築＆デプロイにエラーはつきものですね。何かしらエラー解決の糸口が掴めるのでエラーが出るとまずはエラーコードで検索するようにしてます。
Rails チュートリアル通りに進めても今回のようなエラーでかなり躓いてしまったので少しでも同じような境遇の方の助けになれば幸いです。

# 参考資料

https://qiita.com/kandalog/items/8fd20f79ecf73034795a
https://qiita.com/iloveomelette/items/4f8304cba9601e48db38
https://qiita.com/li2003038/items/2d93b693fa5885c45276
