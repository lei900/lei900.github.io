---
title:  Capybaraで新しいwindowを指定するため、window contextを切り替える方法
category: "Ruby on Rails"
tags: [RSpec, Ruby on Rails, test, capybara]
---

Capybaraでsystemテストを行った時、`can't find element'のエラーが出た。  
`byebug`と`page.body`で確認したら、リンクをクリックした前のページに止まっていることがわかった。

調べてみたら、その原因はページ内のリンクをクリックして、新規window/tabで開く場合は、`window context`は自動切り替えがなく、前のページに止まること。

解決方法は`window context`を指定すること。

主に二つ種類があり、
複数のwindowが開いたまま、使いたいwindow contextを指定するか、
一つのwindow contextを維持するようにする。

まずは複数window contextのままで使いたいcontextを指定する方法
一番簡単なのは、`switch_to_window`メソッドを使う
```ruby
click__on 'the link that opens the new tab'
switch_to_window(windows.last)
```
あるいは
```ruby
switch_to_window(window_opened_by { click_on 'the link that opens the new tab' })
```

二つ目の方法は、｀within_window｀メソッドを使う
```ruby
within_window(windows.last) do
  # code here
end
```
あるいは
```ruby
new_window = window_opened_by do
  click_link 'the link that opens the new tab'
end

within_window(new_window) do
  click_link 'the link inside the new tab'
end

new_window.close # if you want to close the tab that was opened
```

もし複数のwindow contextがいらない、今のwindow contextのみを残すなら、
```ruby
visit find_link('the link that opens the new tab')['href']
```
これで今のwindow contextが維持される。


---
参照:  
[With Capybara, how do I switch to the new window for links with "_blank" targets?](https://stackoverflow.com/questions/7612038/with-capybara-how-do-i-switch-to-the-new-window-for-links-with-blank-targets)  
[Changing the focus of the window on Capybara](https://stackoverflow.com/questions/60198829/changing-the-focus-of-the-window-on-capybara)  
[Clicked link and still on the same page (capybara)](https://selleo.com/til/posts/e5zvbkdptz-clicked-link-and-still-on-the-same-page-capybara)
