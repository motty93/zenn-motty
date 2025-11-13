---
title: "Linux MintでUSB2BTplusを使って有線キーボードを無線化する"
emoji: "⌨"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["keyboard", "usb2btplus", "linuxmint"]
published: true
---

![](/images/use_usb2btplus_on_linux/usb2btplus-main.jpg)

どうもてぃです。
私は現在ARCHISS Quattro TKLという某赤ちく○付きのキーボードを使用しています。
このキーボードは静音赤軸という打鍵音がとても静かで打ちやすくそれでもって赤軸ライクな打ち心地なため、私にとっては快適タイピングライフを送るためには必要不可欠なガジェットなのです。
ですが、このキーボードには致命的な弱点があります。

**有線オンリーなこと**です。

こんなに素晴らしいプロダクトなのに非常に惜しい。
いろいろ調べてみたら**USB2BTplus**という機器があるではないですか！
ということで、今回はUSB2BTplusを用いて有線を無線化した話をしたいと思います。
Linuxだと一癖ありましたので誰かの役に立てれば。

## USB2BTplusとは
>お気に入りのUSBキーボード/マウス/ゲームパッドなどUSB HIDデバイスをBluetoothに変換しPCやスマートフォン等の対応機器を操作可能とする変換アダプタ“USB2BT PLUS”。
>技適対応済みのBluetoothモジュールを本体に内蔵し手軽にUSBからHIDデバイスのBluetooth変換を楽しむことができるほか、キー置き換え機能や3台までのスマホ/PC切り替えに対応し、一層便利なアイテムになりました。 ...公式から引用

![](/images/use_usb2btplus_on_linux/usb2btplus-display.jpg)
そもそもBluetooth化できるのは魅力なのですが、**3台も接続できる**という点に惹かれました。
…値段は可愛くはなくAmazonで6900円前後だったので、Aliexpressで似たような商品を買おうか悩みましたが、ハズレを引くと怖いのとビットトレードワンの商品がかなり高品質とのことで、これが決定だとなり購入へと至りました。

## 前準備
- キーボードとUSB2BTplusのType-A端子を接続
- USB2BTplusのType-B端子とCIOのモバイルバッテリーを接続

:::message
先にwindowsPCに有線接続して専用のソフトウェアでデバイス切り替えスイッチを割り当てたので、もしかしたら先にwindows側で設定する必要があるかもしれません（まあLinux使ってるなら窓PC持ってると思うので問題ないですよね。）
:::

## 普通に接続してみる
1. USB2BTplusのデバイス切り替えスイッチをどれでも良いので1つ押す(Linux Mint専用ボタンになる)
2. ペアリングボタンを3秒押しペアリングモードへ
3. bluetoothのGUIで接続する

これだと失敗します。
理由は全くわからないですが、GUIだとうまくいかない。
接続できた！と表示されても同時に`br-connection-create-socket`と表示されてしまい、接続が解除された状態になります。

これはHuawei Ultrathin keyboardやMX ARGOを接続したときにも起きていたので、`bluetoothctl`コマンドを使用して接続させます。

試したのが以下。

## bluetoothctlを使って接続
`bluetoothctl`でbluetoothのコマンドラインを起動し、`scan on`とするとbluetooth機器を検索します。

![](/images/use_usb2btplus_on_linux/usb2btplus-address.png)

出てきたUSB2BTのアドレスに`connect`する。

![](/images/use_usb2btplus_on_linux/failed-connection.png)

接続できました！が、入力しても全く反応しない。
あとconnectしても成功するときと失敗するときがあり、かなり不安定です。
これも理由はわかりません。ログも出ませんでした…。

同じようにLinux Mint民はbluetoothに悩まされてる人が多いっぽいです。

https://forums.linuxmint.com/viewtopic.php?t=384120

bluetoothのモジュールを入れ直したら直った〜という記事も見かけましたが、以前オーディオやbluetooth周りのモジュールを削除してインストールし直したらPCが起動しなくなった実績を持つ私はそれらを実行することはできませんでした。
ドライバー周りも結構辛いイメージです。

とはいえ、試行錯誤の上、成功させました。

## 成功させる方法

- **（接続済みであれば）USB2BTをbluetooth画面で削除する**
- USB2BTをペアリングモードにする
- bluetoothctlでscan onし、USB2BTが出るまでおとなしく待つ
- **trustする**
- connectする
- **別のデバイス切り替えスイッチを押し、すぐにLinux Mintに割り当てたスイッチを押す**

重要なところは太文字にしてます。

trustすると接続時の成功率が格段に上がりました。
また、最後のスイッチ切り替え。
これをすることによって、入力が効くようになります（とても謎）

## 究極のデメリット

**接続が切れたら`成功させる方法`を順に踏む必要がある**

bluetoothの意味www
さあ、windowsで使おう！さあ、タブレットで使おう！と接続を切り替えたり、モバイルバッテリーの充電が切れたり、仕事の終わりに充電器を切って寝たり…といろんなdisconnectパターンがあると思いますが、どう頑張っても1から接続するしかないです。
とてもとても辛い。
仕事の前準備と思って今は毎日接続し直してますが、これをするために別のキーボードを出さなければならないという本末転倒なことをやっているので、いつか改善したいですね。

## おわりに
現状Linux MintでUSB2BTPlusは使えたもんじゃねぇ。
HHKB Studio買え。

...純Ubuntuでも今度試してみます。
お疲れ様でした。
