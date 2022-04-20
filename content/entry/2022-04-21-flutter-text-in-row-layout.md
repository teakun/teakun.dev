---
title: "【flutter】Rowの中に入れたTextを余っているスペースいっぱいに表示したい"
date: 2022-04-21T00:57:28+09:00
draft: false
---

考えるのは以下のようなケース。
- 3つ要素が並んでいて、左右の要素は左右にピッタリ配置
- 中間の要素はテキストで可変になっていて、スペースが余っている限りいっぱいに表示したい
- テキストは左寄せで表示

{{<figure src="/img/2022-04-21-001.png" class="center" title="成功">}}



### うまく行ったパターン
```dart
  Row(
    crossAxisAlignment: CrossAxisAlignment.center,
    mainAxisAlignment: MainAxisAlignment.spaceBetween,
    children: [
      Container(), // 先頭の要素
      Flexible(
        child: Container(
          width: double.infinity,
          padding: const EdgeInsets.only(left: 8),
          child: Text(
            "あああああああああああああああああああああああああああああああああ",
            overflow: TextOverflow.ellipsis,
          ),
        ),
      ),
      Container(), // 末尾の要素
    ],    
  ),
```

中間のTextをFlexibleで囲んで、Containerの幅をdouble.infinityにする。
infinityを指定しているからめいっぱいに広がるけど、Flexibleで囲んでいるから要素がoverflowを起こさない。

### うまく行かなかったパターン

```dart
 Row(
   crossAxisAlignment: CrossAxisAlignment.center,
   mainAxisAlignment: MainAxisAlignment.spaceBetween,
   children: [
     Container(), // 先頭の要素
     Flexible(
       child: Container(
         width: double.infinity,
         padding: const EdgeInsets.only(left: 8),
         child: Text(
           "あああああああああああああああああああああああああああああああああ",
           overflow: TextOverflow.ellipsis,
         ),
       ),
     ),
     const Spacer(),
     Container(), // 末尾の要素
   ],
 ),
```

SpacerをRowの要素としてかませばテキストが左寄せになるつもりだったが、Rowの中に入れた要素のスペースが想定通りに入らなかった。
Rowで指定しているspaceBetweenがSpacerが入ったときに余計なことをしていそう。

{{<figure src="/img/2022-04-21-002.png" class="center" title="うまくレイアウトができず変なスペースが生まれている様子">}}