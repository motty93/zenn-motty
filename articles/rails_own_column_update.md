---
title: "自身のカラムに値を追加して更新したいとき"
emoji: "🚆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rails"]
published: false
---

どうもてぃ。

## TL;DR

CONCAT関数を使用します。
```ruby
User.update_all("name = CONCAT(name, 'v2')")
```

## よくやる方法

```ruby
User.where(confirmed: true).find_each do |user|
  user.update(name: "#{user.name}v2")
rescue => e
  puts e
  next
end
```
