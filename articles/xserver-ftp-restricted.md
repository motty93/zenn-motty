---
title: "Xserverで接続しているネットワーク全体をFTP接続許可する方法"
emoji: "🖥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["xserver", "ftp", "network"]
published: true
publication_name: ukiyocreate_dev
---


どうもてぃです。
グループ会社のwordpressをXserverに変更しFTP制限設定を行ったところ、フリーアドレスの弊グループ会社ではかなり不便とのことで連絡をいただきまして、仕方なく対応いたしました。
ほんの少しクセがあったので、誰かの助けになれば幸いです。

## XserverのFTP制限設定
>FTPでの接続が可能なIPアドレスを設定し、FTPからのアクセスを制限します。
>これにより、不正アクセスやファイル改ざんのリスクを低減できます。
>許可設定にIPアドレスを設定した場合、設定されたIPアドレスからのアクセスのみを許容し、それ以外からのアクセスはパスワードが一致した場合でも接続を拒否します。

wordpressの管理画面にアクセスする際にもこれを設定していると設定外のIPアドレスではアクセスできなくなり、Xserver専用のエラーページが表示されます。
とても良い機能ですね。

今回はこれを設定している前提で話を進めます。

## CIDR表記が管理画面で使用できない
![](/images/xserver-ftp-restricted/cidr-ip-error.png)

会社のネットワークはいくつかあるのですが、その一つ（ここでは`111.11.11.0`を例とする）をCIDR表記で登録して全体を許可しようとしましたが、そもそもCIDR表記で登録できませんでした。
純粋なIPアドレスしか許容されていないようです。

## CIDR表記したければ .ftpaccessに直接書く
Xserverに割り当てられたユーザーフォルダの直下に `.ftpaccess`があります。
ここに直接CIDR表記で書きましょう。
`/home/xb11111/.ftpaccess`にあるはず(`xb11111`は自分の環境に置き換えてください)なので、エディタで開いて追記します。
```.ftpaccess
<Limit ALL>
Order Allow, Deny
Deny from all
Allow from 111.11.11.0/24
</Limit>
```

こうすることでCIDR表記でipアドレスを許可できます。

ちなみに管理画面のFTP制限設定で確認するとCIDR表記が抜けた状態で表示されます。
設定自体は`.ftpaccess`が優先されるようですね。

## 終わり
最近wordpressばっかりで楽しくないです。
Goで開発してぇー。
おつでした。
