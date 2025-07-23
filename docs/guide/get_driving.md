# 車を運転する

車を[キャリブレーション](/guide/calibrate)したら、運転を始められます。

まだ接続していない場合は、[車両にssh接続](/guide/robot_sbc/setup_raspberry_pi/#step-5-connecting-to-the-pi)してください。

## 車を起動する

> *** 車のタイヤが地面に触れない安全な場所に置いてください ***。

この操作中に車が急発進する可能性があります。

車のフォルダーを開いて起動します。
```
cd ~/mycar
python manage.py drive
```

このスクリプトは車のドライブループを開始し、
その中にはウェブサーバーが含まれます。これにより、
ウェブブラウザーから `<your car's hostname.local>:8887` にアクセスして操作できます。

![drive UI](../assets/web_controller.png)

## ウェブコントローラーで運転する

ウェブコントローラーで車を動かす方法は3つあります。
- **デバイス傾き**:
**Control Mode** セクションで **Device Tilt** を選択し、**Mode** セクションでは **User** を選択します。スマートフォンを前に傾けるとスロットルが上がり、左右に傾けるとハンドルを切れます。
- **ジョイスティック**:
ウェブコントローラーの **Control Mode** セクションで **Joystick** を選び、**Mode** セクションで **User** を選択します。表示される仮想ジョイスティックをドラッグすることで操作できます。上に動かすとスロットルが上がり、下に動かすと減速または後退します。左に動かすと左折、右に動かすと右折します。指を離すと停止します。
- **ゲームパッド**:
ウェブUIを表示しているマシンにゲームコントローラー（ケーブルまたはBluetooth接続）があり、HTML5 Gamepad APIに対応していれば、ウェブコントローラーの **Control Mode** セクションで **Gamepad** を選択できます。その後 **Mode** セクションで **User** を選択し、ゲームコントローラーでDonkeycarを操作できます。

### 機能

* Recording - 「record data」を押すと、画像、ステアリング角度、スロットル値の記録を開始します。
* Throttle mode - スロットルを一定にするオプションです。
レースでパイロットが舵を取るがスロットルを操作しない場合に使用します。
* Pilot mode - パイロットが角度やスロットルを制御する場合に選択します。
* Max throttle - 最高スロットルを設定します。

### キーボードショートカット

* `space` : 車を停止し、記録を停止
* `r` : 記録を切り替え
* `i` : スロットルを上げる
* `k` : スロットルを下げる
* `j` : 左折する
* `l` : 右折する

-----

ジョイスティックを持っていない場合は、次のセクション [自動運転の学習](/guide/train_autopilot/) に進んでください。

-----

## 物理ジョイスティックで運転する

物理的なジョイスティックを使うと車両を操作しやすい場合があります。

### Bluetoothを設定してジョイスティックをペアリングする

Bluetooth接続の設定については[Controllers](/parts/controllers/#physical-joystick-controller)セクションを参照してください。

### 車を起動する

```bash
cd ~/mycar
python manage.py drive --js
```

ジョイスティックの使用を常に有効にし、毎回 --js を付けたくない場合は、`__myconfig.py__` の __USE_JOYSTICK_AS_DEFAULT = True__ を変更します。

```bash
nano myconfig.py
```

### ジョイスティックの操作

* 左アナログスティック - 左右でステアリングを調整
* 右アナログスティック - 前方に倒すと前進スロットルを上げる
* 右アナログスティックを2回後ろに引くと後進

> スロットルがゼロでない限り、Userモードであれば走行データが記録されます。

* Selectボタン - モードを切り替え「User, Local Angle, Local(angle and throttle)」
* Triangle - 最大スロットルを上げる
* X - 最大スロットルを下げる
* Circle - 記録のオン／オフ（デフォルトでは無効。スロットルによる自動記録はデフォルトで有効）
* dpad up - スロットルスケールを増やす
* dpad down - スロットルスケールを減らす
* dpad left - ステアリングスケールを増やす
* dpad right - ステアリングスケールを減らす
* Start - 定速スロットルのオン／オフ。最大スロットルに設定（XとTriangleで調整）。

-----

### 次は自動運転を学習させましょう。
- [ディープラーニング自動運転を学習させる](/guide/deep_learning/train_autopilot)
- [GPSパスフォロー自動運転を学習させる](/guide/path_follow/path_follow)
- [コンピュータビジョン自動運転を構築する](/guide/computer_vision/computer_vision/)
