---
title: Javascript処理を挟むテストの場合、Capybaraを待たせる必要
category: "Coding"
tags: [RSpec, Ruby on Rails, Test, Capybara, JavaScript]
---

# 状況

ローカルや本番で問題なく動いているけど、RSpec テストが通ったり、通れなかったりする状況があった。

test log を確認したところ、個別 example の動きがおかしくなったりしている。

```
Started POST "/questions/1/answer?result=good" for 127.0.0.1 at 2022-09-02 23:25:25 +0900
Processing by QuestionsController#answer as TURBO_STREAM
  Parameters: {"result"=>"good", "id"=>"1"}
```

ここは、/questions/2/answer へ遷移すべきだけど、なぜか controller が受け取った`params[:id]`が`1`になっている

そのせいで、テストが失敗したかもと推測。
なんで parameter が正確に送られていないだろう、その原因が分からなくて、一時詰まってた。

# 原因

チームメンバと相談して、ようやくテストが失敗する原因を特定できた。
Rails 7 はデフォルトで ajax で画面遷移を行なっているため、たまに capybara が ajax 処理が完了するまで待たなくて、前の画面でテストを先に処理してしまった。そのせいで、前の画面で取得したオブジェクト ID を controller に送ってしまった。

# 対策

次の画面操作する前に、capybara をちょっとだけ待たせることで、テストが問題なく通った。
capybara を待たせる方法は主に二つあるそうで、

- `sleep`を使って、capybara を一時停止させる

```
click_on '次の質問'
sleep 1   # capybaraを1秒間sleepさせる
click_on '次の操作ボタン'
```

- ただ`sleep`を大量使用する場合、テスト実行効率が非常に悪くなるので、
  代わりに`find`や`have_xx`句を使って、遷移後の画面にあるべき要素を確認する処理を入れることで、capybara が自動的に画面遷移の処理が完了するまで待ってくれる。
  デフォルトの待ち時間が最大 2 秒。

とにかく`sleep`させるよりは、二つ目の方がより賢い。

```
click_on '次の質問'
expect(page).to have_content "次の画面タイトル"
click_on '次の操作ボタン'
```

---

参照:  
[テストが失敗する場合は sleep 等でテストを停止させてみる](https://qiita.com/jnchito/items/607f956263c38a5fec24#%E3%83%86%E3%82%B9%E3%83%88%E3%81%8C%E5%A4%B1%E6%95%97%E3%81%99%E3%82%8B%E5%A0%B4%E5%90%88%E3%81%AFsleep%E7%AD%89%E3%81%A7%E3%83%86%E3%82%B9%E3%83%88%E3%82%92%E5%81%9C%E6%AD%A2%E3%81%95%E3%81%9B%E3%81%A6%E3%81%BF%E3%82%8B)  
[RSpec の sleep 処理について](https://qiita.com/syossan27/items/78c479a46dcf963f55ef)
