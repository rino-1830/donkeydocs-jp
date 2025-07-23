
# 一時停止標識検出

このパーツは Google Coral アクセラレータと Coral プロジェクトによる事前学習済みの物体検出モデルを用いて、一時停止標識の検出を行います。ドンキー・カーが停止標識を認識すると、`pilot/throttle` を 0 に上書きします。さらに、`cam/image_array` にバウンディングボックスが描画されます。

<video style="width:50%" controls>
  <source src="../../assets/parts/stop_sign_detection/demo.mp4" type="video/mp4">
お使いのブラウザでは video タグがサポートされていません。
</video>

---------------

## 必要なもの
このパーツを使用するには次のものが必要です。

- [Google Coral USB Accelerator](https://coral.ai/products/accelerator/)

## 使用方法
以下の行を `myconfig.py` に追加してください。
```
STOP_SIGN_DETECTOR = True
STOP_SIGN_MIN_SCORE = 0.2
STOP_SIGN_SHOW_BOUNDING_BOX = True
```

### Edge TPU 依存関係のインストール

必要なソフトウェアは Coral Edge TPU の [はじめに](https://coral.ai/docs/accelerator/get-started) の手順に従ってインストールします。Raspberry Pi の場合も Linux 用の手順に従ってください。

停止標識検出器では事前コンパイル済みのモデルを使用するため、推論用ランタイムさえあれば動作します。ただし、自分でモデルを作成する場合は、Raspberry Pi（あるいはトレーニングを行う Linux ノートPC）に [Edge TPU Compiler](https://coral.ai/docs/edgetpu/compiler/) が必要です。コンパイラは Linux でのみ動作する点に注意してください。

## 他の物体を検出する

事前学習済みモデルは COCO データセットで学習されているため、80 種類の物体を検出できます。試してみたい場合は、`stop_sign_detector.py` 内の `STOP_SIGN_CLASS_ID` を変更するだけです。

## 精度

SSD は[小さい物体の検出が得意ではありません](https://medium.com/@jonathan_hui/what-do-we-learn-from-single-shot-object-detectors-ssd-yolo-fpn-focal-loss-3888677c5f4d)ので、遠くの停止標識を検出する精度はあまり高くない可能性があります。改善の方法はいくつかありますが、本パーツの範囲外です。

### Coral Edge TPU なしで動作させる
Coral Edge TPU を使わずに動作させるための [issue が GitHub にあります](https://github.com/autorope/donkeycar/issues/953)。うまく動作した場合はぜひプルリクエストをお送りください。


