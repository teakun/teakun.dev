---
title: "Xcode source editor extensionを試した記録"
date: 2021-01-10T22:16:39+09:00
draft: false
---

試してみたので過程を残しておく。

エディタ上で選択した範囲からその部分がハイライトされるGitHubのリンクを生成するExtensionを作りたかったが、それはできないということが分かって残念。
環境はBigSur11.1でXCode12.3。

# ビルドするまで
まずはこのあたりの資料を参考にシンプルなプロジェクトを作ってみる。2017年とちょっと古い資料だけど、基本的な部分は変わっていない。とても参考になった。

- https://speakerdeck.com/takasek/20170916-number-iosdc
- https://github.com/takasek/XcodeExtensionSample


これを元にシンプルなプロジェクトができたけど、そのままだとXCodeがクラッシュ、コマンドが認識されないなどの問題が起きた。

### XCode12以降でビルドできない問題
extensionを作るためにはXCodeKit.frameworkを使う必要があるけれど、XCode12以降ではそれがembedになっている必要がある。

> For compatibility with new security features in macOS 11, Xcode extensions must be built using Xcode 12 and must embed XcodeKit.framework. An Xcode extension rebuilt with these tools remains compatible with older versions of Xcode and macOS.
- https://developer.apple.com/documentation/xcode-release-notes/xcode-12-release-notes

ということで、Embedの設定を`Embed & Sign`に変えてやるとよい。
{{<figure src="/img/2021-01-10-001.png" class="center">}}

一応他にも[同じ問題で困っている人](https://github.com/Jintin/Swimat/issues/227)がいるようだった。

### Extensionの権限がない問題
上の設定の問題が解消してビルドができるようになったが、自分の環境ではまだ新しく作ったコマンドがXCodeで表示されない問題があった。
どうやら作ったExtensionにmacOS自体の機能拡張の権限が渡っていなかったのが原因。
{{<figure src="/img/2021-01-10-002.png" class="center">}}

システム環境設定 -> 機能拡張 -> XCode Source Editorから作ったExtensionのチェックを入れ直して解決。
- https://ez-net.jp/article/83/1AuGcndM/YL1DSTNKXeYD/

# できたこと
### 選択した行の取得
エディタで選択している行を取得することはできた。
コードはざっくりこんなところ
```swift
    func perform(with invocation: XCSourceEditorCommandInvocation, completionHandler: @escaping (Error?) -> Void ) -> Void {
        if invocation.buffer.selections.count <= 0 {
            completionHandler(nil)
            return
        }
        
        guard let targetSelection = invocation.buffer.selections[0] as? XCSourceTextRange else {
            completionHandler(nil)
            return
        }
        
        let startLine = targetSelection.start.line
        let endLine: Int
        if targetSelection.end.column == 0 && (targetSelection.start.line != targetSelection.end.line) {
            endLine = targetSelection.end.line - 1
        } else {
            endLine = targetSelection.end.line
        }
        
        print("start:",startLine)
        print("end:", endLine)

        completionHandler(nil)
    }
```
行数のカウント(`XCSourceTextPosition.line`)は0から始まるので注意。エディタの左に出る数字とは1つずれるのでややこしい。

1つハマりどころがあって、複数行選択しているかつ、選択範囲の終端が行末にあるときに、上のコードで言うところの、`targetSelection.end.line`が実際に選択している次の行の数になってしまう。
この問題も[よくまとまっている記事](https://qiita.com/funzin/items/e6f2c65351138a980386)があって、とても参考になりました。
正直いうとほとんどこの記事を参考に実装した。

### クリップボードへの保存
```swift
  let pasteboard = NSPasteboard.general
  pasteboard.declareTypes([.string], oner: nil)
  pasteboard.setString(text, forType: .string
```
ざっくりこんな感じ。シンプル。

# できなかったこと
### 開いているファイルの名前を取得
[そもそもサポートされていない](https://stackoverflow.com/questions/56058281/how-to-get-current-workspace-path-in-xcode-source-editor-extension)。ちょっとこれができないのは厳しい。

### gitのリモートリポジトリ、ブランチの取得
Extensionの仕組みとしてはこちらも用意されていない様子。

代案としてshellコマンドをExtension上から実行してリモートリポジトリ、ブランチを取得する方法を考えたが、これも難しかった。
gitコマンドを実行すると、`xcrun: error: cannot be used within an App Sandbox`とエラーが出てしまう。
これは、[App Sandbox](https://developer.apple.com/library/archive/documentation/Security/Conceptual/AppSandboxDesignGuide/AboutAppSandbox/AboutAppSandbox.html#//apple_ref/doc/uid/TP40011183-CH1-SW1)という仕組みがmacOSアプリ開発にあって、あらゆる権限が縛られているらしい。確かに自由にシェルのコマンドが打ててしまうと権限管理的にまずいというのは分かる。AppSandboxでは他にもfinderのアクセス、ハードウェアアクセスなどが縛られていた。もっとよく調べたらこれは解決できそうだったけど、先に書いたファイル名を取得できない問題が解決できないのでこれ以上の調査は断念。

コマンドを実行する方法は[この記事](https://qiita.com/wheel/items/004f94c6e7ecea30bb01)がわかりやすかった。この実装自体はコマンドラインツールを作るときにも使いそう？

# おわり
Extensionは想像以上に何も作れないということが分かった。もうちょっとAPIを開放してくれたらいいのになあ…

あと、このブログ見出しが見にくくて読みにくいなと自分で思った。ブログのデザインもうちょっといい感じにしたい。