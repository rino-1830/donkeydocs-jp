# Donkeyについて&reg;

![Release](https://img.shields.io/github/v/release/autorope/donkeycar)
[![All Contributors](https://img.shields.io/github/contributors/autorope/donkeycar)](#contributors-)
![License](https://img.shields.io/github/license/autorope/donkeycar)
![Discord](https://img.shields.io/discord/662098530411741184.svg?logo=discord&colorB=7289DA)
----

Donkey は Python で書かれた RC カー向けのオープンソース自律走行プラットフォームです。高速な実験と簡単な[コミュニティへの貢献](https://github.com/autorope/donkeycar/blob/main/CONTRIBUTING.md)を重視しており、ニューラルネットワークやコンピュータビジョン、GPS を利用したさまざまなオートパイロットをサポートします。高校や大学での学習・研究用途で活発に利用されており、[リッチなグラフィカルインターフェース](utility/ui/)や[シミュレータ](guide/deep_learning/simulator/)も提供しているため、ロボットを組み立てる前から自動運転を試すことができます。[オンラインコミュニティ](https://discord.gg/PN6kFeA)によるサポートも盛んです。

![donkeycar](assets/build_hardware/donkey2.png)

### Donkeycar を使うとできること
* ロボットを作り、自分で運転させることができます。
* [オートパイロット](guide/train_autopilot/)、ニューラルネットワーク、コンピュータビジョン、GPS を使った実験ができます。
* [DIY Robocars](http://diyrobocars.com) などの自動運転レース、さらには世界中の競合と戦える[オンラインシミュレータレース](guide/deep_learning/virtual_race_league/)に参加できます。
* 最先端技術を学びつつ楽しめる活発な[オンラインコミュニティ](https://discord.gg/PN6kFeA)に参加できます。

---------
### 完成済み Donkeycar を購入する:

すぐに使えるものが欲しい場合は、Waveshare PiRacer Pro を推奨します。RaspberryPi 4 付きで 262 ドル、Pi をお持ちなら 194 ドルとお得です。Donkeycar と相性が良く、[こちら](https://www.waveshare.com/product/robotics/mobile-robots/raspberry-pi-robots/piracer-pro-ai-kit.htm)で購入できます。

### 自分で Donkey を作る

RC カーと Pi、その他少しの部品を用意すれば、比較的簡単に自分の車を組み立てられます。部品代は 250～300 ドルほどで、組み立てには 2 時間程度です。主な手順は以下のとおりです。

1. [ハードウェアを組み立てる](guide/build_hardware)
2. [ソフトウェアをインストールする](guide/install_software)
3. [Donkey アプリを作成する](guide/create_application)
4. [車をキャリブレーションする](guide/calibrate)
5. [運転を開始する](guide/get_driving)
6. [オートパイロットを作成する](guide/train_autopilot)
7. [シミュレータで実験する](guide/deep_learning/simulator)

---------------
### 始める前に知っておきたいこと (TL;DR 特になし)
Donkeycar は自動運転の「Hello World」を目指して設計されており、シンプルながら柔軟で強力です。特別な予備知識は必要ありませんが、次のような知識があると役立ちます。

- **[Python](https://docs.python.org/3.11/) プログラミング**  
  Donkeycar を使うためにプログラミングは必須ではありません。車の設定を行うファイル `myconfig.py` は Python ファイルで、変更したい箇所のコメントを外して編集するだけです。Python の[コメント](https://www.w3schools.com/python/python_comments.asp)や[インデント](https://www.w3schools.com/python/python_syntax.asp)を理解していれば、よくあるミスを避けられます。
- **Raspberry Pi**  
  Donkeycar のオンボードコンピューターとして推奨されます。設定済みの Raspberry Pi を扱った経験があると便利ですが必須ではありません。Donkeycar ドキュメントでは RaspberryPi OS 上へのソフトウェアのインストール方法を説明していますが、[Raspberry Pi Imager](https://www.raspberrypi.com/software/) を使った OS のインストールや [raspi-config](https://www.raspberrypi.com/documentation/computers/configuration.html) での設定方法は、充実した公式ドキュメントに任せています。まず Raspberry Pi のドキュメントに従ってセットアップし、ウェブサイト閲覧や [最初の屋外レース](https://youtu.be/tjWmrCIKgnE) の動画視聴、テキストエディターでのファイル作成、ターミナルでのファイルシステム操作などに慣れておくと、Raspberry Pi と Donkeycar を同時に学ぶ負担が減ります。
- **Linux の[コマンドラインシェル](https://magpi.raspberrypi.com/articles/terminal-help)**  
  コマンドラインシェルはターミナルとも呼ばれ、Donkeycar ソフトウェアのインストールや起動に使います。ファイルシステムの操作、ファイルやディレクトリの一覧・コピー・削除方法を知っておくと便利です。また、[リモートアクセス](https://www.raspberrypi.com/documentation/computers/remote-access.html)を利用する場合、Wi-Fi の有効化や、ホストコンピューターから [SSH](https://www.raspberrypi.com/documentation/computers/remote-access.html#ssh) や [VNC](https://www.raspberrypi.com/documentation/computers/remote-access.html#vnc) セッションを開始する方法を知っておくと役立ちます。

## 運転を始める
[Donkeycar の組み立て](guide/build_hardware/)と[ソフトウェアのインストール](guide/install_software/)が終わったら、[テンプレート](guide/create_application/)を選んで車を[キャリブレーション](guide/calibrate/)し、[運転を開始](guide/get_driving/)しましょう。

## 車の挙動を変更する
Donkeycar にはあらかじめ多くの[テンプレート](guide/create_application/)が用意されており、設定を変更するだけで簡単に始められます。これらのテンプレートだけで十分かもしれませんが、さらに独自の動作をさせたい場合はテンプレートを変更したり新しく作成できます。Donkeycar のテンプレートはソフトウェアの[パーツ](parts/about/)を車のループ内で順に実行するパイプラインとして構成されており、各パーツは実行時に入力を読み取り、出力を車のメモリに書き込みます。一般的な車では次のようなパーツがあります。

- カメラから画像を取得する。Donkeycar は 3D カメラや [lidar](parts/lidar/) を含む多様な[カメラ](parts/cameras/)に対応しています。
- GPS 受信機から位置情報を取得する。
- [ゲームコントローラー](parts/controllers/)や RC コントローラーからステアリングやスロットルの入力を受け取る。Donkeycar は PS3、PS4、XBox、WiiU、Nimbus、Logitech の Bluetooth ゲームコントローラーおよび RaspberryPi で動作するあらゆるコントローラーに対応しています。またブラウザー対応のゲームコントローラーを接続可能な WebUI と、スマートフォンで使えるオンスクリーンタッチコントローラーも提供しています。
- 車の[ドライブトレイン](parts/actuators/)のスロットルとステアリングを制御する。Donkeycar は、RC カーで一般的な [ESC+Servo](parts/actuators/#standard-rc-with-esc-and-steering-servo) 構成をはじめ、さまざまな [差動駆動](parts/actuators/#differential-drive-cars)構成をサポートします。
- カメラ画像、ステアリングやスロットルの入力、lidar データなどのテレメトリ[データ](parts/stores/)を保存する。
- 車を自動運転させる。Donkeycar は[ディープラーニング](guide/deep_learning/train_autopilot/)、[GPS オートパイロット](guide/path_follow/path_follow/)、[コンピュータビジョン](guide/computer_vision/computer_vision/)の 3 種類の[オートパイロット](guide/train_autopilot/)をサポートします。ディープラーニング型では TensorFlow、TensorFlow Lite、Pytorch など多くのモデル[アーキテクチャ](parts/keras/)を利用できます。

もし希望する機能が既存のパーツにない場合は、自分で[パーツ](parts/about/#parts)を作成して車の[テンプレート](parts/about/)に追加してください。

```python
# 毎秒 10 回画像を取得して保存する車両を定義します

import time
from donkeycar import Vehicle
from donkeycar.parts.cv import CvCam
from donkeycar.parts.tub_v2 import TubWriter
V = Vehicle()

IMAGE_W = 160
IMAGE_H = 120
IMAGE_DEPTH = 3

# カメラパーツを追加
cam = CvCam(image_w=IMAGE_W, image_h=IMAGE_H, image_d=IMAGE_DEPTH)
V.add(cam, outputs=['image'], threaded=True)

# 画像保存用の tub パーツを追加
tub = TubWriter(path='./dat', inputs=['image'], types=['image_array'])
V.add(tub, inputs=['image'], outputs=['num_records'])

# 10 Hz でドライブループを開始
V.start(rate_hz=10)
```

詳しくは[ホームページ](http://donkeycar.com)を参照するか、[Discord サーバー](http://www.donkeycar.com/community.html)に参加してみてください。

楽しんでください

-----------------------

### Donkey という名前の由来

このプロジェクトの最終目標は役立つものを作ることです。Donkey（ロバ）は最初に家畜化された荷運び動物の一つで、頑固でありながら子どもにも安全です。車が都市の端から端まで自律走行できるようになるまでは、もっと壮大な名前を付けるのは控えておきます。
