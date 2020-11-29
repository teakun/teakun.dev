---
title: "ブログ作った"
date: 2020-11-24T23:59:35+09:00
draft: false
---

新しくブログを作った。

一年くらい前まで使っていたはてなブログは今はちょっと勢いがなさそうで、使い続けるのは微妙かな、と思うようになってしまった。
他のブログプラットフォームを調べてみたりもしたけど、今どきはnoteだ！notionだ！とか、新興勢力が出てきて何がなんやらよくわからない。
他のプラットフォームに乗り換えたとしても、また新しい勢力が出てくるのは目に見えている。そんなことを気にしながらブログ書くらいなら、ちょうどweb技術のお勉強にもなりそうだしいっそ自分で作るか〜となった。
で、今回の新ブログにいたる。

新しくブログを作るにあたって、以下のようなことを考えた。
- 維持費がかからないか
- メンテナンスが楽かどうか
  - wordpressとかはアップデートとかあるよね？重いしめんどくさい
- いざとなった脱出しやすいかどうか
  - 独自記法だと他のブログに移行するのが難しい
  - markdownで書きたい

ということで新ブログはとりあえずHugo + Github Pagesで作ってみた。
そのうち独自ドメインとかも取って当ててみたい。


しょうもないけど、Hugoでは記事のパスを自分で決めないといけなくて、ここでちょっと悩んだ。
結果的には `$HOME/entry/{記事}` にした。
どこに悩むポイントがあるかというと、`entry` の部分。他の候補としては`posts`、`articles`あたり。
意外とこれ悩みどころじゃない？
自分は文字数の少なさと何となくの好みでentryにした。
調べてると[こういう記事](https://www.dailywritingtips.com/post-entry-or-article/)もあったりする。
でも結局は好みだし好きなやつにしたらいいよね・・・

