# Nvidia Jetson NanoでTensorRTを使用するためのガイド

----

* **注意** このガイドはUbuntu `18.04`を使用していることを前提としています。もしWindowsを使用している場合は[こちらの](https://docs.nvidia.com/deeplearning/sdk/tensorrt-install-guide/index.html)手順を参照してTensorRTを使用するための設定を行ってください。

----

## ステップ1：UbuntuマシンでTensorRTを設定する

[こちら](https://docs.nvidia.com/deeplearning/sdk/tensorrt-install-guide/index.html#installing-tar)の手順に従ってください。以前に`.deb`ファイルでCUDAをインストールしているのでなければ、`tar`ファイルの手順を必ず使用してください。

## ステップ2：Jetson NanoにTensorRTを設定する

* `nvcc`が`$PATH`上にあるよう環境変数を設定します。次の行を`~/.bashrc`ファイルに追加してください。

```bash
# これを .bashrc に追加してください
export CUDA_HOME=/usr/local/cuda
# CUDAコンパイラをPATHに追加
export PATH=$CUDA_HOME/bin:$PATH
# ライブラリを追加
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
```

* `.bashrc`の変更をテストします。

```bash
source ~/.bashrc
nvcc --version
```

次のような表示が得られるはずです：

```text
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on ...
Cuda compilation tools, release 10.0, Vxxxxx
```

* `virtualenv`に切り替えてPyCUDAをインストールします。

```bash
# これは少し時間がかかります
pip install pycuda
```

* その後、`dist-packages`が`virtualenv`の一部として読み込まれるよう、`PYTHONPATH`を設定する必要があります。以下を`.bashrc`に追加してください。これは`tensorrt`のPythonバインディングが`dist-packages`にあり、このフォルダは通常`virtualenv`からは見えないため、`PYTHONPATH`に追加して見えるようにします。

```bash
export PYTHONPATH=/usr/lib/python3.6/dist-packages:$PYTHONPATH
```

* `virtualenv`に切り替えて`tensorrt`をインポートすることでこの変更をテストします。

```python
> import tensorrt as trt
>  # このインポートが成功するはずです
```

## ステップ3：モデルを学習し、凍結してTensorRT形式（`uff`）にエクスポートする

`linear`モデルを学習すると、`.h5`拡張子のファイルが生成されます。

```bash
# モデルフォルダにLinear.h5が生成されます
python manage.py train --model=./models/Linear.h5 --tub=./data/tub_1_19-06-29,...

# （オプション）デスクトップPCからJetson Nanoの作業ディレクトリ(~mycar/models/)に'./models/Linear.h5'をコピーします

# donkeycar/scriptsにあるfreeze_model.pyを使用してモデルを凍結します；凍結されたモデルはプロトコルバッファとして保存されます。
# このコマンドはモデルのメタデータもエクスポートし、./models/Linear.metadataに保存します
python ~/projects/donkeycar/scripts/freeze_model.py --model=~/mycar/models/Linear.h5 --output=~/mycar/models/Linear.pb

# 凍結したモデルをUFFに変換します。以下のコマンドで./models/Linear.uffファイルが作成されます
cd /usr/lib/python3.6/dist-packages/uff/bin/
python convert_to_uff.py ~/mycar/models/Linear.pb
```

変換した`uff`モデルと`metadata`をJetson Nanoにコピーします。

## ステップ4

* `myconfig.py`でモデルタイプを`tensorrt_linear`に設定します。

```python
DEFAULT_MODEL_TYPE = `tensorrt_linear`
```

* 最後に以下を実行します

```bash
# `uff`モデルをNanoにscpした後
python manage.py drive --model=./models/Linear.uff
```