---
title:  Deviseやsorceryを使わないやり方で、パスワードリセット機能の流れを理解する
category: "Coding"
tags: [Ruby, Ruby on Rails]
---

SorceryのReset passwordモジュールを使って、パスワードリセット機能を実装するという課題だったが、sorceryの手順でなんとなくできたけど、メソッドの意味や流れがなかなかうまく理解できなかった。

それで補充情報を検索してみたら、gemを使わないやり方のyoutube解説動画を見つけた。

必要最小限のコードでパスワードリセット機能を作りながら、大体の流れを理解できてから、
それを参照にして、soceryの動きもなんとなくイメージできるようになった。

- 参照した動画: [Rails for Beginners Part 20~22](https://www.youtube.com/watch?v=5azbhq4z3kw&list=PLm8ctt9NhMNV75T9WYIrA6m9I_uw7vS56&index=20)

## 1. まずサンプルアプリの土台を作る
- Rails 6.1の`signed_id`機能を使うので、Rails 6.1以上のバージョンを使う必要
- 簡単なユーザー登録、ログインとログアウト機能を作る。 
  - Userモデルのコラムはemailとpassword_digestだけでも良い
  -  `has_secure_password`使用
- `password_resets_controller`のnewアクション、そしてリセット申請用のnewページとパスワード更新用のeditページを作成
- `password_resets`部分のroutes設定
- `password_mailer`と`reset`というメソッドを作る

ここまで実現できた流れは、パスワードリセットリンクをクリックして、newページが表示され、フォームからemailを記入するまで。

## 2. createアクション
```ruby
  def create
    @user = User.find_by(email: params[:email])   # フォームで記入したemailでユーザーを特定する
    if @user.present?                                                   
      PasswordMailer.with(user: @user).reset.deliver_later  # ユーザーが存在する場合、PasswordMailerでresetの案内メールを送信する
    end
    redirect_to root_path, notice: "パスワードリセット手順を送信しました"  # ユーザーが見つからなかった場合でも、送信したと伝える
  end
```
{: file="password_resets_controller.rb" }


**`PasswordMailer.with(user: @user).reset.deliver_later`について**

`with(user:@user)`でMailerに引数を渡して、Mailerで`params[:user]`を使えるようになる。後でこのuserの`global_id`を使ってtokenを作る。

今すぐ送信する`deliver_now`より、`deliver_later`を使うのは、メール送信するのは時間かかるので、とりあえずcontrollerのアクションを実行完了させて、
メールが送信したとユーザーに知らせることを優先。controllerのアクションが実行完了してから、ActionJobでメール送信を行う。


## 3. PasswordMailerの動き

**resetアクション**
```ruby
  def reset
    @token = params[:user].signed_id(purpose: "password reset", expires_in: 15.minutes)   
# リセットの目的と15分の有効期限を引数で設定し、userのsigned_idを生成して、リセット用のURLに渡すための引数@tokenに代入する

    mail(to:  params[:user].email,
      subject:  'パスワードリセットのご案内' )
  end
```
{: file="password_mailer.rb" }

**signed_idについて**

`signed_id`はRails 6.1での新機能で、モデルのインスタンスの`global_id`を暗号化したハッシュ値（署名）。
Rails consoleで確認してみると、こんな感じ
```zsh
>user = User.last
>user.to_global_id.to_s
=> "gid://sample-app/User/3"   # userインスタンスを定義するURI 
```

- [global_idについて](https://zenn.dev/ooooooo_q/books/rails_deserialize/viewer/globalid))

- [signed_idとfind_signedについて](https://runebook.dev/ja/docs/rails/activerecord/signedid/classmethods))


**メール本文で使うURL**

`password_reset_edit_url(token: @token)`でurlにtokenを渡す。後でパスワードリセットの時、このtokenを`find_signed`メソッドに渡してuserを特定することができる。

ちなみに、ここのURLヘルパは絶対パスを生成する`_url`を使うべき。`_path`だと、相対パスになるので、外からアクセスできないため。

**configでMailerのhostを設定する**

Mailerは、HTTPリクエストとは無関係で、どのドメインを使ってメールを送信するのか、Mailer自体はわからないため、host情報を明示する必要。

hostパラメータを指定しないままで、リセットを申請したら、
`Missing host to link to! Please provide the :host parameter, set default_url_options[:host]`というエラーが出る。

それで、hostを設定する。(課題ではconfigというgemを使って一元管理するのが推奨)
```ruby
  config.action_mailer.default_url_options = { host: "localhost:3000" }
```
{: file="config/environments/development.rb" }

ここまで実現できた流れは、パスワードリセット申請を提出して、対象のemailアドレスにリセット用のURLを告知する案内メールを送る。


## 4. editアクションとupdateアクション(新しいパスワードの設定と更新)
**editアクション**
`password_reset_edit_url(token: @token)`で生成したリンクをクリックしたら、editアクションが動く
```ruby
  def edit
    @user = User.find_signed!(params[:token], purpose: "password reset")　　# もし15分のtoken期限が過ぎた場合、token(つまりsigned_id)がnilになる
  rescue ActiveSupport::MessageVerifier::InvalidSignature       # 発生した例外を処理する
    redirect_to  login_path, alert: "トークンの有効期限が切れました。再度申請してください。"
  end
```
{: file="config/routes.rb" }

まずは、`User.find_signed!(params[:token], purpose: "password reset")`でリセットを申請したユーザーを特定する。

もし15分のtoken期限が過ぎた場合、`ActiveSupport::MessageVerifier::InvalidSignature`エラーが起こる。
その例外の処理として、ログインページにredirectして、再度申請することを提示する。

また、editページのform_withには`url: password_reset_edit_path(token: params[:token]) `を渡す必要。

**updateアクション**

新しいパスワードを記入して、提出したら、updateアクションが動く。
```ruby
  def update
    @user = User.find_signed!(params[:token], purpose: "password reset")

    if @user.update(password_params)
      redirect_to login_path, notice: "パスワードは更新しました"
    else
      render :edit
    end
  end

  private

  def password_params
    params.require(:user).permit(:password, :password_confirmation)
  end
```
{: file="config/routes.rb" }


これで、パスワードリセットリンクをクリックして、tokenが検証通過されてから、パスワード更新することができる。

このやり方はsorceryの難しいメソッドより、だいぶわかりやすくなったので、
もしsorceryの挙動が理解しづらいと感じたら、まずこれを参照にして、sorceryのコードを改めて理解するのが良いかも。