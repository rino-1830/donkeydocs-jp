# LinuxでDonkeycarをインストールする


> 注: Ubuntu 20.04 LTS、22.04 LTSでテスト済み

* ターミナルアプリケーションを開きます。

* 64ビット版のPython 3.11向けminicondaをインストールします。

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-py311_24.4.0-0-Linux-x86_64.sh
bash ./Miniconda3-py311_24.4.0-0-Linux-x86_64.sh
```

`donkey` というconda環境を次のように設定します:

```bash
conda create -n donkey python=3.11
conda activate donkey
```

> **補足**: ターミナルに `conda: command not found` と表示される場合、以下のコマンドを実行するなどしてminicondaのインストール先のパスを通す必要があります。

```bash
echo "export PATH=~/miniconda3/bin:\$PATH" >> ~/.bashrc
echo "source ~/miniconda3/etc/profile.d/conda.sh" >> ~/.bashrc
source ~/.bashrc
```

現在、2通りのインストール方法があります。ほとんどの場合、
ユーザーインストールを行うことになるでしょう。次に
[_User install_](#user-install) を実行します。もし
ソースコードのデバッグや編集をしたい場合は、より高度な
[_Developer install_](#developer-install) を行う必要があります。ただしどちらか一方だけです。

> **注意**: User install か Developer install のどちらか一方だけを実行してください。

### ユーザーインストール

`donkey` 環境をすでに有効化しているので、次のコマンドを入力します:

```bash
pip install donkeycar[pc]
```
これで最新リリースがインストールされます。ZSH を使用している場合は、
`[` と `]` をエスケープする必要があります。つまり、

```bash
pip install donkeycar\[pc\]
```


### 開発者インストール

こちらでは、インストールしたいブランチやタグを選択でき、
GitHubからソースコードをダウンロードすることで、コードの編集やデバッグができます。

プロジェクトのルートとして使いたいディレクトリを作成し、
そこに移動して `donkeycar` をダウンロード・インストールします。
ソースはGitHubから取得します。

```bash
mkdir projects
cd projects
git clone https://github.com/autorope/donkeycar
cd donkeycar
git checkout main
pip install -e .[pc]
```

注: ZSH を使っている場合、このままでは実行できないので、
`pip install -e .[pc]` を実行する際にはブラケットをエスケープして
`pip install -e .\[pc\]` と入力してください。


* すでにインストール済みの場合は、Condaを更新して古いdonkey環境を削除します。

```bash
conda update -n base -c defaults conda
conda env remove -n donkey
```

新しいTensorflowはすでにGPUサポート付きでビルドされています。もし
Nvidia GPUをお持ちなら、Nvidiaのページに従ってCuda 12をインストールしてください
[こちら](https://developer.nvidia.com/cuda-toolkit-archive)

* 省略可能: Coral Edge TPUコンパイラのインストール

Google Coral Edge TPU をお持ちなら、モデルをコンパイルするために
`edgetpu_compiler` 実行ファイルをインストールする必要があります。[こちらの
手順](https://coral.withgoogle.com/docs/edgetpu/compiler/) に従ってください。

* 省略可能: PyTorch をGPUで使えるよう設定する ― Nvidia製GPU専用

NVidiaカードを使用している場合は、最新のドライバに更新し、
[Cuda SDKをインストール](https://www.tensorflow.org/install/gpu#windows_setup) してください。
また、いくつかの箇所でGPUを使用するようコードを変更する必要があるため、
開発者インストールが必要です。

```bash
conda install cudatoolkit=11 -c pytorch
```

`<CUDA Version>` はお使いのCUDAバージョンに置き換えてください。10.0以上であれば
動作します。CUDAのバージョンは
`nvcc --version` または `nvidia-smi` で確認できます。(これらのコマンドが使えない場合は、
インストールされていないということなので、そのエラーの指示に従って
インストールしてください)。この2つのコマンドで表示されるバージョンが一致しない場合は、
`nvidia-smi` の表示を採用してください。

* 作業用のローカルディレクトリを作成します:

```bash
donkey createcar --path ~/mycar
```

> 注: Anaconda Prompt を閉じた後に再度開くときは、
> ```conda activate donkey``` を入力して、donkey 用のライブラリを再度有効に
> してください。

----

### 次に [Donkeycar へのソフトウェアのインストール](/guide/install_software/#step-2-install-software-on-donkeycar) を行います
