この記事は Advent Calendar 2020 「求ム！Cloud Nativeアプリケーション開発のTips！【PR】日本マイクロソフト」 の1日目の記事です。

# 💪 モチベーション

開発をしていると **📖 ドキュメントサイト** を公開したくなることがあると思います。ですが，

- 👩‍💻　**フロントエンドの技術スタックがない**
- ✨　**簡単・いい感じにウェブページを作りたい**
- ⛏　**メンテしたくない、運用コストかけたくない**

みたいなお気持ちになったことはないでしょうか？？私はあります。

この記事では，↑のような方にとって

- **静的サイトジェネレータ MkDocs で作ったドキュメントサイトを**
- **Azure Static Web Apps にデプロイする**

のが一つの解かも知れんよ，という話をしたいと思います。普段フロントエンドに携わっていない人でも「Web アプリ作ってデプロイするの簡単じゃん」と思っていただけたら嬉しいです。

- できるもの: https://brave-rock-07e453310.azurestaticapps.net
- ソースコード: [mkdocs-on-azure](https://github.com/kenchon/mkdocs-on-azure)

<img alt="screenshot" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/182818/37172b7a-5fdc-8391-ae8e-1cda1d10e7f2.png">


# 👨‍👩‍👧 対象読者と前提

**この記事は，以下の人にむけた内容になっています**

- Azure を使っていてドキュメントサイト作りたいな〜という人
- フロントエンドよくわからないけど，
    - いい感じのデザインにしたい！な人
    - 低コストで簡単にウェブページをデプロイ・管理したい人
- Web アプリも Python で作りてえんだ！な人

**また，以下の内容については既知 or 既にあるという前提で話を進めます**

- Python 3.x がインストールしてある
- Git 使ったことある，GitHub のアカウントがある
- VSCode 使ったことある
- Azure のアカウントがある（今回使う Azure Static Web Apps は無料で利用できます）

# 🤷 どうやってやるの？

大まかな流れは以下のようになります：

**1. 静的サイトジェネレータのセットアップ**
**2. リポジトリのセットアップ**
**3. Azure Static Web Apps のリソース作成**
**4. CI/CD のセットアップ**

次に，使用する **静的サイトジェネレーター** と **Azure Static Web Apps** について説明します

## 静的サイトジェネレーター - [MkDocs](https://www.mkdocs.org/)

**静的サイトジェネレータ** [^1]として，本記事では Python 製の OSS である [MkDocs](https://www.mkdocs.org/) を使います。

特にフロントエンド開発未経験の方がドキュメントサイトを構築するのに，MkDocs を使うのが良い選択肢であると感じる理由は以下の通りです。

- **機能面**
    - プラグインが充実 （数式表現，コードのシンタックスハイライトなど）
    - デフォルトで全文検索機能をサポート
    - テーマが充実 (`pip install` で利用可能)
- **手軽さ**
    - 設定ファイルで簡単にカスタマイズ可能（HTML，CSS，JS が隠蔽）
    - 環境構築の簡単さ
        - `pip install mkdocs` && `mkdocs new mysite` && `cd mysite` && `mkdocs serve`

[世の中にはいろんな静的サイトジェネレータがあり](https://jamstack.org/generators/)，その中でもドキュメントサイトに特化した OSS をいくつか使ったことがあるのですが，MkDocs は必要十分な機能を簡単にプラグインとして取り込めるのが良いところです。

> **📚　NOTE**
> 機能を網羅的に知りたい方は，@mebiusbox2 さんの[『MkDocsによるドキュメント作成』](https://qiita.com/mebiusbox2/items/a61d42878266af969e3c) が参考になります。

ちなみに，Python の有名なフロントエンドフレームワーク django や Flask と MkDocs はどのように違うのでしょうか？前者は基本的にサーバー側で動的に HTML を生成するので，Web Apps などを利用した Python 実行環境が必要になります。しかし，MkDocs の場合は HTML を生成するのはビルド時のみなので，あとはビルド時に生成した静的コンテンツをホスティングするだけです。

図にすると以下のようになります。
![スクリーンショット 2020-12-01 1.32.23.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/182818/d77f7e3b-d467-5ed9-3abd-ee6a6e59c7cc.png)

動的に HTML を生成する django や Flask はサービス提供時も常に Python 実行環境が稼働している必要があるので，Web Apps といったリソースを使う必要があります。しかし，MkDocs では， CI/CD などでビルドして一度 HTML を生成したら，それをホスティングするサービスさえあれば十分です。そこで，デプロイ先として Azure Static Web Apps や Azure Storage Blob などの，ホスティングに最適化したサービスを選べばコストも抑えられますし，ホスティングサーバの面倒は Azure がみてくれるので（サーバーレス）便利です。

## [Azure Static Web Apps](https://docs.microsoft.com/ja-jp/azure/static-web-apps/overview)

ちょうど最近リリースされたばかりの Azure のリソースで，現在はプレビュー版となっていますが，Azure を使っているなら **今後フロントエンド開発における最良のリソースの選択肢の1つになると注目されている** [^4] ものです (公式 docs は[こちら](https://docs.microsoft.com/ja-jp/azure/static-web-apps/overview))。

理由は，以下の通りです。

- **無料** [^2]
- **高速** [^3]
- **便利機能の数々！**（認証，ブランチごとの自動デプロイ...）
- **サーバーレス**

無料で使えるホスティングサービスには例えば Netlify や GitHub Pages などがありますが，Azure が優れているのは **地理的分散** による高速化のための仕組みや **認証機能**，**ブランチごとの自動デプロイ機能** などがデフォルトでサポートされている点です。またサーバーレスであり，（定義上）リソースのメンテナンスは不要 [^5]です。

Azure Static Web Apps についてのより詳しい話は miyake さんの [『Azure App Service に Static Web Apps が登場！』](https://k-miyake.github.io/blog/appservice-static-webapps/) をご覧ください。

>**📚　NOTE**
>
現在 **Azure Static Web Apps は preview 版** であり， **GitHub + GitHub Actions 以外のコードベースと CI/CD サービスをサポートしていません**[^6] 。以下に該当する方は，**Azure Storage Blob** を用いる方法 [^7] をオススメします。<br><br>
>
> - production 環境で使用することを検討している場合
> - GitHub を利用していない場合（GitLab, Azure DevOps など）
>
>Azure Storage Blob を使ったデプロイの方法は，この記事の後半の方に簡単に書いています。

## 使用する技術スタック

以上をまとめると，この記事で使用する技術スタックは以下の感じになります。

|名前|用途|
|:--|:--|
|**GitHub Repos**|記事・設定・パイプラインの管理|
|**GitHub Actions**|ビルド・デプロイ用パイプライン|
|**MkDocs**|静的サイトジェネレータ。アプリのソースコード（HTML，CSS，JS）を生成する|
|**Azure Static Web Apps**|静的サイトのホスティング|

それでは早速，やり方に入っていきます。
