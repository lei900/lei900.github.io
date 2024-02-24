---
title: RSpecでdeliver_laterのメールをテストする方法
category: "Coding"
tags: [RSpec, Ruby on Rails, Test]
---

## 背景

RSpec でメール送信をテストする時、メールを確認できなかった。
色々検索したら、`deliver_later`の影響だと気づいた。

## 解決案

いろいろ試して、上手く行けたのは下記の二つ方法

### 1. `have_enqueued_mail`　 matcher を使う

一番簡単な方法は`have_enqueued_mail`　 matcher を使って、メールが実行待ち行列に入っていることを確認する。
先に`ActiveJob::Base.queue_adapter = :test`を追加する必要。

```ruby
RSpec.describe UserMailer do
  it "matches with enqueued mailer" do
    ActiveJob::Base.queue_adapter = :test
    expect { UserMailer.signup.deliver_later }.to have_enqueued_mail(
      UserMailer,
      :signup,
    )
  end
end
```

配信予定時間を含めての確認も簡単

```ruby
RSpec.describe UserMailer do
  it "matches with enqueued mailer" do
    ActiveJob::Base.queue_adapter = :test
    expect {
      UserMailer.signup.deliver_later(wait_until: Date.tomorrow.noon)
    }.to have_enqueued_mail.at(Date.tomorrow.noon)
  end
end
```

### 2. `perform_enqueued_jobs`メソッドを使う

もしメールの内容まで確認必要だったら、`ActiveJob::TestHelper`モジュールが用意する`perform_enqueued_jobs`メソッドを使って、
`deliver_later`のメール配信ジョブをすぐ実行させる。

```ruby
RSpec.describe UserMailer do
  include ActiveJob::TestHelper
  it "matches mail result" do
    perform_enqueued_jobs(only: ActionMailer::MailDeliveryJob) do
      expect(UsersMailer.signup.deliver_later).to change(
        ActionMailer::Base.deliveries,
        :count,
      ).by(1)
      mail = ActionMailer::Base.deliveries.last
      expect(mail.subject).to eq "mail subject"
    end
  end
end
```

先に`ActiveJob::TestHelper`モジュールを include するのが必要。
メールオブジェクトは`ActionMailer::Base.deliveries.last`を使って取得する。
全てのジョブを有効にすると、効率が悪くなるので、`MailDeliveryJob`に限定した方良い。

---

参照(
[ActionMailer の deliver_later でメール送信する場合のユニットテストの書き方](https://shrkw.hatenablog.com/entry/2021/06/16/093000)
[Have_enqueued_mail matcher](https://relishapp.com/rspec/rspec-rails/v/5-0/docs/matchers/have-enqueued-mail-matcher)
[perform_enqueued_jobs](https://edgeapi.rubyonrails.org/classes/ActiveJob/TestHelper.html#method-i-perform_enqueued_jobs)
