---
title: Readable Codeメモ：ループとロジックの単純化
category: "Software Design"
tags: ["Better Code", Refactor, "Reading Notes"]
---

**読みやすさの基本定理とは**

>コードは他の人が最短時間で理解できるように書かなければいけない。

## 第7章：制御フローを読みやすくする

**条件式の引数の並び順**

左側：調査対象の式、変化する；  
右側：比較対象の式、あまり変化しない

例：
```python
# 悪い例
while bytes_received < bytes_expected:
# 良い例
while bytes_expected > bytes_received:
```

**if/else ブロックの並び順**

- 条件は否定形よりも肯定形を使う。`if !debug`ではなく、`if debug`使う
- 単純な条件を先に書く。ifとelseが同じ画面に表示されるので見やすい
- 関心を引く条件や目立つ条件を先に書く

**三項演算子について**

-> 基本的にif/else使う

三項演算子はそれによって簡潔になるときにだけ使う。

考え方：行数を短くするよりも、ほかの人が理解するのにかかる時間を短くする。

**ネストを浅くする**

ネストが増えるたびに「スタックにプッシュ」することが増える。

深いネストを避けるには「直線的」なコードを選択する

## 第8章：巨大な式を分割する

**説明変数を使う**

式を簡単に分割するためには、式を表す「説明変数」を使う。

```python
#　改善前
if line.split(’:’)[0].strip() == “root”:
  ...

#改善後：説明変数usernameを使う
username = line.split(’:’)[0].strip()
if username == “root:
  ...
```

**要約変数**

式を変数に代入しておくと、管理や把握を簡単にできる 

-> 式を要約する変数で、コードを自己文書化する

改善前：
```c++
if request.user.id == document.owner.id)
	// ユーザーはこの文書を編集できる

if (request.user.id != document.owner.id)
	// 文書は読み取り専用
```

改善後：
```c++
user_owns_document = (request.user.id == document.owner_id)

if user_owns_document
	// ユーザーはこの文書を編集できる
if !user_owns_document
	// 文書は読み取り専用
```

**ド・モルガンの法則を使う**

論理式を等価な式に置き換える方法
- not (a or b or c) ↔ (not a) and (not b) and (not c)

- not (a and b and c) ↔ (not a) or (not b ) or not (c)

```c++
// 例1
if (!(a && !b) 
// 下記に書き換えられる
if (!a || b)

// 例2
if (!(file_exists && !is_protected))
// 下記に書き換えられる
if (!file_exists || is_protected)
```

**巨大な文を分割する**

メリット
- タイプミスを減らすために役に立つ。
- 横幅が縮まるのでコードが読みやすくなる。
- クラス名を変更することになれば、一箇所を変更すればよい。

## 第9章：変数と読みやすさ

**邪魔な変数を削除する**

役に立たない一時変数や中間結果を保持するためだけに使っている変数を削除

```c++
now = datetime.dateime.now()
root_message.last_view_time = now
```
`now`変数は役に立たない変数：

  - 複雑な式を分割していない；
  - より明確になっていない；
  - 一度しか使っていないので、重複コードの削除になってない

**変数のスコープをできるだけ小さくする**

変数を数行のコードからしか見えない位置に移動する

**一度だけ書き込む変数を使う**

変数に一度だけ値を設定すれば(或いはconstやfinalなどイミュータブルにする方法)、
コードが理解しやすくなる

---
参照  
[リーダブルコード ―より良いコードを書くためのシンプルで実践的なテクニック](https://www.oreilly.co.jp/books/9784873115658/)
