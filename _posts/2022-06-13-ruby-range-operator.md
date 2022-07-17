---
title:  Rubyの範囲演算子は降順のイテレートに使えないこと
category: "Ruby"
tags: [Ruby]
permalink: "/posts/1/"
---

初めてアルゴリズムを勉強していたとき、バブルソートの方法を使って配列を降順にソートするプログラムを書くという課題で躓いていたところをメモする。


プログラムは下記のように作成したけど、どうしてもソートできなかった。
```ruby
arr = [4, 5, 2, 1, 3]
size = arr.size
max = size - 1

(0..(max-1)).each do |i|
  (max..(i+1)).each do |j|
    if arr[j] < arr[j-1]
      arr[j], arr[j-1] = arr[j-1], arr[j]
    end
  end
end
```
そして下記で試したところ、`  (max..(i+1)).each do |j|`が動かなかったことにようやく気づいた...
```ruby
(0..max).each do |i|
  p "i = " + i.to_s
  (max..(i+1)).each do |j|
    p "j = " + j.to_s
    if arr[j] < arr[j-1]
      arr[j],　arr[j-1] = arr[j-1],arr[j]
    end
    p arr
  end
end
```

調べたら、Rubyでは`(5..1).each do`のように、範囲演算子が降順のイテレートに使いえないそうだ。

下記に修正したら、うまくソートできた。
```ruby
 ((i+1)..max).reverse_each do |j|
```

---
参照:

[Rubyの範囲演算子は降順のイテレートには使えない](https://shin.hateblo.jp/entry/2012/12/20/202641)

[【Ruby】1..4の逆をしたい](https://teratail.com/questions/2087)

[Array#reverse_each](https://docs.ruby-lang.org/ja/latest/method/Array/i/reverse_each.html)