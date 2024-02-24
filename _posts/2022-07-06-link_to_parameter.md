---
title:  link_toメソッドでクエリパラメータを付与する
category: "Coding"
tags: [Ruby, Ruby on Rails]
---

最初は`bookmarks#create`アクションの中に、`params[:board_id]`という書き方に疑問を感じた。
なんで`params`に`board_id`が入ったのか？
```ruby
  def create
    board = Board.find(params[:board_id])
```

`boards/_bookmark.html.erb`の中で、
**`link_to bookmarks_path(board_id: board.id)`**があった。

ブックマークをクリックした瞬間、`(board_id: board.id)`のクエリパラメータがurlに入ってきた。
だからアクセスするURLは`bookmarks/?board_id=xx`になる

それで、createアクションで`params[:board_id]`でboard_id取得して、ブックマーク対象の投稿を取得する

**他のクエリパラメータの付与方法**

```ruby
link_to "Comment wall", profile_path(@profile, anchor: "wall")
 => <a href="/profiles/1#wall">Comment wall</a>

link_to "Ruby on Rails search", controller: "searches", query: "ruby on rails"
=> <a href="/searches?query=ruby+on+rails">Ruby on Rails search</a>

link_to "Nonsense search", searches_path(foo: "bar", baz: "quux")
 => <a href="/searches?foo=bar&baz=quux">Nonsense search</a>
```
[Rails API](https://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to)より

---
他参照

[【Rails】link_toに任意のパラメータを付与する方法](https://310nae.com/linkto-param/)

