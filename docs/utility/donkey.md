# Donkey コマンドラインユーティリティ

`donkey` コマンドは donkeycar の Python パッケージをインストールすると利用できるようになります。これは重要な機能を追加する Python スクリプトで、車両固有のものではなく、どのようなハードウェア構成でも動作します。

## Create Car

このコマンドは、ロボットを動かしたり学習させたりするために必要なファイルを格納する新しいディレクトリを作成します。

Usage:

```bash
donkey createcar --path <dir> [--overwrite] [--template <donkey2>]
```

* このコマンドはどのディレクトリからでも実行できます。
* ホストコンピュータ上でもロボット上でも実行可能です。
* `--path` で指定した場所にディレクトリを作成します。既に `.py` ファイルが存在する場合は上書きしませんが、`--overwrite` オプションを付ければ上書きします。
* `--overwrite` を付けると、`myconfig.py` 以外のファイルを目的のディレクトリに上書きします。donkeycar をアップデートした際に変更を `mycar` フォルダへ反映させたいが、`myconfig.py` は再生成したくない場合に便利です。
* オプションの `--template` で開始時に使用するテンプレートファイルを指定できます。利用可能なテンプレートは `donkeycar/templates` ディレクトリを参照してください。このテンプレートはユーザーの `manage.py` としてコピーされます。よく使われるテンプレートは以下のとおりです：
    - `--template=complete`: the [Deep Learning Autopilot](/guide/train_autopilot/#deep-learning-autopilot)
    - `--template=path_follow`: the [Path Follow Autopilot](/guide/train_autopilot/#path-follow-autopilot)
    - `--template=cv_control`: the [Computer Vision Autopilot](/guide/train_autopilot/#computer-vision-autopilot)

## Find Car

このコマンドは nmap を使用してローカルネットワーク上の車両を探します。

Usage:

```bash
donkey findcar
```

* ホストコンピュータ上で実行します。
* ホストコンピュータの IP アドレスおよび見つかった場合は車両の IP アドレスを表示します。
* 実行には nmap ユーティリティが必要です：

```bash
sudo apt install nmap
```

## Calibrate Car

このコマンドでは、PWM 値を手動で入力しながらインタラクティブにロボットの反応を確認できます。
詳しくは [こちら](/guide/calibrate/) も参照してください。

Usage:

```bash
donkey calibrate --channel <0-15 channel id>
```

* ホストコンピュータ上で実行します。
* `--channel` で指定した PWM チャンネルを開きます。
* 整数値を入力して PWM 値を指定し、Enter キーを押します。
* 終了するには `Ctrl + C` を押します。

## Clean data in Tub

Tub から不要なデータを削除するための Web サーバーを開きます。

Usage:

```bash
donkey tubclean <folder containing tubs>
```

* Raspberry Pi でもホストコンピュータでも実行できます。
* 不良データを削除するための Web サーバーを開きます。
* `Ctrl + C` で終了します。

## Train the model
**注:** _このセクションはバージョン 4.1 以降にのみ適用されます_
このコマンドはモデルを学習させます。詳細は [Deep Learning Autopilot](/guide/deep_learning/train_autopilot/#train-a-model) を参照してください。

```bash
donkey train --tub=<tub_path> [--config=<config.py>] [--model=<model path>] [--type=(linear|categorical|inferred)] [--transfer=<transfer model path>]
```
* `--tub` で指定したデータストアのデータを使用します。複数の Tub を使う場合は `--tub=foo/data,bar/data` のようにカンマ区切りで指定するか、`--tub foo/data bar/data` とスペースで区切って指定できます。
* `--config` で指定した設定ファイルを使用します（省略可能）。
* `--model` で指定したパスにモデルを保存します。省略した場合は自動的にモデル名が生成されます。_**注意:**_ バージョン 4.2 では `--model mypilot.h5` のようにモデル名だけを指定できる回帰がありましたが、4.2.1 で修正されています。最新版を使用してください。
* `--type` でモデルタイプを指定します。
* `--transfer` を使うと既存モデルを引き継いで学習を継続できます。
* `myconfig.py` の `TRAIN_FILTER` 変数に定義した関数でレコードをフィルタリングできます。例：

```bash
def filter_record(record):
    return record.underlying['user/throttle'] > 0

TRAIN_FILTER = filter_record
```
  
  学習にはスロットル値が正のレコードのみを使用します。
 
* バージョン 4.3.0 以降ではすべての 3.x モデルが再びサポートされました：

```bash
donkey train --tub=<tub_path> [--config=<config.py>] [--model=<model path>] [--type=(linear|categorical|inferred|rnn|imu|behavior|localizer|3d)] [--transfer=<transfer model path>]
```

さらに、トレーニング時には自動で Tflite モデルが生成されます。これを抑制したい場合は設定で `CREATE_TF_LITE = False` とします。また Tensorrt モデルも生成可能で、`CREATE_TENSOR_RT = True` を設定します。

* 注意：`createcar` コマンドは後方互換のため `train.py` ファイルを生成しますが、学習には必須ではありません。


## Make Movie from Tub

このコマンドは Tub 内の画像から動画を作成します。

Usage:

```bash
donkey makemovie --tub=<tub_path> [--out=<tub_movie.mp4>] [--config=<config.py>] [--model=<model path>] [--model_type=(linear|categorical|inferred|rnn|imu|behavior|localizer|3d)] [--start=0] [--end=-1] [--scale=2] [--salient]
```

* ホストコンピュータまたはロボット上で実行します。
* 指定した `--tub` ディレクトリの画像レコードを使用します。
* `--out` で指定した名前の動画を作成します。コーデックは拡張子から自動判断され、デフォルトは `tub_movie.mp4` です。
* `config.py` 以外の設定ファイルを使用したい場合は `--config` で指定します。
* `--model` を指定すると Keras モデルを読み込み、予測値を線として動画に重ねます。
* `--model_type` で読み込むモデルタイプを指定できます。省略時は categorical です。
* `--salient` を付けるとニューラルネットワークが特に反応したピクセルを可視化して重ねます。
* `--start` と `--end` を指定すると使用するフレーム番号の範囲を限定できます。
* `--scale` を指定すると出力画像をその倍率で拡大します。


## Plot Predictions

このコマンドは、学習済みモデルの予測と実際のステアリングおよびスロットルを比較するグラフを表示します。

Usage:

```bash
donkey tubplot --tub=<tub_path> --model=<model_path> [--limit=<end_index>] [--type=<model_type>]
```

* `~/mycar` ディレクトリから実行できます。
* ホストコンピュータ上で実行します。
* 指定した tub のステアリング値と学習済みモデルによる予測値を比較したグラフをポップアップ表示します。
* `--limit=<end_index>` を指定すると、そのインデックスまでのレコードのみ使用します。デフォルトは 1000。
* `--type=<model_type>` を指定すると `DEFAULT_MODEL_TYPE` とは異なるモデルタイプを使用できます。


## Tub Histogram

**_注_**: バージョン 4.3 以上が必要です。

このコマンドは tub のデータ（通常はステアリングとスロットル）をヒストグラムとして表示します。

Usage:

```bash
donkey tubhist --tub=<tub_path> --record=<record_name> --out=<output_filename>
```

* `~/mycar` ディレクトリから実行できます。
* ホストコンピュータ上で実行します。
* 指定した tub のデータをヒストグラムとしてポップアップ表示します。
* `--record=<record_name>` を指定すると特定のデータ系列のみを表示します。例: "user/throttle"
* `--out=<output_filename>` を指定するとヒストグラムをその名前で保存し、省略時は tub パスから自動生成されます。


## Joystick Wizard

このコマンドラインウィザードでは、独自のコントローラーを作成する手順を案内します。

Usage:

```bash
donkey createjs
```

* `~/mycar` ディレクトリから実行します。
* まず OS がデバイスへアクセスできることを確認します。`jstest` ユーティリティが便利です。`sudo apt install joystick` でインストールし、コントローラーのデバイスパスを指定して実行します。通常は `/dev/input/js0` ですが異なる場合は正しいパスを調べてください。`createjs` コマンドでも同じパスを指定します。
* `donkey createjs` を実行すると `~/mycar` フォルダに manage.py と並んで `my_joystick.py` が作成されます。
* myconfig.py の `CONTROLLER_TYPE="custom"` を設定して `my_joystick.py` を使用します。

## Visualize CNN filter activations

指定した画像に対してモデル内の各畳み込み層のフィルターごとの特徴マップを表示します。特徴抽出の状態を確認するためのデバッグ用ツールです。

Usage:

```bash
donkey cnnactivations [--tub=<data_path>] [--model=<path to model>]
```

実行するとモデル内の各 `Conv2d` 層ごとにウィンドウが開きます。

Example:

```bash
donkey cnnactivations --model models/model.h5 --image data/tub/1_cam-image_array_.jpg
```

## Show Models database

**_注:_** この機能は donkeycar 4.3.1 以降でのみ利用できます。

`models/database.json` に保存されているモデル一覧を表示します。モデルタイプ、モデル名、学習に使った Tub、転移学習元のモデル、学習時に `--comment` を付けた場合や UI で学習した場合のコメントなどの情報を表示します。


Usage:

```bash
donkey models [--group]
```

* `~/mycar` ディレクトリで実行します。
* `--group` フラグを付けると、モデルごとに使用した tub の組み合わせが異なる場合でも、同じ組み合わせをひとまとめにして表示します。複数の tub を使っていてモデルごとに組み合わせが異なる場合に情報を整理できます。
* 車上でこのコマンドを実行する場合は事前に `pandas` をインストールする必要があります。


## Donkey UI

**注:** _このセクションはバージョン 4.2.0 以降にのみ適用されます_


Usage:

```bash
donkey ui
```

このコマンドで tub データを解析するための UI が起動し、以下の機能を利用できます。

* 選択したデータフィールドの値をリアルタイムに表示し、バーグラフでも確認できます。
* レコードの削除や復元ができます。
* データ選択用のフィルターを試すことができます。
* 選択したデータフィールドのグラフを描画します。

この UI は Web ベースの `donkey tubclean` の代替となります。

![Tub UI](../assets/ui-tub-manager.png)

UI の詳細なドキュメントは[こちら](./ui.md)にあります。
