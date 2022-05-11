---
title: "XcodeのカスタムテンプレートがSwiftPMの中で使えなかったので直す"
date: 2022-05-11T23:32:34+09:00
draft: false
---

Xcodeでの開発効率向上のために用意したカスタムテンプレートが、ローカルに置いたSwiftPMの中で使えない問題があったのでそれに対応したメモ。
自分の場合、ローカルにおいたSwiftPMはマルチモジュール構成のために用意したもの。

### 前提
- 環境
    - Xcode13.3.1
- 起きていた問題
    - ローカルに置いたSwiftPMのSourceに対してNew File → Choose a template your new fileで表示されるテンプレートの一覧に独自のテンプレートが表示されない
    - アプリ本体に対してテンプレートを選択しようとする場合は、独自に用意したテンプレートは表示される
    - (テンプレートの追加は概ねこの記事のとおりにやっている https://zenn.dev/paraches/articles/xcode-custom-template)
### 原因・対応
SwiftPMの中でテンプレートを利用したい場合は、TemplateInfo.plistに専用のパラメータを追加しておく必要があった。
`SupportsSwiftPackage`というパラメータがあるのでこれをtrueにしておくとSwiftPMからもカスタムテンプレートが利用できる。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>SupportsSwiftPackage</key>
	<true/>
    …
</dict>
</plist>
```

独自に追加したテンプレートだけが表示されないのが不思議でデフォルトで用意されているコードテンプレートを覗いてみたところ、いかにも怪しいパラメータを見つけることができて解決した。
デフォルトのテンプレートは`/Applications/Xcode.app/Contents/Developer/Library/Xcode/Templates/File\ Templates/`にあるのでこれを参考にする。
Template.infoの書き方をちゃんと理解したかったけど、公式のドキュメントがないようでちょっと厳しい。。
