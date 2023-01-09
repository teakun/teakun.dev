---
title: "詳解SwiftPM"
date: 2023-01-08T22:12:03+09:00
draft: false
---

Swift Package Manager (以下SwiftPM)中心の構成が最近流行っているが、そのためにはSwiftPMの仕組みや用語を改めてしっかりと理解しておきたい。
公式のドキュメントを参考にしつつ自分が知っていることをまとめてみる。

# SwiftPMとは
## 概要
SwiftPMとはなにか。公式サイトにはこのようにある。
> Swift Package Manager は、Swift コードの配布を管理するためのツールです。依存関係のダウンロード、コンパイル、およびリンクのプロセスを自動化するために、Swiftビルドシステムと統合されています。

SwiftPM自体はツールの名前で、基本的には他の人が作ったライブラリの導入・管理に利用される。
SwiftPMはSwiftにデフォルトでついてくるので余計な外部ツールを入れる必要がないのが嬉しい。
これまではGitHubなどで配布されているライブラリを使うためにはcocoapodsやcarthageなど、外部のツールを利用するのが一般的だった。
出たばかりの頃は不安定だったがだいぶ安定してきてきており、最近はほぼデファクトスタンダートと言ってよい状態。

# 基本的な概念
## Module
[importができる単位](https://speakerdeck.com/d_date/swift-package-centered-project-build-and-practice?slide=19)。SwiftPMによらずcocoapodsやcarthageでもModuleが作られる。
SwiftPMの公式のドキュメントを読んでいるとModuleという言葉が頻発するけどSwiftPMのクラスにModuleというのは存在しない。
## [Package](https://developer.apple.com/documentation/packagedescription/package)
SwiftPMにおける一番大きな単位。PackageはソースファイルとPackage.swiftと呼ばれるマニフェストファイルから構成される。
パッケージは1つ以上のTargetを持っている。
## [Target](https://developer.apple.com/documentation/packagedescription/target)
コードを1つにまとめた単位。この単位で名前空間ができて、どの部分を外部から使えるようにするかを決める。publicを付与したものが外部から参照できる。
Target同士は依存関係を持つことができる。SwiftPMにおいてはModuleに近い存在。
Module/Product/Targetの関係がややこしいなと思っているけど[フォーラム](https://forums.swift.org/t/module-vs-product-vs-target/27640)に同じようなことを質問している人がいた。
## [Products](https://developer.apple.com/documentation/packagedescription/product)
Packageが提供するものを定義して、他のパッケージから見えるようにしたもの。定義できる種類は以下の3種類。
- Library: 大体がこれ、コードをライブラリとして提供する場合はこちら。
- Plugin: ビルドツールなど。Swiftlintなどの。
- Executable: これは使ったことない。実行ファイル？
## [Dependency](https://developer.apple.com/documentation/packagedescription/target/dependency)
モジュール(Target)間の依存関係のこと。依存関係はさらに依存関係を持つことができる。
この依存関係は循環してはいけない。

# 導入
SwiftPMで提供されている外部のライブラリを利用したいとき、導入方法は大きく分けて2つある。
1. Xcodeのプロジェクト上から追加
2. ローカルで追加したSwiftPMのPackage経由で追加

一般的に紹介される方法は1で、ネットで検索すると基本的にはこれが紹介されている。
これはXcode上からポチポチするだけなので非常に簡単。
詳細なやり方は[このあたり](https://qiita.com/hironytic/items/09a4c16857b409c17d2c)が詳しい。

SwiftPM中心の構成にする場合は基本的に2を採用することになる。
1の方法だとライブラリを追加したときの変更がxcodeprojに反映されるので変更の差分が読みにくいが、2だとPackage.swiftに反映されるので変更が追いやすい。
1の方法はネットを探せばいっぱいでてくるのでこの記事では2の導入方法を紹介する。

## プロジェクトへのSwiftPMの追加
プロジェクトにSwiftPMのPackageを追加するところまでを紹介。
まずは対象のプロジェクト上でNew→Packageを選択。
{{<figure src="/img/2023-01-08-001.png" class="center">}}
適当な名前をつけて、Add toとGroupにプロジェクト名を選択してCreateする。
{{<figure src="/img/2023-01-08-002.png" class="center">}}
これでプロジェクト上にSwiftPMのPackageが配置される。
{{<figure src="/img/2023-01-08-003.png" class="center">}}
次はプロジェクト側からSwiftPMのモジュールを参照する設定を追加する。
xcodeproj → targets → general → Framework, Libraries, and Embedded Contentで+を選択、作ったモジュールを選択して追加。
{{<figure src="/img/2023-01-08-004.png" class="center">}}
ここまででプロジェクトからローカルにおいたSwiftPMのPackageを扱う環境が最低限整った。
あとはPackageに対してモジュール・外部のライブラリを追加すればよしなにプロジェクト側から利用することができる。

# Packageの構成要素
SwiftPMのPackageは大きく`Package.swift`, `Sources`, `Tests`の3つの要素でできている。
- `Package.swift`
    - マニフェストファイル。パッケージの設定や構成要素について記述するところ。
- `Sources`
    - ソースコード本体。Package.swiftで記述した内容に従ってディレクトリの構造を作る必要がある。
- `Tests`
    - テスト。

## Package.swiftの読み方
[公式のドキュメント](https://developer.apple.com/documentation/packagedescription/package)が詳しいのでそこから引用する。

```swift
// swift-tools-version:5.3
import PackageDescription

let package = Package(
    name: "MyLibrary",
    platforms: [
        .macOS(.v10_15),
    ],
    products: [
        .library(name: "MyLibrary", targets: ["MyLibrary"])
    ],
    dependencies: [
        .package(url: "https://url/of/another/package/named/utility", from: "1.0.0")
    ],
    targets: [
        .target(name: "MyLibrary", dependencies: ["Utility"]),
        .testTarget(name: "MyLibraryTests", dependencies: ["MyLibrary"])
    ]
)
```
- `name`: このパッケージの名前。
- `platforms`: このパッケージがサポートしているOSの種類、バージョンの指定
- `products`: このパッケージが提供するもの。ここで定義したlibraryが外部からモジュールとして参照できる。
- `dependencies`: このパッケージが依存している外部のパッケージ。githubに加えて、ローカルに配置した他のパッケージを参照したい場合などもここから。
- `targets`: コードを1つにまとめた単位。ここで指定するnameと同じディレクトリが`Sources/`以下に存在している必要がある。target間の依存関係もここで定義できる。

そのほかPackage.swiftについて知っておくと良さそうなことを紹介。
### Package.swiftはswiftファイルである
当たり前だけどPackage.swiftはswiftファイルである。そのため文字列などを変数として定義したりもできる。気を抜くと固定の文字列ばかりになってしまうので適度に変数に置き換えたりして可読性を上げて行きましょう。

### targetのディレクトリはパス指定できる
何もしないと`Sources/`以下にディレクトリが存在する設定になってしまうが、明示的にパスを指定すると任意のディレクトリを参照できるようになる。
```swift
.target(
    name: "FirstFeatureModule",
    dependencies: [],
    path: "Sources/Feature/First"
),
```
パスを指定することで関連するパッケージの配置をまとめたりできる。これが役に立つケースとして、lintやformatの対象を決めるようなケースがある。特定の種類のtargetに対してのみformatをかける、などの操作がパスが切ってあると実現しやすい。

# 参考
- [公式の説明](https://www.swift.org/package-manager/)
- [GitHub](https://github.com/apple/swift-package-manage)

