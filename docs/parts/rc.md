# RC制御
![Donkey RC connections](../assets/parts/rchat.png)
（これはRaspberryPiでのみ動作します。Jetson Nanoでは必要なGPIOピンがサポートされていません）

あなたのRCカーに付属しているであろうRCコントローラーだけでDonkeyを操作できます。Pigpioライブラリのおかげで、RaspberryPiのピンはRC受信機からの信号を読み取り、サーボやモーターコントローラーを駆動するRC信号を生成できます。

これを行うには、RC受信機からRPiのGPIOピンへジャンパー線を接続し、同様にステアリングサーボやモーターコントローラーにも接続します（少し面倒ですが問題なく動作します）。もしくは、プラグアンドプレイで使える[Donkeycar RC Hat](../parts/rc_hat.md)（上図参照）を利用します。これはOLEDスクリーン、ファン、エンコーダーサポート、さらに3チャンネル以上の送信機があればe-stop（緊急停止スイッチ）機能も備えています。


## ハードウェア設定

**上記のRC Hatを使用する場合、このハードウェア設定は不要です。Hatがすべて処理してくれます。**

Donkeycarを操作する前に、RCコントローラーのトリムをしっかり調整しておくと良いでしょう。スロットルトリム、ステアリングトリム、ステアリングの可動範囲を適切に合わせておいてください。調整方法は[こちらの動画](https://www.youtube.com/watch?v=NuVQz7FCAZk)が参考になります。

GPIOピンはRC入力・RC出力のどちらにも、または両方に使用できます。RC入力の場合はBluetoothジョイスティックの代わりとなり、RC出力の場合はI2Cサーボドライバーボードの代わりとして機能します。

最も簡単な接続方法は、上記で紹介した専用の"hat"を使うことです。自分で配線する場合は、以下の配線ガイドに従ってください。入力と出力の両方を行う場合はジャンパー線が多くなりますが、各チャンネルごとではなく、RC受信機には1本だけグラウンドとV+を接続すればよいことを覚えておきましょう。

また、RC受信機は3.3Vピンに接続し、サーボやモーターコントローラーは5Vピンに接続する点に注意してください。

**警告:** RC受信機のPWM信号は受信機の入力電圧から生成されます。したがって、RC受信機をESCからの5Vや6Vに接続するとRPiが故障します。

![Donkey RC connections](../assets/rc.png)

以下はRC受信機を接続したときのイメージです

![Donkey RC connections](../assets/rc.jpg)

## ソフトウェア設定

まず、PIGPIOがインストールされていることを確認してください。[pins](pins.md#pigpio)を参照してください。PIGPIOデーモンをRaspberryPi起動時に自動で開始させたい場合、以下のコマンドで設定します。

```bash
sudo systemctl enable pigpiod & sudo systemctl start pigpiod
```

続いて、`mycar`ディレクトリ内の`myconfig.py`を次のように編集します。

* RC入力を使用する場合、`myconfig.py`のコントローラータイプを`pigpio_rc`に設定します。コメントを外し、次のように記述します。

```python
CONTROLLER_TYPE = 'pigpio_rc'
```

さらに`use joystick`をTrueに設定します。

```python
USE_JOYSTICK_AS_DEFAULT = True
```

* RC出力を使用する場合、`myconfig.py`の駆動方式を`PWM_STEERING_THROTTLE`に設定します。コメントを外し、次のように記述します。

```python
DRIVE_TRAIN_TYPE =  "PWM_STEERING_THROTTLE"
```

いずれの場合も、出力方向を反転させたり接続ピンを変更したりする追加設定が可能です。

入力の設定例:
 
```python
#PIGPIOによるRC制御
STEERING_RC_GPIO = 26
THROTTLE_RC_GPIO = 20
DATA_WIPER_RC_GPIO = 19
PIGPIO_STEERING_MID = 1500         # 真っ直ぐ走らない場合はこの値を調整
PIGPIO_MAX_FORWARD = 2000          # 前進時の最大スロットル値。大きいほど速くなる
PIGPIO_STOPPED_PWM = 1500
PIGPIO_MAX_REVERSE = 1000          # 後退時の最大スロットル値。小さいほど速くなる
PIGPIO_SHOW_STEERING_VALUE = False
PIGPIO_INVERT = False
PIGPIO_JITTER = 0.025   # この値以下の変化は無視する
```

RC Hatを使う場合、以下に示すPWM出力ピン（myconfig.pyのデフォルト値）を必ず使用してください。
RC Hatを使用しない場合は、他のPWM出力ピンを選択しても構いません。
※この設定を使用するにはpigpioのインストールが必要です。[PIGPIO](pins.md#pigpio)を参照してください。

出力の設定例:

```python
PWM_STEERING_PIN = "PIGPIO.BCM.13"           # ステアリングサーボ用PWM出力ピン
PWM_THROTTLE_PIN = "PIGPIO.BCM.18"           # ESC用PWM出力ピン

STEERING_LEFT_PWM = int(4096 * 1 / 20)       # 左一杯のPWM値（1msパルス）
STEERING_RIGHT_PWM = int(4096 * 2 / 20)      # 右一杯のPWM値（2msパルス）

THROTTLE_FORWARD_PWM = int(4096 * 2 / 20)    # 最大前進時のPWM値（2msパルス）
THROTTLE_STOPPED_PWM = int(4096 * 1.5 / 20)  # 停止時のPWM値（1.5msパルス）
THROTTLE_REVERSE_PWM = int(4096 * 1 / 20)    # 最大後退時のPWM値（1msパルス）
```

## トラブルシューティング

ステアリングを左に切ると右に動くなど、チャンネルが逆転してしまう場合は、RC送信機側で該当チャンネルをリバース設定にするか、上記の出力設定でPWM_INVERTEDを`True`に変更してください。
