---
title: "Zennに投稿するために絵文字検索コマンドを作った"
emoji: "🍺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Golang", "emoji", "linuxmint"]
published: true
publication_name: ukiyocreate_dev
---

どうもてぃです。

普段はLinux mintで作業しており、🍎や🪟ユーザーのように簡単に絵文字が出せないです。
Mozcの初期設定を怠っている私のせいでもありますが、なかなかうまくいかないところもあり、Goでhttp requestと常時標準出力の素振りついでに作ってみました。

![](/images/emojjy_golang_emoji/emojjy.gif)

## 環境

```sh
$ go version
go version go1.21.0 linux/amd64
```

## リポジトリ

こちらです〜。

https://github.com/motty93/emojjy

emojjyと名付けました。可愛がってね。

### 使い方
`README.md`みてください。
見たらわかります。ついでに🌟していってください。

## 簡単解説

基礎的なことしかやってないので特に言及することはないのですが、

- 常時標準出力待機（go routine）
- 文字が入力されたらninja v1 emojiへAPI Request

ただそれだけです。

文字数稼ぎにコードをば。

```go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "strings"

    "github.com/joho/godotenv"
)

var (
    apiKey string

    eb []EmojiResponse
)

type EmojiResponse struct {
    Code      string `json:"code,omitempty"`
    Character string `json:"character,omitempty"`
    Image     string `json:"image,omitempty"`
    Name      string `json:"name,omitempty"`
    Group     string `json:"group,omitempty"`
    Subgroup  string `json:"subgroup,omitempty"`
}

func init() {
    if err := godotenv.Load(".env"); err != nil {
        log.Printf("godotenv load error: %v\n", err)
    }

    apiKey = os.Getenv("NINJA_EMOJI_API_KEY")
}

func apiRequest(name string) {
    uri := "https://api.api-ninjas.com/v1/emoji?name=" + name
    req, _ := http.NewRequest("GET", uri, nil)
    req.Header.Add("X-Api-Key", apiKey)

    client := new(http.Client)
    res, err := client.Do(req)
    if err != nil {
        log.Fatal(err)
        return
    }
    defer res.Body.Close()

    bytes, err := io.ReadAll(res.Body)
    if err != nil {
        log.Fatal(err)
        return
    }

    if err := json.Unmarshal(bytes, &eb); err != nil {
        log.Fatal(err)
        return
    }

    for _, e := range eb {
        fmt.Printf("%v%2s%v\n\n", e.Character, "", e.Code)
    }
}

func main() {
    ch := make(chan string)
    fmt.Println("Enter emoji name: ")

    go func(ch chan string) {
        reader := bufio.NewReader(os.Stdin)
        for {
            name, err := reader.ReadString('\n')
            if err != nil {
                log.Fatal(err)
                continue
            }

            name = strings.TrimSpace(name)
            switch name {
            case "exit":
                os.Exit(0)
            case "":
                fmt.Print("Enter emoji name: ")
            default:
                ch <- name
            }
        }
    }(ch)

    for {
        go func(input string) {
            apiRequest(input)
            fmt.Print("Enter emoji name: ")
        }(<-ch)
    }
}
```

## buildしてコマンド化したい

godotenvをなくしてapiKeyを直書きします。
んで、go build。ちなみにですが決してコードを公開しないように...公開してすぐ後悔です。

`func init`と`import`だけいじりbuildすればおｋ。以下がbuildする際のコードです。

```go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "strings"
)

var (
    apiKey string

    eb []EmojiResponse
)

type EmojiResponse struct {
    Code      string `json:"code,omitempty"`
    Character string `json:"character,omitempty"`
    Image     string `json:"image,omitempty"`
    Name      string `json:"name,omitempty"`
    Group     string `json:"group,omitempty"`
    Subgroup  string `json:"subgroup,omitempty"`
}

func init() {
    apiKey = "apiKey"
}

// 以下省略
```

### buildコマンド
`README.md`に書いてますが一応。
```sh
$ go build -o bin/emojjy main.go
```

これを `/bin` だったり `/usr/local/bin` とかにおいてあげれば `emojjy`コマンドとして呼び出せます🎉


## 終わり
Linux mintユーザーの皆さんもこれで楽にZennの記事が投稿できますね！
🍎や🪟ユーザーの皆様も暇なときに使ってみてください。

それではまた次の記事で、さらだばー。
