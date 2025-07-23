# Windows

Donkey Car はかつて Windows への直接インストールをサポートしていましたが、現在は WSL を利用する方法を推奨しています。
## Windows(WSL) に Donkeycar をインストールする

Windows Subsystem for Linux (WSL) を利用すると、従来の仮想マシンやデュアルブートのような負荷なしに、ほぼそのままの GNU/Linux 環境を Windows 上で利用できます。

* [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) をインストールします。
  1. Windows 10 を利用している場合（Windows 11 では不要）、設定 > アプリ > プログラムと機能 > Windows の機能の有効化または無効化 から "Windows Subsystem for Linux" を有効にします。
  2. Microsoft Store から Linux ディストリビューション（最新版の [Ubuntu](https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6?activetab=pivot:overviewtab) を推奨）をダウンロードします。
  3. Ubuntu アプリを起動して初期設定を行います。

* スタートメニューの Ubuntu からターミナルを起動します

* パッケージを更新し、pip と xclip をインストールします:

```bash
sudo apt-get update
sudo apt install python3-pip
sudo apt-get install libmtdev1 libgl1 xclip
```
* `.bashrc` に `LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6` を追加し、読み込み直します。
  ```bash
  source ~/.bashrc
  ```

ここまで終わったら、[Ubuntu 用手順](http://docs.donkeycar.com/guide/host_pc/setup_ubuntu/) に切り替えて続行してください。

* UI 実行時のトラブルシューティング

NVIDIA グラフィックカードで Donkey UI を使った際に画面がぼやけて表示される場合は、設定を変更してください。
設定画面で GPU の 3D レンダリングモードを
`実行中のプログラムに任せる`
から `自分で設定する: 品質` に変更します。

---
## 次に [Donkeycar にソフトウェアをインストール](/guide/install_software/#step-2-install-software-on-donkeycar) しましょう
