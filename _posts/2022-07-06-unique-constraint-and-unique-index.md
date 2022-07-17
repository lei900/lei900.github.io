---
title:  INDEX、複合INDEX、UNIQUE制約とUNIQUE INDEXについて
category: "Ruby on Rails"
tags: [Ruby, Ruby on Rails, Database]
---


## INDEXとは

特定のカラムからデータを取得する際に、テーブルの中の特定のカラムのデータを複製し検索が行いやすいようにしたもの。


  例えばユーザー名を検索する場合、データベースはデフォルトで表の1行目から最後まで探す、いわゆる全表スキャン（Full-table Scan）の方法。

　もしユーザー名にindexをつけると、本の索引のように、アルファベット順などのアルゴリズムを使って途中から探すようになる。検索が早くなる。

　ただデメリットとしては、書き込みの速度が倍かかる。
    だから[闇雲にインデックスを付けてはいけない](https://techracho.bpsinc.jp/hachi8833/2018_05_14/55774)という記事がある。


 Railsではreferences型を指定すると、外部キーとインデックスが自動的に作ってくれる。

（Mysqlなどのデータベースでは外部キーとインデックス作成のセットが必須と要求されているが、そうでもないDBがあるらしい。

ただ外部キーのインデックスを作成した方が良いとの説がある。
理由はインデックスを設置しないと整合性チェックは子表全体のテーブルフルスキャンが実行される。
 そのためにデッドロックが発生する可能性を抱えることになってしまうとのこと）

**複合インデックスとは**

複数のカラムを組み合わせたインデックス。

今回はuserとboardの組み合わせの一意を保証するため、
DB側で`  add_index :bookmarks, [:user_id, :board_id], unique: true`も追加することで、user_idとboard_idの複合インデックスを作成して、ユニーク制約をかけることにした。

「[闇雲にインデックスを付けてはいけない」という記事を読んだので、複合インデックスより、単純なユニーク制約をかけるなど他の方法はないか、と思ったけど、
実はユニーク制約とユニークインデックスについての理解が間違えた。。

**ユニーク制約とユニークインデックス**

　ユニーク制約はコラムのレコードに対して重複を禁止する制約。

　ユニークインデックスは、レコードの重複を禁止するインデックス。一種のオブジェクト。

　あるコラムにユニーク制約をかけると、このコラムのユニークインデックスが自動的に生成される。

  ユニーク制約が重複チェックを行う際に、ユニークインデックスを使ってチェックを行っている。

   つまり、ユニーク制約はユニークインデックスの目的とも言える。

　だから、ユニーク制約をかけるには、ユニークインデックスの作成が必須ということ。
`  add_index :bookmarks, [:user_id, :board_id], unique: true`はもう忘れられない :rolling_on_the_floor_laughing: 

---
参照
[【SQL】UNIQUE制約についてのあれこれ。似ている名前”ユニークインデックス”との違いやカラムの組み合わせについて解説](https://style.potepan.com/articles/23620.html)
[Difference between SQL Server Unique Indexes and Unique Constraints](https://www.mssqltips.com/sqlservertip/4270/difference-between-sql-server-unique-indexes-and-unique-constraints/)
[データベースにindexを張る方法](https://qiita.com/seiya1121/items/fb074d727c6f40a55f22)