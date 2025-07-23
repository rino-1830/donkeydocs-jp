# ドキュメントへの貢献

Donkeycar プロジェクトへの貢献ありがとうございます。ドキュメントはユーザーの成功にとって非常に重要ですので、皆さんの貢献に感謝します。正確さと完全さが重要です。多くのユーザーは初心者ですので、「当然知っているはず」と思い込まずに記述してください。


私たちは [mkdocs パッケージ](https://www.mkdocs.org/user-guide/) を使用して https://docs.donkeycar.com サイト用の HTML を生成しています。リポジトリ内のファイルは Markdown 形式で、mkdocs がそれらを HTML に「コンパイル」することでブラウザで表示できるようにします。変更は自身の fork で行い、プルリクエストを作成してメンテナーによりメインの donkeydocs リポジトリへマージしてもらいます。PR がマージされると変更は自動的にコンパイルされ、https://docs.donkeycar.com サイトへ反映されます。

 1. 自分の GitHub アカウントで [donkeydocs](https://github.com/autorope/donkeydocs) リポジトリを fork します。
 2. fork したリポジトリを自分のコンピューターにクローンし、変更可能なローカルコピーを取得します。
 3. fork のクローンで新しいブランチを作成します。このブランチで変更や追加を行います。メインリポジトリの issue に関連する場合は、ブランチ名を issue 番号から始めます。
 4. 変更や追加を行い、fork にコミットします。編集・作成した Markdown ファイルを HTML にコンパイルするために mkdocs というパッケージを使用しています。Markdown の書式の詳細は [mkdocs ドキュメント](https://www.mkdocs.org/user-guide/writing-your-docs/#writing-with-markdown) を参照してください。mkdocs をインストールすれば、保存するたびに変更を確認できるライブプレビューを生成できます。
    - Windows では git bash コンソールを開き、クローンした donkeydocs プロジェクトフォルダーのルートに移動します。
    - Python の仮想環境を作成してアクティブ化します。
    - アクティブ化した仮想環境に mkdocs パッケージをインストールします。
    - mkdocs サーバーを起動します。これにより、ブラウザでレンダリングされたドキュメントを表示できる URL が得られます。
    ```
    python3 -m venv env
    source env/bin/activate
    pip3 install mkdocs
    mkdocs serve
    ```
    - 次回以降の編集セッションでは、仮想環境を再度アクティブ化して mkdocs サーバーを起動するだけです（環境を作り直す必要はなく、再アクティブ化するだけで構いません）。

 5. ブランチでの変更や追加が完了したら、変更をコミットして fork したリポジトリに push します。さらに変更が必要になった場合は、変更してコミットし push する、を繰り返してください。
 6. 更新内容が正しいことを確認し、fork したリポジトリに push したら、プルリクエストを作成します。メインの donkeydocs リポジトリを fork しているので、そのプルリクエストはメインリポジトリに反映されます。便利な機能として、fork したリポジトリに push したあと GitHub のページに行くと緑色の `Compare & Pull Request` ボタンが表示されます。これを押すとプルリクエストを作成できます。
 7. これでメインの donkeydocs リポジトリにプルリクエストが作成されます。Discord のメンテナーチャンネルに PR を作成したことを知らせて、誰かにレビューしてもらうとよいでしょう。プルリクエストの過程で数回コメントのやりとりがあるはずです。変更依頼を受けた場合は、そのコメントに沿って修正を行い push してください。新しい変更はプルリクエストに反映されます。

このプロセスの詳細は https://docs.github.com/en/get-started/quickstart/contributing-to-projects に記載されています。
