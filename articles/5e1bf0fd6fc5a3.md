---
title: "ZennとGitHub連携して実際に記事投稿してみる"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "github"]
published: true
---

# はじめに

前回記事からの続きです
https://zenn.dev/takuty/articles/ac72d3e7301553

今回は実際に記事ファイルを作成し、記事を md で記述 ⇄ プレビュー閲覧
Git リポジトリへプッシュして実際に投稿されるまでの手順を記載していきたいと思います。

### 手順

- 記事新規作成
- 記事の書き方
- プレビュー閲覧
- 記事公開

# 記事新規作成

Zenn 公式の記事を参照して行いました。
https://zenn.dev/zenn/articles/zenn-cli-guide

## 作成コマンド実行

以下のコマンドで新しい md ファイルが作成されます。

```shell
$ npx zenn new:article
```

作成された md ファイルはリポジトリの「articles」配下に存在します。
※今回は`5e1bf0fd6fc5a3.md`というファイルが作成されました
![articleFolder](/images/articles/articleFolder.png)

# 記事の書き方

作成されたファイルは既に以下のような記載がされています。
![articleHeader](/images/articles/articleHeader.png)

- title：この記事のタイトル
- emoji：タイトルの上に表示される絵文字(初期作成のままで良いと思います)
- type：tech 技術記事 / idea アイデア
- topics：記事のタグ(この記事であれば zenn と GitHub)
- publisjed：true 公開 / false 非公開

これらを編集し終えたら下部から本文を md で記載していきます。

## よく使う記述方法

Zenn 記事は md で記述していきます。
md の基本的な記述方法はこちらのサイトが参考になります。
https://qiita.com/oreo/items/82183bfbaac69971917f

### 1. Web サイトへのリンク

```md
以下のサイトのリンクです
https://zenn.dev/takuty
```

**[表示結果]**

以下のサイトのリンクです
https://zenn.dev/takuty

### 2. 画像の表示

画像はリポジトリ内の images/articles フォルダ内に配置することで表示してくれます。
デフォルトでは images も articles も無いので自身でフォルダ作成しましょう。

```md
画像を表示します
![ダミー画像](/images/articles/20220713234439035_150x150.png)
```

**[表示結果]**

画像を表示します
![ダミー画像](/images/articles/20220713234439035_150x150.png)

### 3. コード記載

````
```HTML　//ここは言語名
<p id="text">テキスト</p>
<Button class="button">ボタン</Button>
```　
````

```HTML
<p id="text">テキスト</p>
<Button class="button">ボタン</Button>
```

言語名を記載すると自動でその言語のカラーリングになってくれます。

# プレビュー閲覧

プレビューを起動すると Zenn 上で記事がどのように表示されるか閲覧することができます。
一般的な表示と Zenn 上の表示は異なることがありますのでプレビューを見ながら作業することをおすすめします。

以下のコマンドでプレビューを起動します。

```shell
$ npx zenn preview
```

すると

```shell
👀 Preview: http://localhost:8000
```

と表示されるのでブラウザで URL に移動するとプレビューが起動しています。
![ZennPreview](/images/articles/ZennPreview.png)

# 記事公開

記事の`publish` を `true` にする必要があります。

```md
---
title: "記事のタイトル"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github"]
published: true
---
```

`published_at` で日時指定すると公開予約もできるみたいですね。

```md
---
title: "記事のタイトル"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github"]
published: true
published_at: 2099-12-31 23:59 # 未来の日時を指定する
---
```

この状態でコミット → リポジトリへプッシュすると記事投稿が完了します。
記事を更新する時もコミット → プッシュするだけで Zenn 上で記事が更新されます。

## 記事の削除

記事の削除はリポジトリ上から削除しても Zenn 上では削除されません。
記事を削除するには、Zenn ダッシュボード＞記事の管理から削除します。
![articleDelete](/images/articles/articleDelete.png)

# 最後に

記事の作成がコミット＆プッシュで出来てしまうのはとても便利ですね。
さらに md を記述していくだけで記事の見た目もかなり綺麗になるのも Zenn の使いやすいところかなと感じています。
慣れてしまえばすぐに記事を書き始めることができるので、まだ Git 連携されていないという方はぜひ環境を構築することをおすすめします！
