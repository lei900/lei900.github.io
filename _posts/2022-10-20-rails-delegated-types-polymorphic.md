---
title: RailsのDelegated Typesで複数モデルを扱うフィード機能を作る
category: "Coding"
tags: [Ruby on Rails, Database]
---

## 背景

個人 PF でニュースフィードみたいな機能を作りたい。そのフィードで表示する情報は複数のモデルから獲得して、時間順や人気順で表示する。

たとえば、Facebook のタイムラインように、友達の投稿だけでなく、誕生日の通知やアクティビティ更新、イベント情報などいろんな情報を表示する。

PF ではまだ二つ種類だけの投稿を表示する予定なので、一番簡単なのは、その二つ種類の投稿モデルを一つのモデルにする方法。

ただコラムが異なる部分が多いのと、今後他の情報を追加表示したいなら、また解決方法を考える必要になるので、今回でも正面から解決したいと思う。

仮に今回のフィードで下記のような二つモデルを扱う予定。

- 投稿記事

```ruby
# == Schema

# Table name: posts

# id          :integer
# user_id     :integer
# title       :string
# content     :text
```

- イベント情報

```ruby
# == Schema

# Table name: events

# id          :integer
# user_id     :integer
# title       :string
# content     :text
# place       :string
```

## 解決策

調べたら、主に四つの解決策があるようで、 single table inheritance (STI、単一テーブル継承) 、 Abstract Classes(抽象クラス)、Polymorphic Association（ポリモーフィック関連）と Delegated Types(委譲型)。

STI や抽象クラスなどそれぞれのデメリットがあるため、  
結論としては、Delegated Types が一番良い。

### 1. STI 単一テーブル継承を使う

STI を使うと、二つのテーブルを一つの feeds テーブルに合併するようになる。

```ruby
# == Schema

# Table name: feeds

# id          :integer
# user_id     :integer
# title       :string
# content     :text
# place       :string
# type        :string #['Post', 'Event']

class Feed < ApplicationRecord
  belongs_to :user
end

class post < Feed; end

class Event < Feed; end
```

データは一つのテーブルに保存しているので、フィードで投稿とイベントを時間順で表示するのは`Feed.order(created_at: :desc)`で簡単にできる。

ただデメリットとしては、テーブルが膨大になるし、NULL 値がいっぱい発生する。

STI なら、やはり性質上が同じで、コラムもほぼ重複するようなモデルを扱う場合の方が良い。

### 2. Abstract Classes(抽象クラス)

これはモデルそれぞれのテーブルを保有して、相互間のつながりは一つの抽象ベースモデルを介して行うやり方。

```ruby
class Feed < ApplicationRecord
  self.abstract_class = true
end

# == Schema

# Table name: posts

# id          :integer
# user_id     :integer
# title       :string
# content     :text

class Post < Feed
  belongs_to :user
end

# == Schema

# Table name: events

# id          :integer
# user_id     :integer
# title       :string
# content     :text
# place       :string

class Event < Feed
  belongs_to :user
end
```

これでユニークな属性を個別のテーブルに分離することができるけど、重複するようなコラム(user, title, content)が削減できていない。
それより、`Feed.all`など DB クエリ操作ができないので、フィード機能自体の実現がややこしくなる。

### 3. Polymorphic Association（ポリモーフィック関連）

これは Rails ガイドで説明があり、以前使ったこともある。  
[Rails ガイド：ポリモーフィック関連付け](https://railsguides.jp/association_basics.html#%E3%83%9D%E3%83%AA%E3%83%A2%E3%83%BC%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E9%96%A2%E9%80%A3%E4%BB%98%E3%81%91)

Feed の実装方法を検索した時、Stack Overflow で出ている回答はほとんどポリモーフィック関連を使うやり方。

フィード生成用の feeds テーブルを作成して、投稿とイベントを作成するたびに、feed レコードも作成する。
ビューで表示するとき、`@feed.feedable`で親のレコードを取得できる。

```ruby
# == Schema

# Table name:  feeds

# id              :integer
# feedable_type   :string
# feedable_id     :string

class Feed < ApplicationRecord
  belongs_to :feedable, polymorphic: true
end

# == Schema

# Table name:  posts

# id          :integer
# user_id     :integer
# title       :string
# content     :text

class Post < ApplicationRecord
  belongs_to :user
  has_many :feeds, as: :feedable
end

# == Schema

# Table name:  events

# id          :integer
# user_id     :integer
# title       :string
# content     :text
# place       :string

class Event < ApplicationRecord
  belongs_to :user
  has_many :feeds, as: :feedable
end
```

これで機能自体が実現できているけど、ちょっとだけ違和感がある。  
一つクエリ操作でレコードを全部取得するため、新しい feeds テーブルを作ったけど、
feed と post、event の間は、一対一の関係になるはずなのに、
関係付けは`has_many`になっている。

### 4. Delegated Types

実は Delegated Types を実装するとき、ポリモーフィックと似たような感じで、この二つの違いを迷っていた。

下記の記事のおかげで、ようやく違いが理解できた。  
[Delegated Types in Rails, handling models with overlapping attributes](https://www.reginavlee.com/delegated-types/)

ポリモーフィック関連が「has many」関係のインタフェースを定義する。
一方、Delegated Types は「has one」関係のインタフェースを定義する。

なので、今回は Delegated Types の方が一番相応しい解決策。
(Delegated Types は Rails 6.1 以降リリースの機能で、Rails 6.1 以前のバージョンが使えない。)

Delegated Types は STI と抽象クラスの良いところを吸収して、一つのソリューションに融合させたもの。　　
それで、共通属性を親テーブルに格納でき、独自の属性だけをそれぞれのテーブルに保存する。

```ruby
# == Schema

# Table name:  feeds

# id              :integer
# user_id         :integer
# title           :string
# content         :text
# feedable_type   :string
# feedable_id     :string

class Feed < ApplicationRecord
  belongs_to :user
  delegated_type :feedable, types: %w[Post Event]
end

# == Schema

# Table name:  posts

# id          :integer

class Post < ApplicationRecord
  has_one :feed, as: :feedable
end

# == Schema

# Table name:  events

# id          :integer
# place       :string

class Event < ApplicationRecord
  has_one :feed, as: :feedable
end
```

Feed モデルに delegated_type :feedable を含めることで、feedable にポリモーフィックな belongs_to リレーションシップを裏で追加している。

構文には少し違いがあるけど、大体は通常のポリモーフィック関連を設定するのと同じ。

ただ delegated_type は一対一の関係なので、`post.feed.title`で親モデルの属性を取得できる。
これは一対多のポリモーフィック関連ではできない。

レコードの作成も簡単になる。

```ruby
Feed.create(
  feedable: Event.create(place: "Tokyo"),
  title: "東京もくもく会",
  content: "Rails勉強会",
  user: current_user,
)
```

Rails 7 から導入された`accepts_nested_attributes_for `を利用してさらに簡単。

```ruby
class Feed < ApplicationRecord
  belongs_to :user
  delegated_type :feedable, types: %w[Post Event]

  accepts_nested_attributes_for :feedable
end
```

```ruby
Feed.create(
  feedable_type: "Event",
  title: "東京もくもく会",
  content: "Rails勉強会",
  user: current_user,
  feedable_attributes: {
    place: "Tokyo",
  },
)
```

ビューで情報表示する時、共通属性と独自属性データをそれぞれ取得する。

```ruby
Feed.first.feedable
# <Event id: 1, place: 'Tokyo'>

Event.first.feed
# <Feed id: 1, feedable_type: 'Event', feedable_id: 1,
#       title: '東京もくもく会', content: 'Rails勉強会', user_id: 1>
```

これで、重複するコラムと大量の NULL 値がなくなり、DB クエリでそれぞれのデータを取得できる。モデルそれぞれの振る舞いも設定できる。完璧な解決案!

参照  
[Delegated Types in Rails, handling models with overlapping attributes](https://www.reginavlee.com/delegated-types/)  
[ActiveRecord::DelegatedType(Rails API ドキュメント)](https://api.rubyonrails.org/classes/ActiveRecord/DelegatedType.html)  
[Feed from multiple models (sorted by the time they were added to the feed)](https://stackoverflow.com/questions/49157298/feed-from-multiple-models-sorted-by-the-time-they-were-added-to-the-feed)
[単一テーブル継承(STI)について](https://lei900.github.io/22/08/rails-sti-single-table-inheritance/)  
[Rails: ActiveRecord::DelegatedType API ドキュメント（翻訳）](https://techracho.bpsinc.jp/hachi8833/2022_04_12/112882)
