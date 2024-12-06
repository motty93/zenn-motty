---
title: "asdfでnodejsのライブラリリンクが切り替わらない件"
emoji: "🐣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nodejs","asdf"]
published: false
---

どうもてぃです。

普段asdfを使ってバージョン管理をしているのですが、ライブラリのリンクが切り替わらないという問題にぶつかりました。

firebase cliをnode v18.12.0、v22.5.1で入れていますが、v22.5.1に切り替えてfirebaseを実行してもv18.12.0のfirebaseが実行されてしまうといった問題です。

以下調査結果を書いていきます。

## 環境
- WSL2 Ubuntu 22.04
- nodejs: v18.12.0, v22.5.1
- asdf: v0.14.0-ccdd47d


## 調査
### v22.5.1で調査
node v22.5.1の状態でグローバルにfirebase cliをインストールし確認します。

```bash
$ node -v
v22.5.1

$ npm i -g firebase

$ firebase --version
No preset version installed for command firebase
Please install a version by running one of the following:

asdf install nodejs 22.5.1

or add one of the following versions in your config file at /home/motty93/.tool-versions
nodejs 18.12.0
```

v22.5.1のfirebase cliがインストールされていないというエラーが出ています。

でも実際にはfirebase cliは前のコマンドでインストールしています…。

パスを確認してみます。

```bash
$ which firebase
/home/motty93/.asdf/shims/firebase
```

`/home/motty93/.asdf/shims/firebase`になっています。問題なさそう。

ちなみに`npm i -g firebase`が実行された後、自動で`asdf reshim nodejs`が実行されています。

### v18.12.0で調査
`~/.tool-versions`へv18.12.0を追加しろとのことなので、このバージョンのfirebase cliが動くのか確認します。

```bash
$ asdf global nodejs 18.12.0

$ firebase --version
13.16.0
```

普通にfirebase cliは動いています…。

## 仮説
`~/.asdf/shims`にある`firebase`がv18.12.0のものを指している可能性があります。

### そもそもv22.5.1のfirebase cliはインストールされているのか

```bash
npm root -g
```

`/home/motty93/.asdf/installs/nodejs/22.5.1/lib/node_modules`を確認します。

```bash
$ ls /home/motty93/.asdf/installs/nodejs/22.5.1/lib/node_modules
corepack  firebase  heroku  ngrok  npm  prettier  typescript  typescript-language-server
```
firebase cli入ってますね。

やはり仮説はあっていそう。


### `~/.asdf/shims/firebase`を確認
`~/.asdf/shims/firebase`を確認してみます。

```bash
$ cat ~/.asdf/shims/firebase
#!/usr/bin/env bash
# asdf-plugin: nodejs 18.12.0
exec /home/motty93/.asdf/bin/asdf exec "firebase" "$@" # asdf_allow: ' asdf '
```
node v22.5.1をglobalで使っているので、`asdf-plugin: nodejs 18.12.0`というのがかなり怪しい。

