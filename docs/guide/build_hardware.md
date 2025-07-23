# Donkey\u00ae を組み立てる方法

&nbsp;

* [概要](build_hardware.md#overview)
* [必要な部品](build_hardware.md#parts-needed)
* [ハードウェア:](build_hardware.md#hardware)
  * [ステップ1: パーツを印刷](build_hardware.md#step-1-print-parts)
  * [ステップ2: パーツを整える](build_hardware.md#step-2-clean-up-parts)
  * [ステップ3: トッププレートとロールケージを組み立てる](build_hardware.md#step-3-assemble-top-plate-and-roll-cage)
  * [ステップ4: サーボシールドをRaspberry Piに接続](build_hardware.md#step-4-connect-servo-shield-to-raspberry-pi)
  * [ステップ5: Raspberry Piを3Dプリントのボトムプレートに取り付け](build_hardware.md#step-5-attach-raspberry-pi-to-3d-printed-bottom-plate)
  * [ステップ6: カメラを取り付け](build_hardware.md#step-6-attach-camera)
  * [ステップ7: すべてを組み立て](build_hardware.md#step-7-put-it-all-together)
* [ソフトウェア](install_software.md)

## 概要

最新のソフトウェアインストール手順は[ソフトウェアの手順](install_software.md)セクションで管理されています。車を組み立てた後は必ずその手順に従ってください。

## 車の選択

車を選ぶ方法は大きく二つあります。組み立て済みの車が欲しい場合は、Raspberry Pi 4 など必要なものがすべてそろった[Waveshare Piracer Pro](https://amzn.to/4eqxpyM)をおすすめします。付属のDonkeycarソフトウェアは古いものですが、[こちら](https://www.diyrobocars.com/2025/06/28/tips-for-installing-donkeycar-on-the-waveshare-piracer-pro/)の手順で簡単に最新版へ更新できます。

自分で車を組み立てたい場合は以下の手順に従ってください。

Donkeycarを作るにはまずベースとなるRCカーを選ぶ必要があります。ほとんどのRCカーで作成できますが、簡単に始めたいなら下記のWL Toys製かExceed製の車を購入するとよいでしょう。

*注意: 現在入手しやすいのはWL Toys 144010とHSP-94186のみです*

[WL Toys 144010](https://amzn.to/3yCVyBI)は現時点で最も入手しやすい車でしょう。ブラシレスモーターで高速ですが初心者には扱いが難しいかもしれません。ブラシモーター版の144011や144001もありますが、ステアリングサーボとESCを[こちらのESC](https://amzn.to/42uvmpt)と[こちらのサーボ](https://amzn.to/40vSDoi)に交換する必要があります。RCに慣れている、または工作が好きな方のみ行ってください。[この短い動画](https://www.youtube.com/watch?v=4LKDjoTKlaE)で組み立て方法が説明されています。アダプターは[Thingiverse](https://www.thingiverse.com/thing:2566276)からダウンロードして自分で印刷するか、外部のプリントサービスを利用してください。

別の選択肢として、やや入手しにくいことがありますがHSP 94186やExceedブランドの車があります。5種類がサポートされており、ほぼ同等と考えて構いません。Amazonで在庫切れの場合は[Exceedの公式サイト](https://www.nitrorcx.com/1rcelca.html)を確認してください。HSP-94186はExceed Magnet 1/16 Truckと同じもので、AliExpressでも購入できますが到着まで約1か月かかります。

*  Exceed Magnet [Blue](https://amzn.to/2BkBRka)、[Red](https://amzn.to/3qKksIC)
*  HSP-94186 [1/16 electric Truck](https://www.aliexpress.us/item/2255799890095973.html)
*  Exceed Desert Monster [Green](https://amzn.to/2MhOfn6)
*  Exceed Short Course Truck  [Green](https://amzn.to/2Bek0ew)、[Red](https://amzn.to/3cmCNkE)
*  Exceed Blaze [Blue](https://amzn.to/3cnP9ci)、[Yellow](https://amzn.to/2zOGthR)、[Wild Blue](https://amzn.to/3dkI4uD)、[Max Red](https://amzn.to/2TWOxE8)

これらの車は電気的には同一ですが、タイヤやマウントなど細部が異なります。Desert Monster、Short Course Truck、Blazeの3車種はアダプターが必要ですが、簡単に印刷するかDonkeyストアで購入できます。これらはブラシモーターで扱いやすく、荒れた路面にも強く価格も手頃なため標準的な組み立て車として推奨されます。

[この動画](https://youtu.be/UucnCmCAGTI)では（WL Toys車を除く）各車種の概要と組み立て方を確認できます。

上級者向けには「Donkey Pro」名義でさらに2台の1/10スケール車がサポートされています。サイズが大きく性能も高い分、価格もやや高めです。

*  HobbyKing Mission-D [入手先](https://hobbyking.com/en_us/1-10-hobbykingr-mission-d-4wd-gtr-drift-car-arr.html?affiliate_code=XFPFGDFDZOPWEHF&_asc=337569952)
*  Tamaya TT01 またはクローン [よくある互換品はこちら](https://amzn.to/2MdNLhZ) - 組み立てキットで販売されることが多く、他の2台より組み立てが難しいかもしれません。

[この動画](https://youtu.be/K-REL9aqPE0)ではそれぞれのモデルを説明しています。Donkey Proモデルはまだ十分にドキュメント化されていない点にご注意ください。

詳細やその他のオプションについては[サポートされている車種](/cars/supported_cars)を参照してください。

![donkey](../assets/build_hardware/donkey.png)

## 自作の車を作る

RCに詳しい、あるいは標準のDonkeyでは満足できない場合は、自分で車を作ることも可能です。参考として[自分で作る](../cars/roll_your_own.md)を参照してください。

## ハードウェア組み立てのビデオ概要

[この動画](https://www.youtube.com/watch?v=OaVqWiR2rS0&t=48s)では、標準的なDonkey Carの組み立て方に加え、Sombrero、Raspberry Pi、nVidia Jetson Nanoの説明も行っています。

[![IMAGE ALT TEXT HERE](../assets/HW_Video.png)](https://www.youtube.com/watch?v=OaVqWiR2rS0&t=48s)


## 必要な部品

以下の説明はRaspberry Pi用です。オプションのアップグレードについては後述のJetson Nano向け手順を参照してください。

| 部品説明 | リンク | 参考価格 |
|--------------------------------------------------------------------|------------------------------------------------|--------------|
| Magnet Car または代替車 | 上記「車の選択」で紹介した車を参照 | $92 |
| M2x6 ネジ (8本) | [Amazon](https://amzn.to/2ZSKa0D) | $4.89 |
| M3x10 ネジ (3本) | [Amazon](https://amzn.to/3gBQuzE) | $7.89 |
| USBバッテリー | [Anker 10,000 mAh](https://amzn.to/4dStvNr) | $26 |
| Raspberry Pi 5B 4GB | [Pi 5B](https://amzn.to/3AmRmqa) | $60 |
| MicroSDカード | [64GB](https://amzn.to/2XP7UAa) | $18.99 |
| 広角Raspberry Piカメラ | [Amazon](https://amzn.to/4bRHCRV) | $18 |
| メス-メス ジャンパワイヤ | [Amazon](https://amzn.to/36RiMlo)) | $7 |
| サーボドライバー PCA9685 | [Amazon](https://amzn.to/2BbVYkj) | $12 |
| 3Dプリント製ロールケージとトッププレート | 自分で印刷するか外部サービスを利用 | $xx |

* これらの部品が見つからない場合は、M2の代わりにM2.2、M2.3、または#4 SAEネジを、M3の代わりに#6 SAEネジを使用するなど多少の融通が利きます。機械ネジを代用することも可能です。
* この部品はAli Expressで約2～4ドルで購入できますが、発送に30～60日かかります。

### 追加のアップグレード

* **LiPoバッテリーとアクセサリー:** LiPoバッテリーはエネルギー密度が高く、放電特性も優れています。以下の図（Traxxas提供）を参照してください。

![donkey](../assets/build_hardware/traxxas.png)

| 部品説明 | リンク | 参考価格 |
|-------------------------------------------------------|------------------------------------------------|--------------|
| LiPoバッテリー | [hobbyking.com/en_us/turnigy-1800mah-2s-20c-lipo-pack.html](https://hobbyking.com/en_us/turnigy-1800mah-2s-20c-lipo-pack.html?affiliate_code=XFPFGDFDZOPWEHF&_asc=1096095044) または [amazon.com/gp/product/B0072AERBE/](https://www.amazon.com/gp/product/B0072AERBE/) | $8.94～$17 |
| LiPo充電器（上記バッテリーの充電に約1時間） | [charger](https://amzn.to/3gE6DVe) | $13 |
| LiPoバッテリーケース（破裂時の被害を防ぐため） | [lipo safe](https://amzn.to/2XkUhcV) | $8 |

## ハードウェア

Donkey Car Storeで部品を購入した場合はステップ3に進んでください。

### ステップ1: パーツを印刷

3Dプリンターを持っていない場合は[Donkey Store](https://donkeycar-701334.square.site)から部品を注文できます。私は黒のPLAで層高2mm、サポートなしで印刷しました。トップロールバーは逆さにして印刷する設計です。「Magnet」でない限りアダプターも印刷してください。

私は別の方法として層高0.3mm、ノズル径0.5mmでも印刷しました。こちらもサポートなしで、トップロールバーは逆向きに印刷します。

### ステップ2: パーツを整える

ほとんどの3Dプリント部品には仕上げが必要です。穴を開け直したり余分なプラスチックを取り除きましょう。

![donkey](../assets/build_hardware/2a.png)

特にロールバーの側面のスロットは下の写真のようにきれいにしてください。

![donkey](../assets/build_hardware/2b.png)

### ステップ3: トッププレートとロールケージを組み立てる

Exceed Short Course Truck、Blaze、Desert Monsterを使う場合は[この動画](https://youtu.be/UucnCmCAGTI)を参照してください。

この工程は比較的簡単で、3mmのタッピングネジを使ってプレートをロールケージに固定するだけです。

ロールケージをトッププレートに取り付ける際は、トッププレートの突起がロールケージ側を向くようにしてください。これで後から取り付ける機材がスムーズに収まります。

### ステップ4: サーボシールドをRaspberry Piに接続

PCA9685サーボコントローラーは最大16個のPWMデバイスを制御できます。Raspberry Pi（またはJetson Nano）の40ピンGPIOにI2Cピンで接続します。

* GPIO I2Cバス1
  * SDAはボードピン03
  * SCLはボードピン05
* 配線
  * 他のデバイスがI2Cバスを使っている場合、SDAとSCLは共有バス経由でも構いません
  * 3.3V VCCはGPIOの3.3Vピン（通常ボードピン01）から供給できます
  * 5V VINはモーターやサーボが大きな電流を必要とするためGPIOから供給しないでください。多くのESCが必要な電力を3ピンケーブル経由で提供します
  * すべてのGNDは共通にしてください。GPIOではボードピン09を使うと簡単です

```
---
    GPIO   ... PCA9685  ... 5v ... ESC ... Servo
    3v3-01 <---> VCC
    pin-03 <---> SDA
    pin-05 <---> SCL
    GND-09 <---> GND
                 VIN  <---> 5v   optional, see above
                 GND  <---> GND
                 CH-0 <---------> ESC
                 CH-1 <------------------> Servo
---
```

* 接続確認
  * PCA9685はI2Cバス1のアドレス0x40に表示されるはずです
  * Donkeycarにsshで入り`i2cdetect`でバス1を確認するとアドレス0x40にデバイスが見えます

```
---
    $ i2cdetect -y -r 1
         0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
    00:          -- -- -- -- -- -- -- -- -- -- -- -- --
    10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    40: 40 -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    70: UU -- -- -- -- -- -- --
---
```

Raspberry Piをボトムプレートに取り付けた後でも接続できますが、作業台で部品を並べた状態の方が見やすいでしょう。以下の写真を参考に接続してください。

![donkey](../assets/build_hardware/4a.png)

参考までにRaspberry Piのピン配置を示します。3.3V、SDA、SCL、GNDを接続しています。

![donkey](../assets/build_hardware/4b.png)

### ステップ5: Raspberry Piを3Dプリントのボトムプレートに取り付け

作業を始める前に、フラッシュ済みのSDカードを挿入して電子部品をテストしておくとよいでしょう。準備ができたら、基板にネジを通してトッププレートのネジ穴に固定します。M2.5x12mmのネジが最適な長さです。ネジの頭は上向き、ナットはトッププレートの裏側に置きます。EthernetとUSBポートは前方を向け、SDカードへのアクセスやカメラリボンケーブルの位置合わせを容易にします。

USBバッテリーはボトムプレートの裏側にケーブルタイかベルクロで固定します。

![donkey](../assets/build_hardware/5ab.png)

### ステップ6: カメラを取り付け

カメラをスロットに差し込みますが、ケーブル側から入れ、**レンズを押さないように**基板を押して挿入してください。

![donkey](../assets/build_hardware/assemble_camera.jpg)

カメラを取り外すときレンズを押したくなりますが、**絶対にレンズを押さず**コネクター部分を押してください。

![donkey](../assets/build_hardware/Remove--good.jpg) ![donkey](../assets/build_hardware/Remove--bad.jpg)

使用前にカメラレンズの保護フィルムやカバーを取り外してください。

![donkey](../assets/build_hardware/6a.png)

ケーブルの向きを間違えやすいので、写真をよく見て正しく接続してください。慣れない場合はYouTubeのチュートリアルも役立ちます。Raspberry Piの公式ドキュメントにも[カメラ接続方法](https://www.raspberrypi.com/documentation/accessories/camera.html#connect-the-camera)が掲載されています。

![donkey](../assets/build_hardware/6b.png)

後の[ソフトウェアをインストール](https://docs.donkeycar.com/guide/install_software/#step-2-install-software-on-donkeycar)するステップでカメラが正しく動作するか確認してください。詳細は[カメラ](https://docs.donkeycar.com/parts/cameras/#raspberry-pi)のドキュメント内「Make sure your camera works」を参照してください。

### ステップ7: すべてを組み立て

*** Desert Monster シャーシの場合は下の7Bセクションを参照してください ***
車体にロールバーアセンブリを取り付けます。付属のピンを使用します。

![donkey](../assets/build_hardware/7a.png)

次にサーボケーブルを車体に接続します。スロットルはチャンネル0、ステアリングはチャンネル1に接続します。

![donkey](../assets/build_hardware/7b.png)

これでハードウェアは完成です!

### ステップ7b: アダプターを取り付ける（Desert Monsterのみ）

Desert Monsterはボディ固定方法が他と異なるため、上記の2つのアダプターが必要です。まず既存のアダプターを外し、同じネジでカスタムアダプターを取り付けます。

![adapter](../assets/build_hardware/Desert_Monster_adapter.png)

取り付けが完了したらステップ7に戻ります。

## ソフトウェア

おめでとうございます！車を動かすための手順は[ソフトウェアの手順](install_software.md)を参照してください。

![donkey](../assets/build_hardware/donkey2.png)

> 当サイトはAmazonアソシエイト・プログラムの参加者です。このプログラムによりAmazon.comおよび関連サイトへのリンクを通じて紹介料を得ることがあります。
