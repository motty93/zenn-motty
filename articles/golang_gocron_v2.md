---
title: "gocron v2で定期実行を走らせる"
emoji: "🦍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Golang", "cron"]
published: true
---

こんにちもてぃ。

つい先日、年末年始に [gocron](https://github.com/go-co-op/gocron)がv2になりv1のときのように単純なcronが使えなくなったので、調査しつつ実装してみました。

## 環境
- go version 1.21.0 linux/amd64

## 実装

コードはこちらです。

https://github.com/motty93/gocron-v2


上記ではservicesにまとめているので、もっと簡易に書くと...

```go
import (
    "fmt"
    "http"
    "time"

    "github.com/go-co-op/gocron/v2"
)

func task() {
    fmt.Println("task!")
}

func main() {
    jst, err := time.LoadLocation("Asia/Tokyo")
    if err != nil {
        log.Fatal(err)
        return
    }

    ns, err := gocron.NewScheduler(gocron.WithLocation(jst))
    if err != nil {
        log.Fatal(err)
        return
    }

    ns.Start()

    nj, err := ns.NewJob(
        gocron.CronJob("0 9 * * 1-5", false),
        gocron.NewTask(task),
    )
    if err != nil {
        log.Fatal(err)
        return
    }

    // ぶっちゃけここのprintは特に意味はない
    fmt.Printf("job ID: %v\n", nj.ID)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

## ちょい解説
v1ではもっとシンプルでした。以下の記事を紹介します。

https://qiita.com/o-magari_kun/items/4cdfda0e9d00cbad3160

コードを引用させていただきます。

```go
scheduler := gocron.NewScheduler(time.Local)
scheduler.Every(1).Day().At("00:40").Do(func)
```

上記のように、v1ではNewSchedulerに対してtimeを渡せばよかったのですが、v2ではgocron.SchedulerOptionを渡さないといけなくなりました。

```go
func NewScheduler(options ...SchedulerOption) (Scheduler, error) {
// 長いので省略
}

type SchedulerOption func(*scheduler) error

type scheduler struct {
    shutdownCtx      context.Context
    shutdownCancel   context.CancelFunc
    exec             executor
    jobs             map[uuid.UUID]internalJob
    location         *time.Location
    clock            clockwork.Clock
    started          bool
    globalJobOptions []JobOption
    logger           Logger

    startCh            chan struct{}
    startedCh          chan struct{}
    stopCh             chan struct{}
    stopErrCh          chan error
    allJobsOutRequest  chan allJobsOutRequest
    jobOutRequestCh    chan jobOutRequest
    runJobRequestCh    chan runJobRequest
    newJobCh           chan newJobIn
    removeJobCh        chan uuid.UUID
    removeJobsByTagsCh chan []string
}
```

そのためv2対応として、`time.LoadLocation`を使用してlocationを渡しています。

```go
jst, err := time.LoadLocation("Asia/Tokyo")
if err != nil {
    log.Fatal(err)
}

ns, err := gocron.NewScheduler(gocron.WithLocation(jst))
if err != nil {
    log.Fatal(err)
}
```

クソめんどくさいですね。

そしてv1ではschedulerに対してEveryメソッドが生えてますが、v2ではそれが消えてます...。
以下画像はservice内のコード補完です。

![method](/images/golang_gocron_v2/method.png)

なんで消したんでしょうね。
理由はよくわかりませんが、[公式のドキュメント](https://github.com/go-co-op/gocron?tab=readme-ov-file#quick-start)を頼りに実装したのが以下です。

```go
nj, err := ns.NewJob(
    gocron.CronJob("0 9 * * 1-5", false),
    gocron.NewTask(task),
)
if err != nil {
    log.Fatal(err)
}
```

`gocron.Scheduler.NewJob`を使用して第1引数に `gocron.JobDefinition`、第2引数に関数を`gocron.NewTask`でラップしたものを渡しています。

公式では`JobDefinition`で`gocron.DurationJob`を使用していますが、今回はcron目的だったので `gocron.CronJob`を使用してます。

公式だと第2引数の関数の渡し方が非常にわかりにくいですが、別関数を渡すようにすればよりシンプルで見やすくなりました。
天才ですね。


## つまづいたところ
### defer shutdown

どこの記事でみたかもう忘れてしまいましたが、実行したjobをdefer shutdownしていたので同じように実装してました。

```go
import (
    "fmt"
    "http"
    "time"

    "github.com/go-co-op/gocron/v2"
)

func task() {
    fmt.Println("task!")
}

func main() {
    jst, err := time.LoadLocation("Asia/Tokyo")
    if err != nil {
        log.Fatal(err)
        return
    }

    ns, err := gocron.NewScheduler(gocron.WithLocation(jst))
    if err != nil {
        log.Fatal(err)
        return
    }

    ns.Start()
    defer ns.Shutdown() // ここ！！！！

    nj, err := ns.NewJob(
        gocron.CronJob("0 9 * * 1-5", false),
        gocron.NewTask(task),
    )
    if err != nil {
        log.Fatal(err)
        return
    }

    // ぶっちゃけここのprintは特に意味はない
    fmt.Printf("job ID: %v\n", nj.ID)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

`defer ns.Shutdown()`を記述していると何故か指定した時刻に実行されないので、削除したところうまく行きました。
そもそも定期実行で止めることはないのでdefer shutdownは不要です。
なぜうまく行かなかったかに関してはまた気分がのったときに調査しようとおもいます（たぶんしないやつ）


### copilot & chatGTPが嘘をつく
めちゃくちゃ嘘つきますし、全然動かないコードを補完してくれました。

1. copilotに実装してくれる
2. LSPがエラーを吐く
3. エラー内容を修正する
4. その先の実装ををchatGPTに聞く
5. その内容を参考にコードを修正する

2~5をループザループ。

やり取り画像を一応。

![chatgpt](/images/golang_gocron_v2/chatgpt.png)

リリースされたばかりだからかgocron v2の情報がまだ学習されてないようです。
もっとがんばってね。
ぶっちゃけめっちゃ辛かった。

## おわりに
とても便利なAIコーディングですが、最新の情報は自分でキャッチアップしましょう。
なんでもかんでもchatGPTやcopilotに頼ると私みたいに足元すくわれます。
私のpublicコードが学習されて皆様のgocron v2ライフが素晴らしいものとなりますように…。
