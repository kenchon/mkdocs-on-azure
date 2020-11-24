前の記事で Azure 上に Static Web Apps の作成までできました。

この時点までで，GitHub のリポジトリには GitHub Actions のパイプライン設定ファイルが自動で生成されます。

自動生成された設定ファイルのままでは，CI/CD を実行してくれる GitHub Actions の仮想マシンさんはビルド方法を知らないので，これを記述してあげる必要があります。

## Python の依存モジュールのインストール

GitHub Actions に Python の環境をセットアップするために，以下の行を追加します:

```yaml
steps:
- uses: actions/checkout@v2
- uses: actions/setup-python@v2
  with:
    python-version: '3.9'
      - name: Build
        run: |
          pip install mkdocs && pip install mkdocs-material && mkdocs build --clean
      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v0.0.1-preview
```



## ビルド