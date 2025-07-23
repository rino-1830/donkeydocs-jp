# 自分の車アプリケーションを作成する

まだ行っていない場合は、[車両にSSH接続してください](/guide/robot_sbc/setup_raspberry_pi/#step-5-connecting-to-the-pi)。

## テンプレートからDonkeycarを作成する

[createcar](/utility/donkey/#create-car) コマンドを使ってDonkeyを制御するための一式のファイルを作成します。

```bash
donkey createcar --path ~/mycar
```

これにより、デフォルトの [deep learning template](/guide/deep_learning/train_autopilot/) を使った車が作成されます。別途、[gps path follow template](/guide/path_follow/path_follow/) を使った車を作成することもできます。

```bash
donkey createcar --template=path_follow --path ~/mycar
```

さらに、[computer vision template](/guide/computer_vision/computer_vision/) を使った車を作成することもできます。

```bash
donkey createcar --template=cv_control --path ~/mycar
```

> `mycar` は特別な名前ではありません。車のフォルダー名は好きなものにして構いません。詳細は [createcar](/utility/donkey/#create-car) のドキュメントを参照してください。

## オプションを設定する

作成したディレクトリ ~/mycar 内の __myconfig.py__ を確認します。
```bash
cd ~/mycar
nano myconfig.py
```

各行にはコメント記号が付いています。コメント内の値はデフォルト値です。デフォルトを上書きしたい場合は、行頭の # とその前の空白を削除してコメントを解除します。

例：

```python
# STEERING_LEFT_PWM = 460
```

こうなります：

```python
STEERING_LEFT_PWM = 500
```

このように編集します。後ほど [calibrate](/guide/calibrate/) の章でこれらを調整します。

### I2C PCA9685 を設定する

PCA9685 サーボドライバボードを使用する場合、I2C 上で認識できることを確認してください。

**Jetson Nano**：

```bash
sudo usermod -aG i2c $USER
sudo reboot
```

再起動後、次を試します：

```bash
sudo i2cdetect -r -y 1
```

**Raspberry Pi**：

```bash
sudo apt-get install -y i2c-tools
sudo i2cdetect -y 1
```

すると、次のようなアドレスの一覧が表示されます：

```text
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: 40 -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: 70 -- -- -- -- -- -- --
```

この例では、PCA9685 ボードのアドレスとして 40 が表示されています。表示されない場合は配線を確認してください。Raspberry Pi では ```sudo raspi-config``` のメニューから I2C を有効にしておきます（再起動を促されます）。

ボードに標準とは異なるアドレスを割り当てた場合は、`myconfig.py` の `PCA9685_I2C_ADDR` 変数でそのアドレスを設定してください。別のバスに接続している場合は `PCA9685_I2C_BUSNUM` を指定できます。

**Jetson Nano**：__myconfig.py__ で ```PCA9685_I2C_BUSNUM = 1``` に設定します。Raspberry Pi では Adafruit のライブラリが自動検出しますが、Jetson Nano ではそうなりません。

## Sombrero の設定

Sombrero ボードを使用する場合は __myconfig.py__ で ```HAVE_SOMBRERO = True``` を設定します。

## Robo HAT MM1 の設定

Robo HAT MM1 ボードを使用する場合は __myconfig.py__ で ```HAVE_ROBOHAT = True``` を設定します。また、以下の変数を環境に合わせて設定してください。ほとんどの方は下記の値を使用しますが、Jetson Nano では `MM1_SERIAL_PORT = '/dev/ttyTHS1'` にしてください。


```python3
#ROBOHAT MM1
HAVE_ROBOHAT = True            # set to true when using the Robo HAT MM1 from Robotics Masters.  This will change to RC Control.
MM1_STEERING_MID = 1500         # Adjust this value if your car cannot run in a straight line
MM1_MAX_FORWARD = 2000          # Max throttle to go fowrward. The bigger the faster
MM1_STOPPED_PWM = 1500
MM1_MAX_REVERSE = 1000          # Max throttle to go reverse. The smaller the faster
MM1_SHOW_STEERING_VALUE = False
# Serial port 
# -- Default Pi: '/dev/ttyS0'
# -- Jetson Nano: '/dev/ttyTHS1'
# -- Google coral: '/dev/ttymxc0'
# -- Windows: 'COM3', Arduino: '/dev/ttyACM0'
# -- MacOS/Linux:please use 'ls /dev/tty.*' to find the correct serial port for mm1 
#  eg.'/dev/tty.usbmodemXXXXXX' and replace the port accordingly
MM1_SERIAL_PORT = '/dev/ttyS0'  # Serial Port for reading and sending MM1 data (raspberry pi default)

# adjust controller type as Robohat MM1
CONTROLLER_TYPE='MM1'
# adjust drive train for web interface
DRIVE_TRAIN_TYPE = 'MM1'
```

Robo HAT MM1 はトレーニング中の走行にRCコントローラーとCircuitPythonスクリプトを使用します。先へ進む前に、CircuitPython スクリプトをパソコンから Robo HAT MM1 へコピーしておく必要があります。

1.  [こちら](https://github.com/autorope/donkeycar/blob/dev/donkeycar/contrib/robohat/code.py) から Robo HAT MM1 用 CircuitPython Donkey Car Driver をパソコンにダウンロードします。
2.  Robo HAT MM1 のMicroUSBコネクターをパソコンのUSBポートに接続します。
3.  パソコンに USB ストレージデバイスとして __CIRCUITPY__ が表示されます。
4.  手順1でダウンロードしたファイルを __CIRCUITPY__ の USB ストレージデバイスにコピーします。ファイル名は __code.py__ とし、ドライブの最上位に配置してください。
5.  Adafruit のロギングライブラリ *adafruit_logging.py* を [ここ](https://github.com/adafruit/Adafruit_CircuitPython_Logging/blob/master/adafruit_logging.py) からダウンロードします。
6.  *adafruit_logging.py* を CIRCUITPY "lib" フォルダーにコピーします。
7.  Robo HAT MM1 のUSBケーブルを外し、他のHATと同様に Raspberry Pi の上に取り付けます。


Raspberry Pi のハードウェアシリアルポートを有効にする必要がある場合があります。Raspberry Pi 上で以下を実行します...

1.  `sudo raspi-config` を実行します。
2.  __5 - Interfaceing options__ セクションへ移動します。
3.  __P6 - Serial__ セクションへ移動します。
4.  `Would you like a login shell to be accessible over serial?` と聞かれたら `NO` を選択します。
5.  `Would you like the serial port hardware to be enabled?` と聞かれたら `YES` を選択します。
6.  raspi-config を終了します。
7.  再起動します。


Robo HAT MM1 に関するさらなるハードウェアやソフトウェアのサポートを知りたい場合、Hackster.io にいくつかのガイドが公開されています。以下に示します。

[Raspberry Pi + Robo HAT MM1](https://www.hackster.io/wallarug/autonomous-cars-with-robo-hat-mm1-8d0e65)

[Jetson Nano + Robo HAT MM1](https://www.hackster.io/wallarug/donkey-car-with-jetson-nano-robo-hat-mm1-e53e21)

[Simulator + Robo HAT MM1](https://www.hackster.io/wallarug/donkey-car-simulator-with-real-rc-controller-628e77)


## ジョイスティックの設定

ジョイスティックを使用する場合は、[こちら](/parts/controllers/#joystick-controller) を参照してください。

## カメラの設定

デフォルトの deep learning テンプレートまたは computer vision テンプレートを使用する場合、カメラが必要です。__myconfig.py__ ではデフォルトで Raspberry Pi カメラを想定しています。`~/mycar` フォルダーの __myconfig.py__ にある `CAMERA_TYPE` の値を編集して変更できます。

gps path follow テンプレートを使用する場合、カメラは不要、むしろ無いほうがよいこともあります。この場合はカメラタイプを `CAMERA_TYPE = "MOCK"` に変更します。

[Cameras](/parts/cameras) で各種カメラと設定の詳細を確認してください。

## Donkey Car ソフトウェアのアップグレード

__myconfig.py__ に行った設定変更は、アップデートしても保持されます。変更を取得したい場合は、最新のコードを取得して次のコマンドを実行します。

```bash
cd projects/donkeycar
git pull
donkey createcar --path ~/mycar --overwrite
```

gps path follow テンプレートで車を作成した場合は、--template 引数を付けるのを忘れないでください。

```bash
donkey createcar --template=path_follow --path ~/mycar --overwrite
```


この操作により ~/mycar/manage.py、~/mycar/config.py などのファイルは更新されますが、__myconfig.py__ は変更されません。__data__ フォルダーと __models__ フォルダーも変更されません。

-------

## 次へ
[車をキャリブレーションする](/guide/calibrate)
