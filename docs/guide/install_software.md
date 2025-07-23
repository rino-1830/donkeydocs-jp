# ソフトウェアのインストール

* [概要](#概要)
* ソフトウェア:
    * [ステップ1: ホストPCにソフトウェアをインストール](install_software.md#step-1-install-software-on-host-pc)
    * [ステップ2: Donkeycarにソフトウェアをインストール](install_software.md#step-2-install-software-on-donkeycar)
* [Donkeycarアプリケーションの作成](/guide/create_application/)

## 概要

DonkeycarはホストPCにインストールするコンポーネントがあります。これはノートPCでもデスクトップマシンでも構いません。高性能である必要はありませんが、より高速なCPU、より多くのRAM、そしてNVidia GPUがあると役立ちます。SSDのハードドライブは学習時間に大きな影響を与えます。

Donkeycarのソフトウェアコンポーネントは、選択したロボットプラットフォームにインストールする必要があります。Raspberry Pi と Jetson Nano にはセットアップドキュメントがありますが、Jetson TX2、Friendly Arm SBC、またはほぼすべてのDebian系SBC（シングルボードコンピュータ）でも動作することが知られています。

インストール後はテンプレートからDonkeycarアプリケーションを作成します。これには、あなたの用途に合わせてカスタマイズできるコードが含まれています。心配しないでください、有用なデフォルトを用意してあるので、すぐに始められます。

次に、あなたの運転スタイルを基にDonkeycarを自律走行させるための学習を行います。これは、一般的に行動クローンと呼ばれる教師あり学習手法を使用します。

<iframe width="560" height="315" src="https://www.youtube.com/embed/BQY9IgAxOO0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Donkeycarを自動運転させる方法はこれだけではありませんが、最小限のハードウェアと知識で始められます。その後、このDonkeycarというAIモバイルラボで他の手法を試すことができます。

## ステップ1: ホストPCにソフトウェアをインストール

行動クローンを用いてDonkeyを操作する場合、ロボットで収集したデータから機械学習モデルをトレーニングするためのホストPCを用意する必要があります。お使いのOSに合ったセットアップを選んでください。

* [LinuxホストPCのセットアップ](host_pc/setup_ubuntu.md)
![donkey](/assets/logos/linux_logo.png)
* [WindowsホストPCのセットアップ](host_pc/setup_windows.md)
![donkey](/assets/logos/windows_logo.png)
* [MacホストPCのセットアップ](host_pc/setup_mac.md)
![donkey](/assets/logos/apple_logo.jpg)

## ステップ2: Donkeycarにソフトウェアをインストール

このガイドは、Raspberry Pi または Jetson Nano 上で Donkeycar を動かすためのソフトウェアをセットアップする方法を解説します。お持ちの SBC（シングルボードコンピュータ） に合わせたセットアップを選んでください。

* [RaspberryPiのセットアップ](robot_sbc/setup_raspberry_pi.md)
![donkey](/assets/logos/rpi_logo.png)

* [Jetson Nanoのセットアップ](robot_sbc/setup_jetson_nano.md)
![donkey](/assets/logos/nvidia_logo.png)

## 【オプション】RPi カメラの代わりに Intel Realsense T265 ローカライゼーションセンサーを使用する

詳細は[こちら](/guide/robot_sbc/intelt265)を参照してください。

## 【オプション】Jetson NanoでTensorRTを使用する

詳細は[こちら](/guide/robot_sbc/tensorrt_jetson_nano)を参照してください。

## 次へ:
[Donkeycarアプリケーションを作成する](/guide/create_application)。
