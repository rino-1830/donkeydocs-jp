# IMU

IMU（慣性計測装置）はロボットにかかる慣性力を検知するパーツである。センサーにより異なるが、一般的には直線加速度と回転加速度を含み、時には磁力計により方位も取得できる。多くの場合温度も取得でき、これはセンサーの感度に影響する。

## MPU6050/MPU9250

これは安価で小型かつそこそこ精度の高いIMUであり、[Amazon](https://www.amazon.com/s/ref=nb_sb_noss_2?url=search-alias%3Dindustrial&field-keywords=MPU6050)などで入手できる。

MPU9250には追加で磁力計が内蔵されている。

* 通常はI2Cインタフェースを使用し、既定のPWM PCA9685ボードから連結して利用できる。この構成では電源供給も行われる。
* MPU6050: X、Y、Z方向の加速度、ジャイロスコープX、Y、Z、および温度を出力する。
* MPU6250: X、Y、Z方向の加速度、ジャイロスコープX、Y、Z、磁力計X、Y、Z、そして温度を出力する。
* チップに16ビットADコンバーターが内蔵されており、16ビットのデータ出力を行う。
* ジャイロスコープの範囲: ±250、500、1000、2000 度/秒
* 加速度範囲: ±2、±4、±8、±16g

### ソフトウェア設定

smbusをインストールする

* パッケージからインストールする場合:

``` bash
 sudo apt install python3-smbus
```

* もしくはソースからインストールする場合:

```bash
sudo apt-get install i2c-tools libi2c-dev python-dev python3-dev
git clone https://github.com/pimoroni/py-smbus.git
cd py-smbus/library
python setup.py build
sudo python setup.py install
```
MPU6050の場合:

`mpu6050`用のpipライブラリをインストール:

```bash
pip install mpu6050-raspberrypi
```

MPU9250の場合:

`mpu9250-jmdev`用のpipライブラリをインストール:

```bash
pip install mpu9250-jmdev
```

### 設定
`myconfig.py`に次の設定を追加する:

``` python
#IMU
HAVE_IMU = True
IMU_SENSOR = 'mpu9250'          # (mpu6050|mpu9250)
IMU_DLP_CONFIG = 3
```
`IMU_SENSOR` は使用するセンサーに応じて `mpu6050` または `mpu9250` を指定できる。

`IMU_DLP_CONFIG` によりIMUのデジタルローパスフィルター設定を変更できる。周波数を下げると（下記参照）高周波ノイズを除去できるが、IMUデータの遅延が増加する。
有効な設定値は0から6まで:

- `0` 250Hz
- `1` 184Hz
- `2` 92Hz
- `3` 41Hz
- `4` 20Hz
- `5` 10Hz
- `6` 5Hz

### MPU9250に関する注意
起動時にMPU9250ドライバーは加速度とジャイロのバイアスをゼロに校正する。通常この処理は10秒未満で終了するので、その間は車体に触れたり動かしたりしないこと。
Donkeyを起動する前に、車体を地面に置いておくこと。
