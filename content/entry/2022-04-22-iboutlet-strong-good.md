---
title: "@IBOutletで接続するときweakを付けるべきなのか"
date: 2022-04-22T23:02:49+09:00
draft: false
---

よくあるこういうコードで、weakを付けるべきか付けないべきか。
```swift
@IBOutlet weak var newButton: UIButton!
```

swiftformatを導入するにあたって[このルール](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#strongoutlets)を入れるべきかという議論になったが、weakの有無によってどのような違いがあるのかわからなかったので調べてみる。


そもそも付ける付けないでどのような違いがあるかを調べる。([参考](https://qiita.com/chocovayashi/items/a96adc1356b7c45524b7))
 - weak付ける(弱参照, weak)
    - removeFromSuperviewしたタイミングでviewが開放される
    - 弱参照なのでretaincountがなくなって開放される
 - weak付けない(強参照, strong)
    - removeFromSuperviewしてもviewが開放されない
    - weakが付いていないのでviewへの参照が残っている

合わせて、swiftformatのルール説明に公式の動画へのリンクがあるのでそれをちゃんと見る。
公式は基本的にIBOutletを接続するときは強参照にすることを推奨している。
https://developer.apple.com/videos/play/wwdc2015/407/ (32:30〜あたり)

> In general you should make your outlet strong, especially if you are connecting an outlet to a sub view or to a constraint that's not always going to be retained by the view hierarchy. The only time you really need to make an outlet weak is if you have a custom view that references something back up the view hierarchy and in general that's not recommended. So I'm going to choose strong and I will click connect which will generate my outlet.


> The only time you really need to make an outlet weak is if you have a custom view that references something back up the view hierarchy and in general that's not recommended.

この部分がミソ。
IBOutletの要素が親のViewControllerなど遡って参照を持つケースでは、強参照でIBOutletの要素を持ってしまうと循環参照を起こしてメモリリークしそう。
それ以外のケースではちゃんと開放されるから、removeしてもIBOutletのviewが開放されないように基本は強参照にしておくのがよいと言っているのだと思う。
これまでずっとweakで書いてきたけど、そもそもIBOutletで接続した要素をViewControllerからremoveすることがないのでweakにしても別に困ることがなかった。

ということで、基本はstrong、viewから親に遡るケースがある場合はweakにしておくのがよいみたいです。間違ってたら教えて下さい。