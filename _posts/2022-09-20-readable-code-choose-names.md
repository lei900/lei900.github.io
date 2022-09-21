---
title: Readable Codeメモ：メソッド名や変数名を正しく選ぶ
category: "Software Design"
tags: ["Better Code", Refactor, "Reading Notes"]
---

**読みやすさの基本定理とは**

>コードは他の人が最短時間で理解できるように書かなければいけない。

### 第二章：名前に情報を詰め込む
**1. 明確な単語を選ぶ**

メソッド名や変数名は曖昧な単語を使うより、類語辞典を使って、カラフルな単語を選んだ方良い。

例：  
  - GetPage -> FetchPage, DownloadPage
  - Stop() -> Kill(), Resume(), Pause()
  - sendの類語: deliver, dispatch, announce, distribute, route
  - findの類語: search, extract, locate, recover
  - startの類語: launch, create, begin, open
  - makeの類語: create, set up, build, generate, compose, add, new

**2. tmpやretvalなど汎用的な名前を避ける**

entityの値や目的を表す名前を選ぶ。

ループイテレータに関しては、通常はi, j, kなどを使うが、  
複雑な場合は `club_i`, `members_i`か, 短縮で`clubs[ci]`, `members[mi]`で使う。

**3. 抽象的な名前より具体的な名前を使う**

`disallow_evil_constructors`よりは、もっと明確的に、`disallow_copy_and_assign`を使った方が良い
`ServerCanStart()`より、`CanListenOnPort()`の方が明確的。

**4. 名前に大切な情報を追加する**

例：
- 16進数の文字列を持つ変数名：`hex_id`
- 値の単位を入れる：
  - ミリ秒を表す変数名に`_ms`つける
  - これからエスケープ必要な変数名に `raw_`をつける
  - `size` → `size_mb`, `limit` → `max_kbps`
- 危険や注意を喚起する重要な属性を入れる
 - `untrustedUrl`, `unsafeMessageBody`,` plaintext_password`, `unescaped_comment`
 - `html_utf8`：htmlの文字コードをUTF-8に変えた
 - `data_urlenc`： 入力されたdataをURLエンコードした

**5. 名前の長さを決める：スコープが小さければ短い名前でもいい**

**6. 名前のフォーマットで情報を使える**

例：
- クラス名はCamelCase形式, 変数名はlower_separated形式
- HTMLのidの区切り文字はアンダースコア `id=”middle_column”`
- classの区切り文字はハイフン、`class=”main-content”`

## 第3章：誤解されない名前

**1. 例：filter()**

```python
results = Database.all_objects.filter(”year ≤ 2011”)
```

`results`は2011以前のオブジェクトなのか、2011年以降なのか、
ちょっと曖昧で、もっと明確にするなら、`select()`か、`exclude()`にする

**2. 例：Clip(text, length)**

段落の内容を切り抜く関数 `Clip(text, length)`
最後から`length`文字を削除するか(`remove`)、最大`length`文字まで切り詰めるか(`truncate`)か、また曖昧。
もし`Truncate(text, length)`にするなら、lengthは`max_length`にした方良い

さらに、lengthはバイト数か、文字数か、単語数か、これも明確にする
改善後：`Truncate(text, max_chars)`

**3. 限界値を含めるときはmin/ max使う**

**4. 範囲を指定するときはfirst, last使う**

**5. 包含・排他的範囲にはbegin/end使う**

**6. Boolean値の名前**

ブール値の変数名は `is`, `has`, `can`, `should`などをつける  
※rubyだと、末尾に`?`を付ける方法がある。

`read_password = ture` -> `need_password`, `user_is_authenticated`

否定形を避ける:   
`disable_ssl = false` -> `use_ssl = true`

**7. ユーザーの期待に合わせる**

単語を別の意味で使っていたとしても、ユーザーが先入観を持っているため、誤解を招いてしまうことがある。

例えば、`get()`や`size()`など軽量なメソッドとして期待されている。

例：`getMean()`

ここの`getMean()`は過去のデータを全てイテレートして、その場で平均値を計算するメソッド。

`get`で始まるメソッドは通常なら軽量アクセスという規約があるけど、ここで使うメソッドは大量計算コストがかかる。  
その計算コストがかかることを知らないメンバーだと、思わずに`getMean()`を呼び出してしまう可能性がある。
そういう誤解を避けるため、`computeMean()`を使った方良い。

例：`list::size()`

```c++
void ShrinkList(list<Node>& list, int max_size) {
	while (list.size() > max_size) {
		FreeNode(list.back());
		list.pop_back();
```
ここの`list.size()`はLinked Listのノード数を事前計算せずに順番にカウントしているので、
計算量はO(n)になっている。  
`ShrinkList()`全体の計算量はO($n^2$)になっている。

元々`size()`という名前は暗黙の計算量はO(1)なので、誤解されないようにするため、

この場合は、`countSize()`や`countElement()`にした方が良い。

---
参照  
[リーダブルコード ―より良いコードを書くためのシンプルで実践的なテクニック](https://www.oreilly.co.jp/books/9784873115658/)