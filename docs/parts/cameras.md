# カメラ

Donkeycar は `CAMERA_TYPE` 設定で多くのカメラをサポートしています。多くのアプリケーションでは広い視野が重要であり、120 度以上の広角レンズを使うべきです。160 度の広角レンズを推奨します。

## カメラのセットアップ

デフォルトのディープラーニングテンプレートまたはコンピュータビジョンテンプレートを使用する場合、カメラが必要です。標準では __myconfig.py__ は RaspberryPi カメラを想定しています。`~/mycar` フォルダーの __myconfig.py__ で `CAMERA_TYPE` の値を編集することで変更できます。

gps パスフォローテンプレートを使用する場合はカメラは必要なく、むしろ無いほうがよいかもしれません。この場合はカメラタイプをモックに変更できます。`CAMERA_TYPE = "MOCK"` と設定してください。

### Raspberry Pi：

Raspberry Pi を使用していて推奨される Pi カメラ（"PICAM"）を利用する場合、__myconfg.py__ を変更する必要はありません。

これはすべての Raspberry Pi カメラで動作します。5 メガピクセルの OV5647 チップセットを採用した初代 Raspberry Pi カメラモジュールや、Sony IMX219 チップを採用した Raspberry Pi Camera Module v2 も含まれます。これらのカメラは入手が容易で、多くのベンダーがクローン品を提供しています。

> **カメラが正常に動作することを確認してください**。[カメラの接続](https://www.raspberrypi.com/documentation/accessories/camera.html#connect-the-camera)の問題はよくあります。特に[新しい Donkeycar の組み立て](https://docs.donkeycar.com/guide/build_hardware/#step-6-attach-camera)後やカメラの取り付け直後、クラッシュ後に発生しがちです。そのような場合や Donkeycar ソフトウェア使用中にカメラエラーに遭遇した場合は、[Discord](https://discord.gg/PN6kFeA)で助けを求める前にカメラが正しく動作するか確認してください。Raspberry Pi OS には写真撮影や動画のストリームができる[カメラソフトウェア](https://www.raspberrypi.com/documentation/computers/camera_software.html)が含まれています。Raspberry Pi にキーボード、マウス、モニターを接続しているなら、[`rpicam-hello`](https://www.raspberrypi.com/documentation/computers/camera_software.html#rpicam-hello)ユーティリティでカメラの映像を表示できます。ssh で Raspberry Pi に接続している場合は、[`rpicam-jpeg`](https://www.raspberrypi.com/documentation/computers/camera_software.html#rpicam-jpeg)ユーティリティで画像を撮影して jpeg として保存し、それをホストコンピューターにコピーして確認してください（エラーなく撮影できれば通常カメラは問題ありません）。

### Jetson Nano：

Jetson には 5 メガピクセル OV5647 を採用した初代 Raspberry Pi カメラ用のドライバーはありませんが、IMX219 チップを採用した v2 カメラ用のドライバーはあります。実際、推奨されるカメラも IMX219 ベースです。

デフォルト設定の `CAMERA_TYPE = "PICAM"` は Jetson では**動作しません**。8MP の RaspberryPi カメラや Sony IMX219 をベースにしたカメラを使用している場合でも同様です。これらの場合は __myconfg.py__ を編集して `CAMERA_TYPE = "CSIC"` と設定してください。

カメラを上下逆に取り付ける必要がある場合は、`CSIC_CAM_GSTREAMER_FLIP_PARM = 6` と設定して画像を上下反転させます。

### USB カメラ

`CAMERA_TYPE = CVCAM` は USB カメラで動作するカメラタイプです。ただし OpenCV の設定が必要で、[Jetson Nano 用の OpenCV](/guide/robot_sbc/setup_jetson_nano/#step-4-install-opencv) または [Raspberry Pi 用の OpenCV](https://www.learnopencv.com/install-opencv-4-on-raspberry-pi/) の追加設定が求められます。

オプションの pygame ライブラリをインストールしていれば、カメラタイプを `CAMERA_TYPE = "WEBCAM"` に変更することでカメラに接続できます。必要な追加設定については[pygame のガイド](https://www.pygame.org/wiki/GettingStarted)を参照してください。

複数のカメラを使用する場合は `CAMERA_INDEX` 設定値を変更する必要があるかもしれません。デフォルトは 0 です。

>> 注意: `CAMERA_TYPE = CVCAM` は GStreamer サポートを組み込んだ OpenCV が必要です。Jetson ではこれがデフォルトであり、Raspberry Pi 用に推奨される OpenCV でもサポートされています。

### Intel Realsense D435

Intel Realsense カメラは RGBD カメラで、RGB 画像と深度を提供します。__myconfig.py__ で `CAMERA_TYPE = "D435"` と設定することで、Deep Learning テンプレートや Computer Vision テンプレートで RGB カメラとして使用できます。また、Intel Realsense カメラ固有の設定も確認してください。

```
# Intel Realsense D435 と D435i 深度センサーカメラ
REALSENSE_D435_RGB = True       # RGB 画像を取得する場合は True
REALSENSE_D435_DEPTH = True     # 深度を画像配列として取得する場合は True
REALSENSE_D435_IMU = False      # IMU データを取得する場合は True（D435i のみ）
REALSENSE_D435_ID = None        # カメラのシリアル番号。1 台だけの場合は None（自動検出）
```

深度を使用しない場合は `REALSENSE_D435_DEPTH = False` と設定し、深度データを保存しないようにします。

## トラブルシューティング

色がおかしく見える場合、カメラが RGB ではなく BGR で出力している可能性があります。`BGR2RGB = True` を設定すると BGR から RGB に変換できます。

今後も他のカメラを追加していく予定です。利用可能なオプションについては __myconfig.py__ のカメラセクションを参照してください。

カメラで問題が発生した場合は、[Discord のハードウェアチャンネル](https://discord.gg/zcyzK69S) をご覧ください。
