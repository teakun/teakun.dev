---
title: "multipart/form-data入門"
date: 2020-12-02T01:43:18+09:00
draft: false
---

仕事で`multipart/form-data`でファイルアップロードをする実装することになったけど、どういうものか全然わかっていないので調べる。

# そもそもなにか
HTTPのリクエストで複数のデータをまとめて送るための仕組み。
例えば、プロフィールを登録するような場合に、名前と画像をセットでアップロードするようなときに使う。

# header
Content-Typeにはこのような指定が入る。
```
Content-Type: multipart/form-data; boundary=--<バウンダリ文字列>\r\n
```
バウンダリ文字列というのがミソで、これが送信したい複数のデータの区切りになる。
なので、この文字列はデータ中には出てこないような無難な乱数が入るのがよい。

# body
bodyの例。1つ目にテキスト、2つ目に画像を送るような場合で考える。

```
--<バウンダリ文字列>\r\n
Content-Disposition: form-data; name="<1つ目の名前>"\r\n
\r\n
<1つ目のデータ>
--<バウンダリ文字列>\r\n
Content-Disposition: form-data; name="<2つ目名前>"; filename="<ファイル名>"\r\n
Content-Type: image/jpeg\r\n
\r\n
<画像のバイナリ>
--<バウンダリ文字列>--\r\n
```

- headerで定義したバウンダリ文字列でデータの区切りを表す。バウンダリ文字列は`--`から始めないといけない制約がある。
- すべての改行は`\r\n`にする。(この改行の方法をCRLFと呼ぶらしい)
- 各データの先頭では`Content-Disposition`を指定していて、これがそのデータの情報を表している。あとは必要に応じて`Content-Type`などを指定する。
- 区切りの最後にもバウンダリ文字列を追加を追加する必要がある。この場合はバウンダリ文字列の最後にも`--`をつけないといけない。

# 参考
- https://satox.hatenadiary.jp/entry/20110726/1311665904
- https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Content-Disposition
- https://marusunrise2.blogspot.com/2014/06/lfcrcrlf.html

