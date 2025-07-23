# Robocar Controller

Robocar Controller は、Donkey Car を手軽に始められるよう「コマンド入力不要」のユーザー体験を提供するモバイルアプリです。

![cover](/docs/assets/mobile_app/cover.png)

## Features
- コマンド不要の操作 ― SSH やテキストエディターは不要
- 内蔵ホットスポット
- ネットワーク上で車両を検索
- リアルタイムキャリブレーション
- バーチャルジョイスティック
- データの可視化
- 走行サマリー
- 無料 GPU トレーニング
- 自動運転
- 詳細設定
- バッテリー残量表示

## Requirements
- Pi 4B を搭載した Donkey Car（Jetson Nano は未対応）
- iOS または Android を搭載したスマートフォン

## Quickstart Guide
<a href="https://medium.com/robocar-store/robocar-controller-quick-start-guide-bdf8cb16d7ce?source=friends_link&sk=8f21a5792f81a1d340abe9433d78cf5b" target="_blank">クイックスタートガイド</a> を参照してください。

> 既成イメージを使用したくない場合は、サーバーコンポーネントを手動で Donkey Car にインストールすることも可能です。詳しくは下記の [Optional Manual Installation](#optional-manual-installation) を参照してください。


## Features Details
### Built-in Hotspot
既知の Wi-Fi ネットワークがない場合、車はホットスポットとして動作します。携帯電話をこのホットスポットに接続した後、アプリを使って車を希望の Wi-Fi ネットワークへ参加させることができます。

### Search vehicle on the network
車がスマートフォンと同じネットワークに接続すると、アプリがネットワーク全体をスキャンして車両を見つけます。SSH で接続したい場合に備えて IP アドレスも表示されます。

![Search Vehicle](/docs/assets/mobile_app/search-vehicle.png)

### Real-time Calibration
車が速すぎたりまっすぐ走らなかったりすると不便です。キャリブレーション UI は適切な設定を見つける手助けをし、車を正しく調整できます。強化されたキャリブレーション機能により変更はリアルタイムで反映され、すぐに効果を確認できます。

![Real-time calibration](/docs/assets/mobile_app/calibration.png)

### Virtual Joystick
物理的なゲームパッドコントローラーがない場合でも、バーチャルジョイスティックを使えばすぐに走行テストができます。カメラからの映像もリアルタイムでストリーミングされるため、画面を見ながらそのまま運転できます。

![Drive UI](/docs/assets/mobile_app/drive-ui.gif)


### Drive Summary
アプリはヒストグラムとともに収集した画像の数や容量をまとめた走行サマリーを表示します。ヒストグラムは Donkey Car ソフトウェアの ```tubhist``` 関数により自動生成されます。

![Drive summary](/docs/assets/mobile_app/drive-summary.png)

### Visualize the data
アプリでは Pi に保存されたすべてのデータ（tub）とメタデータを確認できます。メタデータには画像枚数、tub のサイズ、解像度、ヒストグラム、保存場所が含まれます。`donkey makemovie` コマンドを利用して動画を生成し、データの内容を振り返ることも可能です。

![Data](/docs/assets/mobile_app/data.png)

### Free GPU Training
アプリ利用者は無料で GPU トレーニングを行えます。トレーニングしたいデータ（tub）を選択すると、データがサーバーへアップロードされ学習が開始されます。学習完了後、アプリには損失と精度のグラフが表示され、同時にモデルが車にダウンロードされるので、すぐにテストできます。

注: データとモデルは一定期間のみ保存され、その後ストレージから削除されます。

![Train](/docs/assets/mobile_app/train.png)

#### More on Free GPU Training

トレーニングには AWS の [g4dn.xlarge](https://aws.amazon.com/ec2/instance-types/g4/) インスタンスを利用しています。これは [NVIDIA T4 GPU](https://www.nvidia.com/en-us/data-center/tesla-t4/) を搭載し、最大 16GB の GPU メモリを持ちます。GPU の性能を最大限活用するため、バッチサイズを 256 以上に設定することを推奨します。

#### Limitation
悪用を防ぐため、トレーニングサービスには以下のルールがあります。

- 各トレーニングは最大 15 分までで、それを超えるとタイムアウトになります
- 1 つのデバイスが利用できるのは 24 時間で 5 回まで
- トレーニングに使用できるデータの上限は 100MB

### Autopilot

Pi 内にあるすべてのモデルが一覧表示され、トレーニング機能で生成したものでもコピーしただけのモデルでも利用できます。Drive UI と同様の画面から自動運転モードを開始できます。

![Autopilot](/docs/assets/mobile_app/autopilot.gif)

### Advanced configuration
### Advanced configuration
Donkey Car ソフトウェアには多数の設定項目があり、さまざまな実験が行えます。よく変更されるオプションをいくつかアプリ内に用意しました。

- カメラ解像度
- トレーニング設定
- ドライブトレイン設定

![Advanced configuration](/docs/assets/mobile_app/advanced-configuration.png)



### Battery level

MM1 を使用している場合、アプリは現在のバッテリー残量をパーセンテージで表示します。バッテリー電圧が 7V を下回った際に自動的にシステムをシャットダウンする OS の調整も追加しています。



## Upcoming features
## Upcoming features
- Salient visualization
- バッテリー残量に基づく自動スロットル補正
- 転移学習



## Report a problem
問題が発生した場合は、[こちらの GitHub プロジェクト](https://github.com/robocarstore/donkeycar_controller) で issue を登録してください。

## Optional Manual Installation
## Optional Manual Installation
プリビルドの SD イメージを使用できない、または使用したくない場合は、サーバーコンポーネントを手動で Donkey Car にインストールできます。
[Donkey Car console](https://github.com/robocarstore/donkeycar-console) は Donkey Car を管理するソフトウェアで、モバイルアプリをサポートする REST ベースの API を提供します。

> _**Note**_ このソフトウェアは **RaspberryPi 4B のみ** に対応しています。

#### 1. [RaspberryPi のセットアップ](/docs/guide/robot_sbc/setup_raspberry_pi.md) を完了する
#### 2. Donkey Car Console プロジェクトをクローンする

```bash
git clone https://github.com/robocarstore/donkeycar-console
sudo mv donkeycar-console /opt
cd /opt/donkeycar-console
```

#### 3. 依存関係をインストールする

```bash
pip install -r requirements/production.txt
```

#### 4. データベースを初期化するスクリプトを実行

```bash
python manage.py migrate
```

#### 5. サーバーが正しく動作するかテストする

```bash
python manage.py runserver 0.0.0.0:8000
```

ブラウザーで http://your_pi_ip:8000/vehicle/status にアクセスし、エラーが表示されなければ成功です。

#### 6. サーバーをサービスとしてインストール

```bash
sudo ln -s gunicorn.service /etc/systemd/system/gunicorn.service
```

#### 7. スマートフォンにモバイルアプリをインストール

- [iOS](https://apps.apple.com/app/robocar-controller/id1508125501)
- [Android](https://play.google.com/store/apps/details?id=com.robocarLtd.RobocarController)

携帯電話が Pi と同じネットワークに接続されていることを確認してください（接続できない場合はモバイルデータ通信をオフにしてみてください）。アプリを起動すれば、車両を検索できます。


## FAQ
- なぜアプリ名が Donkeycar Controller ではなく Robocar Controller なのですか？

私たちはアプリを Donkeycar Controller と名付けたいのですが、Apple から許可が得られていません。現在、Adam と協力して Apple に商標使用の証明を提出する準備を進めています。それまでは Robocar Controller の名称を使用します。



## Commercial Usage
このアプリは [Robocar Store](https://www.robocarstore.com) によって開発されています。営利目的で本アプリを使用する場合は、[Donkey Car ガイドライン](https://www.donkeycar.com/make-money.html) に従い、[Robocar Store](mailto:sales@robocarstore.com) までご連絡ください。
