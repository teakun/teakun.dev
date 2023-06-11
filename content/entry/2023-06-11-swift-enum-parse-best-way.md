---
title: "Swiftでの最高のenumパースを考える"
date: 2023-06-11T17:16:19+09:00
draft: false
---

APIから受け取ったJSONをパースし、enumに変換して利用するときの考慮ポイントを整理したい。


例えば次のようなECサイトで商品の状態を表すenumがあったとする。
```swift
enum ItemStatus: String, Decodable {
    case new
    case used
    case junk
}
```

絶対にこのどれかが返却されるならこのままでもいいけど、現実はケアしたいケースがいっぱいある。
- 新機能の追加で新しい種類が返却されるようになった
- 実は想定外の値が返却される仕様だった
- サーバサイドの不具合で全く関係のない文字列が入ってしまった

などなど。

状況に応じてenumのパースをどのように実装していくのがよいかを考えてみる。

# 考えられる実装パターン
取れる作戦を並べると以下のパターンになる。

- エラーに倒す
- 想定外を表現する専用のケースを用意する
- 既存にあるケースに倒す

## エラーに倒す
想定外の値が帰ってきた場合にパース処理自体を失敗にする。
値を厳密に扱いたい場合にやりがち。

```swift
enum ItemStatus: String, Decodable {
    case new
    case used
    case junk
    
    public init(from decoder: Decoder) throws {
        let stringValue = try decoder.singleValueContainer().decode(String.self)
        if let status = ItemStatus(rawValue: stringValue) {
            self = status
        } else {
            throw ParseError() // 好きなエラーを返す
        }
    }
}
```
リクエスト全体を失敗にしてしまうのでenumパースだけに問題があった場合でも全体が影響を受けてしまうが、絶対に想定しているケースだけを扱いたい場合にはこれ。
全部エラーに倒してしまうので、サーバサイドに更新があって新しいケースが追加されたとき強制アップデートなどの旧アプリのケアが必須になってしまうのがデメリット。
お金周りや住所など、クリティカルな情報向き。

## 想定外を表現する専用のケースを用意する

```swift
enum ItemStatus: String, Decodable {
    case new
    case used
    case junk
    case unknown(String) // 追加

    public init(from decoder: Decoder) throws {
        let stringValue = try decoder.singleValueContainer().decode(String.self)
        self = ItemStatus(rawValue: stringValue) ?? .unknown(value) // パースできなかったらunknownに倒す
    }
}
```
パースはできるが新しく処理が増えてしまうのでこのenumを利用する処理に追加の考慮が必要。
ユーザに対して表示するような情報であればその部分を非表示にしたり、専用の文言を用意したりなど。

実際の値をassociated valueで持たせるかどうかはお好み。
エラーで出力したいときは持たせてもいいけど、自分は持たない実装にすることが多いです。

## 既存にあるケースに倒す
想定外の値が来たときに全部既存にあるケースに倒してしまうパターン。

```swift

enum ItemStatus: String, Decodable {
    case new
    case used
    case junk

    public init(from decoder: Decoder) throws {
        let stringValue = try decoder.singleValueContainer().decode(String.self)
        self = ItemStatus(rawValue: stringValue) ?? .new // パースできなかったら既存のケースに倒す
    }
}
```
想定外の値が来てもパースできるし、ケースも増えないから処理の考慮も必要なしで一番楽ちん。
ただし、本当に想定外の値を既存のケースに倒してしまっていいのかは要検討。
上の例だと、想定外の値を`new`として扱うのはかなり怪しくて、本当はボロボロの中古品なのに新品として表示されてしまった！みたいな事故が起きそう。
もともとのレスポンスの中に"その他"を表すようなものがあったときはこのパターンが使いやすいです。


# まとめ
想定外の値をどれだけ厳密に扱いたいかでやり方を決めるのが良さそうです。

厳密: エラーに倒す > 専用のケースを作る > 既存に倒す

簡単: エラーに倒す < 専用のケースを作る < 既存に倒す

# おまけ
配列で返却される要素のうち一部の要素でenumでパースできないものが返却されたとして、パースできないものだけを除外すればいいのでは？となりがちであるが、これはアンチパターンであることが多い。
例えば商品のリストで50の要素が返却されたときに1つパースできないアイテムがあったとして、アプリ側でそれをなかったことにして49個にしてしまうみたいなケース。
これはページングの処理を実装するときに個数がずれるのでバグの原因になりがち。
一発で全ての要素を引くことが確定している場合や、タイムスタンプでページングを管理している場合はこの限りではない。
