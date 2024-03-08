---
title: "Linux mintでUSB2BTplusを使って有線キーボードを無線化する"
emoji: "⌨"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["keyboard", "usb2btplus"]
published: false
publication_name: ukiyocreate_dev
---

どうもてぃです。
私は現在ARCHISS Quattro TKLという某赤ちく○付きのキーボードを使用しています。
このキーボードは静音赤軸という打鍵音がとても静かで打ちやすくそれでもって赤軸ライクな打ち心地なため、私にとっては快適タイピングライフを送るためには必要不可欠なガジェットなのです。
ですが、このキーボードには致命的な弱点があります。

それが**有線オンリーなこと**です。

こんなに素晴らしいプロダクトなのに非常に惜しい。
いろいろ調べてみたら**USB2BTplus**という機器があるではないですか！
ということで、今回はUSB2BTplusを用いて有線を無線化した話をしたいと思います。
Linuxだと一癖ありましたので誰かの役に立てれば。

## USB2BTplusとは
>お気に入りのUSBキーボード/マウス/ゲームパッドなどUSB HIDデバイスをBluetoothに変換しPCやスマートフォン等の対応機器を操作可能とする変換アダプタ“USB2BT PLUS”。
>技適対応済みのBluetoothモジュールを本体に内蔵し手軽にUSBからHIDデバイスのBluetooth変換を楽しむことができるほか、キー置き換え機能や3台までのスマホ/PC切り替えに対応し、一層便利なアイテムになりました。 ...公式から引用

そもそもBluetooth化できるのは魅力なのですが、**3台も接続できる**という点に惹かれました。
…値段は可愛くはなくAmazonで6900円前後だったので、Aliexpressで似たような商品を買おうか悩みましたが、ハズレを引くと怖いのとビットトレードワンの商品がかなり高品質とのことで、これが決定だとなり購入へと至りました。

## 前準備
- キーボードとUSB2BTplusのType-A端子を接続
- USB2BTplusのType-B端子とCIOのモバイルバッテリーを接続

:::message
先にwindowsPCに有線接続して専用のソフトウェアでデバイス切り替えスイッチを割り当てたので、もしかしたら先にwindows側で設定する必要があるかもしれません（まあLinux使ってるなら窓PC持ってると思うので問題ないですよね。）
:::

## 普通に接続してみる
1. USB2BTplusのデバイス切り替えスイッチをどれでも良いので1つ押す(Linux mint専用ボタンになる)
2. ペアリングボタンを3秒押しペアリングモードへ
3. bluetoothのGUIで接続する

これだと失敗します。
理由は全くわからないですが、GUIだとうまくいかない。
接続できた！と表示されても同時に`br-connection-create-socket`と表示されてしまい、接続が解除された状態になります。

これはHuawei Ultrathin keyboardやMX ARGOを接続したときにも起きていたので、


![](/images/use_usb2btplus_on_linux/usb2btplus-address.png)
![](/images/use_usb2btplus_on_linux/failed-connection.png)
