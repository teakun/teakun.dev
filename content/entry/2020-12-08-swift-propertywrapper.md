---
title: "SwiftのPropertyWrapperを知る"
date: 2020-12-08T00:48:36+09:00
draft: false
---

PropertyWrapperについて1から勉強。

### 一番シンプルなパターン
まずは一番かんたんな例から。始めは色のRGBを表す場合を考えてみる。それぞれの値は0~255で表されていて、それ範囲外の値が来た場合は範囲に収まるようにしたい。ここではRGBを表すための`PropertyWrapper`として、`ColorValue`を定義する。

```swift
@propertyWrapper
struct ColorValue {
    private var value: Int

    init() {
        value = 0
    }
    
    var wrappedValue: Int {
        get {
            return value
        }
        set {
            value = min(255, max(0, newValue))
        }
    }
}

struct Color {
    @ColorValue var r: Int // @ColorValueのwrappedValueの型と変数の型が一致しないといけない
    @ColorValue var g: Int
    @ColorValue var b: Int
}

var color = Color()
color.r = 100
print(color.r) // 100

color.g = 999
print(color.g) // 355

color.b = -100
print(color.g) // 0
```

PropertyWrapperの定義は`wrappedValue`が必須になっている。PropertyWrapperを付けた変数にget・setをするときは`wrappedValue`のget, setを経由することになる。そのため、上の例では`color.g`に999を代入しても、ColorValueのwrappedValueのsetを経由するので値としては255が入る。

### 引数をつけられる

PropertyWrapperは初期化のタイミングで変数を入れることもできる。
例えばRGBを任意の範囲で表現できるようにしたいとき(ちょっと例が悪い)はこんな感じ。

```swift
import UIKit

@propertyWrapper
struct ColorValue {
    private let minValue: Int
    private let maxValue: Int
    
    private var value: Int
    
    init(minValue: Int, maxValue: Int) {
        self.minValue = minValue
        self.maxValue = maxValue
        value = 0
    }
    
    var wrappedValue: Int {
        get {
            return value
        }
        set {
            value = min(maxValue, max(minValue, newValue))
        }
    }
}

struct Color {
    @ColorValue(minValue: 0, maxValue: 1000) var r: Int
    @ColorValue(minValue: 0, maxValue: 1000) var g: Int
    @ColorValue(minValue: 0, maxValue: 1000) var b: Int
}

var color = Color()
color.r = 9999
print(color.r) // 1000
```

これで変数の定義のタイミングで任意のしきい値が入れられるようになった。

### projectedValueで追加の値を表現
`wrappedValue`に加えて`projectedValue`という値を定義することで、変数に機能を追加できるようになる。

さっきとは全く違う例を出してみる。今回は気温を表すためのPropertyWrapperを作る。このとき、気温の数字に加えて、氷点下かどうかをBoolで取り出したい。

```swift
@propertyWrapper
struct Temperature {
    private var value: Int
    var projectedValue: Bool = false
    
    init() {
        value = 22
        projectedValue = true
    }
    
    var wrappedValue: Int {
        get {
            return value
        }
        set {
            projectedValue = newValue > 0
            value = newValue
        }
    }
}
class WorldTemperture {
    @Temperature var tokyo: Int
}

let temperture = WorldTemperture()
temperture.tokyo = -22
print(temperture.$tokyo) // false
```

`projectedValue`を定義することで、その値を`$ + プロパティ名`で取り出すことができる。上の例だと`$tokyo`の部分。仕組みは分かったけどこの仕様はあんまりしっくり来てない。`$`をつけると全く違う型になるの不思議すぎない？

### おわり
ちょっと力尽きたので今日はここまで・・・もうちょっと理解したら応用編もまとめたい

## 参考資料
- https://docs.swift.org/swift-book/LanguageGuide/Properties.html
- https://www.2nd-walker.com/2020/02/12/swift-property-wrappers/
- https://dev.classmethod.jp/articles/property-wrappers/


