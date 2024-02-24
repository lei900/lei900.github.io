---
title: Capybaraでheadless_chromeのスクリンサイズを設定する
category: "Coding"
tags: [RSpec, Ruby on Rails, Test, Capybara, Chrome]
---

## 背景

RSpec でシステムテストする時、ログインボタンが見つからないエラーが報告された。

chrome を下記のように headless に設定している。

```ruby
RSpec.configure do |config|
  config.before(:each, type: :system) { driven_by :selenium_chrome_headless }
end
```

エラーキャプチャを見たら、確かにあるはずのログインボタンのところに空白と表示されている。
それより、ヘッダー部分全体もほとんど表示されていない。

そして、一旦 chrome の headless を外して、もう一回テスト実行すると、無事通った。

## エラー原因

最初は画面で何かエラーが出たか、または CSS 設定の問題かと思ったので、headless 外した状態でデバッグしてみた。
すると、偶然に chrome の画面サイズが小さくなっていることに気がついた。
chrome のサイズを大きくすると、消えていたヘッダー部分が通常表示するようになる。

chorome_headless の状態は、画面サイズが 800x600 程度小さくなるけど、
headless 外した時、フルサイズで実行しているようだ。
だから最初はログインボタンを含めヘッダー部分が画面に表示されていなかった。

## 解決策

chrome のスクリンサイズを大きく設定すると、問題解決になる。

```ruby
driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
```

元々どこかで上記の書き方を見たことがあるけど、単純に書き方が違うか、と思っただけで、
screen_size を指定する理由を深く考えなかった。
今後デフォルトで、chrome スクリンサイズを指定するようにする。

---

参照
[Chrome 83 + rspec-rails で画面サイズを変える方法](https://twitter.com/jnchito/status/1266330181142102017)
