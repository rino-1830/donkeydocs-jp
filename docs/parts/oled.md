# OLEDディスプレイ

OLEDディスプレイは車の現在の状態に関する情報を表示するために利用できます。これは特に学習用データを収集するときやレース時に便利です。

OLEDディスプレイには現在、次の情報が表示されます。
* 車のIPアドレス（`eth` と `wlan`）
* 学習用に収集したレコード数
* 走行モード

## 対応ディスプレイ

現在サポートされているディスプレイの例：

* [Adafruit PiOLED - 128X32モノクロOLED](https://www.adafruit.com/product/3527)

## ハードウェア設定

ディスプレイを Raspberry Pi または Jetson Nano の I2C ピンに接続するだけで使用できます。`bus 1` を使うとピンに直接差し込めます。例としては[こちら](https://cdn-shop.adafruit.com/1200x900/3527-04.jpg)をご覧ください。

## ソフトウェア設定

`myconfig.py` の `USE_SSD1306_128_32 = False` 行の先頭にある `#` を削除して `False` を `True` に変更するとディスプレイを有効にできます。128×32 の OLED を使う場合は解像度 1、128×64 を使う場合は解像度 2 を選択し、その行の `#` も忘れずに外してください。

`myconfig.py` の該当部分は次のようになります。

```python
USE_SSD1306_128_32 = True    # Enable the SSD_1306 OLED Display
# SSD1306_128_32_I2C_ROTATION = 0 # 0 = text is right-side up, 1 = rotated 90 degrees clockwise, 2 = 180 degrees (flipped), 3 = 270 degrees
SSD1306_RESOLUTION = 2 # 1 = 128x32; 2 = 128x64
```
## 起動時にIPアドレスを表示する

OLEDスクリーンがあると、起動時に車のIPアドレスを表示して接続しやすくできます。設定手順は[こちら](https://diyrobocars.com/2021/12/29/show-your-raspberrypi-ip-address-on-startup-with-an-oled/)を参照してください。

## トラブルシューティング

車が起動できない場合は、仮想環境に `Adafruit_SSD1306` パッケージがインストールされていることを確認してください。最近の `donkeycar` を使用していれば自動的にインストールされているはずです。

```bash
pip install Adafruit_SSD1306
```
## 既知の問題
- `Adafruit_SSD1306` ライブラリは、`RPI_GPIO` ピンプロバイダーを使用してDuty Cycle/PWMをGPIOヘッダーから直接供給するステアリングやモーター構成と互換性がありません。これはAdafruitライブラリ内部で設定されるGPIOピンモードが当プロジェクトのGPIOライブラリと合わないためです。この場合、次のいずれかを選択できます。
  - 必要なDuty Cycle/PWMを生成するためにPCA9685を使用する。
  - `PIGPIO` ピンプロバイダーを利用してGPIOからスロットルとステアリングのDuty Cycle/PWMを生成する。設定方法は[こちら](pins.md#PIGPIO)を参照。
