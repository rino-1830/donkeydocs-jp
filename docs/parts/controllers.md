# コントローラーのパーツ

## Webコントローラー

スマートフォンやブラウザから車を操縦できる標準コントローラーです。カメラのライブプレビューも表示されます。操作方法は次のとおりです:

1. バーチャルジョイスティック
2. 対応する加速度センサーを搭載したモバイル端末の傾き操作
3. Webアダプター経由の物理ジョイスティック(ブラウザ・OS・ジョイスティックの組み合わせによってサポート状況が異なります)
4. キーボードの'ikjl'キー入力

> 注意: 最近のiOSでは[Safariのモーションアクセスがデフォルトで無効](https://www.macrumors.com/2019/02/04/ios-12-2-safari-motion-orientation-access-toggle/)になりました。
 

## ジョイスティックコントローラー

ゲームコントローラーを使った方が操作しやすいという声も多く、これを実現するパーツが用意されています。

デフォルトのWebコントローラーは1行変更するだけで物理ジョイスティック入力に置き換えられます。標準ではOSのデバイス /dev/input/js0 を使用します。理論上はOSがこのようにマウントするジョイスティックであれば何でも利用できますが、実際にはジョイスティックの種類（純正品や互換品）、XBoxコントローラー、それを支えるBluetoothドライバーによって挙動が変化します。
デフォルトのコードは[Sony製PS3 Sixaxisコントローラー](https://www.ebay.com/sch/i.html?_nkw=Sony+Playstation+Dualshock+PS3+controller+OEM)で作成・テストされています。ほかのコントローラーも使用可能ですが、Bluetoothの設定を変えたり、軸やボタンを正しく合わせるための調整が必要になる場合があります。

### These joysticks are known to work:
### 動作確認済みのジョイスティック:
* [Logitech Gamepad F710](https://www.amazon.com/Logitech-940-000117-Gamepad-F710/dp/B0041RR0TW)
* [Sony PS3 Sixaxis OEM](https://www.ebay.com/sch/i.html?&_nkw=Sony+PS3+Sixaxis+OEM) (Jetson Nanoでは非対応)
* [Sony PS4 Dualshock OEM](https://www.ebay.com/sch/i.html?&_nkw=Sony+PS4+Dualshock+OEM)
* [WiiU Pro](https://www.amazon.com/Nintendo-Wii-U-Pro-Controller-Black/dp/B00MUY0OFU)
* [XBox Controller](https://www.amazon.com/Xbox-Wireless-Controller-Blue-one/dp/B01M0F0OIY)
これらはmyconfig.py内のCONTROLLER_TYPEを該当する識別子に設定することで有効になります(コメントアウトを解除してください)。汎用コントローラーは通常Xbox設定を使用できます。


> 注: 下記に記載のないコントローラーを使用する場合や、正しく動作しない場合、あるいは独自にボタン配置を変更したい場合は[新しいまたはカスタムゲームコントローラーの作成](./#creating-a-new-or-custom-game-controller)を参照してください。

### myconfig.pyを変更するか、--jsオプションを付けて実行する

```bash
python manage.py drive --js
```

Will enable driving with the joystick. This disables the live preview of the camera and the web page features. If you modify myconfig.py to make `USE_JOYSTICK_AS_DEFAULT = True`, then you do not need to run with the `--js`.
ジョイスティックで操縦できるようになります。この場合カメラのライブプレビューやWebページ機能は無効になります。myconfig.pyで`USE_JOYSTICK_AS_DEFAULT = True`と設定しておけば`--js`オプションは不要です。

## PS3コントローラー

### Bluetooth設定

この[ガイド](https://pythonhosted.org/triangula/sixaxis.html)に従ってください。'Accessing the SixAxis from Python'以降の手順は省略できます。念のため手順を以下にも記載します。

``` bash
sudo apt-get install bluetooth libbluetooth3 libusb-dev
sudo systemctl enable bluetooth.service
sudo usermod -G bluetooth -a pi
```

ユーザーグループを変更した後は再起動してください。

PS3をUSBケーブルで接続し、中央のPSロゴボタンを押します。次に、コマンドライン用のペアリングツールを取得・ビルドして実行します:

```bash
wget http://www.pabr.org/sixlinux/sixpair.c
gcc -o sixpair sixpair.c -lusb
sudo ./sixpair
```

bluetoothctl を使ってペアリングします

```bash
bluetoothctl
agent on
devices
trust <MAC ADDRESS>
default-agent
quit
```

USBケーブルを外し、中央のPSロゴボタンを押します。

Bluetooth接続したPS3リモコンが正常に動作しているか確認するには、`/dev/input/js0` の存在をチェックします:

```bash
ls /dev/input/js0
```

#### トラブルシューティング

Raspberry PiでBluetooth接続がうまくいかない場合、`bluetoothctl` で次のような表示が出ることがあります:

```text
[NEW] Controller 00:11:22:33:44:55 super-donkey [default]
[NEW] Device AA:BB:CC:DD:EE:FF PLAYSTATION(R)3 Controller
[CHG] Device AA:BB:CC:DD:EE:FF Connected: yes
[CHG] Device AA:BB:CC:DD:EE:FF Connected: no
[CHG] Device AA:BB:CC:DD:EE:FF Connected: yes
[CHG] Device AA:BB:CC:DD:EE:FF Connected: no
[CHG] Device AA:BB:CC:DD:EE:FF Connected: yes
...
[CHG] Device AA:BB:CC:DD:EE:FF Connected: yes
[CHG] Device AA:BB:CC:DD:EE:FF Connected: no
[bluetooth]#
```

以下のコマンドでLinuxカーネルとファームウェアを更新してみてください:

```bash
sudo rpi-update
```

その後、再起動します:

```bash
sudo reboot
```

### PS3 Sixaxisジョイスティックの充電

何らかの理由で、Bluetooth制御とOSドライバーが動作していない給電USBポートでは充電できません。スマートフォン用のUSB充電器では充電できないため、LinuxまたはMacのノートPCなどの電源付きUSBポートを利用してください。PSロゴボタンを押すとランプが点滅するはずです。

充電後は再度コントローラーをPiに接続し、PSロゴを押してからケーブルを抜き、再びペアリングします。

### PS3 Sixaxisジョイスティックのバッテリー交換

これらのコントローラーは古いものも多いため、[新しいバッテリー](http://a.co/5k1lbns)に交換することをおすすめします。カバーを外す際は注意してください。ネジを5本外し、グリップ間の上部にあるツメを折らないよう前側から開いて下側を前方に引くようにすると良いでしょう。

### LinuxでPS3を接続するとマウスが乗っ取られる問題

PS3ジョイスティックを接続するとマウス操作を乗っ取られることがあります。防ぐには次のコマンドを実行します:

```bash
xinput set-prop "Sony PLAYSTATION(R)3 Controller" "Device Enabled" 0
```

## PS4 DualShock 4 ワイヤレスゲームパッド
以下の手順はRaspberry Pi OS Busterを搭載したRaspberry Pi 3または4を想定しています。
DS4ゲームパッドは追加ソフトなしでBluetooth接続できます。Bluetoothdは起動時に自動で動作するシステムサービスで、bluetoothctlはデバイスの接続やペアリングを管理するプログラムです。
#### sudoを使わずBluetoothctlを利用するための設定
piユーザーをbluetoothグループに追加し、変更を反映させるため再起動します。
```bash
sudo usermod -a -G bluetooth pi
sudo reboot
```

#### PS4ゲームパッドをスキャンする
再起動後、bluetoothctlを起動してスキャナーをオンにし、Bluetoothデバイスを検索します。以下の例では実際のHEX文字列はあなたの環境で異なります。


```bash
bluetoothctl
<response> Agent registered
<response> [bluetooth]# 
scan on
<response>
[CHG] Controller BB:22:EE:77:BB:CC Discovering: yes
[NEW] Device 10:20:30:40:50:60 10-20-30-40-50-60
[NEW] Device 10:20:30:40:50:70 10-20-30-40-50-70
[NEW] Device 10:20:30:40:50:80 10-20-30-40-50-80
[NEW] Device 10:20:30:40:50:90 10-20-30-40-50-90
[NEW] Device 20:AA:88:44:BB:10 WHSCL1
```

数分待ってスキャナーが既存のBluetoothデバイスをすべて見つけるのを待ちます。その後、シェアボタンとPlayStationボタンを同時に長押ししてゲームパッドをペアリングモードにします。
ランプが二回点滅したら、新しい"Wireless Controller"の項目が表示されるはずです。
新しいWireless Controllerの行が現れます。

```bash
<response>
[NEW] Device 1C:AA:BB:99:DD:AA Wireless Controller
```
スキャンを終了して表示を止めます
```bash
scan off
```

#### Connect to your PS4 gamepad
You will now connect, pair and trust the PS4 gamepad wireless controller. Trusting the paired devices will allow you to reconnect to the device after the Raspberry Pi reboots. Copy the wireless controller address. You will
type CONNECT "your wireless controller address", TRUST "your wireless controller address". In this case, "your wireless controller address" is 1C:AA:BB:99:DD:AA

```bash
connect 1C:AA:BB:99:DD:AA
<response>
Attempting to connect to 1C:AA:BB:99:DD:AA
[CHG] Device 1C:AA:BB:99:DD:AA Connected: yes
[CHG] Device 1C:AA:BB:99:DD:AA UUIDs: 00001124-0000-1000-8000-00805f9b34fb
[CHG] Device 1C:AA:BB:99:DD:AA UUIDs: 00001200-0000-1000-8000-00805f9b34fb
[CHG] Device 1C:AA:BB:99:DD:AA ServicesResolved: yes
[CHG] Device 1C:AA:BB:99:DD:AA Paired: yes
Connection successful
```
The PS4 gamepad light should now be solid. Now TRUST the PS4 gamepad wireless controller.
```bash
trust 1C:AA:BB:99:DD:AA
<response>
[CHG] Device 1C:AA:BB:99:DD:AA Trusted: yes
Changing 1C:AA:BB:99:DD:AA trust succeeded
```
Type devices to see the paired-devices.

```bash
paired-devices
<response>
Device 1C:AA:BB:99:DD:AA Wireless Controller
```

Type quit or exit to quit the program bluetoothctl
```bash
quit
```

#### Use your PS4 gamepad wireless controller
After booting your pi, press playstation button once.  The light will flash for about 5 seconds and then turn solid.  If
the light goes off, try again. If this does work, run bluetoothctl and verify devices and paired-devices.

```bash
devices
<response>
Device 1C:AA:BB:99:DD:AA Wireless Controller

paired-devices
<response>
Device 1C:A0:B8:9B:DB:A2 Wireless Controller
```

If it fails to connect, while running bluetoothctl, press the playstation button once.  A good response will be:
```bash
<response>
[CHG] Device 1C:AA:BB:99:DD:AA Connected: yes
```

To disconnect the controller from the Raspberry Pi, press and hold the playstation button for 10 seconds.

## PS4 Controller (for Raspian Stretch)

The following instructions are based on [RetroPie](https://github.com/RetroPie/RetroPie-Setup/wiki/PS4-Controller#installation) and [ds4drv](https://github.com/chrippa/ds4drv).

#### Install `ds4drv`

Running on your pi over ssh, you can directly install it:

```bash
sudo /home/pi/env/bin/pip install ds4drv
```

#### Grant permission to `ds4drv`

```bash
sudo wget https://raw.githubusercontent.com/chrippa/ds4drv/master/udev/50-ds4drv.rules -O /etc/udev/rules.d/50-ds4drv.rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

#### Run `ds4drv`

```bash 
ds4drv --hidraw --led 00ff00
```

If you see `Failed to create input device: "/dev/uinput" cannot be opened for writing`, reboot and retry. Probably granting permission step doesn't take effect until rebooting.
Some controllers don't work with `--hidraw`. If that's the case try the command without it. `--led 00ff00` changes the light bar color, it's optional.

#### Start controller in pairing mode

Press and hold **Share** button, then press and hold **PS** button until the light bar starts blinking. If it goes **green** after a few seconds, pairing is successful.

#### Run `ds4drv` in background on startup once booted

```bash
sudo nano /etc/rc.local
```

paste:

```text
/home/pi/env/bin/ds4drv --led 00ff00
```

Save and exit. Again, with or without `--hidraw`, depending on the particular controller you are using.

To disconnect, kill the process `ds4drv` and hold **PS** for 10 seconds to power off the controller.

## XBox One Controller

### bluetooth pairing

This code presumes the built-in linux driver for 'Xbox Wireless Controller'; this is pre-installed on Raspbian, so there is no need to install any other drivers.  This will generally show up on /dev/input/js0.  There is another userland driver called xboxdrv; this code has not been tested with that driver.

The XBox One controller requires that the bluetooth disable_ertm parameter be set to true; to do this:

#### **Jetson Nano**
Adapted from: https://www.roboticsbuildlog.com/hardware/xbox-one-controller-with-nvidia-jetson-nano

1. Install these python libraries before we disable ertm.
```
sudo apt-get install nano
```

2. Add Non-root access to your input folder:
```
sudo usermod -a -G dialout $USER
sudo reboot
```

3. Install sysfsutils
```
sudo apt-get install sysfsutils
```
4.  Edit the config to disable bluetooth ertm
```
sudo nano /etc/sysfs.conf
```
- Append this to the end of the config
```
/module/bluetooth/parameters/disable_ertm=1
```
5. Reboot your computer
```
sudo reboot
```
6. Re-pair the Xbox One Bluetooth Controller
- Unpair (forget) the controller first if you already tried to pair it, then pair it again.  You can do this with the Bluetooth Manager GUI appliation that ships with Jetpack or if you are using command line, then use bluetoothctl:

  - Open terminal and type:
    ```
    bluetoothctl
    ```
  - then you should see the list of devices you have paired with and their corresponding MAC address. If you do not, type:
    ```
    paired-devices
    ```
  - To un-pair a device type (replace aa:bb:cc:dd:ee:ff with the MAC address of the device to un-pair):
    ```
    remove aa:bb:cc:dd:ee:ff
    exit
    ```
- Pair your device using either Bluetooth Manager GUI or bluetoothctl (see RaspberryPi OS instruction starting with `sudo bluetoothctl`)

Once paired you should have a solid light on the xbox button and a stable bluetooth connection.


#### **RaspberryPi OS**

* edit the file `/etc/modprobe.d/xbox_bt.conf`  (that may create the file; it is commonly not there by default)
* add the line: `options bluetooth disable_ertm=1`
* reboot so that this takes affect.
* after reboot you can verify that `disable_ertm` is set to true entering this
  command in a terminal:

  ```bash
  cat /sys/module/bluetooth/parameters/disable_ertm
  ```

* the result should print 'Y'. If not, make sure the above steps have been done correctly.

Once that is done, you can pair your controller to your Raspberry Pi using the bluetooth tool. Enter the following command into a bash shell prompt:  

```bash
sudo bluetoothctl
```

That will start blue tooth pairing in interactive mode. The remaining commands will be entered in that interactive session. Enter the following commands:

```text
agent on
default-agent
scan on
```

That last command will start the Raspberry Pi scanning for new bluetooth devices. At this point, turn on your XBox One controller using the big round 'X' button on top, then start the pairing mode by pressing the 'sync' button on the front of the controller.  Within a few minutes, you should see the controller show up in the output something like this;

```text
[NEW] Device B8:27:EB:A4:59:08 XBox One Wireless Controller
```

Write down the MAC address, you will need it for the following steps.  Enter this command to pair with your controller:

```text
connect YOUR_MAC_ADDRESS
```

where `YOUR_MAC_ADDRESS` is the MAC address you copied previously.  If it does not connect on the first try, try again.  It can take a few tries.  If your controller connects, but then immediately disconnects, your disable_ertm setting might be wrong (see above).  

Once your controller is connected, the big round 'X' button on the top of your controller should be solid white.  Enter the following commands to finish:

```text
trust YOUR_MAC_ADDRESS
quit
```

Now that your controller is trusted, it should automatically connect with your Raspberry Pi when they are both turned on.  If your controller fails to connect, run the bluetoothctl steps again to reconnect.

## Creating a New or Custom Game Controller

To discover or modify the button and axis mappings for your controller, you can use the [Joystick Wizard](/utility/donkey/#joystick-wizard). The Joystick Wizard will write a custom controller named 'my_joystick.py' to your mycar folder.  To use the custom controller, set `CONTROLLER_TYPE="custom"` in your myconfig.py.
