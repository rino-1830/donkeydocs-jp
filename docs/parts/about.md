# Donkeycar ソフトウェアアーキテクチャ

Donkeycar は非常にシンプルで、コードは入力を受け取り出力を返すパーツに整理されています。これらのパーツは車両に追加されます。車両ループが開始されるとパーツは順番に実行され、車両のメモリを読み書きすることで効果的に通信します。

「テンプレート」とは、「ビークル」と1つ以上の「パーツ」を構築するコードを含む Python ファイルのことです。パーツは車両の機能コンポーネントを包む Python クラスです。パーツは車両に追加されます。パーツは車両メモリの値を入力として受け取り、出力としてそのメモリに値を書き込むことができます。ビークルループが開始されると、パーツは追加された順に実行され、メモリから入力を取得して結果をメモリに出力します。これは車両が停止するまで続き、停止するとすべてのパーツがシャットダウンしてテンプレートが終了します。

## テンプレート
`donkey createcar ...` コマンドを使用して車のアプリケーションを作成すると、ドキュメントの [Create Donkeycar App](https://docs.donkeycar.com/guide/create_application/) セクションで説明されているように、いくつかのファイルが `donkeycar/templates` フォルダからマイカーのフォルダにコピーされます。ここで説明するのは manage.py と myconfig.py の2つです。

mycar フォルダにコピーされるファイルは、テンプレートフォルダ内のテンプレートファイルの名前を変えたものです。ファイルは `createcar` コマンドの `--template` 引数に渡したテンプレート名によって選択されます。何も渡さない場合のデフォルトは `--template = complete` です。したがって `donkey createcar --path=~/mycar` は `donkey createcar --path=~/mycar --template=complete` と同じです。この場合、`~/mycar/manage.py` と `~/mycar/myconfig.py` にコピーされるのは `donkeycar/templates/complete.py` と `donkeycar/templates/cfg_complete.py` です。`--template=path_follow` を指定してアプリケーションを作成した場合は、`donkeycar/templates/path_follow.py` と `donkeycar/templates/cfg_path_follow.py` がコピーされます。

厳密には `donkeycar/template/cfg_xxxx.py` の別コピーが mycar フォルダに `config.py` としてコピーされます。これはデフォルト設定を含むもので、編集すべきではありません。myconfig.py は実際には config.py をコメントアウトしたバージョンです。アプリの設定（カメラや駆動方式など）を変更するには、myconfig.py で必要な部分のコメントを外して編集します。

manage.py ファイルこそが実際に車を動かすコードです。これは myconfig.py の `DRIVE_LOOP_HZ` 値で指定されるレートで動く「ビークルループ」に構成されています。つまりその頻度で車両ループの「パーツ」が更新されます。donkeycar の車両ループは「メモリ」と呼ばれるハッシュマップで状態を取得・設定する「パーツ」のパイプラインです。

complete.py と path_follow.py のテンプレートは非常に設定項目が多いため複雑ですが、特別なものではありません。自分専用のテンプレートを作成することもできますし、テンプレートを使わずに直接 `manage.py` を書くこともできます。次に示すのは、1つのパーツだけを持つ車両ループの例で、数値を受け取りランダムな数値を掛け合わせて結果を返します。ループが回るたびに値はランダム化され続けます。

```python
import random

# the randomizer part
class RandPercent:
    def run(self, x):
        value = x * random.random()
        print(f"{x} -> {value}")
        return value

# create the vehicle and it's internal memory
V = dk.Vehicle()

# initialize the value in the vehicle's memory and give it a name
V.mem['var'] = 4

# add the part to read and write to the same value.
V.add(RandPercent(), inputs=['var'], outputs=['var'])

# start the vehicle loop running; quit after 5 loops
V.start(max_loops=5)
```

## パーツ
パーツとは車両の機能コンポーネントを包む Python クラスです。

以下が例です。

* センサー - カメラ、Lidar、オドメトリ、GPS など
* アクチュエータ - モーターコントローラー
* パイロット - レーン検出器、行動クローンモデルなど
* コントローラー - Web ベースまたは Bluetooth
* ストレージ - Tub などデータを保存する方法

Tawn Kramer は [パーツの作り方](https://www.youtube.com/watch?v=YZ4ESrtfShs) を説明する2部構成の動画を作成しています。また、Arm AIoT カンファレンスで [OLED パーツを作成した方法](https://youtu.be/GOkYPXheWSY?t=1213) を紹介する発表動画もあります。

各パーツは作成された後、名前付きの入力と出力、そしてオプションの `run_condition` とともに車両ループに追加されます。車両のパーツは基本的に追加された順に実行されます。車両ループが回るたびに、パーツの入力は車両メモリから読み込まれて `run()` メソッドに渡され、`run()` メソッドが処理を行い、その戻り値が出力値として割り当てられます。`run_condition` がある場合は、その `run_condition` プロパティが True のときだけ `run()` メソッドが呼び出されます。False のときはそのパーツは「オフ」になります。

- **memory**: 車両メモリは名前付き値のハッシュマップで、車両の状態を表します。入力、出力、条件に使われる値を含み、すべてのパーツで共有されます。
- **inputs**: `run()` メソッドに渡されるメモリ値で、パーツを車両ループに追加するときに宣言します。例として aiLauncher パーツを追加するとき `inputs=['user/mode', 'pilot/throttle']` を指定します。`run()` メソッドが呼ばれる直前に、車両ループは入力値を取得して `run()` メソッドに引数として渡します。従って aiLauncher の `run()` メソッドが呼ばれるとき、最初の引数は車両メモリの `user/mode` の値、2 番目の引数は `pilot/throttle` の値になります。パーツを追加するときに宣言した入力の数と `run()` メソッドの引数の数が一致しないと実行時エラーになります。
- **outputs**: パーツの `run()` メソッドから返されるメモリ値で、パーツを車両ループに追加するときに宣言します。`run()` メソッドが呼ばれた後、その戻り値が名前付きの出力プロパティに割り当てられます。aiLauncher の例では `outputs=['pilot/throttle']` を指定します。aiLauncher が実行を終えると1つの値を返し、その値が車両メモリの `'pilot/throttle'` プロパティに代入されます。パーツを追加するときに宣言した出力の数と `run()` メソッドが返す値の数が一致しないと実行時エラーになります。
- **run_condition**: `run_condition` はパーツの `run()` メソッドを呼び出すかどうかを決める真偽値のメモリ値です。条件が True のときに `run()` が呼び出され、そうでない場合は呼ばれません。これによりパーツをオン・オフできます。たとえば aiLauncher を自動運転モードのときだけ動かしたいなら、`'run_pilot'` という名前のメモリ値を用意して、自動運転時は True、手動運転時は False とします。そして aiLauncher を追加するとき `run_condition='run_pilot'` を指定すれば、`'run_pilot'` が True のときだけ aiLauncher の `run()` メソッドが呼ばれます。

このように入力プロパティの値を変えることでパーツの動作を制御できます。あるパーツが出力した値は、他のパーツの入力や `run_condition` として使われ、相互に影響し合います。

次にパーツを追加する例を示します。[AiLaunch パーツ](https://github.com/autorope/donkeycar/blob/main/donkeycar/parts/launch.py) は、運転モードが手動から自動運転へ切り替わったときにスロットルを上書きし、レース開始直後に短時間だけ大きなスロットルを与えるために使われます。この例では明示的な `run_condition` 引数はなく、デフォルトで True になります。

```python
a iLauncher = AiLaunch(cfg.AI_LAUNCH_DURATION, cfg.AI_LAUNCH_THROTTLE, cfg.AI_LAUNCH_KEEP_ENABLED)
V.add(aiLauncher,
      inputs=['user/mode', 'pilot/throttle'],
      outputs=['pilot/throttle'])
```

この「ランチ」を実装するには、現在の運転モードと自動運転スロットル値を知る必要があります。これらが入力です。ランチしていないときはスロットル値をそのまま通しますが、ランチ中は `cfg.AI_LAUNCH_THROTTLE` と同じスロットル値を出力します。したがって出力はスロットルのみです。パーツの `run()` メソッドは、これら2つの入力を正しい順序で受け取り、1つの出力を返す必要があります。コードを見れば分かります。

```python
import time

class AiLaunch():
    '''
    このパーツは初回の起動時に大きな推力を加えます。これはレースで素早くスタートし、その後すぐに AI が引き継げるようにするためです。
    '''

    def __init__(self, launch_duration=1.0, launch_throttle=1.0, keep_enabled=False):
        self.active = False
        self.enabled = False
        self.timer_start = None
        self.timer_duration = launch_duration
        self.launch_throttle = launch_throttle
        self.prev_mode = None
        self.trigger_on_switch = keep_enabled

    def enable_ai_launch(self):
        self.enabled = True
        print('AiLauncher is enabled.')

    def run(self, mode, ai_throttle):
        new_throttle = ai_throttle

        if mode != self.prev_mode:
            self.prev_mode = mode
            if mode == "local" and self.trigger_on_switch:
                self.enabled = True

        if mode == "local" and self.enabled:
            if not self.active:
                self.active = True
                self.timer_start = time.time()
            else:
                duration = time.time() - self.timer_start
                if duration > self.timer_duration:
                    self.active = False
                    self.enabled = False
        else:
            self.active = False

        if self.active:
            print('AiLauncher is active!!!')
            new_throttle = self.launch_throttle

        return new_throttle
```

設定値をこの例のようにパーツのコンストラクタ引数として渡すことがよくあります。また、パーツがカメラやシリアルポートのようなハードウェアリソースを取得する場合は、donkey を停止したときに適切にリソースを解放する `shutdown()` 関数も用意すべきです。

先ほど述べたように、パーツの `run()` メソッドは車両ループが回るたびに呼び出されます。入力値は車両メモリから読み取られて `run()` メソッドに引数として渡され、処理結果が戻り値として返されて車両メモリに出力として書き込まれます。パーツは基本的に追加された順に実行されるため、ある値を入力として必要とするパーツよりも先に、その値を出力するパーツを追加する必要があります。

## スレッドパーツ

パーツは追加された順に実行されると言いましたが、パーツを独自のスレッドで実行して独自の速度で動かすよう指定することもできます。スレッドパーツは `run()` の代わりに `run_threaded()` メソッドを持ち、入力と出力の扱いは `run()` と同様です。また `run()` 同様、`run_threaded()` も車両ループが回るたびに1回呼び出されます。

`run_threaded()` が `run()` と同様に毎回呼び出され、入力と出力も非スレッドパーツと同じなら、両者の違いは何でしょうか。下の例でも分かるように、スレッドパーツを追加するときは `threaded=True` を渡します。最大の違いは、引数を取らない `update()` メソッドを必ず持たなければならないことです。スレッドパーツが起動されるとスレッドが生成され、そのスレッドで実行されるメソッドとして `update()` が登録されます。`update()` メソッドは車両ループとは別に実行され、Python のスケジューラが許す限り速く実行されるため、通常は車両ループよりずっと高速です。`update()` メソッドは `shutdown()` を指示されるまで戻ってはいけません。TFMini パーツのようにデバイスから読み続けるループなど、同じ処理を繰り返す形にします。スレッドパーツの `run_threaded()` メソッドは通常非常に単純で、`update()` メソッドで使用されるクラスプロパティを設定したり、`update()` メソッドが管理するクラスプロパティを返したりするだけです。

次に、スレッドパーツを車両ループに追加する例を示します。このパーツは TF-Mini 単一ビーム Lidar をシリアルポート経由で使用し、距離を報告します。入力引数はなく、出力は距離値だけです。`inputs=[]` はデフォルト値なので省略できます。

```python
if cfg.HAVE_TFMINI:
    from donkeycar.parts.tfmini import TFMini
    lidar = TFMini(port=cfg.TFMINI_SERIAL_PORT)
    V.add(lidar, inputs=[], outputs=['lidar/dist'], threaded=True)
```

以下は [TFMini パーツ](https://github.com/autorope/donkeycar/blob/main/donkeycar/parts/tfmini.py) のリストです。

```python
class TFMini:
    """
    TFMini および TFMini-Plus 距離センサー用のクラス。
    配線とインストール手順は https://github.com/TFmini/TFmini-RaspberryPi を参照してください。

    距離をセンチメートルで返します。
    """

    def __init__(self, port="/dev/serial0", baudrate=115200, poll_delay=0.01, init_delay=0.1):
        self.ser = serial.Serial(port, baudrate)
        self.poll_delay = poll_delay

        self.dist = 0

        if not self.ser.is_open:
            self.ser.close()  # まだ開いている場合は二重に開かないよう閉じる
            self.ser.open()

        self.logger = logging.getLogger(__name__)

        self.logger.info("Init TFMini")
        time.sleep(init_delay)

    def update(self):
        while self.ser.is_open:
            self.poll()
            if self.poll_delay > 0:
                time.sleep(self.poll_delay)

    def poll(self):
        try:
            count = self.ser.in_waiting
            if count > 8:
                recv = self.ser.read(9)
                self.ser.reset_input_buffer()

                if recv[0] == 0x59 and recv[1] == 0x59:
                    dist = recv[2] + recv[3] * 256
                    strength = recv[4] + recv[5] * 256

                    if strength > 0:
                        self.dist = dist

                    self.ser.reset_input_buffer()

        except Exception as e:
            self.logger.error(e)


    def run_threaded(self):
        return self.dist

    def run(self):
        self.poll()
        return self.dist

    def shutdown(self):
        self.ser.close()
```

> NOTE: TFMini パーツは自分でシリアルポートを管理していますが、シリアルポートから行単位のデータを読み取るには、ポートを自分で扱うのではなく [SerialPort パーツ](https://github.com/autorope/donkeycar/blob/main/donkeycar/parts/serial_port.py) を使うことをお勧めします。SerialPort パーツはシリアルポートの詳細をすべて処理して結果のデータを出力してくれるので、あなたのパーツはそのデータを入力として使うだけで済みます。

TFMini パーツでは `update()` メソッドがシリアルポートが開いている間ずっとループします。シリアルポートはコンストラクタで開かれ、`shutdown()` メソッドで閉じられます。スレッドパーツでは `update()` メソッドはほとんど無限ループのように、Python が実行時間を与える限り何度も実行されます。ここが車両ループよりもはるかに速く実行できる部分です。

スレッドパーツを使う理由は、パーツが車両ループより速く動作する必要がある場合や、デバイスにほぼリアルタイムで応答する必要がある場合です。`update()` メソッド内のループは Python インタプリタが許す限り速く動作し、通常は車両ループよりずっと速いです。`update()` メソッドはパーツのスレッドから呼ばれますが、`run_threaded()` メソッドはメインの車両ループスレッドから呼ばれることを理解することが重要です。つまり、この2つのメソッドは処理の途中で互いに割り込む可能性があります。

データの更新や読み取り、その他のクリティカルセクションを安全に分離して原子性を保つためには、ロックなど適切なスレッドセーフのパターンを使用すべきです。場合によってはリソースへ安全にアクセスするための Lock が必要だったり、複数行のコードを原子的に実行する必要があります。Python では代入が原子的であること（グローバルインタプリタロック、GIL の良いところ）を覚えておく価値があります。したがって次のコードは**原子的ではありません**。

```
x = 12.34
y = 34.56
angle = 1.34
```

なぜならこれらの代入の間でコードが割り込まれる可能性があるからです。次のコードは**原子的**です。

```
pose = (12.34, 34.56, 1.34)
```

このように、スレッド内で変更される可能性のある複合的な内部状態を持つ場合は、それをタプルにまとめればロックなしで原子的に読み書きできます。
