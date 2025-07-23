# ピン指定子

制御信号はRaspberry Pi、Jetson NanoおよびPCA9685サーボコントローラのような周辺機器のピンを介して送受信されます。バージョン5.xからDonkeycarでは、基盤ハードウェアやライブラリ実装固有の設定を含めてピンを指定する「ピン指定子」を使用します。これにより、モーターコントローラがスロットル値を取得してモーターへ出力するなどの内部ロジックを、信号生成に利用するハードウェアやライブラリに依存せずに構築できます。

## ピンの種類

PWMピンはPWMパルスとも呼ばれる矩形波を生成し、サーボモーターや電子スピードコントローラー、LEDの制御に使用されます。

TTL出力ピンは高(1)または低(0)の値を出力できます。

TTL入力ピンは高(1)または低(0)の値を読み取ります。

## ピンプロバイダー
Donkeycarではピンを指定するために複数の技術を利用できます。ピンは、プロバイダー名、ピン番号、技術固有の設定を表す文字列として指定します。

#### PCA9685
PCA9685サーボコントローラは16個のPWMおよびTTL出力ピンを備えています。PCA9685は出力専用で入力ピンはサポートしていません。PCA9685のピン指定子には次の情報を含めます。

- PCA9685が接続されているI2Cバス
- I2Cバス上でのPCA9685の16進アドレス
- チャネル番号（0〜15）

例えば `"PCA9685.1:40.13"` はI2Cバス1のアドレス0x40にあるPCA9685のチャネル13を指定します。

また `"PCA9685.0:60.1"` はI2Cバス0のアドレス0x60にあるPCA9685のチャネル1を指定します。

#### RPI_GPIO
DonkeycarではRaspberryPiに標準でRPi.GPIOライブラリがインストールされます。Jetson Nanoには互換ライブラリであるJetson.GPIOが既定でインストールされています。これらのライブラリはいずれも同様に動作し、RaspberryPiやJetson Nanoの40ピンGPIOバスでPWM、入力、出力ピンを扱えます。ピン指定子には次の情報を含めます。

- ピンのアドレス指定方式
  - "BOARD" はRaspberryPiやJetson Nanoの基板に印字されている0〜39の番号に基づく方式を示します。
  - "BCM" はRaspberryPiで採用されているBroadcom GPIO番号方式を示します。Jetson側ではこの方式がエミュレーションされているため、Jetsonでも"BCM"番号を使用できます。
- ピン番号（アドレス指定方式によって異なります）

RaspberryPiの40ピンヘッダーの詳細はこちらを参照してください:  https://www.raspberrypi.com/documentation/computers/os.html#gpio-and-the-40-pin-header

Jetson Nanoの40ピンヘッダーも同じ基板番号方式を採用していますが、物理的に裏返しに配置されているため、基板上に印字された番号を確認してください。Jetson NanoはPWMピンを2本しかサポートしておらず、これらは有効化が必要です。詳細は[Jetson NanoでのPWM生成](#generating-pwm-from-the-jetson-nano)を参照してください。

例えば `"RPI_GPIO.BOARD.33"` はRPi.GPIOライブラリを利用して基板番号33のピンを指定します。

また `"RPI_GPIO.BCM.13"` はRPi.GPIOライブラリでBroadcom GPIO-13を指定します。上記のヘッダ図を見ると分かるように、これは物理的には`"RPI_GPIO.BOARD.33"`と同じピンであり、基板番号33の別名です。

RPI_GPIOプロバイダーを使用する場合、BOARD方式とBCM方式のどちらかを選択できますが、すべてのピンで同一の方式を使う必要があります。方式を混在させることはできません。

#### PIGPIO
RaspberryPiユーザーは、40ピンGPIOヘッダーのピンを制御するためにPiGPIOライブラリとデーモンを任意でインストールできます。ただし、このライブラリはJetson Nanoでは動作しません。PiGPIOはPWM、入力、出力ピンをサポートします。

##### PiGPIOのインストールと起動
- システムデーモンのインストール
```bash
sudo apt-get update
sudo apt-get install pigpio
```
- Pythonサポートのインストール（donkey環境を有効にした状態で）
```bash
pip install pigpio
```
- デーモンの起動
```bash
sudo systemctl start pigpiod
```
- 起動時にデーモンを有効化
```
sudo systemctl enable pigpiod
```

PIGPIOのピン指定子には以下が含まれます。
- "BCM" — PiGPIOはBroadcom(BCM)方式のピン番号のみを使用するため、この方式が指定子に組み込まれています。
- BCMピン番号

例えば `"PIGPIO.BCM.13"` はBroadcom GPIO-13を指定します。前述およびリンク先のヘッダ図のとおり、これは基板ピン33に対応しています。


## Jetson NanoでPWMを生成する

Jetson NanoとRaspberryPi4の双方ともハードウェアPWMピンを2本備えています。Jetson Nanoではこれらを設定する必要があります。


#### Jetson拡張ヘッダーでPWMを有効化

- donkeycarへsshして次のコマンドを実行します: `sudo /opt/nvidia/jetson-io/jetson-io.py`。GPIOピンの機能を変更できるJetson Expansion Header Toolが表示されます（下図参照）。

- Jetsonの拡張ヘッダー設定にPWMピンが表示されていない場合は、有効化する必要があります。


```
---
     =================== Jetson Expansion Header Tool ===================
     |                                                                    |
     |                                                                    |
     |                        3.3V ( 1)  ( 2) 5V                          |
     |                        i2c2 ( 3)  ( 4) 5V                          |
     |                        i2c2 ( 5)  ( 6) GND                         |
     |                      unused ( 7)  ( 8) uartb                       |
     |                         GND ( 9)  (10) uartb                       |
     |                      unused (11)  (12) unused                      |
     |                      unused (13)  (14) GND                         |
     |                      unused (15)  (16) unused                      |
     |                        3.3V (17)  (18) unused                      |
     |                      unused (19)  (20) GND                         |
     |                      unused (21)  (22) unused                      |
     |                      unused (23)  (24) unused                      |
     |                         GND (25)  (26) unused                      |
     |                        i2c1 (27)  (28) i2c1                        |
     |                      unused (29)  (30) GND                         |
     |                      unused (31)  (32) unused                      |
     |                      unused (33)  (34) GND                         |
     |                      unused (35)  (36) unused                      |
     |                      unused (37)  (38) unused                      |
     |                         GND (39)  (40) unused                      |
     |                                                                    |
      ====================================================================
---
```


`Configure the 40 pin expansion header` を選択して pwm0 と pwm2 を有効化します。


```
---
     =================== Jetson Expansion Header Tool ===================
     |                                                                    |
     |                                                                    |
     |                        3.3V ( 1)  ( 2) 5V                          |
     |                        i2c2 ( 3)  ( 4) 5V                          |
     |                        i2c2 ( 5)  ( 6) GND                         |
     |                      unused ( 7)  ( 8) uartb                       |
     |                         GND ( 9)  (10) uartb                       |
     |                      unused (11)  (12) unused                      |
     |                      unused (13)  (14) GND                         |
     |                      unused (15)  (16) unused                      |
     |                        3.3V (17)  (18) unused                      |
     |                      unused (19)  (20) GND                         |
     |                      unused (21)  (22) unused                      |
     |                      unused (23)  (24) unused                      |
     |                         GND (25)  (26) unused                      |
     |                        i2c1 (27)  (28) i2c1                        |
     |                      unused (29)  (30) GND                         |
     |                      unused (31)  (32) pwm0                        |
     |                        pwm2 (33)  (34) GND                         |
     |                      unused (35)  (36) unused                      |
     |                      unused (37)  (38) unused                      |
     |                         GND (39)  (40) unused                      |
     |                                                                    |
      ====================================================================
---
```


有効化後、pwm0 は基板ピン32、pwm2 は基板ピン33となります。



