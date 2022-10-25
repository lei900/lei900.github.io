---
title: Rails 7 でReactを使う方法まとめ
category: "Ruby on Rails"
tags: [Ruby on Rails, JavaScript, React]
---

Rail 7 API + React で PF を作りたいけど、関連チュートリアを探してたとき、やり方が複数あるようで、一時迷っていた。

## Rails 7 で react を使う方法まとめ

Rails で React を使う方法は、下記にまとめてみた。

1. Rails 7 デフォルトの [import-map](https://github.com/rails/importmap-rails) を使って React を Rails に組み込む。

2. JavaScript bundler を使って React を Rails に組み込む。

   - Rails 7 デフォルトの [jsbundling-rails](https://github.com/rails/jsbundling-rails) で、esbuild, rollup.js, Webpack のいずれかを使う。
   - webpacker を使う。(webpacker は既に開発終了)
     - webpacker 後継の [Shakapacker](https://github.com/shakacode/shakapacker) を使う。
     - ほか gem 経由で webpacker / Shakapacker を使う。(主に [react-rails](https://github.com/reactjs/react-rails) と [react_on_rails](https://github.com/shakacode/react_on_rails))

3. React アプリと Rails API サーバ をそれぞれ作る。フロントエンドとバックエンドを完全分離。

### 1. Rails 7 デフォルトの import-map を使う

やり方については、DHH 自身による説明動画がある。
[Alpha preview: Using React with importmaps on Rails 7](https://www.youtube.com/watch?v=k73LKxim6tw)

惜しいことは、React 使うなら、基本的にこの方法を使わない方が良いと思われる。

なぜなら、React の特徴でもある**JSX 構文**が使えないから。

import-map はトランスパイルやコンパイルを経由せず、JavaScript モジュールを CDN などから直接インポートしている。なので、JavaScript へのコンパイルが必要な JSX 構文が使えない。

import-map を使って、React コードを書くと、こんな感じになる。

```javascript
import React from "react";
import ReactDOM from "react-dom";

const Hello = (props) =>
  React.createElement("div", null, `Hello ${props.name}`);

Hello.defaultProps = {
  name: "David",
};

document.addEventListener("DOMContentLoaded", () => {
  ReactDOM.render(
    React.createElement(Hello, { name: "Rails 7" }, null),
    document.getElementById("app")
  );
});
```

糖衣構文を使わず、`React.createElement(component, props, ...children)`そのまま書く必要。

参照： [How to use React with Rails 7](https://learnetto.com/tutorials/how-to-use-react-with-rails-7)

### 2. JavaScript bundler を使う

**`jsbundling-rails`(esbuild)、または`Shakapacker`を使う**

この二つの違いとやり方について、下記のチュートリアを参照できる。

[How to Create a CRUD App with Rails and React](https://hibbard.eu/rails-react-crud-app/)  
[(日本語翻訳版)Rails 7 と React による CRUD アプリ作成チュートリアル](https://techracho.bpsinc.jp/hachi8833/2022_05_26/118202)

また、 import-map、jsbundling と shakapacker の違いについて、下記の記事を参照できる。

[JavaScript in Rails 7](https://ricostacruz.com/posts/javascript-in-rails-7#comparing-them-all)  
[Comparison with webpacker (shakapacker)](https://github.com/rails/jsbundling-rails/blob/main/docs/comparison_with_webpacker.md)

**`react-rails` / `react-on-rails`を使う**

どちらも setup が簡単で、`Shakapacker`をサポートしている。  
ちなみに、`react-on-rails`の開発チームは`Shakapacker`の開発者であり、`react-rails`の保守もサポートしている。
Rails 内で React を使うなら、こちらの gem を使うのも良いと思う。

参照:  
[react-rails](https://github.com/reactjs/react-rails)  
[react_on_rails](https://github.com/shakacode/react_on_rails)

### 3. 独立した React アプリを作る

このやり方では、React と Rails との統合を考慮する必要がなく、Rails API 側はどのようなデータを返すのかを考えれば良い、そしてデータの表現や画面遷移は React 側に任せる。

それぞれの役割にフォーカスできるので、個人的にこの方法が一番やりやすいと思う。

完全分離するやり方について、日本語のチュートリアルは下記を参考できる。

[Rails と React で SPA 開発+AWS（Fargate・CloudFront）デプロイ解説チュートリアル](https://zenn.dev/prune/books/28c2d690e11e45)  
[【Rails×React】UberEats 風アプリを作りながら、SPA 開発を学ぼう](https://www.techpit.jp/courses/138)

ただ、単純に API サーバにするなら、Rails の優位点は何なのか？という質問が自分の中で出ている...

その他参照  
[React and Ruby on Rails Integration Tips](https://sloboda-studio.com/blog/how-to-integrate-react-with-ruby-on-rails/)
