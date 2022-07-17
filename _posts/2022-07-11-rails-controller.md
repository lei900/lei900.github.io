---
title:  Rails controllerの動きについて
category: [Ruby on Rails]
tags: [Ruby, Ruby on Rails]
---

routeの設定やcontrollerの記述だけで、viewのレンダリングやデータの渡しなどRailsが自動的に動いてくれるとはわかるけど、でもどうやってできたの？って素朴な疑問があったのでちょっと調べてみた。

面白いと感じたので、メモしておく。

- **URLにアクセスした後、controllerの動き**

  - 仮に/boards/indexにアクセスしたら、Rails は関連のcontrollerインスタンスを作成する。イメージ例：`kontroller = BoardsController.new`

  -  routingのマッピングによって、この`kontroller`インスタンスは`index`メソッドを実行する。

  - `@boards`などのインスタンス変数に関連データを保存する。

  - `index`メソッドの最後の動きとしてデフォルトで`index.html.erb`をレンダリングしようとする。


- **controllerメソッドの戻り値**

  - Rubyではメソッドの最後にreturnを明記する必要がないが、indexメソッドは最後に評価する式は  `return render 'index'`と思っても良い
  - つまり戻り値はレンダリングしたviewページとなる
  - もし他のviewページをメソッドの最後に明記したら、index以外のページをレンダリングすることになる。


- **conrollerからデータをviewに渡す仕組み**

  - 他に定義したRubyメソッドを使ってcontrollerのインスタンス変数名と値をコピーしてviewのインスタンスに持たせること
  - 普通のローカル変数だと、indexメソッド外から参照できないので、ここでインスタンス変数を使うわけ
  - っていうことは、viewファイルで使っている@boardsは、実はviewクラスのインスタンスがコピした同じ名前と同じ値を持つviewのインスタンス変数?

Railsではインスタンス変数を使って、controllerからviewへのデータ共有を実現しているが、通常のOOPでのインスタンス変数の使い方とはズレているので、ちょっと議論があるけど..

他のMVCモデルを使うフレームワークはデータの共有はどうやって実現しているだろう

Railsの動きについて、いつも魔法的な感じだったけど、実は全てクラスやインスタンス、メソッドなどで実現していることは改めてわかった。 

---
参照:

[How are Rails instance variables passed to views?](https://stackoverflow.com/questions/18855178/how-are-rails-instance-variables-passed-to-views)

[Howcome an Instance variable @variable defined in the controller's action can be called from its views?](https://stackoverflow.com/questions/10035103/howcome-an-instance-variable-variable-defined-in-the-controllers-action-can-be?noredirect=1&lq=1)

[Sending Data Between Rails Controllers and Views](https://www.youtube.com/watch?v=mRJSovhdzWc&list=PLm8ctt9NhMNWh7KBtkeJbyVP8vgSkKWqz&index=5)
