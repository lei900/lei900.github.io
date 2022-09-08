---
title: insert_allやupdate_allを使って、SQLクエリの発行を削減する
category: "Ruby on Rails"
tags: [Ruby on Rails, SQL, Database]
---

今回は東京都知事杯ハッカソンに参加して、二日間のチームワークの下で、なんと投票マッチングアプリのMVPが出来上がった。  
ただ、小さいアプリだけどめっちゃSQL走っていると気がついたので、もっと改善できないかなと思った。

コード全体をもう一度整理して、特にSQL大量発行する箇所に目を留めた。  
- ユーザーと政党テーブル、質問テーブルを紐付けるところ
- ユーザーの選択回答を保存するところ
- 政党ポイントを計算するところ
- 政党マッチング結果一覧を表示するところ

以上の箇所について、
- 紐付けるタイミングを変更できないか、まだ必要のない紐付けだったら一旦保留したら？
- 紐付けは個々のレコードではなく、一括でできないか?
- レコードのupdateは個別で行うより、一括で更新できないか?
- 計算ロジックは、個々の政党レコードを取得して、一つ一つ比較して計算するよりは、二つグループに分けて一括更新するのが可能か?  

と考えながら、コードを書き直してみた。

結果的に、全体的なSQLクエリ発行はほぼ8割程度削減できた！ 

# 修正前後の比較

## ユーザーと政党の紐付け

元々のコードは`each`メソッドを使って、政党レコードを一つ一つ取得して、中間テーブルのレコードを新規作成する形式で、政党の数分でSQL文が発行される。

```ruby
  def create_party_relation
    parties = Party.all
    parties.each do |party|
      UserParty.create!(user_id: self.id, party_id: party.id, point: 0)
    end
  end
```
<details> 
<summary>修正前SQLクエリ発行状況</summary>

<pre style="background-color:whitesmoke;">
Party Load (0.2ms)  SELECT "parties".* FROM "parties"
  ↳ app/models/user.rb:17:in `party_relation'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user.rb:18:in `block in party_relation'
  User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  Party Load (0.2ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  UserParty Create (0.5ms)  INSERT INTO "user_parties" ("user_id", "party_id", "point", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 70], ["party_id", 1], ["point", 0], ["created_at", "2022-09-01 13:19:53.973751"], ["updated_at", "2022-09-01 13:19:53.973751"]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.7ms)  COMMIT
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.2ms)  BEGIN
  ↳ app/models/user.rb:18:in `block in party_relation'
  User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  UserParty Create (0.3ms)  INSERT INTO "user_parties" ("user_id", "party_id", "point", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 70], ["party_id", 2], ["point", 0], ["created_at", "2022-09-01 13:19:53.983360"], ["updated_at", "2022-09-01 13:19:53.983360"]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user.rb:18:in `block in party_relation'
  User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  UserParty Create (0.3ms)  INSERT INTO "user_parties" ("user_id", "party_id", "point", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 70], ["party_id", 3], ["point", 0], ["created_at", "2022-09-01 13:19:53.989469"], ["updated_at", "2022-09-01 13:19:53.989469"]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.2ms)  BEGIN
  ↳ app/models/user.rb:18:in `block in party_relation'
  User Load (0.4ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  Party Load (0.2ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 4], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  UserParty Create (0.3ms)  INSERT INTO "user_parties" ("user_id", "party_id", "point", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 70], ["party_id", 4], ["point", 0], ["created_at", "2022-09-01 13:19:53.998626"], ["updated_at", "2022-09-01 13:19:53.998626"]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.5ms)  COMMIT
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.2ms)  BEGIN
  ↳ app/models/user.rb:18:in `block in party_relation'
  User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 5], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  UserParty Create (0.4ms)  INSERT INTO "user_parties" ("user_id", "party_id", "point", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 70], ["party_id", 5], ["point", 0], ["created_at", "2022-09-01 13:19:54.005710"], ["updated_at", "2022-09-01 13:19:54.005710"]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.5ms)  COMMIT
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user.rb:18:in `block in party_relation'
  User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  Party Load (0.2ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 6], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  UserParty Create (0.3ms)  INSERT INTO "user_parties" ("user_id", "party_id", "point", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 70], ["party_id", 6], ["point", 0], ["created_at", "2022-09-01 13:19:54.012852"], ["updated_at", "2022-09-01 13:19:54.012852"]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.5ms)  COMMIT
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user.rb:18:in `block in party_relation'
  User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  Party Load (0.2ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 7], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  UserParty Create (0.3ms)  INSERT INTO "user_parties" ("user_id", "party_id", "point", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 70], ["party_id", 7], ["point", 0], ["created_at", "2022-09-01 13:19:54.019201"], ["updated_at", "2022-09-01 13:19:54.019201"]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user.rb:18:in `block in party_relation'
  User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 8], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  UserParty Create (0.4ms)  INSERT INTO "user_parties" ("user_id", "party_id", "point", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 70], ["party_id", 8], ["point", 0], ["created_at", "2022-09-01 13:19:54.026128"], ["updated_at", "2022-09-01 13:19:54.026128"]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user.rb:18:in `block in party_relation'
  User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 9], ["LIMIT", 1]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  UserParty Create (0.3ms)  INSERT INTO "user_parties" ("user_id", "party_id", "point", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 70], ["party_id", 9], ["point", 0], ["created_at", "2022-09-01 13:19:54.033751"], ["updated_at", "2022-09-01 13:19:54.033751"]]
  ↳ app/models/user.rb:18:in `block in party_relation'
  TRANSACTION (0.5ms)  COMMIT
  ↳ app/models/user.rb:18:in `block in party_relation'
</pre>
</details>

改善後のやり方は`pluck`を使って、政党IDを一括取得して、｀user.user_parties｀の中身となるデータを配列形式で一括生成する。  
そして`inser_all`メソッドを使ってUserPartyテーブルに一括挿入DBへ挿入する。  
これでSQLクエリが一回で済む。
```ruby
  def create_party_relation
    record_array = Party.pluck(:id).map { |party_id| { user_id: self.id, 
                                                        party_id: party_id, point: 0 } }
    UserParty.insert_all(record_array)
  end
```
<details> 
<summary>修正後SQLクエリ発行状況</summary>

```ruby
Party Pluck (0.2ms)  SELECT "parties"."id" FROM "parties"
  ↳ app/models/user.rb:14:in `create_party_relation'
  UserParty Bulk Insert (1.1ms)  INSERT INTO "user_parties" ("user_id","party_id","point","created_at","updated_at") VALUES (60, 1, 0, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP), (60, 2, 0, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP), (60, 3, 0, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP), (60, 4, 0, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP), (60, 5, 0, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP), (60, 6, 0, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP), (60, 7, 0, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP), (60, 8, 0, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP), (60, 9, 0, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP) ON CONFLICT  DO NOTHING RETURNING "id"
  ↳ app/models/user.rb:15:in `create_party_relation'
```

</details>

## ユーザーと質問の紐付け
元々ゲストユーザーが作成された時点で、ユーザーを全部の質問と紐付けるというやり方だったけど、  
そもそも最初から全部の質問と紐づける必要があるのか、と疑問に感じた。
もし今後ユーザーが質問をskipする機能を追加したら、その質問と紐づける自体も必要がなくなる。  

元々のコード
```ruby
# ユーザーと全部の質問を紐付けるつもり
  def question_relation
    questions = Question.all
    questions.each do |question|
      UserQuestion.create!(user_id: self.id, question_id: question.id)
    end
  end
```
これも質問の数分でSQLクエリが発行される。

<details> 
<summary>修正前SQLクエリ発行状況</summary>

```ruby
 Question Load (0.2ms)  SELECT "questions".* FROM "questions"
  ↳ app/models/user.rb:24:in `question_relation'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user.rb:25:in `block in question_relation'
  User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 69], ["LIMIT", 1]]
  ↳ app/models/user.rb:25:in `block in question_relation'
  Question Load (0.1ms)  SELECT "questions".* FROM "questions" WHERE "questions"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  ↳ app/models/user.rb:25:in `block in question_relation'
  UserQuestion Create (0.5ms)  INSERT INTO "user_questions" ("user_id", "question_id", "result", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 69], ["question_id", 2], ["result", nil], ["created_at", "2022-09-01 13:16:31.900255"], ["updated_at", "2022-09-01 13:16:31.900255"]]
  ↳ app/models/user.rb:25:in `block in question_relation'
  TRANSACTION (40.2ms)  COMMIT
  ↳ app/models/user.rb:25:in `block in question_relation'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user.rb:25:in `block in question_relation'
  User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 69], ["LIMIT", 1]]
  ↳ app/models/user.rb:25:in `block in question_relation'
  Question Load (0.2ms)  SELECT "questions".* FROM "questions" WHERE "questions"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
  ↳ app/models/user.rb:25:in `block in question_relation'
  UserQuestion Create (0.4ms)  INSERT INTO "user_questions" ("user_id", "question_id", "result", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 69], ["question_id", 3], ["result", nil], ["created_at", "2022-09-01 13:16:31.955346"], ["updated_at", "2022-09-01 13:16:31.955346"]]
  ↳ app/models/user.rb:25:in `block in question_relation'
  TRANSACTION (1.8ms)  COMMIT
  ↳ app/models/user.rb:25:in `block in question_relation'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user.rb:25:in `block in question_relation'
  User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 69], ["LIMIT", 1]]
  ↳ app/models/user.rb:25:in `block in question_relation'
  Question Load (0.1ms)  SELECT "questions".* FROM "questions" WHERE "questions"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  ↳ app/models/user.rb:25:in `block in question_relation'
  UserQuestion Create (0.3ms)  INSERT INTO "user_questions" ("user_id", "question_id", "result", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 69], ["question_id", 1], ["result", nil], ["created_at", "2022-09-01 13:16:31.962690"], ["updated_at", "2022-09-01 13:16:31.962690"]]
  ↳ app/models/user.rb:25:in `block in question_relation'
  TRANSACTION (0.5ms)  COMMIT
  ↳ app/models/user.rb:25:in `block in question_relation'
```

</details>

もう一つのデメリットとしては、最初から全部の質問と紐づいて、後でユーザーが質問を答えたら、また質問レコードを取得して、データを更新する操作が必要。それで質問レコードを取得するクエリ自体も重複発行になる。

修正後のコードは、まず一括紐付けるのでなく、ユーザーが質問回答してから、該当の質問と紐付けるようにする。
ユーザーが選択した回答も紐付けと一緒に記録する。  
修正後コード
```ruby
  def save_result(question, result)
    user_questions.create(question_id: question.id, result: result)
  end
```
<details> 
<summary>修正後SQLクエリ発行状況</summary>

```ruby
  Question Load (0.2ms)  SELECT "questions".* FROM "questions" WHERE "questions"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  ↳ app/models/user.rb:19:in `save_result'
  UserQuestion Create (0.5ms)  INSERT INTO "user_questions" ("user_id", "question_id", "result", "created_at", "updated_at") VALUES ($1, $2, $3, $4, $5) RETURNING "id"  [["user_id", 60], ["question_id", 1], ["result", 0], ["created_at", "2022-09-01 11:57:34.573782"], ["updated_at", "2022-09-01 11:57:34.573782"]]
  ↳ app/models/user.rb:19:in `save_result'
  TRANSACTION (5.8ms)  COMMIT
  ↳ app/models/user.rb:19:in `save_result'
```

</details>

## 政党ポイント計算メソッド修正

政党ポイント計算のロジックとしては、  
- ユーザー選択が超賛成の議題に対して、政党意見が--  
  賛成の場合、該当政党との相性度+2点  
  反対の場合、相性度-1点  
- ユーザー選択が賛成の議題に対して、政党意見が--  
  賛成の場合、政党との相性度+1点  
  反対の場合、相性度-1点  
- ユーザー選択が反対の議題に対して、政党意見が--  
  賛成の場合、該当政党との相性度-1点  
  反対の場合、相性度+1点  
- ユーザー選択が無関心の議題に対して、政党との相性度が加減しない  

つまり、政党とユーザー意見が一致したら、加点になる、正反対だったら減点になること。

最初から政党を賛成派と反対派を分けて、データ更新したいつもりだったけど、
必要な参照データがuser_questionsテーブル、user_partiesテーブルとparty_quesionsテーブルの三つのテーブルに分散しているので、うまくデータを一括操作する方法が思い出さなかった。  
とりあえず、政党意見を一つ一つ取り出して、ユーザー意見と比較する方法にした。
作成したコードもややこしい感じで、チームメンバーに理解してもらうのが時間かかったそう。

<details> 
<summary>修正前のコード</summary>

1. まずcontrollerのアクション内で、ユーザーと政党の中間テーブルuser_partiesからをユーザーと紐づいた政党レコードを一つ一つ取り出して、ポイント計算を行う

```ruby
    current_user.user_parties.each do |user_party|
      user_party.calculate_point(user_question)
    end
```

2. ポイント計算のメソッドの中身として、まず政党と質問の中間テーブルparty_questionsから政党意見を取得し、  
そしてユーザー意見と比較して、一致したら、加点する;不一致だったら、減点する。

```ruby
  def calculate_point(user_question)
    # 政党意見を参照するため、party_questionsテーブルから政党を取得してopinionを参照する
    party_questions = PartyQuestion.where(question_id: user_question.question_id)
    party_question = party_questions.find_by(party_id: party_id)
    # ユーザーの選択意見
    user_result = user_question.result

    #　ユーザーと政党の意見が一致する場合、政党pointを加点する
    if user_question.agree_with_party?(party_question)
      add_point(user_result)
    # ユーザーと政党の意見が正反対の場合、政党pointを減点する
    # 政党が中立やユーザー意見が無回答の場合は、処理なし
    elsif user_question.disagree_with_party?(party_question)
      reduce_point(user_result)
    end
  end
```

3. さらにユーザーと政党意見の一致か不一致かを判断するメソッドを追加する  
ここは中間テーブル名を挟んで、見た目もややこしい感じになっている。

```ruby
  # ユーザ意見が政党と一致する場合
  # つまり両方とも賛成、あるいはともに反対する場合
  def agree_with_party?(party_question)
    both_agree?(party_question) || both_disagree?(party_question)
  end

  # ユーザ意見が政党と不一致する場合
  # つまり一方が賛成で、もう一方は反対
  def disagree_with_party?(party_question)
    user_positive_but_party_disagree?(party_question) || user_negative_but_party_agree?(party_question)
  end

  # 議題に対して、ユーザも政党も賛成する場合
  def both_agree?(party_question)
    result_positive? && party_question.agree?
  end

  # 議題に対して、ユーザも政党も反対する場合
  def both_disagree?(party_question)
    result_negative? && party_question.disagree?
  end

  # 議題に対して、ユーザが賛成する場合
  # つまり選択意見が超賛成か賛成の場合
  def result_positive?
    self.great? || self.good?
  end

  # ユーザ意見が反対の場合
  def result_negative?
    self.bad?
  end

  # 議題に対して、ユーザが賛成で、政党が反対する場合
  def user_positive_but_party_disagree?(party_question)
    result_positive? && party_question.disagree?
  end

  # 議題に対して、ユーザが反対で、政党が賛成する場合
  def user_negative_but_party_agree?(party_question)
    result_negative? && party_question.agree?
  end
```

4. 最後に加点と減点を実行するメソッド

```ruby
  # ユーザー意見が超賛成の場合、政党point+2
  # ユーザー意見が賛成の場合、政党point+1
  def add_point(user_result)
    if user_result == "great"
      self.point += 2
      save
    elsif user_result == "good"
      self.point += 1
      save
    end
  end

  # ユーザーと政党が正反対の場合合、政党point-1
  def reduce_point(user_result)
    self.point -= 1
    save
  end
```

</details>

修正前のSQLクエリ発行も政党の数分で政党意見取得と政党ポイント更新の2重発行になっている。
<details> 
<summary>修正前SQLクエリ発行状況</summary>

```ruby
# 修正前
User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 70], ["LIMIT", 1]]
  ↳ app/controllers/application_controller.rb:5:in `current_user'
  UserParty Load (0.2ms)  SELECT "user_parties".* FROM "user_parties" WHERE "user_parties"."user_id" = $1  [["user_id", 70]]
  ↳ app/controllers/questions_controller.rb:15:in `answer'
  PartyQuestion Load (0.2ms)  SELECT "party_questions".* FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."party_id" = $2 LIMIT $3  [["question_id", 2], ["party_id", 1], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:11:in `calculate_point'
  PartyQuestion Load (0.2ms)  SELECT "party_questions".* FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."party_id" = $2 LIMIT $3  [["question_id", 2], ["party_id", 2], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:11:in `calculate_point'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user_party.rb:30:in `add_point'
  Party Load (0.2ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:30:in `add_point'
  UserParty Update (0.4ms)  UPDATE "user_parties" SET "point" = $1, "updated_at" = $2 WHERE "user_parties"."id" = $3  [["point", 4], ["updated_at", "2022-09-01 13:23:01.801733"], ["id", 513]]
  ↳ app/models/user_party.rb:30:in `add_point'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user_party.rb:30:in `add_point'
  PartyQuestion Load (0.2ms)  SELECT "party_questions".* FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."party_id" = $2 LIMIT $3  [["question_id", 2], ["party_id", 3], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:11:in `calculate_point'
  TRANSACTION (0.2ms)  BEGIN
  ↳ app/models/user_party.rb:30:in `add_point'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:30:in `add_point'
  UserParty Update (0.3ms)  UPDATE "user_parties" SET "point" = $1, "updated_at" = $2 WHERE "user_parties"."id" = $3  [["point", 4], ["updated_at", "2022-09-01 13:23:01.809307"], ["id", 514]]
  ↳ app/models/user_party.rb:30:in `add_point'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user_party.rb:30:in `add_point'
  PartyQuestion Load (0.2ms)  SELECT "party_questions".* FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."party_id" = $2 LIMIT $3  [["question_id", 2], ["party_id", 4], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:11:in `calculate_point'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user_party.rb:30:in `add_point'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 4], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:30:in `add_point'
  UserParty Update (0.3ms)  UPDATE "user_parties" SET "point" = $1, "updated_at" = $2 WHERE "user_parties"."id" = $3  [["point", 1], ["updated_at", "2022-09-01 13:23:01.815549"], ["id", 515]]
  ↳ app/models/user_party.rb:30:in `add_point'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user_party.rb:30:in `add_point'
  PartyQuestion Load (0.2ms)  SELECT "party_questions".* FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."party_id" = $2 LIMIT $3  [["question_id", 2], ["party_id", 5], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:11:in `calculate_point'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user_party.rb:30:in `add_point'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 5], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:30:in `add_point'
  UserParty Update (0.4ms)  UPDATE "user_parties" SET "point" = $1, "updated_at" = $2 WHERE "user_parties"."id" = $3  [["point", 1], ["updated_at", "2022-09-01 13:23:01.822521"], ["id", 516]]
  ↳ app/models/user_party.rb:30:in `add_point'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user_party.rb:30:in `add_point'
  PartyQuestion Load (0.2ms)  SELECT "party_questions".* FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."party_id" = $2 LIMIT $3  [["question_id", 2], ["party_id", 6], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:11:in `calculate_point'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user_party.rb:30:in `add_point'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 6], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:30:in `add_point'
  UserParty Update (0.2ms)  UPDATE "user_parties" SET "point" = $1, "updated_at" = $2 WHERE "user_parties"."id" = $3  [["point", 4], ["updated_at", "2022-09-01 13:23:01.829242"], ["id", 517]]
  ↳ app/models/user_party.rb:30:in `add_point'
  TRANSACTION (0.5ms)  COMMIT
  ↳ app/models/user_party.rb:30:in `add_point'
  PartyQuestion Load (0.2ms)  SELECT "party_questions".* FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."party_id" = $2 LIMIT $3  [["question_id", 2], ["party_id", 7], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:11:in `calculate_point'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user_party.rb:30:in `add_point'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 7], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:30:in `add_point'
  UserParty Update (0.4ms)  UPDATE "user_parties" SET "point" = $1, "updated_at" = $2 WHERE "user_parties"."id" = $3  [["point", 1], ["updated_at", "2022-09-01 13:23:01.835753"], ["id", 518]]
  ↳ app/models/user_party.rb:30:in `add_point'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user_party.rb:30:in `add_point'
  PartyQuestion Load (0.2ms)  SELECT "party_questions".* FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."party_id" = $2 LIMIT $3  [["question_id", 2], ["party_id", 8], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:11:in `calculate_point'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user_party.rb:30:in `add_point'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 8], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:30:in `add_point'
  UserParty Update (0.2ms)  UPDATE "user_parties" SET "point" = $1, "updated_at" = $2 WHERE "user_parties"."id" = $3  [["point", 1], ["updated_at", "2022-09-01 13:23:01.842407"], ["id", 519]]
  ↳ app/models/user_party.rb:30:in `add_point'
  TRANSACTION (0.5ms)  COMMIT
  ↳ app/models/user_party.rb:30:in `add_point'
  PartyQuestion Load (0.1ms)  SELECT "party_questions".* FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."party_id" = $2 LIMIT $3  [["question_id", 2], ["party_id", 9], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:11:in `calculate_point'
  TRANSACTION (0.1ms)  BEGIN
  ↳ app/models/user_party.rb:30:in `add_point'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 9], ["LIMIT", 1]]
  ↳ app/models/user_party.rb:30:in `add_point'
  UserParty Update (0.3ms)  UPDATE "user_parties" SET "point" = $1, "updated_at" = $2 WHERE "user_parties"."id" = $3  [["point", 2], ["updated_at", "2022-09-01 13:23:01.848200"], ["id", 520]]
  ↳ app/models/user_party.rb:30:in `add_point'
  TRANSACTION (0.4ms)  COMMIT
  ↳ app/models/user_party.rb:30:in `add_point'
```

</details>

修正後のロジックはなるべく設計の時の考え方をそのまま反映するようにした。
まず個々の政党でなく、ユーザーと紐づいた政党レコードを一括更新できるようにする。
```ruby
current_user.user_parties.calculate_point(@question, result)
```
そして賛成意見を持つ政党リストと反対意見を持つ政党リストを取得する。
最後に`update_all`メソッドを使って、ふたつグループの政党ポイントを一括更新する。
```ruby
  def self.calculate_point(question, user_result)
    # 該当質問に賛成意見を持つ政党リストを取得
    agree_party_ids = question.party_questions.where(opinion: :agree).pluck(:party_id)
    agree_user_parties = self.where('party_id IN (?)', agree_party_ids)

    # 該当質問に反対意見を持つ政党リストを取得
    disagree_party_ids = question.party_questions.where(opinion: :disagree).pluck(:party_id)
    disagree_user_parties = self.where('party_id IN (?)', disagree_party_ids)

    case user_result
    when "great" # ユーザーが超賛成の場合
      agree_user_parties.update_all("point = point + 2")
      disagree_user_parties.update_all("point = point - 1")
    when "good" # ユーザーが賛成の場合
      agree_user_parties.update_all("point = point + 1")
      disagree_user_parties.update_all("point = point - 1")
    when "bad" # ユーザーが反対の場合
      agree_user_parties.update_all("point = point - 1")
      disagree_user_parties.update_all("point = point + 1")
    end
  end
```

<details> 
<summary>修正後SQLクエリ発行状況</summary>

```ruby
# 修正後
  User Load (0.3ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 60], ["LIMIT", 1]]
  ↳ app/controllers/application_controller.rb:5:in `current_user'
  PartyQuestion Pluck (0.3ms)  SELECT "party_questions"."party_id" FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."opinion" = $2  [["question_id", 1], ["opinion", 0]]
  ↳ app/models/user_party.rb:9:in `calculate_point'
  PartyQuestion Pluck (0.2ms)  SELECT "party_questions"."party_id" FROM "party_questions" WHERE "party_questions"."question_id" = $1 AND "party_questions"."opinion" = $2  [["question_id", 1], ["opinion", 1]]
  ↳ app/models/user_party.rb:12:in `calculate_point'
  UserParty Update All (0.6ms)  UPDATE "user_parties" SET point = point + 2 WHERE "user_parties"."user_id" = $1 AND (party_id IN (1,2,3,6))  [["user_id", 60]]
  ↳ app/models/user_party.rb:17:in `calculate_point'
  UserParty Update All (0.7ms)  UPDATE "user_parties" SET point = point - 1 WHERE "user_parties"."user_id" = $1 AND (party_id IN (4,5,7,8))  [["user_id", 60]]
  ↳ app/models/user_party.rb:18:in `calculate_point'
```

</details>

## 結果一覧メソッドの修正

結果ページでは政党ポイントランキングを表示する。
ここでchartjsを使うので、政党名と政党ポイントを分けて取得する必要
最初のやり方としては、まず政党とポイントをそれぞれ空の配列を作成し、user_partiesテーブルから一つ一つレコードを取得して、配列にデータをpushする。  
これも政党の数分でSQLが発行される。  
修正前コード
```ruby
def result
    user_parties = UserParty.where(user_id: current_user.id)
    @results = user_parties.ranking
    parties = []
    points = []
    @results.each do |result|
      point = result.point
      party_name = result.party.name
      parties.push(party_name)
      points.push(point)
    end
```

<details> 
<summary>修正前SQLクエリ発行状況</summary>

```ruby
# 修正前
Processing by StaticPagesController#result as TURBO_STREAM
  User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 60], ["LIMIT", 1]]
  ↳ app/controllers/application_controller.rb:5:in `current_user'
  UserParty Load (0.3ms)  SELECT "user_parties".* FROM "user_parties" WHERE "user_parties"."user_id" = $1 ORDER BY "user_parties"."point" DESC  [["user_id", 60]]
  ↳ app/controllers/static_pages_controller.rb:11:in `result'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 6], ["LIMIT", 1]]
  ↳ app/controllers/static_pages_controller.rb:13:in `block in result'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 9], ["LIMIT", 1]]
  ↳ app/controllers/static_pages_controller.rb:13:in `block in result'
  Party Load (0.2ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
  ↳ app/controllers/static_pages_controller.rb:13:in `block in result'
  Party Load (0.2ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 8], ["LIMIT", 1]]
  ↳ app/controllers/static_pages_controller.rb:13:in `block in result'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 4], ["LIMIT", 1]]
  ↳ app/controllers/static_pages_controller.rb:13:in `block in result'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 7], ["LIMIT", 1]]
  ↳ app/controllers/static_pages_controller.rb:13:in `block in result'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
  ↳ app/controllers/static_pages_controller.rb:13:in `block in result'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
  ↳ app/controllers/static_pages_controller.rb:13:in `block in result'
  Party Load (0.1ms)  SELECT "parties".* FROM "parties" WHERE "parties"."id" = $1 LIMIT $2  [["id", 5], ["LIMIT", 1]]
  ↳ app/controllers/static_pages_controller.rb:13:in `block in result'
  Rendering layout layouts/application.html.erb
```

</details>

修正後は、`joins`と`pluck`メソッド使って、政党ポイントデータを一括取得する
```ruby
  def result
    user_parties = current_user.user_parties.ranking
    parties = user_parties.joins(:party).pluck(:name)
    points = user_parties.pluck(:point)
  end
```
<details> 
<summary>修正後SQLクエリ発行状況</summary>

```ruby
# 修正後
User Load (0.2ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 66], ["LIMIT", 1]]
  ↳ app/controllers/application_controller.rb:5:in `current_user'
  UserParty Pluck (0.4ms)  SELECT "name" FROM "user_parties" INNER JOIN "parties" ON "parties"."id" = "user_parties"."party_id" WHERE "user_parties"."user_id" = $1 ORDER BY "user_parties"."point" DESC  [["user_id", 66]]
  ↳ app/controllers/static_pages_controller.rb:9:in `result'
  UserParty Pluck (0.2ms)  SELECT "user_parties"."point" FROM "user_parties" WHERE "user_parties"."user_id" = $1 ORDER BY "user_parties"."point" DESC  [["user_id", 66]]
  ↳ app/controllers/static_pages_controller.rb:10:in `result'
```

</details>

以上でSQLクエリが大量発行したコード部分をリファクタリングした。
ただ政党ポイント計算のところはまだ改善の余地があるかなと思った。後日またやってみたい。

---
参照  

[https://stackoverflow.com/questions/4967191/mass-increment-field-by-1](https://stackoverflow.com/questions/4967191/mass-increment-field-by-1)

[https://phrase.com/blog/posts/activerecord-speed-up-your-sql-queries/](https://phrase.com/blog/posts/activerecord-speed-up-your-sql-queries/)

[https://stackoverflow.com/questions/2509320/saving-multiple-objects-in-a-single-call-in-rails](https://stackoverflow.com/questions/2509320/saving-multiple-objects-in-a-single-call-in-rails)
