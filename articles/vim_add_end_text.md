---
title: "【vim】検索した文字を含む行末に文字を追加する"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vim", "正規表現"]
published: true
---

## TL;DR

```bash
# hogehogeを含む文字行の末尾に testtestを追加
:'<,>'s/^\(.*hogehoge.*\)$/\1 testtest/g
```

もしくは

```bash
:'<,>'s;hogehoge.*;& testtest
```

## 解説

1. 行の頭へ移動(gg)
2. visual mode
3. 行の末尾へ移動(shift+g)
4. コロン(:)でコマンドラインモード(自動で正規表現)
5. 正規表現(`/^\(.*hogehoge.*\)$/`)でhogehogeを含む行を取得
6. 取得した列は`\1`で受け取れるので、続けて追加したい文字列を入れる
7. 終わり


## 実践
今回は直近で触ってる`schema.prisma`を使います。

updatedAtカラムを含む行の末尾に`@updatedAt`を追加したかったので。

```prisma
model Health {
  id     String  @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  status Boolean @default(true)

  createdAt DateTime @default(now()) @map("created_at") @db.Timestamptz(6)
  updatedAt DateTime @default(now()) @map("updated_at") @db.Timestamptz(6)
}

model Book {
  id      String @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  title   String
  content String
  author  String
  userId  String @map("user_id") @db.Uuid

  createdAt DateTime @default(now()) @map("created_at") @db.Timestamptz(6)
  updatedAt DateTime @default(now()) @map("updated_at") @db.Timestamptz(6)
}

model User {
  id             String     @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  email          String?    @unique() @default("")
  phoneNumber    String?    @unique() @default("") @map("phone_number")

  createdAt DateTime @default(now()) @map("created_at") @db.Timestamptz(6)
  updatedAt DateTime @default(now()) @map("updated_at") @db.Timestamptz(6)
}
```
この子に対して…
```bash
:'<,>'s/^\(.*updatedAt.*\)$/\1 @updatedAt/g
```
を解説のとおりに実行すると


```prisma
model Health {
  id     String  @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  status Boolean @default(true)

  createdAt DateTime @default(now()) @map("created_at") @db.Timestamptz(6)
  updatedAt DateTime @default(now()) @map("updated_at") @db.Timestamptz(6) @updatedAt
}

model Book {
  id      String @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  title   String
  content String
  author  String
  userId  String @map("user_id") @db.Uuid

  createdAt DateTime @default(now()) @map("created_at") @db.Timestamptz(6)
  updatedAt DateTime @default(now()) @map("updated_at") @db.Timestamptz(6) @updatedAt
}

model User {
  id             String     @id @default(dbgenerated("gen_random_uuid()")) @db.Uuid
  email          String?    @unique() @default("")
  phoneNumber    String?    @unique() @default("") @map("phone_number")

  createdAt DateTime @default(now()) @map("created_at") @db.Timestamptz(6)
  updatedAt DateTime @default(now()) @map("updated_at") @db.Timestamptz(6) @updatedAt
}
```

## 終わり
個人的にはノーマルモード時に`#`を文字列上で押せばハイライトしてくれるので、その状態からコマンドラインモードに入ると`:'<,>'s/^\(.*updatedAt.*\)$/\1`まで入力してくれるといいんすけど。


## もっと簡単な方法があった
教えていただきました 🦍

```bash
:'<,>'s;updateAt.*;& @updatedAt
```

超かんたんだわ…感謝です 🦍
