# Donkey シミュレーター

[Donkey Gym](https://github.com/tawnkramer/gym-donkeycar) プロジェクトは、[Self Driving Sandbox](https://github.com/tawnkramer/sdsandbox/tree/donkey) ドンキーシミュレータ（`sdsandbox`）を OpenAI Gym で扱えるようにしたラッパーです。ソースからシミュレーターをビルドする場合は `sdsandbox` プロジェクトの `donkey` ブランチをチェックアウトしてください。

このシミュレーターは [Unity](https://unity.com/) ゲームプラットフォーム上に構築されており、内部の物理演算とグラフィックスを利用し、学習済みモデルで制御する Python プロセスへ接続します。

## インストール動画:

インストール手順を説明したビデオです。

Linux:
https://youtu.be/J6Ll5Obtuxk

Windows:
https://youtu.be/wqQMmHVT8qw

## My Virtual Donkey

目的に応じてシミュレーターの使い方はさまざまです。ここではシミュレーターを仮想ハードウェアとして扱い、通常の Donkeycar の drive/train/test サイクルを体験します。実機と__同じコマンド__でデータ収集、走行、学習を行います。まずこの使い方から説明します。

![sim_screen_shot](../../assets/sim_screen_shot.png)

## インストール

* [Donkey Gym Release](https://github.com/tawnkramer/gym-donkeycar/releases) からホスト PC 用のシミュレーターをダウンロードし解凍します。
* 好きな場所にシミュレーターを置きます。ここでは ~/projects/DonkeySimLinux を例にしますが、プラットフォームにより名前は異なります。
* [ホスト PC への Donkey インストール](https://docs.donkeycar.com/guide/install_software/#step-1-install-software-on-host-pc) の手順をすべて実行します。
* DonkeyGym を設定します:

```bash
cd ~/projects
git clone https://github.com/tawnkramer/gym-donkeycar
cd gym-donkeycar
conda activate donkey
pip install -e .[gym-donkeycar]
```

* 既存の ~/mycar アプリケーションを使うことも新しく作成することもできます。ここでは新規作成します:

```bash
donkey createcar --path ~/mysim
cd ~/mysim
```

* myconfig.py を編集して Donkey Gym シミュレーターラッパーを有効にし、`<user-name>` などを自分の環境に合わせます:

```bash
DONKEY_GYM = True
DONKEY_SIM_PATH = "/home/<user-name>/projects/DonkeySimLinux/donkey_sim.x86_64"
DONKEY_GYM_ENV_NAME = "donkey-generated-track-v0"
```

> 注: 実行ファイルのパスはプラットフォームとユーザーによって異なります。
>  Windows: DonkeySimWin/donkey_sim.exe
>  Mac OS: DonkeySimMac/donkey_sim.app/Contents/MacOS/donkey_sim
>  Linux: DonkeySimLinux/donkey_sim.x86_64

## ドライブ

manage.py の通常のコマンドが使えます。例えば:

```bash
python manage.py drive
```

自動的にシミュレーターが起動し接続されます。デフォルトではウェブインターフェースで操作でき、[http://localhost:8887/drive](http://localhost:8887/drive) にアクセスすると操作ページが表示されます。

Ubuntu Linux ではジョイスティックを接続して使用することも可能です。
`/dev/input/js0` として認識されれば動作する可能性があります。
myconfig.py でジョイスティックモデルを設定し `--js` オプションを付けて起動します。

```bash
python manage.py drive --js
```

走行すると通常通り data ディレクトリに記録が保存されます。

## 学習
You will not need to rsync your data, as it was recorded and resides locally. You can train as usual:

```bash
donkey train --tub ./data --model models/mypilot.h5
```

## テスト

You can use the model as usual:

```bash
python manage.py drive --model models/mypilot.h5
```

Then navigate to web control page. Set `Mode and Pilot` to `Local Pilot(d)`. The car should start driving.

## サンプル走行データ

Here's some sample driving data to get you started. [Download this](https://drive.google.com/open?id=1A5sTSddFsf494UDtnvYQBaEPYX87_LMp) and unpack it into your data dir. This should train to a slow but stable driver.

---

## API

Here's some info on the api to talk to the sim server. Make a TCP client and connect to port 9091 on whichever host the sim is running. The server sends and receives UTF-8 encoded JSON packets. Each message must have a "msg_type" field. The sim will end all JSON packets with a newline character for termination. You don't have to end each packet with a newline when sending to the server. But if it gets too many messages too quickly it may have troubles. Check the player log file for JSON parse errors if you are having troubles.

---

### プロトコルバージョン取得

Client=>Sim。プロトコルのバージョンを問い合わせます。メッセージ変更の確認に使用します。

Fields: *None*

Example:
```bash
    {
    "msg_type" : "get_protocol_version"
    }
```

---

### プロトコルバージョン

Sim=>Client。プロトコルのバージョンを返します。現在はバージョン 2 です。

Fields:

* *version* : string integer

Example:
```bash
    {
    "msg_type" : "protocol_version",
    "version" : "2",
    }
```

---

### シーン選択準備完了

Sim=>Client。メニューシーンのロードが完了すると送信されます。この後にシーンロードメッセージが有効になります。（メニューのみ）

Fields: *None*

Example:
```bash
    {
    "msg_type" : "scene_selection_ready"
    }
```

---

### シーン名取得

Client=>Sim。ロード可能なシーン名を尋ねます。（メニューのみ）

Fields: *None*

Example:
```bash
    {
    "msg_type" : "get_scene_names"
    }
```

---

### シーン名一覧

Sim=>Client。シーン名の一覧を返します。

Fields:

* *scene_names* : array of scene names

Example:
```bash
    {
    "msg_type" : "scene_names",
    "scene_names" : [ "generated_road", "warehouse", "sparkfun_avc", "generated_track" ]
    }
```

---

### シーン読み込み

Client=>Sim。メニュー画面からシーンを読み込むよう要求します。（メニューのみ）

Fields:

*scene_name* : **generated_road | warehouse | sparkfun_avc | generated_track** ( or whatever list the sim returns from get_scene_names)

Example:
```bash
    {
        "msg_type" : "load_scene",
        "scene_name" : "generated_track"
    }
```

---

### シーン読み込み完了

Sim=>Client。シーンが読み込まれると次のメッセージが返ります:

```bash
    {
        "msg_type" : "scene_loaded"
    }
```

---

### 車両読み込み完了

Sim=>Client。シミュレータが車両のロードを完了すると送信されます。シーン読み込み時にクライアントが接続されていれば自動的に車両がロードされます。

Fields: *None*

Example:
```bash
    {
    "msg_type" : "car_loaded"
    }
```

---
### 車両設定

Client=>Sim。読み込み後、車両の外観を設定できます（シーンのみ）。

Fields:

* *body_style* :  **donkey | bare | car01 | cybertruck | f1**
* *body_r* :  0〜255 の整数値（文字列）
* *body_g* :  0〜255 の整数値（文字列）
* *body_b* :  0〜255 の整数値（文字列）
* *car_name* :  車名を表示する文字列。改行を入れると複数行で表示されます。
* *font_size* :  10〜100 の整数値（文字列）で車名表示のフォントサイズを指定

Example:
```bash
    {
        "msg_type" : "car_config",
        "body_style" : "car01",
        "body_r" : "128",
        "body_g" : "0",
        "body_b" : "255",
        "car_name" : "Your Name",
        "font_size" : "100"
    }
```

---

### カメラ設定

Client=>Sim。シーン読み込み後、車載カメラのセンサー設定を行えます。

Fields:

* *fov* :  10〜200 の浮動小数点値（文字列）。カメラの視野角を度で指定。
* *fish_eye_x* :  0〜1 の浮動小数点値（文字列）。X 軸方向の歪み量。
* *fish_eye_y* :  0〜1 の浮動小数点値（文字列）。Y 軸方向の歪み量。
* *img_w* :  16〜512 の整数値（文字列）。カメラ画像の幅。
* *img_h* :  16〜512 の整数値（文字列）。カメラ画像の高さ。
* *img_d* :  1 または 3 の整数値（文字列）。画像の深さ。1 の場合でもチャンネル数は 3 で、シミュレータ側でグレースケール変換が行われます。
* *img_enc* :  画像フォーマット **JPG | PNG | TGA**
* *offset_x* : 浮動小数点値（文字列）。カメラを左右方向に移動。
* *offset_y* : 浮動小数点値（文字列）。カメラを上下方向に移動。
* *offset_z* : 浮動小数点値（文字列）。カメラを前後方向に移動。
* *rot_x* : 浮動小数点値（文字列）。度数法。カメラを X 軸回りに回転。

Example:
```bash
    {
    "msg_type" : "cam_config",
    "fov" : "150",
    "fish_eye_x" : "1.0",
    "fish_eye_y" : "1.0",
    "img_w" : "255",
    "img_h" : "255",
    "img_d" : "1",
    "img_enc" : "PNG",
    "offset_x" : "0.0",
    "offset_y" : "3.0",
    "offset_z" : "0.0",
    "rot_x" : "90.0"
    }
```

注意:
msg_type を "cam_config_b" に変更すると別のカメラを追加できます。

---

### 車両操作

Client=>Sim。スロットルとステアリングを制御します。

Fields:

* *steering* :  -1〜1 の浮動小数点値（文字列）。-1 が左いっぱい、1 が右いっぱいで中心から約16度。
* *throttle* :  -1〜1 の浮動小数点値（文字列）。前進または後退のトルクを指定。
* *brake* :  0〜1 の浮動小数点値（文字列）。

Example:
```bash
    {
    "msg_type" : "control",
    "steering" : "0.0",
    "throttle" : "0.3",
    "brake" : "0.0"
    }
```

---

### テレメトリ

Sim=>Client。カメラ画像と車両状態を含むメッセージを送ります。通常 1 秒間に約 20 回の頻度で送られます。

Fields:

* *steering_angle* :  最後に適用されたステアリング角。なぜ control と同じ名前ではないのかは不明。
* *throttle* :  最後に適用されたスロットル。
* *speed* :  車両速度の大きさ。
* *image* :  BinHex でエンコードされた画像。`PIL.Image.open(BytesIO(base64.b64decode(imgString)))` で読み込めます。
* *imageb* :  （オプション）2 台目のカメラ画像も同様に取得。
* *lidar* :  （オプション）{d: distanceToObject, rx: rayRotationX, ry: rayRotationY} 形式の LiDAR 点群リスト。
* *hit* :  最後に衝突したオブジェクト名。衝突していない場合は None。
* *accel_x* :  車両の X 軸加速度。
* *accel_y* :  車両の Y 軸加速度。
* *accel_z* :  車両の Z 軸加速度。
* *gyro_x* :  X 軸ジャイロ。
* *gyro_y* :  Y 軸ジャイロ。
* *gyro_z* :  Z 軸ジャイロ。
* *gyro_w* :  W 成分のジャイロ。
* *pitch* :  車両のピッチ角（度）。
* *roll* :  車両のロール角（度）。
* *yaw* :  車両のヨー角（度）。
* *activeNode* :  コース上の進行状況（複数車両では正しく動きません）。
* *totalNodes* :  コースのノード総数。
* *pos_x* :  （学習時のみ）車両の X 座標。
* *pos_y* :  （学習時のみ）車両の Y 座標。
* *pos_z* :  （学習時のみ）車両の Z 座標。
* *vel_x* :  （学習時のみ）車両の X 速度。
* *vel_y* :  （学習時のみ）車両の Y 速度。
* *vel_z* :  （学習時のみ）車両の Z 速度。
* *cte* :  （学習時のみ）クロストラック誤差。右車線中央またはコース中央から車両までの距離（コースによる）。

Example:
```bash
    {
    "msg_type" : "telemetry",
    "steering_angle" : "0.0",
    "throttle" : "0.0",
    "speed" : "1.0",
    "image" : "0x123...",
    "hit" : "None",
    "pos_x" : "0.0",
    "pos_y" : "0.0",
    "pos_z" : "0.0",
    "accel_x" : "0.0",
    "accel_y" : "0.0",
    "accel_z" : "0.0",
    "gyro_x" : "0.0",
    "gyro_y" : "0.0",
    "gyro_z" : "0.0",
    "gyro_w" : "0.0",
    "pitch" : "0.0",
    "roll" : "0.0",
    "yaw" : "0.0",
    "activeNode" : "5",
    "totalNodes" : "26",
    "cte" : "0.5"
    }
```

---

### 車両リセット

Client=>Sim。車両をスタート地点に戻します。

Fields: *None*

Example:
```bash
    {
    "msg_type" : "reset_car"
    }
```

---
### 車両位置設定
Client=>Sim。指定した位置に車両を移動させます（学習時のみ）。

Fields:

* *pos_x* :  X 座標。
* *pos_y* :  Y 座標。
* *pos_z* :  Z 座標。
* *qx* :  （オプション）クォータニオン x
* *qy* :  （オプション）クォータニオン y
* *qz* :  （オプション）クォータニオン z
* *qw* :  （オプション）クォータニオン w

Example:
```bash
    {
    "msg_type" : "set_position",
    "pos_x" : "0.0",
    "pos_y" : "0.0",
    "pos_z" : "0.0"
    }
```
または:

```bash
    {
    "msg_type" : "set_position",
    "pos_x" : "0.0",
    "pos_y" : "0.0",
    "pos_z" : "0.0",
    "qx" : "0.0",
    "qy" : "0.2",
    "qz" : "0.0",
    "qw" : "1.0"
    }
```

---

### ノード位置と回転の取得
Client=>Sim。node_position パケットを要求します。

Fields:

* *index* :  ノードのインデックス

Example:
```bash
    {
    "msg_type": "node_position",
    "index": "0"
    }
```

---

### ノード位置と回転
Sim=>Client。node_position パケット（node_position メッセージ送信後に受信）。

Fields:

* *pos_x* :  X 座標。
* *pos_y* :  Y 座標。
* *pos_z* :  Z 座標。
* *qx* :  （オプション）クォータニオン x
* *qy* :  （オプション）クォータニオン y
* *qz* :  （オプション）クォータニオン z
* *qw* :  （オプション）クォータニオン w

Example:
```bash
    {
    "msg_type": "node_position",
    "Qx": "0",
    "Qy": "0",
    "Qz": "0",
    "Qw": "1",
    "pos_x": "0",
    "pos_y": "0",
    "pos_z": "0"
    }
```

---

### シーン終了

Client=>Sim。シーンを離れてメインメニューに戻ります。

Fields: *None*

Example:
```bash
    {
    "msg_type" : "exit_scene"
    }
```

---

### アプリ終了

Client=>Sim。シミュレータアプリを終了します。（メニューのみ）

Fields: *None*

Example:
```bash
    {
    "msg_type" : "quit_app"
    }
```





















