# Donkey UI

`donkey ui` と入力するとグラフィカルな学習インターフェースが起動します。Linux、Mac、Windows で動作しますが、Windows では Ubuntu 20 を実行する WSL（[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install)）を使用することを推奨します。

Donkey UI には現在、次のワークフローを支援する 4 つの画面があります。

1. **Tub Manager** - `donkey tubclean` で起動する Web アプリの代替となる画面です。

1. **Trainer** - パイロットを学習させるための UI です。大きな Tub やバッチで長時間の学習を行う場合は、シェルから `donkey train` を実行することを推奨します。UI での学習は実験的かつ素早い分析サイクルを目的としています。
     * data manipulation / selection
     * training
     * pilot benchmarking

1. **Pilot Arena** - 2 つのパイロットを比較し、性能を測定できます。
1. **Car Connector** - 車両から Tub データを取得したり、学習済みパイロットを送り返したり、さらには車両の起動や停止も行えます。この画面は Windows では動作しません。

**注意:** Linux ではアプリが `xclip` に依存しています。インストールされていない場合は次を実行してください。
```bash
sudo apt-get install xclip
```

## The tub manager
![Tub_manager UI](../assets/ui-tub-manager.png)

Tub Manager ではまず `Load car directory` ボタンで `myconfig.py` を含む車両ディレクトリを指定します。続いて `Load tub` で作業したい Tub を選択します（Tub は車両ディレクトリ内にある必要があります）。アプリケーションは最後に読み込んだ設定と Tub を記憶します。

画像左側のデータパネルにある `Add/remove` ドロップダウンで `user/angle` や `user/throttle` などのレコードフィールドを選択できます。

**注意:** Tub に `user/angle` や `user/throttle` 以外のデータが含まれていてプログレスバーに正しい値を表示したい場合、ホームディレクトリにある `.donkeyrc` ファイルへエントリーを追加する必要があります。このファイルは Donkey UI が自動で作成します。例を示します。
```yaml
field_mapping:
- centered: true
  field: car/accel
  max_value_id: IMU_ACCEL_NORM
```

この `field_mapping` への登録では、Tub のフィールド名、値が 0 を中心としているかどうか、そして `myconfig.py` に定義するそのデータの最大値の名前を指定します。上記の例では IMU6050 の加速度データを表しており、範囲は ±2g（約 ±20 m/s<sup>2</sup>）です。`IMU_ACCEL_NORM` を 20 に設定することでプログレスバーが値を正しく表示できます。そのため `myconfig.py` には次のように記述します。
```python
IMU_ACCEL_NORM = 20
```

**注:** ベクトル（リストや配列）は UI が自動的に要素に分解します。

例えば `car/accel` と `car/gyro` という IMU データの配列に加え、`car/distance` や `car/m_in_lap` を含む Tub では、前者 2 つは `field_mapping` にエントリーがあるためプログレスバーが表示されます。
![Tub_manager UI_more_data](../assets/ui-tub-manager-2.png)

コントロールパネルでは <, > で一コマずつ、<<, >> で連続的に前後へ移動できます。これらのボタンはキーボードの < left >, < right >, < space > にも対応しています。

不要なレコードを削除するには `Set left` と `Set right` で範囲を指定してから `Delete` を押します。削除結果を確認するには `Reload tub` を押します。誤って削除したレコードを復活させたい場合は、削除範囲外の left/right 値を指定して `Restore` を押します。

**注意:** left と right の値は入れ替えて指定することもでき、left > right の場合は [left, right) 以外のすべてのレコードが対象になります。

フィルターセクションではレコードを絞り込めます。たとえば右カーブのみで次の学習を行いたい場合は `user/angle > 0` と入力して該当レコードを選択します。
**注意:** このフィルターは Tub Manager 上での表示にのみ適用されます。実際の学習に適用するには [utility](../utility/donkey.md) で説明されているように述語を記述してください。

下部のパネルにはデータパネルで選択した項目のグラフが表示されます。何も選択されていない場合はレコード内のすべてのフィールドが表示されます。各フィールドの最小値から最大値まででスケーリングされるため、絶対値での測定はできません。より高度なグラフ表示を行いたい場合は `Browser Graph` を押すとブラウザで plotly の履歴グラフが開きます。

## The trainer
![Trainer UI](../assets/ui-trainer.png)

Trainer 画面では Tub データを使ったモデルの学習ができます。`Overwrite config` セクションで右側のテキストフィールドに値を入力しリターンを押すと任意の設定パラメータを上書きできます。

パイロットを学習するにはモデルタイプを選択し、コメントを入力して `Train` を押します。学習が終わるとパイロットは下部のデータベースに表示されます。ゼロから学習したくない場合は転移学習用のモデルも選択できます。tensorflow はオプティマイザの状態も保存するため、保存した地点から学習が再開されます。

パイロットを複数の Tub で学習することもありますが、Trainer では現状対応していません。`donkey train` で複数の Tub を指定した場合でもデータベースには表示されます。画面を見やすく整理するためには `Group multiple tubs` ボタンを押して、2 つ以上の Tub グループをひとまとめにし、代わりにグループ名を表示できます。グループ名の対応表はウィンドウ下部に表示されます。

## The pilot arena
![Pilot Arena UI](../assets/ui-pilot-arena.png)

ここでは 2 つのパイロットを比較してベンチマークできます。オプティマイザのパラメータやモデルタイプの変更、特定レコードの削除、画像の拡張によってパイロットが良くなったか悪くなったかを確認するのに利用します。最後に選択したパイロットはアプリに記憶されます。

`Model type` を選択し `Choose pilot` で Keras モデルを読み込んでパイロットを選びます。コントロールパネルは Tub Manager と同じです。右下のデータパネルには Tub レコードのデータが表示されます。車速で学習している場合などはスロットルではなく速度のフィールドを選択できるよう `.donkeyrc` の該当セクションにフィールド名を追加します。例を以下に示します。

```yaml
user_pilot_map:
  car/speed: pilot/speed
  user/angle: pilot/angle
  user/throttle: pilot/throttle
```

`user/angle` と `user/throttle` のマッピングはアプリが自動で読み込みます。`car/speed` を表示して AI が生成した `pilot/speed` と比較したい場合は、マップに対応するエントリーを追加する必要があります。

2 つのパイロットの下には、あらかじめ定義された画像拡張（Brightness と Blur）のスライダーがあります。明るさやぼかしを画像に加えて、テストデータの変更に対してパイロットがどの程度対応できるかを比較できます。ボタンを押してスライダーを有効にしてください。

アプリケーションは最後に選択した 2 つのパイロットを記憶します。

## The car connection
![Car_Connector_UI](../assets/ui-car-connector-1.png)

**注:** この画面はバックグラウンドで `ssh` と `rsync` を使用するため Linux/OSX でのみ動作します。PC から車両へパスワードなしでログインできるよう SSH を設定する必要があります。PC で次のコマンドを実行してください。
```bash
ssh-keygen
```
パスフレーズを尋ねられたらそのまま Enter を押します。`./ssh` ディレクトリに公開鍵と秘密鍵が作成されます。次のコマンドで鍵を車両へコピーします。ここでは車両のホスト名を `donkeypi` としていますが、適宜置き換えてください。
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub pi@donkeypi.local
```
Log into your car using:
```bash
ssh pi@donkeypi.local
```
初回接続時にそのホストを known_hosts に追加するか聞かれたら Enter を押してください。これ以降はパスワードを求められずに ssh 接続できます。**このパスワード不要の設定がこの画面の動作には必須です。**

* `myconfig.py` を編集し、`PI_USERNAME` と `PI_HOSTNAME` が車両のユーザー名とホスト名になっていることを確認してください。

Car Connector を使えば車両から PC へ Tub データを転送し、学習済みパイロットを車両へ送り返すことができます。

* `Car directory` に車両のフォルダーを入力してリターンを押します。`Select tub` のドロップダウンが更新されます。通常は `data/` ディレクトリを選びますが、サブフォルダーに Tub がある場合は `Car directory` に `~/mycar/data` を入力し、取得したい Tub を選択して `Create new folder` を有効にします。これにより `~/mycar/data/tub_21-04-09_11` のような Tub が PC 側の同じ場所にコピーされます。`Create new folder` を有効にしない場合、車両の Tub フォルダー内の内容が PC の `~/mycar/data` にコピーされ、既存データが上書きされる可能性があります。

* `Pull tub data/` を押すと車両から Tub をコピーします。
* `Send pilots` を押すとローカルの `models/` フォルダーと車両側の `models/` フォルダーを同期します。ローカルにあるすべてのパイロットが同期されます。
* `Drive car` セクションでは車両を起動し、自動運転用のモデルを選択できます。起動後は通常通りウェブやジョイスティックのコントローラーを使用してください。



## Future plans
1. Car Connector 画面をすべての OS で使える Web インターフェースへ移行する。
1. 複数 Tub の取り扱いに対応する。
1. `myconfig.py` を編集せずにトレーニングでフィルターを利用できるようにする。
1. `~/.donkeyrc` ファイルを Kivy の内部設定へ移行する。
1. パイロットアリーナで 1 体のみ（または 2 体以上）でも利用できるようにする。

## Video tutorial
UI のビデオチュートリアルは以下から視聴できます。

[![Video tutorial](https://img.youtube.com/vi/J5-zHNeNebQ/0.jpg
)](https://www.youtube.com/watch?v=J5-zHNeNebQ)


