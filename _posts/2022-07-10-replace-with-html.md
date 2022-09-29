---
title:  replaceWith()とhtml()の違い
category: "JavaScript"
tags: [JavaScript, jQuery, Ruby on Rails]
---

- replaceWith()はタグとタグの中身内容を含めて置き換える
- html()はタグの中身内容だけを置き換える

例えば、置き換え前のコードは
```html
<a id="js-bookmark-button-for-board-1" class="float-right" data-remote="true" rel="nofollow" data-method="post" href="/bookmarks?board_id=1">
  <i class="far fa-star"></i>
</a>
```

`replaceWith()`でaタグを含めて置き換えたらこうなる
```html
<a id="js-bookmark-button-for-board-1" class="float-right" data-remote="true" rel="nofollow" data-method="delete" href="/bookmarks/28">
　<i class="fas fa-star"></i>
</a>
```

一方で、`html()`なら、aタグの中身だけが置き換えられる、つまり`<i class="far fa-star"></i>`部分だけが置き換えられる。
```html
<a id="js-bookmark-button-for-board-1" class="float-right" data-remote="true" rel="nofollow" data-method="post" href="/bookmarks?board_id=1">
    <a id="js-bookmark-button-for-board-1" class="float-right" data-remote="true" rel="nofollow" data-method="delete" href="/bookmarks/28">
　　  <i class="fas fa-star"></i>
　  </a> 
</a>
```
二重のaタグ構造になってしまった。

だから今回は`replaceWith()`を使う。