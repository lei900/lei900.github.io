---
title:  ルーティングのcollectionとmemberについて
category: "Ruby on Rails"
tags: [Ruby, Ruby on Rails]
---

`collection`と`member`の使い分けについて、ID取得必要かどうかで判断するとの説明が多く見かけて、理解が曖昧だったけど、stackoverflowの回答を見て、ようやく納得した内容をメモする。

### 結論
違いはrouteにidが付くか付かないかと言うより、個別リソースに対してのアクションなのか、リソース全体に対してのアクションなのか、ということ。

### 特定のリソースに対してのアクションの場合、`member`を使う、そのリソースのIDを取得する
```
resources :photos do
  member do
    get 'preview'
  end
end
```
{: file="config/routes.rb" }

- 生成URL例：　`/photos/1/preview`　　

  特定の写真のプレビューのため、その写真のidを取得する必要

- 生成するルーティングヘルパー:`preview_photo_path(photo)`　　　

  特定の写真リソースのため、単数形の`photo`になる


### リソース全体に対してのアクションの場合、 `collection`を使う、個別リソースのID取得は不要
```
resources :photos do
  collection do
    get 'search'
  end
end
```
{: file="config/routes.rb" }


- 生成URL例：　
`/photos/search`　　　

  写真全体に対しての検索行動のため、個別写真のidは取得不要

- 生成するルーティングヘルパー:
`search_photos_path`　　

  写真リソース全体のため、複数形の`photos`になる

---
参照：
[difference between collection route and member route in ruby on rails?](https://stackoverflow.com/questions/3028653/difference-between-collection-route-and-member-route-in-ruby-on-rails)