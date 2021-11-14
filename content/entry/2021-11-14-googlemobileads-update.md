---
title: "GoogleMobileAdsアップデート(7.67.0 -> 8.12.0)"
date: 2021-11-14T16:29:01+09:00
draft: false
---

iOSの話。
久しぶりにGoogleMobileAdsSDKのアップデートをしたらIFが変わっていたのでメモ。

## interstatial
- https://developers.google.com/admob/ios/full-screen-migration?hl=ja
    - ベータ版のときのドキュメント
    - `GADInterstitialAdBeta`を`GADInterstitialAd`に読み替えるとそのまま使える
    - これまではinterstatialが読込中・読み込み完了の状態を持つようになっていたが、広告の読み込みが完了したときだけインスタンスが取得できるようになった
## banner
- デリゲートメソットのネーミングが刷新
```swift
# before
func adViewDidReceiveAd(_ bannerView: GADBannerView) {}
# after
func bannerViewDidReceiveAd(_ bannerView: GADBannerView) {}
```
手元の環境では大体対応するメソッドがあるので置き換えて完了した。