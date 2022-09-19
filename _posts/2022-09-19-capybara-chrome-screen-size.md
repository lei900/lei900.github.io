---
title:  Capybaraでheadless_chromeのスクリンサイズを設定する
category: "Ruby on Rails"
tags: [RSpec, Ruby on Rails, test, capybara, chrome]
---

## 背景
RSpecでシステムテストする時、ログインボタンが見つからないエラーが報告された。

chromeを下記のようにheadlessに設定している。

```ruby
RSpec.configure do |config|
  config.before(:each, type: :system) do
    driven_by :selenium_chrome_headless
  end
end
```
エラーキャプチャを見たら、確かにあるはずのログインボタンのところに空白と表示されている。
それより、ヘッダー部分全体もほとんど表示されていない。

そして、一旦chromeのheadlessを外して、もう一回テスト実行すると、無事通った。

## エラー原因

最初は画面で何かエラーが出たか、またはCSS設定の問題かと思ったので、headless外した状態でデバッグしてみた。
すると、偶然にchromeの画面サイズが小さくなっていることに気がついた。
chromeのサイズを大きくすると、消えていたヘッダー部分が通常表示するようになる。

chorome_headlessの状態は、画面サイズが800x600程度小さくなるけど、
headless外した時、フルサイズで実行しているようだ。
だから最初はログインボタンを含めヘッダー部分が画面に表示されていなかった。

## 解決策
chromeのスクリンサイズを大きく設定すると、問題解決になる。
```ruby
driven_by :selenium, using: :headless_chrome, screen_size: [1400, 1400]
```

元々どこかで上記の書き方を見たことがあるけど、単純に書き方が違うか、と思っただけで、
screen_sizeを指定する理由を深く考えなかった。
今後デフォルトで、chromeスクリンサイズを指定するようにする。

---
参照 
[Chrome 83 + rspec-railsで画面サイズを変える方法](https://twitter.com/jnchito/status/1266330181142102017)