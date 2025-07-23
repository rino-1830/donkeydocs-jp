# バーチャルレースリーグ

DIY Robocars の競技において次の段階に進みました。現在、特別イベントをオンラインで開催しています！ 世界中からの参加者を歓迎します。イベントは Chris Anderson と Donkeycar のメンテナによってスケジュールされます。ただし、これは Donkeycar 専用というわけではありません。引き続き読んでいただければ、Donkeycar フレームワークを使用してレースを行うかどうかで2通りの方法を案内します。

![race_previe](/docs/assets/virtual_race_league.jpg) 

レースの模様は Twitch で配信します。イベント告知で URL を確認してください。参加者は Zoom のグループチャットに参加します。Tawn がレースサーバーをホストし、Zoom/Twitch 経由で映像を共有します。うまくいくか見てみましょう。

## シムサーバーについて

[SDSandbox](https://github.com/tawnkramer/sdsandbox) というオープンソースプロジェクトをレース用シミュレータとして利用しています。これは [Unity](https://unity.com/) を使って 3D 環境を作成します。また、[NVidia PhysX](https://developer.nvidia.com/physx-sdk) オープンソース物理エンジンを使用して四輪車の動力学をシミュレートします。このシムはサーバーとしても機能し、TCP ポート 9091 を待ち受けます。ここでは JSON パケットを送受信します。API については後述します。

サーバーとのインターフェースには OpenAI GYM 形式のラッパーを使用します。このラッパーのプロジェクトは [gym-donkeycar](https://github.com/tawnkramer/gym-donkeycar) です。

サーバーは上記のソースからビルドすることもできますし、Ubuntu、Mac、Windows 用の [事前ビルド済みバイナリ](https://github.com/tawnkramer/gym-donkeycar/releases) を利用することもできます。これは Ubuntu 18.04、Mac 10.13、Windows 10 でテストされています。

## Donkeycar ユーザー向けセットアップ

Donkeycar フレームワークでレースをする場合は、[シミュレータのセットアップ](/docs/guide/deep_learning/simulator.md)のガイドに従ってください。映像での説明が役立つ場合は、[Windows Sim Setup Screen-Cast on Youtube](https://youtu.be/wqQMmHVT8qw) を参照してください。レース前の練習に利用しましょう。レース本番では、myconfig.py に次の 2 つの変更を加えます。

```
DONKEY_SIM_PATH = "remote"
SIM_HOST = "trainmydonkey.com"
```

このレースサーバーは常時稼働しているわけではありません。テストイベントやレース当日に起動します。レースの 1 週間前から毎晩 7pm～9pm（太平洋時間）に稼働させる予定です。サーバーが起動していない場合は Discord で声をかけてください。できるだけ対応します。

> 注: Donkey のモデルをトレーニングしたものの、Jetson Nano など依存関係のインストールが難しい環境で実行したい場合は、[この単体スクリプト](https://gist.github.com/tawnkramer/a74938653ab70e3fd22af1e4788a5001) を利用できます。モデルファイル名、ホスト名、車両名を指定するだけで、donkeycar や gym-donkeycar の依存なしにレースシムのクライアントとして動作します。

## Donkeycar を使わないユーザー向けセットアップ

独自のクライアントを作成したい場合、始めるための Python コードを用意しています。

* まずは、ご利用のプラットフォーム向けのシム[事前ビルドバイナリ](https://github.com/tawnkramer/gym-donkeycar/releases)をダウンロードし、任意の場所に展開してください。

* 続いて gym-donkeycar の Python プロジェクトをクローンしてインストールします。仮想環境を使用している場合は、先に有効化するのを忘れずに。
```bash
git clone https://github.com/tawnkramer/gym-donkeycar
pip install -e gym-donkeycar
```

* テストクライアントを取得します。Mac や Linux では次のように wget でダウンロードできます。
```
wget https://raw.githubusercontent.com/tawnkramer/sdsandbox/master/src/test_client.py
```

 * Windows の場合はブラウザーで [https://github.com/tawnkramer/sdsandbox/tree/master/src](https://github.com/tawnkramer/sdsandbox/tree/master/src) を開きます。
 * test_client.py を右クリックし「名前を付けてリンクを保存」を選び、任意の場所に保存してください。

 * シミュレータを起動し、メニュー画面まで進めます。
 * 以下のようにテストクライアントを実行します。
 
 ```
 python3 test_client.py
 ```

test_client.py を確認すると何をしているか分かります。SimpleClient クラスは指定したホストに接続し、試したいコースに応じてシーン読み込みコマンドを送信します。その後、車両の外観設定とカメラ設定を送り、更新ループに入ります。

num_clients 変数を 2 以上に変更して、シムが複数クライアントをどう処理するか試してみてください。

テストクライアントは time_to_drive = 1.0 秒の間ランダムなステアリングコマンドを送信し、その後終了します。

その間、テレメトリメッセージは SimpleClient::on_msg_recv に届きます。これらが表示される様子を確認してください。また、生成される 'test.png' を見ればカメラ映像の雰囲気が分かります。

コード中のコメントではカメラ設定について詳細に説明しています。独自のカメラ構成をお持ちなら、これらの設定で近い構成にできるはずです。

レース本番の際は次の変数を変更します。
```
host = "trainmydonkey.com"
```

自分の合図で車を走らせられるよう、コントロールを有効にしておいてください。ビデオチャットで昔ながらの「3、2、1、GO!」と掛け声をかけることになりそうです。

## ヘルプを得るには

学ぶことはたくさんあります。不明点があれば [Discord](https://discord.gg/JGQUU8w) に参加して助けを求めてください。#virtual-racing-league チャンネルも確認しましょう。





