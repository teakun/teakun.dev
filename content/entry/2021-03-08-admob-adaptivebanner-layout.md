---
title: "【iOS】admobのアダプティブバナーでレイアウトが可変にならなかった問題"
date: 2021-03-08T00:03:52+09:00
draft: false
---

タイトルのまま。
基本的に[公式のマニュアル](https://developers.google.com/admob/ios/banner/adaptive?hl=ja)の通りにやれば間違いないがしょうもないハマりどころがあった。

## 起きた問題
- 前提: iOS, UIKit,autolayout
- 以下のように`adSize`を更新してもレイアウトが320*50で固定されてしまう

```swift
bannerView.adSize = GADCurrentOrientationAnchoredAdaptiveBannerAdSizeWithWidth(viewWidth)
```

## 原因
autolayoutでバナーのviewに対してwidth=320,height=50の制約を貼ってしまっていたため。元々のバナーの320*50の制約が残っていて`adSize`を更新してもその制約が優先されるようになっていた。
勝手に更新してくれるのかと思ったけどそんなことはない。

## 対応
width=320,height=50の制約を剥がす。そのままではviewのレイアウトが定まらずにエラーが出てしまうのでintrinsic sizeのPlaceholderに仮の値を入れておくとよい。
intrinsic sizeについては[この記事](https://qiita.com/shtnkgm/items/f0b189e4184fe6c90707)が詳しくて分かりやすい。
{{<figure src="/img/2021-03-08-001.png" class="center">}}

ちなみに[サンプルコード](https://github.com/googleads/googleads-mobile-ios-examples/tree/master/Swift/admob/AdaptiveBannerExample)を実際に開いてみるとintrinsic sizeを入れる方法で実装してあった。