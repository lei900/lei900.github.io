---
title: ActiveModelでは`text`型は指定不可のことについて
category: "Ruby on Rails"
tags: [Ruby on Rails, SQL, MySQL, PostgresSQL]
---

今回のFormObjectでうっかり`body`のタイプを`text`に指定してしまい、`Unknown type :text`のエラーが出た。  
調べたら、ActiveModelでは`text`タイプをサポートしてないことがわかった。

サポートしているタイプはドキュメントでの明確な説明は見つからなかったが、ソースコードでのファイル名を見れば何とか推測できる  
Rails APIでのファイル名リストを見ると、textが入ってないことがわかった。
[ActiveModel::Type](https://api.rubyonrails.org/classes/ActiveModel/Type.html)

一方、`ActiveRecord::Type`ではtextファイルがあって、中身を見ると、typeを`:text`に指定することができることがわかる。

調べているところ、そもそも`string`と`text`は本質上で違いがないことがわかった。  
ActiveRecordでのtextはActiveModelのStringから継承している。  
([`class Text < ActiveModel::Type::String`](https://github.com/rails/rails/blob/04972d9b9ef60796dc8f0917817b5392d61fcf09/activerecord/lib/active_record/type/text.rb)

ActiveRecordでtext型を指定しても、取扱方はstringとはほぼ一緒。(違いはformでtext_areaタグを生成するぐらい？)

そのため、ActiveModelでtext型を使いたい場合、単にstringにすれば良い。

MySQLやPostgresSQLなどのDBでも、text型とvarchar型は本質上の違いがないらしい。  
例えば、PostgresSQLでは、varchar(n), char(n)とtextが格納できる最大文字長(1GB)は同じで、違いはtextは文字数上限指定がない、残りの二つは上限指定ができること。  
MySQLでは、似たような状況で、VARCHARとTEXTどちらも上限は65,535文字で、ただindex設定したい時や上限指定したい時は、VARCHARが良い。  
それより、元々SQL標準仕様では文字列のデータ型はCHARとVARCHARのみで、text型がない。

参照  
[Mimic SQL table with ActiveModel Attributes](https://stackoverflow.com/questions/63910342/mimic-sql-table-with-activemodel-attributes)  
[PostgreSQL: Difference between text and varchar (character varying)](https://stackoverflow.com/questions/4848964/difference-between-text-and-varchar-character-varying)  
[Difference between VARCHAR and TEXT in MySQL](https://stackoverflow.com/questions/25300821/difference-between-varchar-and-text-in-mysql)  
[PostgreSQL Character Types](https://www.postgresql.org/docs/14/datatype-character.html)  
[MySQL 11.3 String Data Types](https://dev.mysql.com/doc/refman/5.7/en/string-types.html)
