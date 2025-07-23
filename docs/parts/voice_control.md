# Alexaサポート
## 概要
このパーツは、私たちが公開しているAlexaスキルと連携して動作します。
あなたがコマンドを話すと、Alexaスキルは当社がホストするサーバーにこのコマンドを転送し、
そこで一時的に保存します。このパーツを組み込み、正しく設定されたあなたのドンキー・カーは、
Alexaからの新しいコマンドがないか、当社のサーバーをポーリングします。

![概要](../assets/parts/voice_control/alexa_overview.png)


## デモ
下の画像をクリックすると、YouTubeで動画が開きます。

[![デモ](https://img.youtube.com/vi/Q3kYmy0yjmc/0.jpg)](https://www.youtube.com/watch?v=Q3kYmy0yjmc)

## 対応コマンド
- デバイスコードを報告
- 自動運転
- 減速
- 加速
- 停止/マニュアル

## はじめに
1. `Alexaアプリ`を使用し、「スキルとゲーム」に移動します。
2. "Donkey Car Control" を検索します。
3. このスキルを有効にします。
4. "Open car control and report device code" と話します。表示されたデバイスコードを
   メモしてください。
5. 以下の手順に従って、Raspberry Pi上で動作するドンキー・カーソフトウェアに
   このパーツをインストールします。


## インストール
このパーツをインストールするには、`manage.py`の`controller`設定直後に以下の行を追加します。
manage.py での設定例:
```python

if cfg.USE_ALEXA_CONTROL:
  from donkeycar.parts.voice_control.alexa import AlexaController
  V.add(AlexaController(ctr, cfg), threaded=True)
```

myconfig.py に次の設定を追加します:
```python
USE_ALEXA_CONTROL = True
ALEXA_DEVICE_CODE = "123456"
```

## コマンド
### 自動運転
`フレーズ: autopilot, start autopilot`

このコマンドを使用する場合、ドンキー・カーはモデルを読み込んで起動していることが前提です。
このコマンドはコントローラーの変数 `mode` を `local` に設定します。

### 減速 / 加速
`フレーズ: slow down, speed up, go faster, go slower`

このコマンドはコンストラクタから渡された `cfg.AI_THROTTLE_MULT` 変数を変更します。
コマンドを受け取るたびに、`AI_THROTTLE_MULT` が
0.05 ずつ増減します。

注意: このコマンドは `AI_THROTTLE_MULT` を変更するため、
`user` または `local_angle` モードでは加速しません。

### 停止/マニュアル
`フレーズ: human control, user mode, stop autopilot, manual`

このコマンドはコントローラーの変数 `mode` を `user` に設定します。

### デバイスコードを報告
`フレーズ: report device code, what is your device code, device code`

デバイスコードは、あなたのAlexaデバイスIDからハッシュ関数によって生成される6桁の数字です。
複数のAlexaデバイスから送られるコマンドを区別するため、
当社のサーバーへ送信されるコマンドには、このデバイスコードという識別子が必要です。
ドンキー・カーが新しいコマンドをポーリングする際、このデバイスコードを使用して
問い合わせを行います。

## バックエンド
こちらでWebサービスのソースコードを公開しています。

[https://github.com/robocarstore/donkeycar-alexa-backend](https://github.com/robocarstore/donkeycar-alexa-backend)

## 著作権
Copyright (c) 2020 [Robocar Ltd](https://robocarstore.com)
