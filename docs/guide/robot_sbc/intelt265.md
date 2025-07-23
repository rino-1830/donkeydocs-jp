# Intel Realsense T265 センサーを Donkeycar で利用するためのガイド

----

* **注意** Realsense T265 は Jetson Nano でも利用できますが、Raspberry Pi（RPi4、4GB以上推奨）の方がセットアップが簡単です。Realsense D4XX シリーズもカメラとして使用でき、深度データ対応手順は後日公開予定です。

## Original T265 path follower code by [Tawn Kramer](https://github.com/tawnkramer/donkey)

* 手順1: Donkeycar をセットアップ

* 手順2: Ubuntu マシンで Librealsense をセットアップ。
  Raspbian Buster を使用している場合は [こちらの手順](https://github.com/IntelRealSense/librealsense/blob/master/doc/installation_raspbian.md) に従って Librealsense と依存パッケージをインストールします。

* 手順3: Jetson Nano で TensorRT をセットアップ

準備ができたら次のコマンドでディレクトリを作成します。

```bash
donkey createcar --path ~/follow --template path_follower
```

```bash
cd ~/follow
python3 manage.py drive
```

起動したらノート PC のブラウザで `http://<Nano の IP アドレス>:8887` にアクセスします。

以下は Tawn 氏のリポジトリにある追加手順です。

走行すると赤線で経路が、緑の円でロボット位置が表示されます。
開始位置を決め、毎回そこからスタートします。
車を周回させると赤線で経路が表示されます。
PS3/4 コントローラーの X ボタンで経路を保存します。
車をスタート地点に戻します。
PS3 では select、PS4 では share ボタンを2回押すとパイロットモードになります。速度を変えたい場合は `myconfig.py` の `THROTTLE_FORWARD_PWM = 530` を変更してください。
`myconfig.py` の末尾には PID 値やオフセット、スケールなど調整用設定があります。私のリポジトリからサンプル `myconfig.py` を入手して利用するのがおすすめです。
### いくつかのヒント

起動すると緑のドットは画面左上に表示されます。中央にしたい場合は `myconfig.py` の `PATH_OFFSET = (0, 0)` を `PATH_OFFSET = (250, 250)` に変更します。

コースが小さいと経路が見づらい場合があります。その際は `PATH_SCALE = 5.0` を `PATH_SCALE = 10.0`（必要に応じてさらに大きく）に変更してください。

赤線が表示されない場合は既に経路ファイルが作成されています。`donkey_path.pkl` を削除すると表示されます。

デフォルトでは 0.3m ごとに経路ポイントを記録します。より滑らかにしたい場合は `myconfig.py` の `PATH_MIN_DIST = 0.3` を小さな値に変更してください。
