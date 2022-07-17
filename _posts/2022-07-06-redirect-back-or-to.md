---
title:  redirect_backとredirect_back_or_toについて
category: "Ruby on Rails"
tags: [Ruby, Ruby on Rails]
---

`redirect_back fallback_location: root_path`は初めて見た使い方で、redirect_back_or_toとの違いを調べていた。

Rails APIでは`redirect_back`は`redirect_back_or_to`のエイリアスで、後者よりやや非推奨と書いてた。

理由はfallback_locationの引数は `redirect_back`でキーワード引数の形式で、`redirect_back_or_to`では一番目の位置引数になっているとのこと。

>Soft deprecated alias for redirect_back_or_to where the fallback_location location is supplied as a keyword argument instead of the first positional argument.

こう見ると、ここでは`redirect_back_or_to root_path`を使った方が良いではないかと思い、やって見たら、`redirect_back`と同じ挙動はしなかった。

おかしいなと思っていたところ、`redirect_back_or_to`はsorceryにより提供された便利なメソッドだとの話を思い出した。

だとしたら、Railsでの`redirect_back_or_to`は新しくリリースしたメソッド？

Railsドキュメントでは5.0から7.0まで適用と書いたので、新しいメソッドではないみたいだけど...

ちょっと迷っているところ、こんなissueを見つけた。
>Should Sorcery rename redirect_back_or_to? #296
Rails 7 released a new method called redirect_back_or_to added by DHH as a replacement for the now soft-deprecated redirect_back: rails/rails#40671.
That may conflict with the method by the same name defined by Sorcery. Should it be renamed? I'm opening an issue instead of a PR because I guess it's not a trivial decision to make.

なるほど！`redirect_back_or_to`はRails 7でリリースしたメソッドで、元々Sorceryが提供しているメソッド名と被ったため、Sorceryのメソッド名の変更が必要になったと。

(Railsドキュメントも完全に信用してはいけないね。。)

だから今回課題のアプリでは、`redirect_back_or_to`はまだ使えない、従来の`redirect_back`を使うべきということで疑問が解消した。

ちなみに、先日Sorceryの開発者がこの問題を着手し始めてるそうで、Sorceryで使う`redirect_back_or_to`メソッド名が変更されるかも。

---
参照
[Should Sorcery rename redirect_back_or_to? #296](https://github.com/Sorcery/sorcery/issues/296)
[Rails API](https://github.com/Sorcery/sorcery/issues/296)