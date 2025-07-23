# Donkeycar RCハット
![RaspberryPi用RCハット](../assets/PXL_20250527_030121313-600x338.jpg "RaspberryPi用Donkey RCハット")

走行準備が整ったRCカーを購入した場合、たいていRC送信機が付属しています。朗報として、その送信機を使ってDonkeycarで手動走行が可能です。またPCA9685モーター／サーボ制御基板を用いなくても、車両のサーボとモーターコントローラーを直接Raspberry Piに接続できます。

RCハットを使用する前に送信機のトリムを調整しておきましょう。スロットルトリム、ステアリングトリム、ステアリング範囲を適切に合わせてください。調整方法は[こちらの動画](https://www.youtube.com/watch?v=NuVQz7FCAZk)を参照してください。

配線する方法は、[このチュートリアル](rc.md)で説明されているように手作業で行うこともできますが、多くの細い配線が外れやすくなります。上記のDonkeycar RCハットを使用すれば、OLEDスクリーンも搭載されており、配線をすべてきれいにまとめられます。

![RaspberryPi用RCハット](../assets/PXL_20250527_030450600-600x509.jpg)

Donkeycar RCハットは[Donkeycarストア](https://www.diyrobocars.com/shop/)で購入できます。

標準的な[ホイールエンコーダ](odometry.md)を使う場合は「Encoder」ピンに接続してください。また「Optional 5v power in」ピンから5Vを入力すれば、この基板からRaspberry Piに給電することも可能です。

すべてのケーブルを接続したらソフトウェアの設定に進みます。設定方法は[こちら](https://www.diyrobocars.com/2024/12/22/using-the-rc-hat/)を参照してください。


### トラブルシューティング

車が起動しない場合は、仮想環境に`Adafruit_SSD1306`パッケージがインストールされているか確認してください。新しいバージョンの`donkeycar`を使用していれば自動的にインストールされているはずです。

```bash
pip install Adafruit_SSD1306
```

## エンコーダ
標準的な[ホイールエンコーダ](odometry.md)を使う場合は「Encoder」ピンに接続し、続いて`myconfig.py`でRCハットのエンコーダ用ヘッダーが公開しているピンを使用するよう設定してください。
