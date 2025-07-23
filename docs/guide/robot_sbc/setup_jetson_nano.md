# Jetson Nano/Xavier NX を使えるようにする

![donkey](/assets/logos/nvidia_logo.png)

Donkey Car のバージョンによってソフトウェアのインストール方法が異なります。
Donkey Car <= 4.5.X では Jetpack 4.5.X を使用します。
これは Tensorflow 2.3.1 が同梱されており、Python はシステムの Python 3.6 を基にした
virtualenv を利用します。古い Jetson Nano で動作するのはこのバージョンだけです。

`main` ブランチでは Tensorflow 2.9 と Python 3.8 もしくは 3.9 を使用しています。
これはより新しい Jetpack 上で動作し、利用するには Jetson Xavier や Orin といった
新しい Jetson が必要で、Jetpack 5.0.2 を使います。システム Python から切り離すため、
arm アーキテクチャで動作する mamba ベースの Miniconda である Miniforge を利用します。

Donkey Car <= 4.5.X の場合は次のセクションへ進んでください。`main` ブランチの最新
バージョンを使用する場合は[こちらのセクション](#installation-for-donkey-car-main)
を参照してください。

ソフトウェアを問題なく動かすためには Jetson Nano 4GB 版または Jetson Xavier の使用を
推奨します。また U3 スピードの 128GB microSD カードを用意してください。
例としては[この SanDisk SD カード](https://www.amazon.com/SanDisk-128GB-Extreme-microSD-Adapter/dp/B07FCMKK5X/ref=sr_1_4?crid=1J19V1ZZ4EVQ5&keywords=SanDisk+128GB+Extreme+microSDXC+UHS-I&qid=1676908353&sprefix=sandisk+128gb+extreme+microsdxc+uhs-i%2Caps%2C121&sr=8-4)などがあります。


以下のバージョンがサポートされています:

| Jetson      | Jetpack | Python | Donkey   | Tensorflow |
|-------------|---------|--------|----------|------------|
| Nano        | 4.5.1   | 3.6    | <= 4.5.X | 2.3.1      |    
| Xavier/Orin | 5.0.2   | 3.8    | >= 5.X   | 2.9        | 


次に [Donkeycar アプリケーションを作成](/guide/create_application/) してください。


## Donkey Car <= 4.5.X のインストール

> 注: この手順は現在 DC 4.3.6 と DC 4.4.0 のみで動作します。4.5.X への対応も進行中です。


* [Step 1a: OS の書き込み](#step-1a-flash-operating-system)
* [Step 2a: シリアルポートを開放する](#step-2a-free-up-the-serial-port-optional-only-needed-if-youre-using-the-robohat-mm1)
* [Step 3a: 依存パッケージのインストール](#step-3a-install-system-wide-dependencies)
* [Step 4a: Python 環境の設定](#step-4a-setup-python-environment)
* [Step 5a: (任意) USB カメラ用に PyGame をインストール](#step-5a-optional-install-pygame-for-usb-camera)


### Step 1a: OS を書き込む


以下の手順は Jetpack 4.5.1 で動作します。

* 4GB版 Jetson Nano をお持ちの場合は [jetson-nano-jp451-sd-card-image.zip](https://developer.nvidia.com/embedded/l4t/r32_release_v5.1/r32_release_v5.1/jeston_nano/jetson-nano-jp451-sd-card-image.zip) をダウンロードしてください。
* 2GB版 Jetson Nano の場合は [jetson-nano-2gb-jp451-sd-card-image.zip](https://developer.nvidia.com/embedded/l4t/r32_release_v5.1/r32_release_v5.1/jeston_nano_2gb/jetson-nano-2gb-jp451-sd-card-image.zip) をダウンロードしてください。

これにより公式の Nvidia 製 Tensorflow 2.3.1 がインストールされます。ホスト PC を使う場合は同じバージョンの Tensorflow を使用してください。異なるバージョンで学習したネットワークを使用すると、オートパイロットとして利用する際にエラーが発生することがあります。

公式の [Nvidia Jetson Nano Getting Started Guide](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit#prepare)
または [Nvidia Xavier NX Getting Started Guide](https://developer.nvidia.com/embedded/learn/get-started-jetson-xavier-nx-devkit) を参照し、
__Prepare for Setup__、__Writing Image to the microSD Card__、__Setup and First Boot__ の各手順を実行してから戻ってきてください。

セットアップが完了したら車体へ SSH 接続します。Ubuntu や Mac ではターミナルを、Windows では [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) を使用してください。

Libre Office を削除します:

```bash
sudo apt-get remove --purge libreoffice*
sudo apt-get clean
sudo apt-get autoremove
```

続いて 8GB のスワップファイルを追加します:

```bash
git clone https://github.com/JetsonHacksNano/installSwapfile
cd installSwapfile
./installSwapfile.sh
sudo reboot now 
```

### Step 2a: シリアルポートを開放する (任意、RoboHat MM1 を使用する場合のみ)

```bash
sudo usermod -aG dialout <your username>
sudo systemctl disable nvgetty
```

### Step 3a: システム全体の依存パッケージをインストール

まず `apt-get` で以下のパッケージをインストールします。

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install -y libhdf5-serial-dev hdf5-tools libhdf5-dev zlib1g-dev zip libjpeg8-dev liblapack-dev libblas-dev gfortran
sudo apt-get install -y python3-dev python3-pip
sudo apt-get install -y libxslt1-dev libxml2-dev libffi-dev libcurl4-openssl-dev libssl-dev libpng-dev libopenblas-dev
sudo apt-get install -y git nano
sudo apt-get install -y openmpi-doc openmpi-bin libopenmpi-dev libopenblas-dev
```

### Step 4a: Python 環境の設定

#### 仮想環境の準備

```bash
pip3 install virtualenv
python3 -m virtualenv -p python3 env --system-site-packages
echo "source ~/env/bin/activate" >> ~/.bashrc
source ~/.bashrc
```

#### Python 依存パッケージのインストール

次に `pip` でパッケージをインストールします:

```bash
pip3 install -U pip testresources setuptools
pip3 install -U futures==3.1.1 protobuf==3.12.2 pybind11==2.5.0
pip3 install -U cython==0.29.21 pyserial
pip3 install -U future==0.18.2 mock==4.0.2 h5py==2.10.0 keras_preprocessing==1.1.2 keras_applications==1.0.8 gast==0.3.3
pip3 install -U absl-py==0.9.0 py-cpuinfo==7.0.0 psutil==5.7.2 portpicker==1.3.1 six requests==2.24.0 astor==0.8.1 termcolor==1.1.0 wrapt==1.12.1 google-pasta==0.2.0
pip3 install -U gdown

# This will install tensorflow as a system package
pip3 install --pre --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v45 tensorflow==2.5
```

#### Donkeycar の Python コードをインストール

プロジェクトのルートにしたいディレクトリへ移動します。前述の `projects` ディレクトリを使い、最新の 4.5.X リリースを取得して仮想環境にインストールします。

```bash
mkdir projects
cd ~/projects
git clone https://github.com/autorope/donkeycar
cd donkeycar
git fetch --all --tags -f
git checkout 4.5.1
pip install -e .[nano]

```

### Step 5a: (任意) USB カメラ用に PyGame をインストール

USB カメラを使用する予定がある場合は pygame をセットアップします:

```bash
sudo apt-get install python-dev libsdl1.2-dev libsdl-image1.2-dev libsdl-mixer1.2-dev libsdl-ttf2.0-dev libsdl1.2-dev libsmpeg-dev python-numpy subversion libportmidi-dev ffmpeg libswscale-dev libavformat-dev libavcodec-dev libfreetype6-dev
pip install pygame
```

後で `myconfig.py` に `CAMERA_TYPE="WEBCAM"` を追加してください。



## Donkey Car >= 5.X のインストール

`main` ブランチの最新コードや 5.X 以上のリリース向けの手順です。利用可能な 2 種類の OS でインストール方法が異なります。Jetson では Jetpack 5.0.2 をインストールする必要があります。


### Jetson Xavier (またはそれ以降の Jetson ボード) へのインストール

* [Step 1b: OS の書き込み](#step-1c-flash-operating-system)
* [Step 2b: シリアルポートを開放する](#step-2c-free-up-the-serial-port-optional-only-needed-if-youre-using-the-robohat-mm1)
* [Step 3b: Python 環境を設定](#step-3c-setup-python-environment)
* [Step 4b: (任意) USB カメラ用に PyGame をインストール](#step-4c-optional-install-pygame-for-usb-camera)


#### Step 1b: OS を書き込む

これらの手順は Jetpack 5.0.2 用です。

[jetson-nx-developer-kit-sd-card-image.zip](https://developer.nvidia.com/jetson-nx-developer-kit-sd-card-image) から Jetpack イメージをインストールしてください。


公式の [Nvidia Xavier NX Getting Started Guide](https://developer.nvidia.com/embedded/learn/get-started-jetson-xavier-nx-devkit) を参照し、
__Prepare for Setup__、__Writing Image to the microSD Card__、__Setup and First Boot__ の手順を実行したらここへ戻ってきてください。

セットアップが完了したら車体へ SSH 接続します。Ubuntu または Mac ではターミナルを、Windows では [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) を使用してください。

Libre Office を削除します:

```bash
sudo apt-get remove --purge libreoffice*
sudo apt-get clean
sudo apt-get autoremove
```

続いて 8GB のスワップファイルを追加します。SSD から起動する予定の場合は、SSD で起動した後にこの作業を行ってください。

```bash
git clone https://github.com/JetsonHacksNano/installSwapfile
cd installSwapfile
./installSwapfile.sh -s 8
reboot 
```

#### Step 2b: シリアルポートを開放する (任意、RoboHat MM1 を使用する場合のみ)

```bash
sudo usermod -aG dialout <your username>
sudo systemctl disable nvgetty
```

#### Step 3b: Python 環境を設定

* Step 3b-1: システム Python 環境に tensorflow をインストール

JP5.1.2 向け tensorflow とその依存関係のインストールは、NVIDIA の[こちらの手順](https://docs.nvidia.com/deeplearning/frameworks/install-tf-jetson-platform/index.html#overview__section_z4r_vjd_v2c)に従ってください。

* Step 3b-2: venv を設定

```bash
python3 -m venv env --system-site-packages
echo "source ~/env/bin/activate" >> ~/.bashrc
source ~/.bashrc
```

* Step 3b-3: Donkey Car をインストール

インストール方法は 2 種類あります。通常はユーザーインストールを行い Step 2a を実行します。ソースコードをデバッグや編集したい場合は、より高度な開発者インストールを行いますが、両方を同時には実施できません。

> _**注意**_: Step 3b-4 と 3b-5 のどちらか一方のみを実行してください。

* Step 3b-4: ユーザーインストール

`env` をすでに有効にした状態で次を実行します:

```bash
pip install donkeycar[nano]
pip install -U albumentations --no-binary qudida,albumentations
pip uninstall opencv-python-headless
pip uninstall scikit-learn
git clone https://github.com/scikit-learn/scikit-learn.git
cd scikit-learn/
python setup.py install
sudo chmod 666 /dev/gpiochip*
```
これで最新リリースがインストールされます。

* Step 3b-5: 開発者インストール

3b-4 のユーザーインストールを行っていない場合のみ実施してください。

インストールするブランチやタグを選択し、GitHub からソースコードを取得してコードの編集やデバッグを行うことができます。`main` ブランチの最新版を入手する場合はこちらを行います。

```bash
mkdir projects
cd projects
git clone https://github.com/autorope/donkeycar
cd donkeycar
git checkout main
pip install -e .[nano]
pip install -U albumentations --no-binary qudida,albumentations
pip uninstall opencv-python-headless
pip uninstall scikit-learn
git clone https://github.com/scikit-learn/scikit-learn.git
cd scikit-learn/
python setup.py install
sudo chmod 666 /dev/gpiochip*
```

* Step 3b-6: TensorFlow と OpenCV のインストールを確認

Python を実行し、TensorFlow がバージョン 2.9、TensorRT がバージョン 8.2.1 であることを確認します。
TensorRT の共有ライブラリを正しく読み込むため、次のように環境変数 `LD_PRELOAD` を設定します:

```bash
export LD_PRELOAD=/usr/lib/aarch64-linux-gnu/libnvinfer.so.8:/usr/lib/aarch64-linux-gnu/libgomp.so.1
```

この設定は donkeycar や tensorflow を実行するたびに行うか、`.bashrc` に上記の行を追加してください。

```bash
python
>>> import tensorflow as tf
>>> tf.__version__
>>> from tensorflow.python.compiler.tensorrt import trt_convert as trt
>>> trt._check_trt_version_compatibility()
>>> import cv2
>>> print(cv2.getBuildInformation())
```

#### Step 4b: (任意) USB カメラ用に PyGame をインストール

USB カメラを使用する予定がある場合は pygame をインストールします:

```bash
pip install pygame
```
後で `myconfig.py` に `CAMERA_TYPE="WEBCAM"` を追加してください。


## (任意) CSIC カメラのピンク色問題を修正する

上記のいずれのインストール (JP 4.6.X または 5.0.X) にも適用できます。CSIC カメラを使用している場合、画像がピンク色になることがあります。[こちら](https://jonathantse.medium.com/fix-pink-tint-on-jetson-nano-wide-angle-camera-a8ce5fbd797f) に記載されている方法でこの問題を解決できます。

```bash
wget https://www.dropbox.com/s/u80hr1o8n9hqeaj/camera_overrides.isp
sudo cp camera_overrides.isp /var/nvidia/nvcam/settings/
sudo chmod 664 /var/nvidia/nvcam/settings/camera_overrides.isp
sudo chown root:root /var/nvidia/nvcam/settings/camera_overrides.isp
```


----

## 次に、[Donkeycar アプリケーションを作成](/guide/create_application)します。
