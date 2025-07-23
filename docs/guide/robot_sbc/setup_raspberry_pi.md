# Raspberry Pi を動作させる

![donkey](/assets/logos/rpi_logo.png)

Donkey Carは現在インストール方法が以前と異なっています。以下をよくお読みください。
バージョンに応じてインストール方法が変わります。最新のバージョン5.1では
64ビット版 Raspberry Pi OS Bookworm が必要です。5.0を使用している場合のみ
64ビット版Bullseyeをインストールしてください。

Donkey Carの4.Xなど古いバージョンを使用している場合は
古い Raspberry Pi OS(Raspbian) のBuster版を使用する必要があります。
その手順は[こちら](#step-1-flash-operating-system)をご覧ください。

Tubデータや車のテンプレートなどは両方のバージョン間で互換性があります。
keras形式の`.h5`モデルも同様ですが、Tensorflow Lite形式の`.tflite`モデルは
互換性がないため再生成する必要があります。

一般的には、4GBのRAMを搭載したRPi 4または5を推奨します。
また、U3速度の128GB microSDカードの使用を推奨します。例として
[このSanDisk SDカード](https://www.amazon.com/SanDisk-128GB-Extreme-microSD-Adapter/dp/B07FCMKK5X/ref=sr_1_4?crid=1J19V1ZZ4EVQ5&keywords=SanDisk+128GB+Extreme+microSDXC+UHS-I&qid=1676908353&sprefix=sandisk+128gb+extreme+microsdxc+uhs-i%2Caps%2C121&sr=8-4) などがあります。


## 最新のDonkey Car (>= 5.1) をRaspberry Pi OS Bookwormでインストールする

ここでは64ビット版のRaspberry Pi OS Bookwormを使用します。

* [ステップ1: OSのインストール](#step-1-install-raspberry-pi-os)
* [ステップ2: 更新とアップグレード](#step-2-update-and-upgrade)
* [ステップ3: Raspi-config](#step-3-raspi-config)
* [ステップ4: 仮想環境のセットアップ](#step-4-setup-virtual-environment)
* [ステップ5: DonkeycarのPythonコードをインストール](#step-5-install-donkeycar-python-code)
* その後で [Donkeycarアプリケーションを作成](/guide/create_application/)


### ステップ1: Raspberry Pi OSをインストール

Raspberry Pi OSはグラフィカルインストーラー _Raspberry Pi Imager_ を用いてインストールできます。
インストーラーは[こちら](https://www.raspberrypi.com/software/)からダウンロードしてください。
使用するSDカードをパソコンのカードリーダーに挿入した状態で、アプリケーションをダウンロードして起動します。
RaspberryPi用に使用するSDカードをコンピュータのSDカードリーダーに挿入してください。

最初に使用するデバイス（Raspberry Pi 5 または Raspberry Pi 4）を選択します。

次に「Operating System」をクリックして「Raspberry Pi OS (64 bit)」を選択します。

続いて「Storage」をクリックしてSDカードを選びます。

「NEXT」を押すと「OS customization settings」を適用するオプションが表示されるので、「Edit Settings」を選択します。

ここでユーザー名やパスワード、WiFiの詳細を入力します。ホスト名（ここでは「donkeycar」とします）やパスワード、WiFi、地域などを設定します。

次のような画面になります:
![imager_advanced_dialog](/assets/imager.png)

他の項目はデフォルトのままで構いません。終了したら「Save」をクリックするとOSカスタマイズのダイアログに戻ります。「Yes」をクリックするとSDカードへOSが書き込まれます。

完了したらSDカードをPiに挿入し、電源を入れます。初回起動には1分ほどかかりますが、緑のランプの点滅が止まったら起動が完了です。

ネットワークを介してホスト名 "donkeycar.local" （もしくは設定した名前）でPiにSSH接続できるはずです。例として ```ssh mydonkey@donkeycar.local``` のように接続します。


### ステップ2: 更新とアップグレード

```bash
sudo apt-get update --allow-releaseinfo-change
sudo apt-get upgrade
```

### ステップ3: Raspi-config

Raspi-configユーティリティを起動します:

```bash
sudo raspi-config
```

* `Interfacing Options` - `I2C` を有効にします
* `Advanced Options` - `Expand Filesystem` を選んでSDカード全体を使用できるようにします
* レガシーカメラを有効にしないでください（デフォルトで無効なので変更しないでください）

`<Finish>` を選択してEnterキーを押します。

> 注意: これらの設定変更後は再起動が必要です。`yes` を選ぶと再起動します。

VNCでデスクトップに接続している場合は、「Raspberry -> Preferences -> Raspberry Pi Configuration」で同じ設定を行うこともできます。

> 注意: ラズパイOSのヘッドレス版をインストールする場合は、[こちら](https://www.raspberrypi.com/documentation/computers/configuration.html#setting-up-a-headless-raspberry-pi)の手順に従ってください。その後 `sudo apt -y install pip git` を実行する必要があります。


### ステップ4: 仮想環境のセットアップ

自宅ディレクトリで次のコマンドを実行して仮想環境を作成します:
```bash
python3 -m venv env --system-site-packages
echo "source ~/env/bin/activate" >> ~/.bashrc
source ~/.bashrc
```

必要なライブラリをインストールします
```bash
sudo apt install libcap-dev libhdf5-dev libhdf5-serial-dev
```

### ステップ5: Donkeycar Pythonコードのインストール

#### ユーザーインストール（推奨）
以下を実行します:
```bash
pip install donkeycar[pi]
```
完了まで5〜10分かかります。少々お待ちください。

#### 開発者向けインストール（別のフォークやブランチ、タグを使用したい場合のみ）

プロジェクトの基点となるディレクトリを作成して移動し、GitHubから `donkeycar` をダウンロードしてインストールします。`donkey` 環境が有効になっていることを確認してください。

```bash
mkdir projects
cd projects
git clone https://github.com/autorope/donkeycar
cd donkeycar
git checkout main
pip install -e .[pi]
```

### 追加の手順

- **カメラが正常に動作することを確認してください**。[カメラ接続](https://www.raspberrypi.com/documentation/accessories/camera.html#connect-the-camera)はよく問題になります。特に[新しいDonkeycarの組み立て](https://docs.donkeycar.com/guide/build_hardware/#step-6-attach-camera)直後やカメラの交換、クラッシュ後によく発生します。こうした状況やその他の理由でDonkeycarのソフトウェア使用時にカメラエラーが発生した場合、[Discord](https://discord.gg/PN6kFeA)で質問する前にカメラが正常に動作しているか確認してください。Raspberry Pi OSには[カメラソフトウェア](https://www.raspberrypi.com/documentation/computers/camera_software.html)が含まれており、写真撮影やビデオストリームを行えます。キーボード、マウス、モニタをRaspberry Piに接続している場合は、[`rpicam-hello`](https://www.raspberrypi.com/documentation/computers/camera_software.html#rpicam-hello)ユーティリティを実行してカメラの映像を確認できます。SSH接続している場合は、[`rpicam-jpeg`](https://www.raspberrypi.com/documentation/computers/camera_software.html#rpicam-jpeg)を使ってJPEG画像を保存し、そのファイルをホストPCにコピーして確認できます（エラーなく撮影できればカメラは問題ないと考えられます）。

- **TensorFlowが動作することを確認してください**。以下のコマンドを実行してTensorFlowのインストールを確認できます（donkey環境が有効になっていることが前提です。ステップ4を完了していれば有効なはずです）。エラーが出ないことを確認し、表示されるバージョン番号を書き留めておいてください。後の[深層学習自動運転の学習手順](https://docs.donkeycar.com/guide/train_autopilot/)で問題が発生した場合、このバージョン情報がDiscordで助けを求める際に非常に重要になります。

```bash
python -c "import tensorflow; print(tensorflow.__version__)"
```
* [ステップ12: (任意) OpenCV のインストール](#step-12-optional-install-opencv)
* [ステップ13: (任意) モバイルアプリのインストール](#step-13-optional-install-mobile-app)
* その後で [Donkeycar アプリケーションの作成](/guide/create_application/)

### ステップ1: オペレーティングシステムの書き込み

> 注意: モバイルアプリを使用する予定がある場合は、あらかじめ構築されたイメージの利用を検討してください。
> 詳細は [モバイルアプリのユーザーガイド](../deep_learning/mobile_app.md) を参照してください。

microSDカードにオペレーティングシステムのイメージを書き込みます。

> 注意: 最新のRaspbian(bullseye)はPythonカメラバインディングと互換性がありません。カメラシステムが変更されたためです。`main` ブランチから最新バージョンをインストールするには以下の手順に従ってください。

1. [Raspian Legacy (Buster)](https://downloads.raspberrypi.org/raspios_oldstable_lite_armhf/images/raspios_oldstable_lite_armhf-2021-12-02/2021-12-02-raspios-buster-armhf-lite.zip) をダウンロードします。
2. OS ごとの手順は [こちら](https://www.raspberrypi.org/documentation/installation/installing-images/) を参照してください。
3. microSDカードは挿したままにして、以下のファイルを編集／作成します:


### ステップ2: 初回起動用の WiFi 設定

初回起動時にWiFiへ接続するための特別なファイルを作成できます。
詳しくは[こちら](https://raspberrypi.stackexchange.com/questions/10251/prepare-sd-card-for-wifi-on-headless-pi)を参照してくださいが、以下の手順を説明します。

Windows ではメモリカードにイメージを書き込み、カードを挿したままにすると 2 つのドライブ（実際には 2 つのパーティション）が表示されます。
そのうちの一つは __boot__ とラベル付けされています。Mac や Linux でも __boot__ パーティションにアクセス可能です。
このパーティションは一般的な FAT 形式でフォーマットされており、ここでファイルを編集して初回起動時に WiFi に接続できるようにします。

> 注意: __boot__ がすぐに表示されない場合は、カードリーダーを抜き差ししてください。

* テキストエディタを起動します
    * Linux では `gedit`
    * Windows では Notepad++
    * Mac では `vi /Volumes/boot/wpa_supplicant.conf` (`boot` はSDカード名)
* 使用可能な `country` コードは[ここ](https://www.thinkpenguin.com/gnu-linux/country-code-list)で確認できます
* 以下の内容を貼り付け、WiFiに合わせて編集します:

```text
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="<your network name>"
    psk="<your password>"
}

```

`country` は利用可能なWiFiチャネルを指定します。お住まいの地域とハードウェアに合わせて正しく設定してください。

`<your network name>` にはネットワークのIDを入力します。引用符は残します。ネットワーク名にアポストロフィ（例: "Joe's iPhone"）が含まれていると問題になることがあります。
`<your password>` にはパスワードを入力します。引用符は付けたままにしてください。パスワードを平文で置いておくのが気になる場合は、起動後に
[こちら](https://unix.stackexchange.com/questions/278946/hiding-passwords-in-wpa-supplicant-conf-with-wpa-eap-and-mschap-v2)の方法で内容を変更できます。

* このファイルを __boot__ パーティションのルートに `wpa_supplicant.conf` という名前で保存します。
  初回起動時にこのファイルは `/etc/wpa_supplicant/wpa_supplicant.conf` に移動され、後から編集できます。Windows の Notepad を使用している場合は拡張子 `.txt` が付かないよう注意してください。


### ステップ3: Pi のホスト名設定

> 注意: この手順は Linux ホスト PC でのみ可能です。そうでない場合は Pi にログイン後 `raspi-config` で設定できます。

Pi をネットワーク上で見つけやすくするため、ホスト名を設定します。ネットワーク上に自分の Pi だけがあるなら、次のようにして見つけられます。

```bash
ping raspberrypi.local
```

起動したらこのコマンドを実行します。ネットワークに他の Pi が多数ある場合はうまくいかないことがあります。
Linux マシンを使用している、または UUID パーティションを編集できる場合は、今のうちに `/etc/hostname` と `/etc/hosts` を編集しておくと、起動後に Pi を見つけやすくなります。
これらのファイルで `raspberrypi` を任意の名前に変更します。
小文字のみを使用し、特殊文字やハイフンは避け、アンダースコア `_` は使用できます。
複数の Pi デバイスが同じネットワークにある場合は `pi-<MAC_ADDRESS>` のようにするのがよいでしょう。

```bash
sudo vi /media/userID/UUID/etc/hostname
sudo vi /media/userID/UUID/etc/hosts
```

### ステップ4: 起動時に SSH を有効化

__boot__ パーティションのルートに __ssh__ という名前のファイルを置きます。Mac や Linux では `touch` コマンドで作成できます。例: `touch /Volumes/boot/ssh` (`boot` は SD カード名)。

これで SD カードの準備は完了です。書き込みが終了するまで待ってからカードを安全に取り外し、Pi の電源がオフになっていることを確認してカードを挿入し、電源を入れます。

### ステップ5: Pi へ接続する

上記の手順で WiFi 設定を行った場合、Pi は既にネットワークに接続されているはずです。次に SSH で接続するための IP アドレスを調べます。

Ubuntu では `findcar` コマンドを使うのが最も簡単です。
`ping raspberrypi.local` を試すこともできます。ホスト名を変更している場合は次のようにします:

```bash
ping <your hostname>.local
```

この方法は Windows では失敗します。Windows ユーザーは完全な IP アドレスが必要です（cygwin を使用していない場合）。

ネットワーク上で Pi を見つけられないときは、Pi に HDMI モニターと USB キーボードを接続して起動し、次の情報でログインします:

* Username: `pi`
* Password: `raspberry`

その後、以下のコマンドを試します:

```bash
ifconfig wlan0
```

または、Pi に割り当てられているすべての IP アドレスを確認するには次を実行します:

```bash
ip -br a
```

ここに IPv4 アドレス（数字4組）が表示されていれば、そのアドレスで SSH 接続を試せます。表示されない場合は WiFi 設定が間違っている可能性があります。以下で修正してみてください。

```bash
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

HDMI モニターとキーボードがない場合は、CAT5 ケーブルで Pi を DHCP ルーターに接続します。
そのルーターが PC と同じネットワークにあるなら次のコマンドを試せます:

```bash
ping raspberrypi.local
```

うまくいけば、これらの方法のいずれかで Pi に SSH 接続できるはずです。Mac や Linux ではターミナルを開きます。
Windows の場合は [Putty](http://www.putty.org/) や[他の選択肢](https://www.htpcbeginner.com/best-ssh-clients-windows-putty-alternatives/2/)を使用するか、Windows 10 ならコマンドプロンプトから ssh が利用できることもあります。

コマンドプロンプトがある場合は次を試してみてください:

```bash
ssh pi@raspberrypi.local
```

また、

```bash
ssh pi@<your pi ip address>
```

と入力しても接続できます。Putty を使うことも可能です。

* ユーザー名: `pi`
* パスワード: `raspberry`
* ホスト名: `<your pi IP address>`


### ステップ6: 更新とアップグレード

```bash
sudo apt-get update --allow-releaseinfo-change
sudo apt-get upgrade
```

### ステップ7: Raspi-config

```bash
sudo raspi-config
```

* pi のデフォルトパスワードを変更
* ホスト名を変更
* `Interfacing Options` - `I2C` を有効化
* `Interfacing Options` - `Camera` を有効化
* `Advanced Options` - `Expand Filesystem` を選択して SD カード全体を使用可能にする

`<Finish>` を選択して Enter を押します。

> 注意: これらの設定を変更した後は再起動が必要です。`yes` を選ぶと再起動します。

### ステップ8: 依存関係をインストール

```bash
sudo apt-get install build-essential python3 python3-dev python3-pip python3-virtualenv python3-numpy python3-picamera python3-pandas python3-rpi.gpio i2c-tools avahi-utils joystick libopenjp2-7-dev libtiff5-dev gfortran libatlas-base-dev libopenblas-dev libhdf5-serial-dev libgeos-dev git ntp
```

### ステップ9: (任意) OpenCV 依存関係のインストール

最小構成でインストールする場合はこれらを省略できますが、OpenCV があると便利です。

```bash
sudo apt-get install libilmbase-dev libopenexr-dev libgstreamer1.0-dev libjasper-dev libwebp-dev libatlas-base-dev libavcodec-dev libavformat-dev libswscale-dev
```

### ステップ10: 仮想環境の設定

この作業は一度だけ行えば十分です:

```bash
python3 -m virtualenv -p python3 env --system-site-packages
echo "source ~/env/bin/activate" >> ~/.bashrc
source ~/.bashrc
```

`.bashrc` をこのように変更しておくと、ログインするたびにこの環境が自動で有効になります。システムの Python に戻るには `deactivate` と入力します。

### ステップ11: Donkeycar の Python コードをインストール

* プロジェクトの作業ディレクトリとなるフォルダを作成して移動します。

```bash
mkdir projects
cd projects
```

* 最新の安定版リリース（4.4 系）を取得します

```bash
git clone https://github.com/autorope/donkeycar
cd donkeycar
git fetch --all --tags -f
git checkout 4.5.1
pip install -e .[pi]
pip install https://github.com/lhelontra/tensorflow-on-arm/releases/download/v2.2.0/tensorflow-2.2.0-cp37-none-linux_armv7l.whl
```

TensorFlow のインストールを確認するには次を実行します

```bash
python -c "import tensorflow; print(tensorflow.__version__)"
```

### ステップ12: (任意) OpenCV をインストール

先に OpenCV の依存関係をインストールした場合は、次のコマンドで Python 用 OpenCV バインディングをインストールできます:

```bash
sudo apt install python3-opencv
```

うまくいかないときは pip を試します:

```bash
pip install opencv-python
```

その後、インポートできるかテストします。

``` bash
python -c "import cv2"
```

エラーが出なければ OpenCV のインストール完了です。

### ステップ13: (任意) モバイルアプリのインストール

iPhone と Android 向けにモバイルアプリが提供されています。SD カードイメージをダウンロードするか、手動でインストールできます。手動インストールの方法は[こちら](/guide/deep_learning/mobile_app/#optional-manual-installation)を参照してください。

> **注意** サーバーコンポーネントは **RaspberryPi 4B** のみをサポートしています。


## 次に、[Donkeycar アプリケーションを作成](/guide/create_application/) してください。
