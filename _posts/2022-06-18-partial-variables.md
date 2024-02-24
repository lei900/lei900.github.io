---
title:  パーシャルに渡す変数はなるべくローカル変数にした方が良い
category: "Coding"
tags: [Ruby, Ruby on Rails]
---

`form_with model: post`と`from_with model: @post`の2種類記法で戸惑っていたので、パーシャルに渡す変数について、調べていた。

### 結論

インスタンス変数か、ローカル変数か、form_withの挙動に特に影響がなさそうだが、
パーシャルにはなるべくローカル変数を渡すのが良いことがわかった。

### 理由
インスタンス変数を使うと、コントローラとも結びついてしまい、パーシャルの再利用性が低くなる。

例えば、

controller側でインスタンス変数の名前や挙動を変更したとき、partial側も変更しなければならなくなる。

特定のモデルのデータと関連づけられてしまうので、フレキシブルに使うことができない。

---
参照:

[partialではインスタンス変数を参照しない方がいい](https://qiita.com/mom0tomo/items/e1e3fd29729b2d112a48)

[【パーシャル】インスタンス変数の直接参照ではなく、localsで値を渡す](https://forest-valley17.hatenablog.com/entry/2018/10/15/104936)

[form_with model:について](https://www.sejuku.net/plus/question/detail/14961)