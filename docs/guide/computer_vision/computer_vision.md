
# コンピュータビジョンオートパイロット

コンピュータビジョンオートパイロットはディープラーニングオートパイロットと同様にカメラ画像を解釈してステアリングとスロットルを決定します。ただし、ディープラーニングモデルではなく、Canny エッジ検出などの従来型コンピュータビジョンアルゴリズムを用いてトラックの画像を解析します。このオートパイロットは自分のアルゴリズムを簡単に作成し、組み込みのアルゴリズムの代わりに使用できるよう特別に設計されています。

![The Computer Vision Autopilot](/assets/cv_track.png)

組み込みのアルゴリズムはライン追従型で、検出可能なセンターライン（できれば実線）がトラック上にあることを想定しています。ラインの色は設定で調整でき、デフォルトでは黄色を使用します。アルゴリズムは画像中央からラインまでの距離を計算し、その値をPIDコントローラーでステアリング角に変換します。車がラインの左側にあれば右に、右側にあれば左に曲がります。選択されるステアリング角はラインからの距離に比例し、スロットルはステアリング角に反比例するため、直線では速く走り、カーブでは減速します。アルゴリズムと設定パラメータの詳細は後述します。

しかし、走行路にセンターラインが無い場合や左右のレーン境界線だけの場合、あるいは歩道などの場合、自分でアルゴリズムを作りたいこともあるでしょう。このコンピュータビジョン用テンプレートはそのようなときに簡単に利用できるよう作られています。Python で独自のパートを作成し、`myconfig.py` の設定を変更するだけでそのパートをオートパイロットとして使用できます。パートでは [cv.py](https://github.com/autorope/donkeycar/blob/main/donkeycar/parts/cv.py) のCV部品を利用することも、OpenCV の Python API を直接呼び出すことも可能です。以下に簡単な例を示します。

>> **重要**: コンピュータビジョンテンプレートを使用するには OpenCV がインストールされている必要があります。Jetson Nano にはあらかじめ OpenCV が入っていますが、Raspberry Pi では明示的にインストールしなければなりません。Raspberry Pi のインストール手順は[ステップ9](/guide/robot_sbc/setup_raspberry_pi/#step-9-optional-install-opencv-dependencies)と[ステップ11](/guide/robot_sbc/setup_raspberry_pi/#step-11-install-donkeycar-python-code)を参照してください。

## コンピュータビジョンアプリケーションを作成する

ディープラーニングアプリケーションを作成するときと同様に、**cv_control** テンプレートを指定することでコンピュータビジョンアプリケーションを作成できます。まず donkeycar の Python 環境を有効化し、続いて **createcar** コマンドでアプリケーションフォルダーを作成します。

```bash
donkey createcar --template=cv_control --path=~/mycar
```

When updating to a new version of donkeycar, you will want to refresh your application folder.  You can do this with the same command, but add `--overwrite` so that it does not erase your **myconfig.py** file.

```bash
donkey createcar --template=cv_control --path=~/mycar --overwrite
```

## ラインフォロワー

この組み込みアルゴリズムはカメラ映像からラインを追従します。デフォルトでは黄色のラインに合わせて調整されていますが、追跡する色は設定で変更できます。その他の多くの要素も調整可能です。以下にアルゴリズムの概要と設定値の利用方法を説明し、続いて各値を列挙します。


0. `TARGET_PIXEL` が `None` の場合は、1～5 の手順でラインの想定位置を推定します。
1. 画像の `SCAN_Y` 行から `SCAN_HEIGHT` 行分を取り出し、画像幅と同じ幅で `SCAN_HEIGHT` 高のブロックを作成します。
2. そのブロックを `RGB` から `HSV`（色相・彩度・明度）空間に変換します。
3. `COLOR_THRESHOLD_LOW` と `COLOR_THRESHOLD_HIGH` の範囲内にある HSV 値を持つ画素を抽出します。
4. 抽出した画素からヒストグラムを作成し、`SCAN_HEIGHT` ピクセル高、幅1ピクセルごとに左から右へ黄色画素の数を数えます。
5. 黄色画素が最も多いスライスの x 座標（水平位置）を求めます。ここが黄線の位置だと判断します。
6. この x 値と `TARGET_PIXEL` の差を PID アルゴリズムに入力して新しいステアリング値を計算します。差が `TARGET_PIXEL` より左に `TARGET_THRESHOLD` ピクセル以上あれば右に、右に `TARGET_THRESHOLD` ピクセル以上あれば左に曲がります。差が `TARGET_THRESHOLD` 以内ならステアリングは変更しません。
7. 求めたステアリング値を元に加速・減速を判断します。ステアリングが変化しない場合は `THROTTLE_STEP` ずつスロットルを上げますが `THROTTLE_MAX` を超えません。ステアリングを変更した場合は `THROTTLE_STEP` ずつ下げますが `THROTTLE_MIN` を下回りません。

>> この[pyimagesearch の記事](https://pyimagesearch.com/2021/04/28/opencv-color-spaces-cv2-cvtcolor/)と動画では、利用可能なさまざまな色空間と OpenCV の特徴が解説されています。


完全なソースコードはこのページ末尾近くの [LineFollower クラス](#linefollower-class) セクションで紹介しています。


### カメラのセットアップ
The image at the top of the page shows the camera setup approximately how it would be using the standard Donkeycar cage.  It is angled to see to the horizon so that it can see turns from far away.  This is good when going very fast because you can see far ahead.  However if the detected line is very thin then it could have artifacting (noise) that could lead to false positives that cause the vehicle to move off the line. If you are not going fast and you want to be as accurate as possible then pointing the camera down at the line is a good idea.  So if your camera can be adjusted then you can make trade-offs between accuracy (point it down) and speed (point to to the horizon).

### Choosing Parameters for the LineFollower
The computer vision template is a little different than the deep learning and path follow templates; there is no data recording.  After setting your configuration parameters you just put your car on the track that has the line that you want to follow and then change from user mode to one of the auto-pilot modes; full-auto or auto-steering.  The complete set of configuration parameters can be found in the [LineFollower Configuration](#linefollower-configuration) section below; we will discuss the most important configuration in more detail in this section.

#### SCAN_Y and SCAN_HEIGHT
ラインをスキャンする矩形領域（検出エリア）は `SCAN_Y` と `SCAN_HEIGHT` で決まります。

オートパイロットモードでは検出エリアが水平の黒い帯として表示されます。色閾値範囲内（次節参照）の画素は白で描かれます。理想的にはライン上の画素だけが白く表示されますが、ライン以外の白い画素は誤検出です。誤検出が点在する程度なら問題ありませんが、大きな白い領域があるとアルゴリズムが惑わされる可能性があります。誤検出を減らすには次節の色閾値設定を調整してください。

下の画像は検出エリアと検出されたラインを示しています。

![The Detection Area](/assets/cv_track_telemetry.png)

#### COLOR_THRESHOLD_LOW と COLOR_THRESHOLD_HIGH
これらの値はライン検出に用いる色の範囲を表します。検出バーを通過するラインの色を含み、かつ他の色を含まないように設定します。値は RGB ではなく HSV（色相・彩度・明度）形式で指定します。HSV は人間の色の感じ方に近く、影や照明の影響を受けにくい色相値のみで色を見つけられるため便利です。

>> RGB と HSV を変換するオンラインツールは多数あります。ここでは [peko-step](https://www.peko-step.com/en/tool/hsvrgb_en.html) を利用しました。彩度と明度を 0～255 の範囲で取得できるため便利です。注意点として、一般的な HSV は色相 0～359 度、彩度・明度 0～100% ですが、OpenCV では色相 0～179、彩度・明度 0～255 で表現するため、設定を変更する際は変換が必要です。

閾値の色を選ぶ際は、カメラが実際に見る環境や照明を考慮することが重要です。Donkeycar にはそれを簡単に行うための `hsv_picker.sh` スクリプトが用意されています。デスクトップ環境であれば、カメラ映像を直接表示させて調整できます。ヘッドレス環境の場合は、車を走らせて Web ビューを開きスクリーンショットを取得し、その画像をスクリプトに読み込ませます。いずれの場合も、実際にオートパイロットで走行する状況を再現した配置でラインを撮影してください。

You can run the `hsv_picker.sh` script to view a screen shot image; with the donkey python environment activated run the script from the root of your donkeycar repo folder;
```
python scripts/hsv_picker.sh --file=<path-to-image>
```

To view the camera stream, again with donkey python environment activated, run the script from the root of your donkeycar repo folder;

```
python scripts/hsv_picker.sh
```

If you have more than one camera and it is not showing the correct one, you can choose the camera index and/or set the image size

```
python scripts/hsv_picker.sh --camera=2  --width=320 --height=240
```

![A screenshot in hsv_script.sh](/assets/hsv_picker_no_mask.png)

上の画像は `hsv_script.sh` に Web UI のスクリーンショットを読み込んだ例です。中央の青い線が追従したいラインで、カメラ画像に映る水平の黒い帯が検出バーです。`SCAN_Y` と `SCAN_HEIGHT` で指定されたこの領域にマスクを適用し、ラインの画素を抽出します。検出された画素は白で表示されます。


画面下部には6つのトラックバーがあり、マスクに使用するHSVの下限3値と上限3値を調整できます。バーを動かすと即座にマスクに反映され、画像上でどの画素が抽出されているか確認できます。通常はHueが最も重要です。Escapeキーを押せばバーをリセットしてマスクをクリアできます。

トラックバーでも調整できますが、矩形選択による自動設定も可能です。画像上で範囲をドラッグするとその領域の最小値・最大値が計算され、トラックバーに反映されます。ライン上を選択すると簡単にマスクを求められるので、微調整だけバーで行うと便利です。

下の画像は青いライン上を矩形選択して得られたマスクの例です。

![A masked screenshot in the hsv_picker.sh script](/assets/hsv_picker_mask.png)

`hsv_picker.sh` の主な機能:

- 画面下部のトラックバーでマスクの上下限を変更
- 画像上で範囲をドラッグして上下限を自動設定
- Escape キーでマスクをクリア
- `p` キーで現在の値をコンソールに表示
- `q` キーで値を表示して終了


#### TARGET_PIXEL
`TARGET_PIXEL` は画像内でラインが存在すると想定する水平位置を表します。ラインフォローアルゴリズムはこの位置にラインが来るようステアリングを調整します。具体的には、実際に検出されたライン位置と `TARGET_PIXEL` の差を PID コントローラに渡してステアリングを計算します（詳しくは[PID コントローラ](#the-pid-controller)参照）。

単独で走行する場合はライン上をそのまま走りたいでしょう。そのときは `TARGET_PIXEL` を画像の中央 `(IMAGE_W / 2)` に設定すると、ラインが中央にあると仮定して車が中心へ寄るよう制御します。スタート時にラインから外れていても素早く中央へ戻ります。

一方、2台同時に走るコースなどでは自車のレーンを維持したいはずです。その場合 `TARGET_PIXEL` を `None` に設定すると、起動時にラインの位置を自動で検出し、その位置を基準として走行します。

>> さらに凝る場合は、レーン変更を行うために `TARGET_PIXEL` を動的に切り替えるアルゴリズムを実装してみてもよいでしょう。

### LineFollower Configuration
The complete set of configuration values and their defaults can be found in [donkeycar/templates/cfg_cv_control.py](https://github.com/autorope/donkeycar/blob/main/donkeycar/templates/cfg_cv_control.py#L556) and is copied here for convenience.

```
# configure which part is used as the autopilot - change to use your own autopilot
CV_CONTROLLER_MODULE = "donkeycar.parts.line_follower"
CV_CONTROLLER_CLASS = "LineFollower"
CV_CONTROLLER_INPUTS = ['cam/image_array']
CV_CONTROLLER_OUTPUTS = ['pilot/steering', 'pilot/throttle', 'cv/image_array']
CV_CONTROLLER_CONDITION = "run_pilot"

# LineFollower - line color and detection area
SCAN_Y = 120          # num pixels from the top to start horiz scan
SCAN_HEIGHT = 20      # num pixels high to grab from horiz scan
COLOR_THRESHOLD_LOW  = (0, 50, 50)    # HSV dark yellow (opencv HSV hue value is 0..179, saturation and value are both 0..255)
COLOR_THRESHOLD_HIGH = (50, 255, 255) # HSV light yellow (opencv HSV hue value is 0..179, saturation and value are both 0..255)

# LineFollower - target (expected) line position and detection thresholds
TARGET_PIXEL = None   # In not None, then this is the expected horizontal position in pixels of the yellow line.
                      # If None, then detect the position yellow line at startup;
                      # so this assumes you have positioned the car prior to starting.
TARGET_THRESHOLD = 10 # number of pixels from TARGET_PIXEL that vehicle must be pointing
                      # before a steering change will be made; this prevents algorithm
                      # from being too twitchy when it is on or near the line.
CONFIDENCE_THRESHOLD = (1 / IMAGE_W) / 3  # The fraction of total sampled pixels that must be yellow in the sample slice.
                                          # The sample slice will have SCAN_HEIGHT pixels and the total number
                                          # of sampled pixels is IMAGE_W x SCAN_HEIGHT, so if you want to make sure
                                          # that all the pixels in the sample slice are yellow, then the confidence
                                          # threshold should be SCAN_HEIGHT / (IMAGE_W x SCAN_HEIGHT) or (1 / IMAGE_W).
                                          # If you keep getting `No line detected` logs in the console then you
                                          # may want to lower the threshold.

# LineFollower - throttle step controller; increase throttle on straights, descrease on turns
THROTTLE_MAX = 0.3    # maximum throttle value the controller will produce
THROTTLE_MIN = 0.15   # minimum throttle value the controller will produce
THROTTLE_INITIAL = THROTTLE_MIN  # initial throttle value
THROTTLE_STEP = 0.05  # how much to change throttle when off the line

# These three PID constants are crucial to the way the car drives. If you are tuning them
# start by setting the others zero and focus on first Kp, then Kd, and then Ki.
PID_P = -0.01         # proportional mult for PID path follower
PID_I = 0.000         # integral mult for PID path follower
PID_D = -0.0001       # differential mult for PID path follower

OVERLAY_IMAGE = True  # True to draw computer vision overlay on camera image in web ui
                      # NOTE: this does not affect what is saved to the data

```


## The PID Controller
It is very common to use a Proportional Integral Derivative (PID) controller to control throttle and/or steering in a wheeled robot.  For example, the [Path Follow](/guide/path_follow/path_follow) autopilot uses a PID algorithm to modify steering based on how far away from the desired path the robot is.  In the Computer Vision template, the built-in Line Follower algorithm uses a PID in a similar way; the line follow algorithm outputs a value that is proportional to how far the car is from the center line and whose sign indicates which side of the line it is on.  The PID controller uses the magnitude and sign of the distance from the center line to calculate a steering value that will move the car towards the center line.  

>> The path_follow autopilot also uses a PID controller.  There is a good description of how to tune a controller for driving at [Determining PID Coefficients](/guide/path_follow/path_follow/#determining-pid-coefficients)


## Writing a Computer Vision Autopilot
You can use the `CV_CONTROLLER_*` configuration values to point to a python file and class that implements your own computer vision autopilot part.  Your autopilot class must conform to the [donkeycar part](/parts/about/#parts) standard.  You can also determine the name of the input values, output values and run_condition.  The default configuration values point the the included LineFollower part.  At a minimum computer vistion autpilot part takes the camera image as an input and outputs the autopilot's throttle and steering values.

Let's create a simple custom computer vision part.  It won't be much of an autopilot because it will just output a constant throttle and steering value and an image that counts that frames.

A computer vision is a [donkeycar part](/parts/about/#parts), so at a minimum it must be a Python class with a `run(self)` method.  An autpilot needs a little more than that, which we will see, but here is a minimal structure;

```
import cv2
import numpy as np
from simple_pid import PID
import logging

logger = logging.getLogger(__name__)


class MockCvPilot:
    def __init__(self, pid, cfg):
        # initialize instance properties
        pass

    def run(self, img):
        # use img to determine a steering and throttle value
        return 0, 0, None  # steering, throttle, image
```

The constructor, `__init__(self, pid, cfg)` takes a PID controller instance and the vehicle configuration properties.  It is very common for autopilots to use a PID controller, so the framework provides one.  Your autopilot may have values that you want to adjust to tune the algorithm; you should put those values in the `myconfig.py` configuration file, then retrieve them in the constructor.  In our `MockCvPilot` we want to know if the user wants to see the telemetry image or just the camera image.  We do the same thing in the built-in `LineFollower` autopilot part, so we can just re-use that configuration value, `OVERLAY_IMAGE`, in our autpilot.  We can add that in our constructor;

```
    def __init__(self, pid, cfg):
        self.pid_st = pid
        self.overlay_image = cfg.OVERLAY_IMAGE
        self.counter = 0
```


The `run(self, img)` method is called each time through the loop.  This is where you will interpret the image that is passed and determine a steering and throttle value that the car should use.  The Computer Vision template also allows for showing an image in the web ui that is different in autopilot mode; typically you would add telemetry information to the camera image that is passed to run; such as the new steering and throttle values and perhaps a other alterations to the image so the user can better understand how the algorithm is working.  For instance, if your algorithm did edge detection using the Canny algorithm, then you might want to show the processed image with the edges.  So the minimal autopilot part returns a tuple of (steering, throttle, image).

To keep things simple, the MockCvPilot won't actually predict a steering and throttle, it will just return zero for each.  However it will maintain a counter and display that in the telemetry image.  We can see that in the `run()` method.

```
    def run(self, cam_img):
        if cam_img is None:
            return 0, 0, None
        
        self.counter += 1

        # show some diagnostics
        if self.overlay_image:
            # draw onto a COPY of the image so we don't alter the original
            cam_img = self.overlay_display(np.copy(cam_img))

        return self.steering, self.throttle, cam_img
```

There are a couple of things to note here:

- The `run()` method is protected against an empty camera image - this can happen, espcially during startup.  So in this case we stop the car.
- We only produce a telemetry image if the original configuration value, `OVERLAY_IMAGE`, that we copied into `self.overlay_image`is `True`.  If it is not `True` then we just pass throught the original camera image.
- Note that we make a copy of the original camera image so that we do not alter the original.  This is called a _defensive copy_; we don't know what other parts in the vehicle need to do with the orignal image, so we don't want t alter it.

We put the logic that draws the telemetry image into it's own method so keep the both it and the `run()` method clean and cohesive.  Also because we `run()` method has made a defensive copy of the original image, the method can do anything it wants to the image; even overwrite it completely.  In our case we just draw some text on it to show the steering, throttle and counter values.  We know the steering and throttle values will be zero in our mock autopilot, but it is instructive to show how you might display them.  In this case we are showing them as text, but you might prefer to show them as bars, like we do in the webui, or some other visualization.  This is the display method we use in our mock autopilot;

```
    def overlay_display(self, img):
        display_str = []
        display_str.append(f"STEERING:{self.steering:.1f}")
        display_str.append(f"THROTTLE:{self.throttle:.2f}")
        display_str.append(f"COUNTER:{self.counter}")

        lineheight = 25
        y = lineheight
        x = lineheight
        for s in display_str:
            cv2.putText(img, s, color=(0, 0, 0), org=(x ,y), fontFace=cv2.FONT_HERSHEY_SIMPLEX, fontScale=1, thickness=3)
            cv2.putText(img, s, color=(0, 255, 0), org=(x ,y), fontFace=cv2.FONT_HERSHEY_SIMPLEX, fontScale=1, thickness=1)
            y += lineheight

        return img
```

There are a couple of things to note:
- We organize the text as an array of strings; that makes it easy to process the line when we are drawing the text.  You might even want to have a separate method to create this list of text string and possibly pass them into the display() methed if that simplies the display method or makes it more versatile (it can do both).
- We draw the text twice, once with a thick black stroke and then again with a thinner green stroke.  This creates green text with a black outline; this makes it easier to read on an unpredictable background.


Here is the complete custom computer vision autopilot part:

```
import cv2
import numpy as np
from simple_pid import PID
import logging

logger = logging.getLogger(__name__)


class MockCvPilot:
    '''
    OpenCV based MOCK controller; just draws a counter and 
    returns 0 for thottle and steering.

    :param pid: a PID controller that can be used to estimate steering and/or throttle
    :param cfg: the vehicle configuration properties
    '''
    def __init__(self, pid, cfg):
        self.pid_st = pid
        self.overlay_image = cfg.OVERLAY_IMAGE
        self.steering = 0
        self.throttle = 0
        self.counter = 0


    def run(self, cam_img):
        '''
        main runloop of the CV controller.

        :param cam_img: the camerate image, an RGB numpy array
        :return: tuple of steering, throttle, and the telemetry image.

        If overlay_image is True, then the output image
        includes an overlay that shows how the 
        algorithm is working; otherwise the image
        is just passed-through untouched. 
        '''
        if cam_img is None:
            return 0, 0, None
        
        self.counter += 1

        # show some diagnostics
        if self.overlay_image:
            # draw onto a COPY of the image so we don't alter the original
            cam_img = self.overlay_display(np.copy(cam_img))

        return self.steering, self.throttle, cam_img

    def overlay_display(self, img):
        '''
        draw on top the given image.
        show some values we are using for control

        :param img: the image to draw on as a numpy array
        :return: the image with overlay drawn
        '''
        # some text to show on the overlay
        display_str = []
        display_str.append(f"STEERING:{self.steering:.1f}")
        display_str.append(f"THROTTLE:{self.throttle:.2f}")
        display_str.append(f"COUNTER:{self.counter}")

        lineheight = 25
        y = lineheight
        x = lineheight
        for s in display_str:
            # green text with black outline so it shows up on any background
            cv2.putText(img, s, color=(0, 0, 0), org=(x ,y), fontFace=cv2.FONT_HERSHEY_SIMPLEX, fontScale=1, thickness=3)
            cv2.putText(img, s, color=(0, 255, 0), org=(x ,y), fontFace=cv2.FONT_HERSHEY_SIMPLEX, fontScale=1, thickness=1)
            y += lineheight

        return img
```

To use the custom part, we must modify the myconfig.py file in the mycar folder to locate the python file and the class within it and to specify the inputs, outputs and run_condition that should be used when adding the part to the vehicle loop:

```
# # configure which part is used as the autopilot - change to use your own autopilot
CV_CONTROLLER_MODULE = "my_cv_pilot"
CV_CONTROLLER_CLASS = "MockCvPilot"
CV_CONTROLLER_INPUTS = ['cam/image_array']
CV_CONTROLLER_OUTPUTS = ['pilot/steering', 'pilot/throttle', 'cv/image_array']
CV_CONTROLLER_CONDITION = "run_pilot"
```

`CV_CONTROLLER_MODULE` is the package path to the `my_cv_autpilot.py` file.  It is generally is convenient to have this in the mycar folder and this is what we have done here.  However, if you are developing this in your own repository, then if you are a Mac or Linux machine you can create a symbolic link to the file or the folder in which the file or files are.

`CV_CONTROLLER_CLASS` is the name of the part's class in the python file to which `CV_CONTROLLER_MODULE` points.  In our case this is `MockCvPilot`.

`CV_CONTROLLER_INPUTS` is an array of the named inputs to the part that are passed when the part is added to the vehicle loop.  For a computer vision autopilot the image is the minimum required.  However you can pass any named values in the vehicle's memory.  These correspond in a one-to-one fashion to the arguments (ignore the self argument) to the autopilot's run() method.  So our mock example expects only an image `run(self, cam_img)` and we only declare an image in the inputs, `['cam/image_array']`.

`CV_CONTROLLER_OUTPUTS` is an array of named outputs to the part that are passed when the part is added to vehicle loop.  These correspond to the return values from the autopilot's `run()`.  This is an autopilot, so we return a steering value and a throttle value.  We also produce a new image with telemetry information drawn on it.  So our mock autopilot returns `return self.steering, self.throttle, cam_img` which corresponds to the declared output values, `['pilot/steering', 'pilot/throttle', 'cv/image_array']`.

`CV_CONTROLLER_CONDITION` is the named value that decides if the autopilot part will run or not run.  If you always want it to run, then pass `None`, otherwise this should be the name of a boolean value; when it is `True` the part's `run()` method will be called; when it if `False` the `run()` method is not called.  The templates maintain such a boolean value named `"run_pilot"`, so we use that.


## LineFollower Class

Now that you understand the structure of an autopilot part, it is worth reviewing the pseudocode in [The Line Follower](#the-line-follower) section above and compare that to the actual implementation.  The python file is located at https://github.com/autorope/donkeycar/blob/main/donkeycar/parts/line_follower.py and is copied below.  In particular:

- get_i_color() uses SCAN_Y and SCAN_HEIGHT to copy a section of the camera image, convert it to HSV and apply the mask created by the low and high HSV mask values.  Then it finds the x (horizontal) index in the area that has the highest amount of the positive pixels; that is where the autopilot thinks the line is.
```
    def get_i_color(self, cam_img):
        # take a horizontal slice of the image
        iSlice = self.scan_y
        scan_line = cam_img[iSlice : iSlice + self.scan_height, :, :]

        # convert to HSV color space
        img_hsv = cv2.cvtColor(scan_line, cv2.COLOR_RGB2HSV)

        # make a mask of the colors in our range we are looking for
        mask = cv2.inRange(img_hsv, self.color_thr_low, self.color_thr_hi)

        # which index of the range has the highest amount of yellow?
        hist = np.sum(mask, axis=0)
        max_yellow = np.argmax(hist)

        return max_yellow, hist[max_yellow], mask
```

- Note how target value is initialized by reading the image if it is not already initialized in the configuration:
```
        max_yellow, confidence, mask = self.get_i_color(cam_img)
        if self.target_pixel is None:
            self.target_pixel = max_yellow
```

- Note how the PID controller's set point (the target value) in initialized in the `run()` method with the `TARGET_PIXEL` value.
```
        if self.pid_st.setpoint != self.target_pixel:
            # this is the target of our steering PID controller
            self.pid_st.setpoint = self.target_pixel
```

- If we got a good reading from the image, then we use it to predict a new steering value based on the horizontal distance of the detected line from the target_pixel.
```
        if confidence >= self.confidence_threshold:
            # invoke the controller with the current yellow line position
            # get the new steering value as it chases the ideal target_value
            self.steering = self.pid_st(max_yellow)
```

- Slow down if we are turning
```
            if abs(max_yellow - self.target_pixel) > self.target_threshold:
                # we will be turning, so slow down
                if self.throttle > self.throttle_min:
                    self.throttle -= self.delta_th
```

- Or speed up if we are going straight
```
            else:
                # we are going straight, so speed up
                if self.throttle < self.throttle_max:
                    self.throttle += self.delta_th

```


Here is the complete source to the `LineFollower` part.

```
import cv2
import numpy as np
from simple_pid import PID
import logging

logger = logging.getLogger(__name__)


class LineFollower:
    '''
    OpenCV based controller
    This controller takes a horizontal slice of the image at a set Y coordinate.
    Then it converts to HSV and does a color thresh hold to find the yellow pixels.
    It does a histogram to find the pixel of maximum yellow. Then is uses that iPxel
    to guid a PID controller which seeks to maintain the max yellow at the same point
    in the image.
    '''
    def __init__(self, pid, cfg):
        self.overlay_image = cfg.OVERLAY_IMAGE
        self.scan_y = cfg.SCAN_Y   # num pixels from the top to start horiz scan
        self.scan_height = cfg.SCAN_HEIGHT  # num pixels high to grab from horiz scan
        self.color_thr_low = np.asarray(cfg.COLOR_THRESHOLD_LOW)  # hsv dark yellow
        self.color_thr_hi = np.asarray(cfg.COLOR_THRESHOLD_HIGH)  # hsv light yellow
        self.target_pixel = cfg.TARGET_PIXEL  # of the N slots above, which is the ideal relationship target
        self.target_threshold = cfg.TARGET_THRESHOLD # minimum distance from target_pixel before a steering change is made.
        self.confidence_threshold = cfg.CONFIDENCE_THRESHOLD  # percentage of yellow pixels that must be in target_pixel slice
        self.steering = 0.0 # from -1 to 1
        self.throttle = cfg.THROTTLE_INITIAL # from -1 to 1
        self.delta_th = cfg.THROTTLE_STEP  # how much to change throttle when off
        self.throttle_max = cfg.THROTTLE_MAX
        self.throttle_min = cfg.THROTTLE_MIN

        self.pid_st = pid


    def get_i_color(self, cam_img):
        '''
        get the horizontal index of the color at the given slice of the image
        input: cam_image, an RGB numpy array
        output: index of max color, value of cumulative color at that index, and mask of pixels in range
        '''
        # take a horizontal slice of the image
        iSlice = self.scan_y
        scan_line = cam_img[iSlice : iSlice + self.scan_height, :, :]

        # convert to HSV color space
        img_hsv = cv2.cvtColor(scan_line, cv2.COLOR_RGB2HSV)

        # make a mask of the colors in our range we are looking for
        mask = cv2.inRange(img_hsv, self.color_thr_low, self.color_thr_hi)

        # which index of the range has the highest amount of yellow?
        hist = np.sum(mask, axis=0)
        max_yellow = np.argmax(hist)

        return max_yellow, hist[max_yellow], mask


    def run(self, cam_img):
        '''
        main runloop of the CV controller
        input: cam_image, an RGB numpy array
        output: steering, throttle, and the image.
        If overlay_image is True, then the output image
        includes and overlay that shows how the 
        algorithm is working; otherwise the image
        is just passed-through untouched. 
        '''
        if cam_img is None:
            return 0, 0, False, None

        max_yellow, confidence, mask = self.get_i_color(cam_img)
        conf_thresh = 0.001

        if self.target_pixel is None:
            # Use the first run of get_i_color to set our relationship with the yellow line.
            # You could optionally init the target_pixel with the desired value.
            self.target_pixel = max_yellow
            logger.info(f"Automatically chosen line position = {self.target_pixel}")

        if self.pid_st.setpoint != self.target_pixel:
            # this is the target of our steering PID controller
            self.pid_st.setpoint = self.target_pixel

        if confidence >= self.confidence_threshold:
            # invoke the controller with the current yellow line position
            # get the new steering value as it chases the ideal target_value
            self.steering = self.pid_st(max_yellow)

            # slow down linearly when away from ideal, and speed up when close
            if abs(max_yellow - self.target_pixel) > self.target_threshold:
                # we will be turning, so slow down
                if self.throttle > self.throttle_min:
                    self.throttle -= self.delta_th
                if self.throttle < self.throttle_min:
                    self.throttle = self.throttle_min
            else:
                # we are going straight, so speed up
                if self.throttle < self.throttle_max:
                    self.throttle += self.delta_th
                if self.throttle > self.throttle_max:
                    self.throttle = self.throttle_max
        else:
            logger.info(f"No line detected: confidence {confidence} < {self.confidence_threshold}")

        # show some diagnostics
        if self.overlay_image:
            cam_img = self.overlay_display(cam_img, mask, max_yellow, confidence)

        return self.steering, self.throttle, cam_img

    def overlay_display(self, cam_img, mask, max_yellow, confidense):
        '''
        composite mask on top the original image.
        show some values we are using for control
        '''

        mask_exp = np.stack((mask, ) * 3, axis=-1)
        iSlice = self.scan_y
        img = np.copy(cam_img)
        img[iSlice : iSlice + self.scan_height, :, :] = mask_exp
        # img = cv2.cvtColor(img, cv2.COLOR_RGB2BGR)

        display_str = []
        display_str.append("STEERING:{:.1f}".format(self.steering))
        display_str.append("THROTTLE:{:.2f}".format(self.throttle))
        display_str.append("I YELLOW:{:d}".format(max_yellow))
        display_str.append("CONF:{:.2f}".format(confidense))

        y = 10
        x = 10

        for s in display_str:
            cv2.putText(img, s, color=(0, 0, 0), org=(x ,y), fontFace=cv2.FONT_HERSHEY_SIMPLEX, fontScale=0.4)
            y += 10

        return img
```

