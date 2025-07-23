# アクチュエータ

車両が前後に動き、左右に曲がるためには仕組みが必要です。ロボットで物理的な動作を生み出す装置を一般に「アクチュエータ」と呼びます。代表的なアクチュエータにはDCモーター、サーボモーター、連続回転サーボモーター、ステッピングモーターがあります。これらのアクチュエータを組み合わせてロボットを走らせたり曲げたりする方法は無数に存在します。Donkeycar ではさまざまなアクチュエータで実装できる2つの一般的な構成をサポートしています。

- **車型の車両** は前輪を左右に角度を付けて操舵し、駆動輪を前後に回して走行します。一般的なRCカーはこの分類です。
- **差動走行車両** は個別に制御できる2つの駆動輪を持ち、これによって前進とステアリングの両方を行います。たとえば両方の駆動輪を同じ速度で前進させれば直進し、一方を速く回せばその反対側へ弧を描いて曲がります。[差動走行車](#differential-drive-cars)についてはこのページの後半で解説します。

アクチュエータは Donkeycar から送られる制御信号によって動作します。これらの制御信号を生成する方法はいくつかあります。

- RaspberryPi の上に載せて既存のRC受信機を利用する「RC Hat」
- PCA9685 サーボコントローラボード
- RPi/Jetson の40ピンGPIOヘッダー
    - Jetson Nano 40ピンGPIOヘッダーからPWM出力を有効化する方法は[Jetson NanoからのPWM生成](./pins.md#generating-pwm-from-the-jetson-nano)を参照してください。
- Arduino

以下では、サポートされているアクチュエータ構成とその制御信号のソフトウェア設定について説明します。

## ESC とステアリングサーボを用いた標準的なRC

一般的なRCカーには前輪を操舵するステアリングサーボと、駆動用DCモーターの速度を制御するESC(Electronic Speed Controller)が搭載されています。ステアリングサーボもESCもともにPWM(Pulse Width Modulation)信号で制御されます。PWM信号とは一定の周期である幅のパルスを送るもので、ステアリングサーボではパルス幅でサーボアームの角度(おおよそ0度=右一杯から180度=左一杯まで)を決めます。ESCではパルス幅で駆動モーターの回転方向と速度を決め、全開後退から停止、全開前進までを指示します。

- 標準的なRCサーボのパルス幅は1ミリ秒(ESCでは全開後退、サーボでは左一杯)から2ミリ秒(ESCでは全開前進、サーボでは右一杯)までで、1.5ミリ秒が中立(ESCでは停止、サーボでは直進)です。
- このパルスは通常50ヘルツ(20ミリ秒周期)で送信されます。1周期とは信号をHIGHにした後LOWに戻すまでで、50ヘルツの場合1ミリ秒パルス(1ms HIGH/19ms LOW)はデューティ比5%、2ミリ秒パルスはデューティ比10%となります。
- 重要なのはパルスの長さで、1〜2ミリ秒の範囲に収まっている必要があります。

![サーボモーターの典型的なPWMタイミング図 (Wikipedia)](../assets/parts/Servomotor_Timing_Diagram.svg "A diagram showing typical PWM timing for a servomotor (Wikipedia)")

- つまり別の周波数を使用する場合は、1〜2ミリ秒のパルス幅を得るためにデューティ比を調整する必要があります。
- 例えば60ヘルツであれば、1ミリ秒パルスは0.05 * 60 / 50 = 0.06(6%)のデューティ比となります。
- PCA9685 の周波数はデフォルトで60ヘルツなので、設定値も60ヘルツかつ12ビット値を基準としています。PCA9685は12ビット解像度のため、1ミリ秒パルスは0.06 * 4096 ≒ 246、中立の0.09デューティ比は0.09 * 4096 ≒ 367、全開前進の0.12デューティ比は0.12 * 4096 ≒ 492 となります。
- これらはAPI呼び出しの引数やキャリブレーション時に生成される値を理解する上で役立つ概念です。最終的なデューティ比やパルス幅の選択はハードウェアや方針によって決まります(あまり速くしたくないなら最大スロットルのPWMを低めにするなど)。

### RC Hat を使ってコマンドを生成する
- [Donkeycar Store](https://www.diyrobocars.com/shop/)でRC Hatを購入できます。既存のRC送受信機を使う場合、これが最も簡単な方法です。使い方は商品ページに記載されています。

### PCA9685 サーボコントローラでPWMパルスを生成する
- PCA9685 I2Cサーボドライバボードのハードウェア接続は[こちら](../guide/build_hardware.md)のセットアップ手順に詳しく説明されています。
- PCA9685 サーボコントローラはRaspberryPiまたはJetson Nanoと40ピンバスのI2Cピンで接続し、ESCとステアリングサーボからの3ピンケーブルを一般的にはそれぞれチャンネル0とチャンネル1へ接続します。[ステップ4: サーボシールドを接続](../guide/build_hardware.md#step-4-connect-servo-shield-to-raspberry-pi) を参照してください。Jetson Nanoへの接続も同様です。

**設定**

- myconfig.py で `DRIVE_TRAIN_TYPE = "PWM_STEERING_THROTTLE"` を使用します。
  - myconfig.py の `PWM_STEERING_THROTTLE` セクションで、`PWM_STEERING_PIN` と `PWM_THROTTLE_PIN` に PCA9685 のピン指定子を設定します。例:

```python
    DRIVE_TRAIN_TYPE = "PWM_STEERING_THROTTLE"

    #
    # PWM_STEERING_THROTTLE
    #
    # ステアリングサーボとESCを備えたRCカー向けドライブトレイン
    # ステアリング用に1つの PwmPin、スロットル用にもう1つの PwmPin を使用
    # PWMの基本周波数は60Hzを想定。異なる周波数を使用する場合は PWM_xxxx_SCALE でパルス幅を調整
    #
    PWM_STEERING_THROTTLE = {
        "PWM_STEERING_PIN": "PCA9685.1:40.0",   # PCA9685、I2Cバス1、アドレス0x40、チャンネル0
        "PWM_STEERING_SCALE": 1.0,              # PWM周波数が60Hzと異なる場合の補正。ステアリング範囲調整用ではない
        "PWM_STEERING_INVERTED": False,         # ハードウェアが反転PWMパルスを要求する場合はTrue
        "PWM_THROTTLE_PIN": "PCA9685.1:40.1",   # PCA9685、I2Cバス1、アドレス0x40、チャンネル1
        "PWM_THROTTLE_SCALE": 1.0,              # PWM周波数が60Hzと異なる場合の補正。速度制限/増加用ではない
        "PWM_THROTTLE_INVERTED": False,         # ハードウェアが反転PWMパルスを要求する場合はTrue
        "STEERING_LEFT_PWM": 460,               # 左一杯ステアのPWM値
        "STEERING_RIGHT_PWM": 290,              # 右一杯ステアのPWM値
        "THROTTLE_FORWARD_PWM": 500,            # 最大前進スロットルのPWM値
        "THROTTLE_STOPPED_PWM": 370,            # 停止時のPWM値
        "THROTTLE_REVERSE_PWM": 220,            # 最大後退スロットルのPWM値
    }
```
> 注: PWM値(`STEERING_LEFT_PWM` など)は車ごとに異なり、キャリブレーション手順で求めます。詳しくは[車のキャリブレーション](http://docs.donkeycar.com/guide/calibrate/)を参照してください。

> ピンプロバイダやピン指定子の詳細は [pins](pins.md) を参照してください。

### 40ピンGPIOヘッダーからPWMパルスを生成する
- PWM信号を40ピンGPIOヘッダーから直接出力する方法です。ESCとサーボの3ピンコネクタのデータピンをGPIOのPWMピンに接続し、3ピンコネクタのGNDピンは共通のGNDに接続します。5VピンはGPIOの5Vに接続しますが、ESCの3ピンコネクタが一般に5Vを供給するため、それをサーボの電源として利用できます。

**設定**

- myconfig.py で `DRIVE_TRAIN_TYPE = "PWM_STEERING_THROTTLE"` を使用します。
  - `# PWM_STEERING_THROTTLE` セクションでGPIOのピン指定子を設定します。各ピンにはBOARDモード番号とBCM(Broadcom)モード番号があり、どちらでも利用できますが、すべて同じモードで指定してください。
  - RaspberryPi 4b にはハードウェアPWM出力が4つあり、そのうち3つが40ピンヘッダーに割り当てられています(https://linuxhint.com/gpio-pinout-raspberry-pi/ を参照)。ピンは基板番号または内部gpio番号で指定できます(http://docs.donkeycar.com/parts/pins/ を参照)。ハードウェアPWMピンの場合、BOARDピン12("RPI_GPIO.BOARD.12") は GPIO18("RPI_GPIO.BCM.18")、BOARDピン33("RPI_GPIO.BOARD.33") は GPIO13("RPI_GPIO.BCM.13")、BOARDピン32("RPI_GPIO.BOARD.32") は GPIO12("RPI_GPIO.BCM.12") と同じです。したがって、`DRIVE_TRAIN_TYPE = "PWM_STEERING_THROTTLE"` とし、`PWM_STEERING_PIN` と `PWM_THROTTLE_PIN` にハードウェアPWMピンを指定してください。例:

```python
    DRIVE_TRAIN_TYPE = "PWM_STEERING_THROTTLE"

    #
    # PWM_STEERING_THROTTLE
    #
    # ステアリングサーボとESCを備えたRCカー向けドライブトレイン
    # ステアリング用に1つの PwmPin、スロットル用にもう1つの PwmPin を使用
    # PWMの基本周波数は60Hzを想定。異なる周波数を使用する場合は PWM_xxxx_SCALE でパルス幅を調整
    #
    PWM_STEERING_THROTTLE = {
        "PWM_STEERING_PIN": "RPI_GPIO.BOARD.33",# BOARDモード33番ピン == BCMモード13番ピン
        "PWM_STEERING_SCALE": 1.0,              # PWM周波数が60Hzと異なる場合の補正。ステアリング範囲調整用ではない
        "PWM_STEERING_INVERTED": False,         # ハードウェアが反転PWMパルスを要求する場合はTrue
        "PWM_THROTTLE_PIN": "RPI_GPIO.BOARD.12",# BOARDモード12番ピン == BCMモード18番ピン
        "PWM_THROTTLE_SCALE": 1.0,              # PWM周波数が60Hzと異なる場合の補正。速度制限/増加用ではない
        "PWM_THROTTLE_INVERTED": False,         # ハードウェアが反転PWMパルスを要求する場合はTrue
        "STEERING_LEFT_PWM": 460,               # 左一杯ステアのPWM値
        "STEERING_RIGHT_PWM": 290,              # 右一杯ステアのPWM値
        "THROTTLE_FORWARD_PWM": 500,            # 最大前進スロットルのPWM値
        "THROTTLE_STOPPED_PWM": 370,            # 停止時のPWM値
        "THROTTLE_REVERSE_PWM": 220,            # 最大後退スロットルのPWM値
    }
```
> 注: PWM値(`STEERING_LEFT_PWM` など)は車ごとに異なり、キャリブレーション手順で求めます。詳しくは[車のキャリブレーション](http://docs.donkeycar.com/guide/calibrate/)を参照してください。

> ピンプロバイダやピン指定子の詳細は [pins](pins.md) を参照してください。

### RaspberryPi のGPIOピンで直接制御する

詳しい手順は[こちら](../parts/rc.md)を参照してください。

### Robo HAT MM1 ボードで制御する

詳しい手順は[こちら](https://robohatmm1-docs.readthedocs.io/en/latest/)を参照してください。

### Arduino
Arduino を利用してステアリングとスロットルを制御するPWM信号を生成することもできます。

現在 Arduino モードは [Latte Panda Delta (LP-D)](https://www.lattepanda.com/products/lattepanda-delta-432.html) ボードでのみテストされていますが、Raspberry Pi や Jetson Nano でも(PCA9685の代わりとして)簡単に利用できるはずです。

以下のブロック図で各部の接続位置を確認してください。

![ブロック図](../assets/Arduino_actuator_blk_dgm.jpg)

Arduino ボードには標準のFirmataスケッチを実行させてください(Arduinoツールをダウンロードするとデフォルトで付属しています)。Arduinoへは _Examples > Firmata > StandardFirmata_ にあるスケッチを書き込みます。
![配線図](../assets/Arduino_firmata_sketch.jpg)
さらに車両側コンピュータに **pymata_aio** Pythonパッケージを _pip3 install pymata_aio_ でインストールしておく必要があります。

上のブロック図の通り、LattePanda は x86 CPU と接続済みの Arduino を1枚のボードに統合したものです。

次の図は Arduino のピンをステアリングサーボと ESC にどのように接続するかを示しています。

![配線図](../assets/ArduinoWiring.png)
サーボへの電源は、多くのESCが備えるBEC(Battery Eliminator Circuit)から供給します。これは Arduino の5Vからサーボに全ての電力を供給しないためです。大型RCカーではサーボが最大2Aを消費することがあり、それにより Arduino が破損する恐れがあります。

### キャリブレーション
Arduino では( PCA9685とは)若干異なるキャリブレーション手順/値を用います。通常の中央値は90(50Hzでパルス幅1.5ms)なので、まず90から開始し、左右/前後それぞれ±5ずつ調整して望む範囲を見つけてください。
```bash
(env1) jithu@jithu-lp:~/master/pred_mt/lp/001/donkey$ donkey calibrate --arduino --channel 6
using donkey v2.6.0t ...

pymata_aio Version 2.33 Copyright (c) 2015-2018 Alan Yorinks All rights reserved.

Using COM Port:/dev/ttyACM0

Initializing Arduino - Please wait...
Arduino Firmware ID: 2.5 StandardFirmata.ino
Auto-discovery complete. Found 30 Digital Pins and 12 Analog Pins


Enter a PWM setting to test(0-180)95
Enter a PWM setting to test(0-180)90
Enter a PWM setting to test(0-180)85
...
```
キャリブレーションコマンドには **--arduino** オプションを付け、さらにキャリブレーションする Arduino ピンを **--channel** パラメータで指定します。

### Arduino アクチュエータパートの使用

drive() ループ内で Arduino アクチュエータを利用する例を以下に示します。

```python
    #Drive train setup
    arduino_controller = ArduinoFirmata(
                                    servo_pin=cfg.STEERING_ARDUINO_PIN,
                                    esc_pin=cfg.THROTTLE_ARDUINO_PIN)

    steering = ArdPWMSteering(controller=arduino_controller,
                        left_pulse=cfg.STEERING_ARDUINO_LEFT_PWM,
                        right_pulse=cfg.STEERING_ARDUINO_RIGHT_PWM)

    throttle = ArdPWMThrottle(controller=arduino_controller,
                        max_pulse=cfg.THROTTLE_ARDUINO_FORWARD_PWM,
                        zero_pulse=cfg.THROTTLE_ARDUINO_STOPPED_PWM,
                        min_pulse=cfg.THROTTLE_ARDUINO_REVERSE_PWM)

    V.add(steering, inputs=['user/angle'])
    V.add(throttle, inputs=['user/throttle'])
```

templates/arduino_drive.py も参照してください。

## HBridge モータードライバとステアリングサーボ
この構成では、車輪を駆動するDCモーターを L298N HBridge モータードライバ(互換品含む)で制御します。前輪の操舵はPWMパルスを受けるステアリングサーボで行います。モータードライバの配線方法には3ピン方式と2ピン方式の2通りがあります。

### 3ピン HBridge とステアリングサーボ
単一のDCギアモーターをL298Nで制御する際、方向選択に2本のTTL出力ピン、モーターへの電力制御にPWMピン1本を使用します。

![L298N Motor Driver Module](../assets/L298N-Driver-Board-Module.png "L298N Motor Driver Module")
**L298N モータードライバモジュール**

3ピンモードでL298N HBridgeモジュールをRaspberryPiのGPIOに配線する方法については、https://www.electronicshub.org/raspberry-pi-l298n-interface-tutorial-control-dc-motor-l298n-raspberry-pi/ を参照してください。これはTB6612FNGなどL298N互換のドライバチップにも当てはまります。

**設定**

- myconfig.py で `DRIVETRAIN_TYPE = "SERVO_HBRIDGE_3PIN"` を使用します。
  - 40ピンGPIOヘッダーから信号を生成する場合のピン指定例:
```python
HBRIDGE_3PIN_FWD = "RPI_GPIO.BOARD.18"   # TTLピン、高で前進
HBRIDGE_3PIN_BWD = "RPI_GPIO.BOARD.16"   # TTLピン、高で後退
HBRIDGE_3PIN_DUTY = "RPI_GPIO.BOARD.35"  # モーターへのデューティ比
PWM_STEERING_PIN = "RPI_GPIO.BOARD.33"   # ステアリングサーボへのパルス
STEERING_LEFT_PWM = 460         # 左一杯ステアのPWM値(`donkey calibrate`で測定)
STEERING_RIGHT_PWM = 290        # 右一杯ステアのPWM値(`donkey calibrate`で測定)
```

PCA9685 を用いてすべての制御信号を生成することも可能です。詳細は [pins](pins.md) を参照してください。

### 2ピン HBridge とステアリングサーボ
ミニL298d HBridge(またはL9110S HBridge)で単一のDCギアモーターを制御する場合、2本のPWMピンを用います。1本は前進速度用、もう1本は後退速度用です。

![L293D Motor Driver Module](../assets/MINI-L293D-Motor-driver-module.png "L293D Motor Driver Module")
**L293D モータードライバモジュール**

L298d ミニHBridgeモジュールを2ピンモードで配線する方法については https://www.instructables.com/Tutorial-for-Dual-Channel-DC-Motor-Driver-Board-PW/ を参照してください。
L9110S/HG7881 モータードライバモジュールの配線方法は https://electropeak.com/learn/interfacing-l9110s-dual-channel-h-bridge-motor-driver-module-with-arduino/ を参照してください。

**設定**

- myconfig.py で `DRIVETRAIN_TYPE = "SERVO_HBRIDGE_2PIN"` を使用します。
  - 40ピンGPIOヘッダーから信号を生成する場合のピン指定例:
```python
  HBRIDGE_2PIN_DUTY_FWD = "RPI_GPIO.BOARD.18"  # 前進用デューティ比
  HBRIDGE_2PIN_DUTY_BWD = "RPI_GPIO.BOARD.16"  # 後退用デューティ比
  PWM_STEERING_PIN = "RPI_GPIO.BOARD.33"       # ステアリングサーボへのパルス
  STEERING_LEFT_PWM = 460         # 左一杯ステアのPWM値(`donkey calibrate`で測定)
  STEERING_RIGHT_PWM = 290        # 右一杯ステアのPWM値(`donkey calibrate`で測定)
```
PCA9685 を用いてすべての制御信号を生成することも可能です。詳細は [pins](pins.md) を参照してください。

## ステアリングとスロットルの両方をHBridgeで制御する
ごく安価な玩具の車では、後輪駆動用のDCモーターと、前輪操舵用の別のDCモーターを使うことがあります。これら2つのモーターは1つのL298N HBridge(またはL9110S HBridge)で制御できます。このドライバでは2ピン配線を想定しており、各モーターにつき2本のPWMピン(方向ごとに1本)を使用します。

**設定**

- myconfig.py で `DRIVETRAIN_TYPE = "DC_STEER_THROTTLE"` を使用します。
  - 40ピンGPIOヘッダーから信号を生成する場合のピン指定例:
```python
  HBRIDGE_PIN_LEFT = "RPI_GPIO.BOARD.18"   # 左旋回用デューティ比
  HBRIDGE_PIN_RIGHT = "RPI_GPIO.BOARD.16"  # 右旋回用デューティ比
  HBRIDGE_PIN_FWD = "RPI_GPIO.BOARD.15"    # 前進用デューティ比
  HBRIDGE_PIN_BWD = "RPI_GPIO.BOARD.13"    # 後退用デューティ比
```
PCA9685 を用いてすべての制御信号を生成することも可能です。詳細は [pins](pins.md) を参照してください。

## ステアリングとスロットルを両方VESCで制御する

VESC はESCの高度なバージョンで、再生ブレーキや温度管理など、多くのカスタマイズ機能を提供します。

これはVESC 6とTraxxas ブラシレスモーターでテストされています。VESCのファームウェア更新とキャリブレーションは F1Tenth のチュートリアル [https://f1tenth.readthedocs.io/en/stable/getting_started/firmware/firmware_vesc.html](https://f1tenth.readthedocs.io/en/stable/getting_started/firmware/firmware_vesc.html) に従ってください。ステアリングもVESCで制御できるよう、servo out bin を使用することが重要です。

サーボ制御には PyVESC をソースからインストールする必要があります(`pip install git+https://github.com/LiamBindle/PyVESC.git@master`)。
**設定**

- myconfig.py で `DRIVETRAIN_TYPE = "VESC"` を使用します。
  - パラメータ例
```python
  VESC_MAX_SPEED_PERCENT =.2  # 実際の速度に対する最大速度の割合
  VESC_SERIAL_PORT= "/dev/ttyACM0" # 通信用シリアルデバイス。ls /dev/tty* で確認可能
  VESC_HAS_SENSOR= True # BLDCモーターがホールセンサーを使用しているか
  VESC_START_HEARTBEAT= True # コマンドを生かし続けるハートビートスレッドを自動開始するか
  VESC_BAUDRATE= 115200 # シリアル通信のボーレート。変更の必要はほぼない
  VESC_TIMEOUT= 0.05 # シリアル通信のタイムアウト
  VESC_STEERING_SCALE= 0.5 # VESCは0〜1のステア入力を受け付ける。ジョイスティックは通常-1〜1なので-0.5〜0.5に変換
  VESC_STEERING_OFFSET = 0.5 # 上記変換に伴いジョイスティックの範囲を0〜1にシフト
```

## 差動駆動車
安価なスマートカー用シャーシにはDCギアモーター2個と、駆動用モーターを制御するL298Nモータードライバ(互換品含む)が付属しており、これでDonkeycar互換のロボットを作ることができます。ステアリングは片方のモーターを他方より速く回すことで行い、車体は弧を描いて走行します。モータードライバの配線方法には3ピン方式と2ピン方式の2通りがあります。差動駆動用のDonkeycarドライブトレイン名はすべて `DC_TWO_WHEEL` で始まります。

### 3ピン HBridge 差動駆動
2つのDCギアモーターを L298N で制御します。各モーターは方向選択用に2本のTTL出力ピン、速度制御用に1本のPWMピンを使用します。各モーターが3本使うため、差動駆動全体では合計6本のピンを使用します。この方式の利点は、必要なPWMピンが2本だけで済むことです。これはJetson Nano がサポートする最大PWM数と一致します。

L298N HBridge モジュールを3ピンモードでRaspberryPiのGPIOに配線する方法については https://www.electronicshub.org/raspberry-pi-l298n-interface-tutorial-control-dc-motor-l298n-raspberry-pi/ を参照してください。これはTB6612FNGなどL298N互換のドライバチップにも当てはまります。

**設定**

- myconfig.py で `DRIVETRAIN_TYPE = "DC_TWO_WHEEL_L298N"` を使用します。
  - 40ピンGPIOヘッダーから信号を生成する場合のピン指定例:

```python
DC_TWO_WHEEL_L298N = {
    "LEFT_FWD_PIN": "RPI_GPIO.BOARD.16",        # 左ホイール前進用TTL出力
    "LEFT_BWD_PIN": "RPI_GPIO.BOARD.18",        # 左ホイール後退用TTL出力
    "LEFT_EN_DUTY_PIN": "RPI_GPIO.BOARD.22",    # 左モーター速度用PWM出力

    "RIGHT_FWD_PIN": "RPI_GPIO.BOARD.15",       # 右ホイール前進用TTL出力
    "RIGHT_BWD_PIN": "RPI_GPIO.BOARD.13",       # 右ホイール後退用TTL出力
    "RIGHT_EN_DUTY_PIN": "RPI_GPIO.BOARD.11",   # 右モーター速度用PWM出力
}
```

  - PCA9685 を用いて信号を生成する場合のピン指定例:

```python
DC_TWO_WHEEL_L298N = {
    "LEFT_FWD_PIN": "PCA9685.1:40.3",        # 左ホイール前進用TTL出力
    "LEFT_BWD_PIN": "PCA9685.1:40.2",        # 左ホイール後退用TTL出力
    "LEFT_EN_DUTY_PIN": "PCA9685.1:40.1",    # 左モーター速度用PWM出力

    "RIGHT_FWD_PIN": "PCA9685.1:40.6",       # 右ホイール前進用TTL出力
    "RIGHT_BWD_PIN": "PCA9685.1:40.5",       # 右ホイール後退用TTL出力
    "RIGHT_EN_DUTY_PIN": "PCA9685.1:40.4",   # 右モーター速度用PWM出力
}
```

  - 設定では HBRIDGE_L298N_PIN_xxxx_EN ピンでモーターの回転速度を決定します。これらのピンはPWM出力に対応している必要があります。Jetson Nano では `/opt/nvidia/jetson-io/jetson-io.py` を使って有効化した場合のみPWM出力が2ピン利用可能です。[Jetson Nano からのPWM生成](pins.md#generating-pwm-from-the-jetson-nano) を参照してください。
  - HBRIDGE_L298N_PIN_xxxx_FWD および HBRIDGE_L298N_PIN_xxxx_BWD ピンはモーターの回転方向を指定するTTL出力ピンです。

> ピンプロバイダやピン指定子の詳細は [pins](pins.md) を参照してください。

### 2ピン HBridge 差動駆動
ミニ L293D HBridge を使用して2つのDCモーターを制御します。各モーターは2本のPWMピンを用い、1本が前進速度用、もう1本が後退速度用です。この配線方式の利点は合計4本のピンで済むことですが、使用するピンはすべてPWM出力に対応している必要があります。

- L293D ミニHBridgeモジュールを2ピンモードで配線する方法は[こちら](https://www.instructables.com/Tutorial-for-Dual-Channel-DC-Motor-Driver-Board-PW/)を参照してください。
- このドライバはL9110S/HG7881モータードライバでも利用できます。配線方法は[Interfacing L9110S](https://electropeak.com/learn/interfacing-l9110s-dual-channel-h-bridge-motor-driver-module-with-arduino/)を参照してください。
- DRV8833でも使用可能です。[DRV8833 HBridge](https://electropeak.com/learn/interfacing-drv8833-dual-motor-driver-module-with-arduino/)を参照してArduinoとの接続方法を確認してください。

**設定**

- myconfig.py で `DRIVETRAIN_TYPE = "DC_TWO_WHEEL"` を使用します。
  - 40ピンGPIOヘッダーから信号を生成する場合のピン指定例:

```python
DC_TWO_WHEEL = {
    "LEFT_FWD_DUTY_PIN": "RPI_GPIO.BCM.16",  # BCM.16 == BOARD.36 左前進用PWM
    "LEFT_BWD_DUTY_PIN": "RPI_GPIO.BCM.20",  # BCM.20 == BOARD.38 左後退用PWM
    "RIGHT_FWD_DUTY_PIN": "RPI_GPIO.BCM.5",  # BCM.5 == BOARD.29 右前進用PWM
    "RIGHT_BWD_DUTY_PIN": "RPI_GPIO.BCM.6",  # BCM.6 == BOARD.31 右後退用PWM
}
```
  - PCA9685 を用いて信号を生成する場合のピン指定例:

```python
DC_TWO_WHEEL = {
    "LEFT_FWD_DUTY_PIN": "PCA9685.1:40.0",  # 左前進用PWM
    "LEFT_BWD_DUTY_PIN": "PCA9685.1:40.1",  # 左後退用PWM
    "RIGHT_FWD_DUTY_PIN": "PCA9685.1:40.5",  # 右前進用PWM
    "RIGHT_BWD_DUTY_PIN": "PCA9685.1:40.6",  # 右後退用PWM
}
```

> ピンプロバイダやピン指定子の詳細は [pins](pins.md) を参照してください。
