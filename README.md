
<!-- markdownlint-disable MD026 -->
# Donkey Docs&reg;
<!-- markdownlint-restore -->

このドキュメントのソースは `./site` フォルダーでビルドされ、[公式サイト](http://docs.donkeycar.com/) に公開されています。
ドキュメントでは MkDocs による拡張 Markdown を利用しています。

## ドキュメントのビルド

* Python3 環境を作成します: `python -m venv env`
* Python 環境を有効化します: `source env/bin/activate`
* MkDocs をインストールします: `pip install mkdocs`
* MkDocs Redirect をインストールします: `pin install mkdocs-redirects`
* `mkdocs serve` を実行するとローカルのポート 8000 で Web サーバーが起動します。これはライブサーバーで、docs フォルダー内の .md ファイルを保存すると更新されます。変更を確認するため、編集中はこのサーバーを起動しておくと便利です。
* `mkdocs build` を実行すると `./site` ディレクトリに静的サイトが生成されます。
* `./mkdocs.yml` を編集してドキュメントを設定します。

## Lint の実行

ドキュメントのフォーマットチェックには markdownlint を使用してください。
[markdownlint の公式リポジトリ](https://github.com/DavidAnson/markdownlint)

Docker を使ってローカルで実行できます:

```bash
docker run -v $PWD:/markdown:ro 06kellyjac/markdownlint-cli .
```

Lint ルールは `.markdownlint` を参照してください。現在はかなり緩い設定になっています。
