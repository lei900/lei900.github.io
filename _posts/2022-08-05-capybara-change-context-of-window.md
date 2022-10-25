---
title: Capybaraで新しいwindowを指定するため、window contextを切り替える方法
category: "Ruby on Rails"
tags: [RSpec, Ruby on Rails, Test, Capybara]
---

Capybara で system テストを行った時、`can't find element'のエラーが出た。 `byebug`と`page.body`で確認したら、リンクをクリックした前のページに止まっていることがわかった。

調べてみたら、その原因はページ内のリンクをクリックして、新規 window/tab で開く場合は、`window context`は自動切り替えがなく、前のページに止まること。

解決方法は`window context`を指定すること。

主に二つ種類があり、
複数の window が開いたまま、使いたい window context を指定するか、
一つの window context を維持するようにする。

まずは複数 window context のままで使いたい context を指定する方法
一番簡単なのは、`switch_to_window`メソッドを使う

```ruby
click__on "the link that opens the new tab"
switch_to_window(windows.last)
```

あるいは

```ruby
switch_to_window(
  window_opened_by { click_on "the link that opens the new tab" },
)
```

二つ目の方法は、｀ within_window ｀メソッドを使う

```ruby
within_window(windows.last) do
  # code here
end
```

あるいは

```ruby
new_window = window_opened_by { click_link "the link that opens the new tab" }

within_window(new_window) { click_link "the link inside the new tab" }

new_window.close # if you want to close the tab that was opened
```

もし複数の window context がいらない、今の window context のみを残すなら、

```ruby
visit find_link("the link that opens the new tab")["href"]
```

これで今の window context が維持される。

---

参照:  
[With Capybara, how do I switch to the new window for links with "\_blank" targets?](https://stackoverflow.com/questions/7612038/with-capybara-how-do-i-switch-to-the-new-window-for-links-with-blank-targets)  
[Changing the focus of the window on Capybara](https://stackoverflow.com/questions/60198829/changing-the-focus-of-the-window-on-capybara)  
[Clicked link and still on the same page (capybara)](https://selleo.com/til/posts/e5zvbkdptz-clicked-link-and-still-on-the-same-page-capybara)
