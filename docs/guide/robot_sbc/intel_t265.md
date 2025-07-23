# Intel Realsense T265 センサーを Donkeycar で利用するためのガイド

----

* **注意** Realsense T265 は Jetson Nano でも利用できますが、Raspberry Pi（最低 4GB メモリの RPi4 推奨）の方がセットアップが容易です。Realsense D4XX シリーズもカメラとして使用でき、深度データへの対応は今後追加予定です。

T265 用パスフォロープログラムは [Tawn Kramer](https://github.com/tawnkramer/donkey) 氏の提供です。
----


## 手順1: Ubuntu マシンで Librealsense をセットアップ

Raspberry Pi（Raspbian Buster で確認済み）を利用する場合は、[こちらの手順](https://github.com/IntelRealSense/librealsense/blob/master/doc/installation_raspbian.md) に従い Librealsense と依存パッケージをインストールします。

## 手順2: Donkeycar をセットアップ

セットアップ手順は [こちら](https://docs.donkeycar.com/guide/install_software/) を参照してください。

## 手順3: Donkeycar パスフォローアプリを実行

ここまで終わったら、次のコマンドでディレクトリを準備します。

```bash
donkey createcar --path ~/follow --template path_follower

cd ~/follow
python3 manage.py drive
```

起動後、ノート PC のブラウザで `http://<Nano の IP アドレス>:8890` にアクセスします。

以下は Tawn 氏のリポジトリに記載されている追加手順です。

走行すると経路が赤線、ロボット位置が緑の円で表示されます。

1) 開始位置を決め、毎回そこへ戻してスタートします。
2) 車を周回させると赤い線で経路が表示されます。
3) PS3/4 コントローラーの X ボタンで経路を保存します。
4) 車をスタート地点に戻します。
5) PS3 は select、PS4 は share ボタンを2回押すとパイロットモードになります。速度調整は `myconfig.py` の `THROTTLE_FORWARD_PWM = 530` を変更してください。

`myconfig.py` の末尾には PID 値やオフセット、スケールなど調整用の設定があります。まずは私のリポジトリから `myconfig.py` を取得して利用すると良いでしょう。
### いくつかのヒント

起動すると緑のドットは画面左上に表示されます。中心に移動したい場合は `myconfig.py` の `PATH_OFFSET = (0, 0)` を `PATH_OFFSET = (250, 250)` に変更します。

コースが小さいと経路が見づらいことがあります。その場合は `PATH_SCALE = 5.0` を `PATH_SCALE = 10.0`（必要ならさらに大きく）に変更してください。

赤線が表示されない場合は、既に経路ファイルが存在している可能性があります。`donkey_path.pkl` を削除すると表示されるはずです。

自動運転モードでは緑のドットが青に変わります。

デフォルトでは 0.3 メートルごとに経路ポイントを記録します。より滑らかにしたい場合は `myconfig.py` の `PATH_MIN_DIST = 0.3` を小さな値に変更してください。