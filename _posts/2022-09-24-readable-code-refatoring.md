---
title: Readable Codeメモ：コードをリファクタリングする
category: "Software Design"
tags: ["Better Code", Refactor, "Reading Notes"]
---

**読みやすさの基本定理とは**

>コードは他の人が最短時間で理解できるように書かなければいけない。

## 第10章：無関係の下位問題を抽出

**プロジェクト固有のコードから汎用コードを分離すること**

一般的な問題を解決するライブラリやヘルパー関数を作っていれば、プログラムに固有の**小さな核**だけが残る

- 関数やコードブロックを見て「このコードの高いレベルの目標は何か？」と自問する
- コードの各行に対して「高い目標に直接的に効果があるのか？、或いは、無関係の下位問題を解決しているのか？」と自問する
- 無関係の下位問題を解決しているコードが相当量あれば、それらを抽出して別の関数を作る


例： `findClosestLocation()`

関数の高い目標は与えられた地点から最も近い場所を見つける

```javascript
// 与えられた緯度経度に最も近い'array'の要素を返す
// 地球が完全な球体であることを前提としている
var findClosestLocation = function(lat, lng, array) {
    var closest;
    var closest_dist = Number.MAX_VALUE;
    for(var i = 0;i < array.length;i++) {
        var lat_rad = radians(lat);
        var lng_rad = radians(lng);
        var lat2_rad = radians(array[i].latitude);
        var lng2_rad = radians(array[i].lngitude);

        // 「球面三角法の第二余弦定理」を使う
        var dist = Math.acos(
            Math.sin(lat_rad) * Math.sin(lat2_rad) + 
            Math.cos(lat_rad) * Math.cos(lat2_rad) *
            Math.cos(lng2_rad - lng_rad)
        );

        if(dist < closest_dist) {
            closest = array[i];
            closest_dist = dist;
        }
    }
    return closest;
}
```

二つ地点（経度緯度）の球面距離を算出するというのが下位問題として、新しい関数`spherical_distance()`を作る

```javascript
var spherical_distance = function(lat1, lng1, lat1, lng2) {
    var lat1_rad = radians(lat1);
    var lng1_rad = radians(lng1);
    var lat2_rad = radians(lat2);
    var lng2_rad = radians(lng2);
    // 「球面三角法の第二余弦定理」を使う
    return Math.acos(
        Math.sin(lat1_rad) * Math.sin(lat2_rad) +
        Math.cos(lat1_rad) * Math.cos(lat2_rad) *
        Math.cos(lng2_rad - lng1_rad);
    );
}
```
改善後
```javascript
// 与えられた緯度経度に最も近い'array'の要素を返す
// 地球が完全な球体であることを前提としている
var findClosestLocation = function(lat, lng, array) {
    var closest;
    var closest_dist = Number.MAX_VALUE;
    for(var i = 0;i < array.length;i++) {
        var lat_rad = radians(lat);
        var lng_rad = radians(lng);
        var lat2_rad = radians(array[i].latitude);
        var lng2_rad = radians(array[i].lngitude);

        // 「球面三角法の第二余弦定理」を使う
        var dist = spherical_distance(lat, lng, array[i].latitude, array[i].longtude);

        if(dist < closest_dist) {
            closest = array[i];
            closest_dist = dist;
        }
    }
    return closest;
}
```

**純粋なユーティリティコード**

プログラムの核となる基本的なタスク、文字列の操作、ファイルの読み書きなどは「基本的なユーティリティ」と呼ぶ。  
基本的にプログラミング言語の組み込みライブラリとして実装されている。

たまに自分でこの溝を埋めなきゃいけないこととき、例えば、C++では、ファイルの中身を全て読み込む方法が用意されていない。  
その場合は自分で書くことになる。  
その場合は、複数のプロジェクトで使えるユーティリティコードにしておく。

**汎用的なコードをたくさん作る**

コードの高レベルの目標と、コードの大部分の処理が異なっている場合は、コードを分割する。

コードが独立していれば、そのコードの改善が楽になる。

汎用コードをたくさん作ることにより、プロジェクトから完全に切り離されたコードを作ることができる。

## 第11章： 一度に一つのことをする

**コードは一つずつタスクを行うようにする**

1. コードが行なっているタスクを全て列挙する
2. タスクをできるだけ異なる関数に分割する。少なくとも異なる領域に分割する

**例1：投票のスコアを計算する**

ユーザーが賛成(up),反対(down)、無選択("")のいずれかを選択できる、選択の更新ができる

```javascript
var vote_changed = functoin(old_vote, new_vote) {
    var score = get_score();

    if(new_vote !== old_vote) {
        if(new_vote === 'Up') {
            score += (old_vote === 'Down' ? 2 : 1);
        } else if(new_vote === 'Down') {
            score -= (old_vote === 'Up' ? 2 : 1);
        } else if(new_vote === '') {
            score += (old_vote === 'Up' ? -1 : 1);
        }
    }

    set_score(score);
}
```
タスクの分解：
1. old_voteとnew_voteを数値に「パース」する。
```javascript
var vote_value = function(value) {
    if(vote === 'Up') {
        return +1;
    }
    if(vote === 'Down') {
        return -1;
    }
    return 0;
}
```
2. scoreを更新する
```javascript
var vote_chagned = function(old_value, new_vote) {
    var score = get_score();
    score -= vote_value(old_vote); // 古い値を削除する
    score += vote_value(new_vote); // 新しい値を追加する
}
```

**例2：ユーザー所在地を読みやすい文字列に整形する**

location_infoの情報から"Sanata Monica, USA"のような「都市、国」の形に整形する

```ruby
location_info = { locality_name:                "Santa Monica", 
									sub_administrative_area_name: "Los Angeles",
									administrative_area_name:     "Califolnia",
									country_name:                 "USA",
								}
```

ロジックは
- 都市を選ぶときに、locality_name -> sub_administrative_area_name -> administrative_area_name の順番で使用可能なものを使う
- 以上三つ全て使えない場合は、都市に[Middle-of-Nowhere]というデフォルト値を使う
- 国名はcountry_nameが使えない場合、[Planet Earth]というデフォルト値を使う

実装コード
```ruby
place = location_info[:locality_name] # 例: "Santa Monica"

if !place
  place = location_info[:sub_administrative_area_name] # 例: "Los Angeles"
end

if!place
  place = location_info[:administrative_area_name] # 例: "California"
end

if !place
  place = "Middle-of-Nowhere"
end

if location_info[:country_name]
  place += ", " + location_info[:country_name] # 例: "USA"
else
  palce += ", Plaent Earth"
end

return place
```
もしアメリカ国内の住所に対して、国ではなく、「州/state」レベルを表示可能という機能を追加すると、コードがややこしくなる

そして、今のタスクを分解すると
1. location_infoハッシュから値を抽出しゅる
2. 「都市」の優先順位を調べる。何も見つからない場合、デフォルト値使う
3. 「国」を取得する、なければデフォルト値使う
4. placeを更新する

```ruby
town = location_info[:locality_name]
city = location_info[:sub_administrative_area_name]
state = location_info[:administrative_area_name]
country = location_info[:country_name]

first_half = "Middle-of-Nowhere" # 先にデフォルト値を設定する
if state && (country != "USA")
  first_half = state
end

if city
  first_half = city
end

if town
  first_half = town
end

second_half = "Planet Earth"
if country
  second_half = country
end

if state && (country == "USA") 
  second_half = state
end

retun first_half + "," + second_half
```

他の手法：　
```ruby
a = 1 || 2 || 3 # 最初に「真」と評価できる値を取り出す
=> 1
```
上記によって、
```ruby
if country == "USA"
    first_half = town || city || "Middle-of-Nowhere"
    second_half = state || "USA"
else 
    first_half = town || city || state || "Middle-of-Nowhere"
    second_half = contry || "Planet Earth"
end
```

## 第12章：コードは簡単な言葉で書くべき

1. コードの動作を簡単な言葉で同僚にもわかるように説明する
2. その説明の中で使っているキーワードやフレーズに注目する
3. その説明に合わせてコードを書く

問題や設計をうまく言葉で説明できないのであれば、何かを見落としているか、詳細が明確になっていないといことだ。

## 第13章：短いコードを書く

- 不必要な機能をプロダクトから削除する。過剰な機能は持たせない
- 最も簡単に問題を解決できるような要求を考える
- 定期的に全てのAPIを読んで、表示ライブラリに慣れ親しんでおく

## 第14章：テストと読みやすさ

- テストのトップレベルはできるだけ簡潔にする。入出力のテストはコード1行で記述できるといい
- テストが失敗したらバグの発見や修正がしやすいようなエラーメッセージを表示する
- テストに有効な最も単純な入力値を使う
- テスト関数に説明的な名前をつけて、何をテストしているのかを明らかにする。`Test_1()`ではなく、`Test_<関数名>_<状況>`のような名前にする


---
参照  
[リーダブルコード ―より良いコードを書くためのシンプルで実践的なテクニック](https://www.oreilly.co.jp/books/9784873115658/)
