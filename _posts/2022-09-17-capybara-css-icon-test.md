---
title: Capybaraでfontawesomeのiconリンクをテストする
category: "Ruby on Rails"
tags: [RSpec, Ruby on Rails, Test, Capybara, CSS, Fontawesome]
---

## 背景

仮にブログ対してのブックマーク機能をテストする。

ブックマーク関係の erb ファイル内容は下記で

```ruby
<div id="js-blog-bookmark-<%= blog.id %>">
  <% if current_user.bookmarked?(blog) %>
    <%= render 'blogs/bookmarks/unbookmark', blog: blog %>
  <% else %>
    <%= render 'blogs/bookmarks/bookmark', blog: blog %>
  <% end %>
</div>
```

{: file="\_blog.html.erb" }

```ruby
<%= link_to blog_bookmark_path(blog), remote: true, method: :post do  %>
  <i class="bi bi-star"></i>
<% end %>
```

{: file="\_bookmark.html.erb" }

```ruby
<%= link_to blog_bookmark_path(blog), remote: true, method: :delete do  %>
  <i class="bi bi-star-fill"></i>
<% end %>
```

{: file="\_unbookmark.html.erb" }

生成する HTML ファイルは下記となる

- ブックマーク icon クリック前

```html
# blog_idが1のブログ
<div id="js-blog-bookmark-1">
  <a
    data-remote="true"
    rel="nofollow"
    data-method="post"
    href="/blogs/1/bookmark"
  >
    <i class="bi bi-star"></i>
  </a>
</div>
```

- ブックマーク icon クリックした後

```html
# blog_idが1のブログ
<div id="js-blog-bookmark-1">
  <a
    data-remote="true"
    rel="nofollow"
    data-method="delete"
    href="/blogs/1/bookmark"
  >
    <i class="bi bi-star-fill"></i>
  </a>
</div>
```

## capybara でのテスト方法

まずは一番目のブログのブックマーク icon をクリックする  
(任意のブログでも良いけど)

```ruby
find("#js-blog-bookmark-#{blog.id}").find(:css, "i.bi.bi-star").click
```

ブックマークされた icon の表示を確認する

- 方法 1: `have_css`を使う

```ruby
expect(page).to have_css "i.bi.bi-star-fill"
```

- 方法 2: `has_css?`メソッドを使う

```ruby
expect(has_css?("i.bi.bi-star-fill", count: 1)).to eq true
```

- 方法 3: `within`メソッドを使う

```ruby
within("#js-blog-bookmark-#{blog.id}") do
  expect(page).to have_css "i.bi.bi-star-fill"
end
```

これで、特定のブログがブックマークされたかどうかを確認できる

### 補足：

icon をクリックする方法について、
直接リンクに class や id、title をつけて icon をクリックするのも一つの方法。

html 部分が下記に修正する

```html
<div id="js-blog-bookmark-1">
  <a
    class="bookmark-link"
    title="Create bookmark"
    data-remote="true"
    rel="nofollow"
    data-method="post"
    href="/blogs/1/bookmark"
  >
    <i class="bi bi-star"></i>
  </a>
</div>
```

そうすると、下記のようにクリックできる

```ruby
# ブックマークiconが複数存在する場合、一番目に指定
find_link("Create bookmark", match: :first).click

# ブックマークiconが一つのみの場合
find_link("Create bookmark").click
```

### 一つ目の要素を指定する方法について

ちなみに、一つ目の要素を指定する方法としては、下記の中に、`match`を使った方が良いだそう。

```ruby
find(".my-selector", match: :first)
# or
all(".my-selector").first
# or
first(".my-selector")
```

なぜなら、`match`を使うと、capybara がマッチする要素が少なくとも一つが現れるまで待ってくれる。

---

参照

[Test css(icon) in Capybara](https://dev.to/n350071/test-css-icon-with-capybara-5fb8)  
[Rspec Target Font Awesome Icon Link](https://stackoverflow.com/questions/25891438/rspec-target-font-awesome-icon-link)  
[Capybara: click on element found by the icon class](https://stackoverflow.com/questions/32863603/capybara-click-on-element-found-by-the-icon-class)  
[Capybara - select first element (Ambiguity Resolution)](https://coderwall.com/p/bfaqwa/capybara-select-first-element-ambiguity-resolution)
