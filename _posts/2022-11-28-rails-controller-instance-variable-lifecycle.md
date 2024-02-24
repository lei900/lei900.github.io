---
title: Railsのリクエストライフサイクルのについて
category: "Coding"
tags: [Ruby on Rails, API]
---

# 背景: Rails API での current_user について

この間、Rails API を勉強した時、下記のコードに引っかかった。

```ruby
class Api::V1::BaseController < ApplicationController
  include ActionController::HttpAuthentication::Token::ControllerMethods

  before_action :authenticate

  def authenticate
    authenticate_or_request_with_http_token do |token, _options|
      @_current_user ||= ApiKey.active.find_by(access_token: token)&.user
    end
  end

  def current_user
    @_current_user
  end
end
```

このコードの意図は、API 側で `current_user` を使えるようにすること。

気になるのは、ここの`||=`演算子。

> `a ||= xxx` > `「||」`演算子の自己代入演算子。a が 偽 か 未定義 なら a に xxx を代入する、という意味になります。

```ruby
@_current_user ||= ApiKey.active.find_by(access_token: token)&.user
```

直感で考えると、ユーザーがログインしている状態だったら、  
同じユーザーからリクエストが来る時、毎回 token を検証し、 DB にクエリしてユーザーを特定する必要がなくなるではないか。

だったら、疑問がある。

確か、HTTP リクエストはステートレスで、session や token などを使って、ユーザー状態を保持する仕組みになっていると思うけど、

もし、ここで`access_token`を都度検証しない場合、どうやって同じユーザーだとわかるのか?

他のユーザーからリクエストがきたら、同じ`current_user`と判断されないのか？

もしかしたら、Rails controller は他の仕組みがあって、同じユーザーからのリクエストだとわかっているのか?

## 通常の`current_user`の使い方について

似たような書き方はは Rails Tutorial の時に見たことがある。

```ruby
module SessionsHelper
  def current_user
    if (user_id = session[:user_id])
      @current_user ||= User.find_by(id: user_id)
    end
  end
end
```

ここは、View 側で`current_user`を使うため、helper method として作ったもの。

> current_user メソッドが１リクエスト内の処理で何度も呼び出されてしまうと、呼び出された回数と同じだけデータベースへの問い合わせが発生してしまい、結果として処理が完了するまでに時間がかかってしまうからです。

> User.find_by の実行結果をインスタンス変数に保存することで、１リクエスト内におけるデータベースへの問い合わせは最初の１回だけになり、以後の呼び出しではインスタンス変数の結果を再利用するようになります

Rails Tutorial の説明によると、ここの`current_user`は同じリクエスト内に使うもの。

つまり、同じリクエスト内で、ユーザーが特定できたら、View 側で何度も`current_user`を呼び出しても、都度の DB クエリが必要なくなる。

だとしたら、やはり異なるリクエストなら、current_user が同じユーザーかどうかという判断はできない?

# リクエストが来るとき、Rails はどう動いているのか

それなら、ユーザーからリクエストが来るとき、Rails は実際にどう対応しているのか。

調べたら、下記のリソースが見つけた。

[The Lifecycle of a Request](https://blog.skylight.io/the-lifecycle-of-a-request/)

[Storing state across requests from the same user](https://stackoverflow.com/questions/18006097/storing-state-across-requests-from-the-same-user)

仮に、`get '/posts' => 'posts#index'`という route がある。

そして、`get '/posts'`というリクエストが来るとき、Rails の対応は簡単にまとめると、こんな感じになる

1. `controller = PostsController.new`で PostsController のインスタンスが作成される。 この controller には、三つのインスタンス変数が持つ - `@request`、`@response`、`@params`

2. controller は`index`アクションを実行する。

3. もし`index`アクションで view テンプレート を render するなら、controller が持つインスタンス変数が view のインスタンスにコピーして渡される。

4. view インスタンスというのは、そのテンプレートの`self`。view ファイル内で使う変数は view インスタンスが controller または他の view インスタンスらコピした変数。

5. `index`アクションの内容を実行完了後、controller インスタンスの任務が完了したので、そのまま廃棄される

6. API の場合だと、もし最後の実行内容は JSON 形式のレスポンスを返すことなら、それを返した時点で、このリクエストの対応が完了になったので、controller インスタンスが捨てられる。

7. 次のリクエストが来る時、また新しい controller インスタンスが作成られ、そのリクエストを対応する。

## Rails controller のインスタンス変数のスコープとライフサイクル

そうしたら、ここでの実行流れはこんな感じになると思う。

```ruby
class Api::V1::BaseController < ApplicationController
  include ActionController::HttpAuthentication::Token::ControllerMethods

  before_action :authenticate

  def authenticate
    authenticate_or_request_with_http_token do |token, _options|
      @_current_user ||= ApiKey.active.find_by(access_token: token)&.user
    end
  end

  def current_user
    @_current_user
  end
end
```

もし`get '/posts'`というリクエストが来たら、

1. PostController のインスタンスが生成される。
2. `index`アクションを実行する前に、親クラスから継承した`before_action`を実行する。
3. そして、`@_current_user`が初期化され、初めて値が代入される。
4. このリクエストが処理終了するまで、PostController の`index`アクション内及び`index`アクションによる呼び出される関連のメソッド内で、`current_user`を使って、ユーザーを取得できる
5. `index`アクションが実行完了されたら、PostController のインスタンスが廃棄される。その同時に、`@_current_user`インスタンス変数も当然捨てられる。

controller が持つインスタンス変数は、同じリクエスト内でも、関連する view にコピして渡すことができる以外

他の controller や他の controller の view、またはモデルでも利用することができない。

## ここの`||=`演算子について

最初の疑問に戻ると、

- Rails controller は異なるリクエストが同じユーザーから来たかどうか、token を検証する前に判断できない
- つまり、リクエストが来る度に、token の都度検証が必要。
- そのため、新しいリクエストが来たら、 `@_current_user`が nil になっているので、`ApiKey.active.find_by(access_token: token)&.user`が必ず実行される

こう見ると、ここでの`||=`演算子は普通の`=`と同じ結果..

参照

[why cannot reassign class variables in controller of rails](https://stackoverflow.com/questions/25914553/why-cannot-reassign-class-variables-in-controller-of-rails)

[Is Rails shared-nothing or can separate requests access the same runtime variables?](https://stackoverflow.com/questions/1025432/is-rails-shared-nothing-or-can-separate-requests-access-the-same-runtime-variabl/1029798#1029798)

[What is the scope of an instance variable in a Rails controller?](https://stackoverflow.com/questions/35828121/what-is-the-scope-of-an-instance-variable-in-a-rails-controller)

[Lifecycle of Instance variables in Ruby on Rails](https://stackoverflow.com/questions/53670979/lifecycle-of-instance-variables-in-ruby-on-rails)

[Ruby on Rails Controller Instance Variable not shared](https://stackoverflow.com/questions/52309062/ruby-on-rails-controller-instance-variable-not-shared)

[Sharing data between threads](https://livebook.manning.com/book/c-plus-plus-concurrency-in-action/chapter-3/)
