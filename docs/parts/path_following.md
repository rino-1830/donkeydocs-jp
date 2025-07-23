
# Intel Realsense T265 センサーによる経路追従

通常のカメラを使って運転用ネットワークを学習させる代わりに、Donkeycar では [Intel Realsense T265 "tracking camera"](https://www.intelrealsense.com/tracking-camera-t265/) を利用して経路を追従させることができます。この仕組みでは、一度手動で経路を走らせるだけで Donkeycar がその経路を「記憶」し、自律的に繰り返して走行します。

Intel T265 は、ステレオカメラと内蔵の慣性計測装置（IMU）、そして独自の Myriad X プロセッサを組み合わせて Visual Inertial Odometry を行います。これは周囲の風景を見ながら移動し、IMU の情報と照合することで自身の位置を特定し、GPS センサーのように X、Y、Z 座標を Donkeycar へ出力します（ただし理想的にはより高精度で、センチメートル単位まで測位します）。

---------------
* **注意** Realsense T265 は Nvidia Jetson Nano でも利用できますが、Raspberry Pi（最低でも 4GB メモリの RPi 4 を推奨）で設定する方が少し簡単です。また、Intel Realsense D4XX シリーズも通常のカメラとして Donkeycar で使えます（深度センサーのデータ利用は近日対応予定）。準備ができ次第、手順を追加します。

T265 の経路追従コードは [Tawn Kramer](https://github.com/tawnkramer/donkey) によるものです。
----


## ステップ 1: Ubuntu マシンで Librealsense をセットアップする

Raspberry Pi 上で最新の Raspian（Raspian Buster で確認済み）を使用し、Intel の Realsense ライブラリ（Librealsense）とその依存関係をセットアップするには [こちらの手順](https://github.com/acrobotic/Ai_Demos_RPi/wiki/Raspberry-Pi-4-and-Intel-RealSense-D435) を参照してください。これらの手順は別の Realsense センサーを扱っていますが、T265 でも同様に利用できます。また [ビデオによる解説](https://www.youtube.com/watch?v=LBIBUntnxp8) もあります。

## ステップ 2: Donkeycar のセットアップ

標準的な手順は [こちら](https://docs.donkeycar.com/guide/install_software/) を参照してください。この Path Follower 構成では Tensorflow のインストールは不要ですが、"pip install -e .[pi]" を実行する前に numpy をインストール／アップグレードしておいてください。

## ステップ 3: Donkeycar の Path Follower アプリを作成する

```donkey createcar --path ~/follow --template path_follow

## ステップ 4: 設定を確認・変更する

```cd ~follow```
```sudo nano myconfig.py```

デフォルト値を確認し、必要に応じて好みに合わせて調整してください（「throttle」「steering」や PID など）。変更した行はコメントアウト記号「#」を外して有効にします。Nano では Ctrl+O で保存、Ctrl+X で終了します。

## ステップ 5: Donkeycar Path Follower アプリを実行する

実行
``ssh pi@<your pi’s IP address or "raspberrypi.local">``
``` cd ~/follow```
```python3 manage.py drive```

アプリが動作している間、端末は開いたままにして出力を確認してください。

T265 が見つからないというエラーが出た場合は、一度センサーを抜いて差し直してから再度試してください。ゲームパッドが電源オンで接続されていること（コントローラーの青いランプが点灯していること）も確認してください。

起動したら、ノート PC のブラウザで次の URL を入力します: http://<your pi’s IP address or "raspberrypi.local">:8890

走行中、Web インターフェースには経路を示す赤線とロボット位置を示す緑の円が描かれます。緑の点だけで赤線が表示されない場合は、すでにパスファイルが作成されているということです。「donkey_path.pkl」を削除（rm donkey_path.pkl）し、再起動すると赤線が表示されるはずです。


PS4 ゲームパッドの操作は次の通りです：
+------------------+--------------------------+
|     操作        |          動作          |
+------------------+--------------------------+
|      share       | toggle auto/manual mode  |
|      circle      |        save_path         |
|     triangle     |        erase_path        |
|      cross       |      emergency_stop      |
|        L1        |  increase_max_throttle   |
|        R1        |  decrease_max_throttle   |
|     options      | toggle_constant_throttle |
|      square      |       reset_origin       |
|        L2        |        dec_pid_d         |
|        R2        |        inc_pid_d         |
| left_stick_horz  |       set_steering       |
| right_stick_vert |       set_throttle       |
+------------------+--------------------------+

## ステップ 6: 走行の手順

1) ロボットのスタート地点を決め、毎回必ずそこに戻してから開始してください。
2) 車を一周させるように走らせます。赤い線で経路が表示されます。
3) PS3/4 コントローラの circle ボタンを押して経路を保存します。
4) ロボットをスタート地点に戻します。
5) その後、PS3 コントローラなら「select」、PS4 コントローラなら「share」を 2 回押してパイロットモードに切り替えます。これで経路を走り始めます。速度を調整したい場合は、myconfig.py の ```THROTTLE_FORWARD_PWM = 400``` を変更してください。

myconfig.py の末尾には PID 値やマップオフセット、スケールなど調整できる設定があります。私のリポジトリにある myconfig.py をダウンロードして使うと、既に実績のある設定が入っているので出発点として便利です。

いくつかのヒント：

起動すると、緑の点はボックスの左上に表示されます。中央にしたい場合は、myconfig.py の PATH_OFFSET = (0, 0) を PATH_OFFSET = (250, 250) に変更してください。

小さいコースでは経路が見づらい場合があります。そのときは PATH_SCALE = 5.0 を PATH_SCALE = 10.0（必要に応じてそれ以上）に変更します。

自動モードで走行中は、緑の点が青に変わります。

デフォルトでは 0.3 メートルごとに経路ポイントを記録します。より滑らかにしたい場合は、myconfig.py の PATH_MIN_DIST = 0.3 を小さい値に変更してください。
