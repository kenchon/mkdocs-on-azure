## 📝 Azure Static Web Apps の作成

Azure Portal からリソース作成することもできますが，ここではより簡単な VSCode を用いた方法を紹介します。

1. VSCode の Azure Static Web Apps 拡張機能をインストールする

    ![vscode azure static web apps](./images/vscode-azure-static-web-apps.png)

    以降の手順では，VSCode の Azure Static Web Apps 拡張機能の [README.md](https://github.com/microsoft/vscode-azurestaticwebapps) に書いてある手順を実行します。

2. VSCode の左側タブの Azure アイコンをクリックして `+` アイコンをクリックすると，リソース作成ダイアログに進みます。

    ![](./images/create-azure-static-web-apps.png)

    ダイアログには下記を順に入力します。

    - Azure のリソース名 → 例）`document-site`
    - デプロイするブランチ名 → 例）`main`
    - ソースコードの場所 → `/`
    - Functions コードの場所 → "Skip for now"
    - ビルド生成物が配置される場所 → `/site`

    !!! Note
        Functions コードはここでは使わないためスキップしますが，簡単に説明しておきます。

        Azure Static Web Apps では Azure Functions のコードも取り扱うことができます（Functions との統合）。これにより，例えばウェブページに表示する値を Azure Functions で作成した API から動的に取得できるようにしたい！と思った時に，その API の実装とデプロイもまとめて行うことができるというものです。

3. リソース作成をしたら，GitHub 上のリモートリポジトリをみにいきましょう。
   
    すると Azure が自動でウェブサイトをデプロイするためのパイプラインを作成し，それが走っているのが見えると思います。
   
    しかし Azure が自動で作成してくれたパイプラインは失敗します。ソースコードのビルド方法や Python の依存パッケージに関する情報が `github-workflow.yml` に記述されていないからです。

    ということで，次はこのパイプライン（CI/CD）の設定を行いましょう！