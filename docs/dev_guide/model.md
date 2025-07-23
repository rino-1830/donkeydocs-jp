# 独自モデルの構築方法

---
 **注意:** _この手順にはバージョン >= 4.1.X が必要です_

---

* [概要](model.md#overview)
* [コンストラクター](model.md#constructor)
* [学習インターフェース](model.md#training-interface)
* [パーツインターフェース](model.md#parts-interface)
* [例](model.md#example)

## 概要

以下のような理由で独自モデルを作成したい場合があります。

* Donkey に付属するモデルでは不足しており、自分のモデル基盤で実験したい場合
* 車にセンサーが多く、より多くの入力データをモデルに追加したい場合

## コンストラクター

モデルは `donkeycar/parts/keras.py` にあります。独自モデルは `KerasPilot` を継承し、次のように初期化します。

```python
class KerasSensors(KerasPilot):
    def __init__(self, input_shape=(120, 160, 3), num_sensors=2):
        super().__init__()
        self.num_sensors = num_sensors
        self.model = self.create_model(input_shape)
```
ここではメンバー関数 `create_model()` 内で[keras のモデル](https://www.tensorflow.org/guide/keras/sequential_model)を実装します。学習を正常に行うためには、入力テンソルと出力テンソルに名前を付けておく必要があります。


## 学習インターフェース

モデルを機能させるためには、次の関数が必要です。

```python
def compile(self):
    self.model.compile(optimizer=self.optimizer, metrics=['accuracy'],
                       loss={'angle_out': 'categorical_crossentropy',
                             'throttle_out': 'categorical_crossentropy'},
                       loss_weights={'angle_out': 0.5, 'throttle_out': 0.5})
```

`compile` 関数は学習時の損失関数を定義する方法を keras に伝えます。ここでは `KerasCategorical` モデルを例にしています。この損失関数ではモデルの出力テンソル（`angle_out`, `throttle_out`）を明示的に利用しています。

```python
def x_transform(self, record: TubRecord):
    img_arr = record.image(cached=True)
    return img_arr
```

この関数では、記録されたデータから入力データを抽出する方法を定義します。このデータは機械学習では一般に `X` と呼ばれます。画像のみを入力とするモデルであれば、基底クラスの実装が利用できます。

モデルが入力を一つだけ持つ場合、この関数は単一のデータ項目を返します。複数の入力を利用するモデルではタプルを返す必要があります。


**注意:** _入力が複数ある場合、タプルの先頭には画像を置く必要があります_

```python
def y_transform(self, record: TubRecord):
    angle: float = record.underlying['user/angle']
    throttle: float = record.underlying['user/throttle']
    return angle, throttle
```
この関数では、記録データから `y` 値（ターゲット値）を取り出す方法を定義します。


```python
def x_translate(self, x: XY) -> Dict[str, Union[float, np.ndarray]]:
    return {'img_in': x}
```
ここでは上で抽出した `X` の値を `tf.data` に渡す形に変換する必要があります。`tf.data` は入力が複数ある場合は辞書を期待するため、一貫性を保つために入力が1つだけの場合も辞書を使います。画像のみを入力にするモデルであれば基底クラスの実装がそのまま使用でき、`x_transform` と `x_translate` をオーバーライドする必要はありません。


**注意:** _辞書のキーはモデルの**入力**レイヤー名と一致していなければなりません_

```python
def y_translate(self, y: XY) -> Dict[str, Union[float, np.ndarray]]:
    if isinstance(y, tuple):
        angle, throttle = y
        return {'angle_out': angle, 'throttle_out': throttle}
    else:
        raise TypeError('Expected tuple')
```
同様に、`y` データを `tf.data` 用の辞書に変換します。この例は `KerasLinear` の実装です。


**注意:** _辞書のキーはモデルの**出力**レイヤー名と一致していなければなりません_

```python
def output_shapes(self):
    # need to cut off None from [None, 120, 160, 3] tensor shape
    img_shape = self.get_input_shape()[1:]
    shapes = ({'img_in': tf.TensorShape(img_shape)},
              {'angle_out': tf.TensorShape([15]),
               'throttle_out': tf.TensorShape([20])})
    return shapes
```
この関数は、モデルで使用されるテンソル形状を TensorFlow に伝える 2 つの辞書からなるタプルを返します。ここでは `KerasCategorical` モデルの例を示しています。


**注意 1:** _上記の通り、2 つの辞書のキーはモデルの**入力**および**出力**レイヤー名と一致していなければなりません_


**注意 2:** _モデルがスカラー値を返す場合、その型は `tf.TensorShape([])` とする必要があります_


## Parts interface

車のアプリケーションでは、`run()` 関数を通じてモデルが呼び出されます。この関数は基底クラスで提供されており、入力画像の正規化処理も中央で行われます。派生クラスでは正規化済みデータを扱う `inference()` を実装します。追加のデータも正規化が必要な場合は、`run()` をオーバーライドすることも考慮してください。
```python
def inference(self, img_arr, other_arr):
    img_arr = img_arr.reshape((1,) + img_arr.shape)
    outputs = self.model.predict(img_arr)
    steering = outputs[0]
    throttle = outputs[1]
    return steering[0][0], throttle[0][0]
```
ここでは線形モデルの実装例を示しています。入力テンソルの形状には常にバッチ次元が先頭に含まれるため、入力画像の形状は `(120, 160, 3)` から `(1, 120, 160, 3)` へと調整されています。


**注意:** _`other_arr` 変数で別の配列を渡す場合も同様の形状変換が必要です_


## 例
標準的な線形モデルを基に、入力データとネットワーク設計に以下の変更を加えた新しい Donkey モデルを作成してみましょう。

1. 車の前部に取り付けた距離センサーから取得した値を表すベクトルを追加の入力として受け取る
   
2. ビジョンシステムの CNN 層と距離センサーのデータを組み合わせるために、いくつかのフィードフォワード層を追加する

### keras を用いたモデルの構築
以下に例となるモデルを示します。
```python
class KerasSensors(KerasPilot):
    def __init__(self, input_shape=(120, 160, 3), num_sensors=2):
        super().__init__()
        self.num_sensors = num_sensors
        self.model = self.create_model(input_shape)

    def create_model(self, input_shape):
        drop = 0.2
        img_in = Input(shape=input_shape, name='img_in')
        x = core_cnn_layers(img_in, drop)
        x = Dense(100, activation='relu', name='dense_1')(x)
        x = Dropout(drop)(x)
        x = Dense(50, activation='relu', name='dense_2')(x)
        x = Dropout(drop)(x)
        # ここまでは標準的な線形モデル。ここからセンサーのデータを追加する
        sensor_in = Input(shape=(self.num_sensors, ), name='sensor_in')
        y = sensor_in
        z = concatenate([x, y])
        # ここでさらに 2 層の Dense を追加
        z = Dense(50, activation='relu', name='dense_3')(z)
        z = Dropout(drop)(z)
        z = Dense(50, activation='relu', name='dense_4')(z)
        z = Dropout(drop)(z)
        # 角度とスロットルの 2 つの出力
        outputs = [
            Dense(1, activation='linear', name='n_outputs' + str(i))(z)
            for i in range(2)]

        # 追加の入力をモデルに指定する
        model = Model(inputs=[img_in, sensor_in], outputs=outputs)
        return model

    def compile(self):
        self.model.compile(optimizer=self.optimizer, loss='mse')

    def inference(self, img_arr, other_arr):
        img_arr = img_arr.reshape((1,) + img_arr.shape)
        sens_arr = other_arr.reshape((1,) + other_arr.shape)
        outputs = self.model.predict([img_arr, sens_arr])
        steering = outputs[0]
        throttle = outputs[1]
        return steering[0][0], throttle[0][0]

    def x_transform(self, record: TubRecord) -> XY:
        img_arr = super().x_transform(record)
        # 簡単のため、ここではセンサーのデータが正規化済みであると仮定する
        sensor_arr = np.array(record.underlying['sensor'])
        # 画像データを先に返す必要がある
        return img_arr, sensor_arr

    def x_translate(self, x: XY) -> Dict[str, Union[float, np.ndarray]]:
        assert isinstance(x, tuple), 'Requires tuple as input'
        # キーはモデルの入力レイヤー名
        return {'img_in': x[0], 'sensor_in': x[1]}

    def y_transform(self, record: TubRecord):
        angle: float = record.underlying['user/angle']
        throttle: float = record.underlying['user/throttle']
        return angle, throttle

    def y_translate(self, y: XY) -> Dict[str, Union[float, np.ndarray]]:
        if isinstance(y, tuple):
            angle, throttle = y
            # キーはモデルの出力レイヤー名
            return {'n_outputs0': angle, 'n_outputs1': throttle}
        else:
            raise TypeError('Expected tuple')

    def output_shapes(self):
        # [None, 120, 160, 3] の None を取り除く必要がある
        img_shape = self.get_input_shape()[1:]
        # キーはモデルの入出力レイヤーに合わせる必要がある
        shapes = ({'img_in': tf.TensorShape(img_shape),
                   'sensor_in': tf.TensorShape([self.num_sensors])},
                  {'n_outputs0': tf.TensorShape([]),
                   'n_outputs1': tf.TensorShape([])})
        return shapes
```
`KerasLinear` を継承すれば `y_transform()`, `y_translate()`, `compile()` の実装は
すでに提供されていますが、一般的なケースを明示するためここではすべての関数を実装しています。
モデルでは、センサーデータが TubRecord の `"sensor"` というキーの配列として存在することが求められます。

### チューブの作成

センサーのデータを含むチューブがないため、ダミーのセンサーエントリーでチューブを作成してみましょう。
```python
import os
import tarfile
import numpy as np
from donkeycar.parts.tub_v2 import Tub
from donkeycar.pipeline.types import TubRecord
from donkeycar.config import load_config


if __name__ == '__main__':
    # あなたの車アプリへのパスを指定
    my_car = os.path.expanduser('~/mycar')
    cfg = load_config(os.path.join(my_car, 'config.py'))
    # donkey プロジェクトへのパスを指定
    tar = tarfile.open(os.path.expanduser(
        '~/Python/donkeycar/donkeycar/tests/tub/tub.tar.gz'))
    tub_parent = os.path.join(my_car, 'data2/')
    tar.extractall(tub_parent)
    tub_path = os.path.join(tub_parent, 'tub')
    tub1 = Tub(tub_path)
    tub2 = Tub(os.path.join(my_car, 'data2/tub_sensor'),
               inputs=['cam/image_array', 'user/angle', 'user/throttle',
                       'sensor'],
               types=['image_array', 'float', 'float', 'list'])

    for record in tub1:
        t_record = TubRecord(config=cfg,
                             base_path=tub1.base_path,
                             underlying=record)
        img_arr = t_record.image(cached=False)
        record['sensor'] = list(np.random.uniform(size=2))
        record['cam/image_array'] = img_arr
        tub2.write_record(record)
```

### モデルを利用可能にする
まだ動的なファクトリーはないため、`donkeycar/utils.py` の `get_model_by_type()` 関数にこの新しいモデルを追加する必要があります。
```python
...
elif model_type == 'sensor':
    kl = KerasSensors(input_shape=input_shape)
...
```

### 学習を実行
車アプリのフォルダーで以下を実行すると、学習が始まります。
`donkey train --tub data2/tub_sensor --model models/pilot.h5 --type sensor`
データがランダム値なのでモデルの収束は早くありませんが、ここではフレームワークで動作させることが目的です。


## サポートとディスカッション
サポートや議論には [Discord](https://discord.gg/dpvYHhpV2w) の Donkey Car グループにご参加ください。



