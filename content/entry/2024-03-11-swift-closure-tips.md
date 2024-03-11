---
title: "クロージャを使ったちょっと嬉しいSwiftの書き方"
date: 2024-03-11T22:43:12+09:00
draft: false
---

こういう場面。
```swift
if foo == true {
    if bar == true {
        hoge()
        fuga()
    }
} else {
    hoge()
    fuga()
}
```

`hoge()` `fuga()`を一緒に呼ぶ処理を何度も書きたくないけどそれをまとめた関数を作るほどでもない、というときクロージャを使うとちょっときれいに書ける。
```swift
 let closure = {
    hoge()
    fuga()
 }

 if foo == true {
    if bar == true {
        closure()
    }
 } else {
    closure() 
 }
 ```

ちょっと考えてみれば当然できる書き方ではあるのだけれど、意外とこれまでやってこなかったなと思ってメモ。