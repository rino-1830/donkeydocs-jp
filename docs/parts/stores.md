## Tub

これは標準的な Donkey のデータストアです。`data` フォルダーは「tub」と呼ばれます。

### 受け入れられる型

以下のデータ型をサポートしています。

* `str`
* `int`
* `float` / `np.float`
* `image_array` や `array` (`np.ndarray`)
* `image` (jpeg / png)

`Tub` は追加のみ可能な形式で、学習モデルの高速化のために読み出しに最適化されています。
レコードのインデックスを保持し、メモリマップドファイルを使用します。

`Tub` はレコードを読み出すための `Iterator` を提供します。これらのイテレータは `Pipeline` と組み合わせて、学習前にデータ拡張など任意の変換を行うことができます。

### 例

```python
from donkeycar.parts.tub_v2 import Tub

# ここでは単一の `input` 型 `int` を持つレコードを定義します。
inputs = ['input']
types = ['int']
tub = Tub(path, inputs, types)

```
