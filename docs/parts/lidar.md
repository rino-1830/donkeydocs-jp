# Lidar

LiDARセンサーはDonkeycarで障害物回避を行ったり、壁のあるトラックでの走行を補助したりするために利用できます。トレーニング中はカメラと同時にデータを記録し、これを学習に活用できます。

![Donkey lidar](../assets/lidar.jpg)
## サポートされているLiDAR

現在はRPLidarシリーズのセンサーのみをサポートしていますが、近いうちに同様のYDLidarシリーズへの対応を追加する予定です。

[$99 A1M8](https://amzn.to/3vCabyN)（射程12m）を推奨します。


## ハードウェアセットアップ

上図のようにカメラキャノピーの下にLiDARを取り付けます（そこではRPLidar A2M8を使用していますが、A1M8の取り付けも同じです）。USBアダプタはDonkeyプレートの下にベルクロで固定し、短いUSBケーブルを使ってRPiまたはNanoのUSBポートに接続します。USBポートから給電できるため追加の電源は不要です。

## ソフトウェアセットアップ

LiDARを使用するには`glob`ライブラリをインストールする必要があります。まだ入っていない場合は `pip3 install glob2` でインストールしてください。

また、LiDARドライバを `pip install Adafruit_CircuitPython_RPLIDAR` でインストールしてください。


次に`lidarcar`ディレクトリに移動し、`myconfig.py`ファイルを編集してLiDARが有効になっていることを確認します。上限角度と下限角度は、車体で遮られる領域を除外しつつLiDARに"見て"ほしい範囲を反映するように設定します。以下は例です。RPLidarシリーズでは、0度がA1M8ではモーター方向、A2M8ではケーブル方向を指します。

```
# LIDAR
USE_LIDAR = True
LIDAR_TYPE = 'RP' #(RP|YD)
LIDAR_LOWER_LIMIT = 90 # 記録される角度。この値を使って車体で遮られた領域を除外したり後方を見るのを避けたりします。RP A1M8 Lidarでは「0」がモーター方向になります。
LIDAR_UPPER_LIMIT = 270
```
![Lidar limits](../assets/lidar_angle.png)

## テンプレートサポート
 [deep learning template](/guide/train_autopilot/#deep-earning-autopilot) と [path follow template](/guide/path_follow/path_follow/) は現在LiDARデータを直接サポートしていません。 [deep learning templateへのLiDARデータ追加](https://github.com/autorope/donkeycar/issues/910) の課題があります。 LiDARは[ path follow template](/guide/path_follow/path_follow/)での障害物検知と回避にも非常に有用です。 このようなプロジェクトに興味がある場合は、ぜひ [Discordコミュニティ](https://www.donkeycar.com/community.html) に参加してお知らせください。喜んでサポートいたします。



