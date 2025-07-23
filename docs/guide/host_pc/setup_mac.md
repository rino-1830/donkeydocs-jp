## MacでDonkeycarをインストールする

![donkey](/assets/logos/apple_logo.jpg)

* [miniconda Python 3.11 64 bit](https://conda.io/miniconda.html) をインストールする

* Terminal を起動する

`donkey` conda 環境を以下のように設定する:

```bash
conda create -n donkey python=3.11
conda activate donkey
```

ここからは2種類のインストール方法が選択できます。多くの場合
ユーザーインストールを行うことになるでしょう。その場合は手順
[_ユーザーインストール_](#user-install) を実行します。もし
ソースコードをデバッグしたり編集したい場合は、より高度な
[_開発者インストール_](#developer-install) を行う必要があります。ただし、どちらか一方しか実行できません。

> _**注意**_: ユーザーインストールか開発者インストールのどちらか一方のみ実行してください!

### ユーザーインストール

すでに新しい `donkey` 環境を有効化しているので次を入力します:

```bash
pip install donkeycar[pc]
```
MacOS のデフォルトである ZSH を使用している場合は `[` と `]` をエスケープする必要があります
そのため次のように入力してください:

```bash
pip install donkeycar\[pc\]
```
Intel Mac を使用している場合、あるいは
ZSH を使用している場合の変更点は上記を参照して次を入力します:

```bash
pip install donkeycar[macos]
```
Apple Silicon を使用している場合はこちらを入力します。これにより最新リリースがインストールされます。

### 開発者インストール

ここではインストールしたいブランチやタグを選択できます。
GitHub からソースコードをダウンロードしてコードを編集・デバッグできます。
[git 64 bit](https://www.atlassian.com/git/tutorials/install-git) をインストールし、
プロジェクトのルートとしたいディレクトリへ移動します。
```bash
mkdir projects
cd projects
git clone https://github.com/autorope/donkeycar
cd donkeycar
git checkout main
pip install -e .[pc]
```

注意: ZSH を使用している場合、そのままでは `pip install -e .[pc]` を実行できません。
ブラケットをエスケープする必要があるため、次のように入力してください。
`pip install -e .\[pc\]`

### 追加の手順

* 初めてのインストールでない場合は Conda を更新し、古い donkey 環境を削除します

```bash
conda update -n base -c defaults conda
conda env remove -n donkey
```

* Tensorflow GPU

現在のところ、[mac 用の tensorflow](https://www.tensorflow.org/install#install-tensorflow) には NVidia GPU のサポートがありません。

* 作業用のローカルディレクトリを作成します:

```bash
donkey createcar --path ~/mycar
```

> 注意: Terminal を閉じた後に再度開いた場合は、
> 次のコマンドを入力してください
> `conda activate donkey`
> donkey 用の Python ライブラリへのパスを
> 再設定する必要があります

----

### 次は [Donkeycar にソフトウェアをインストールする](/guide/install_software/#step-2-install-software-on-donkeycar)
