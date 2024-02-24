---
title: Reactの条件付きレンダーで`&&`演算子使う時の注意点
category: "Coding"
tags: [JavaScript, React]
---

今朝 medium で[Stop Using “&&” for Conditional Rendering in React](https://medium.com/geekculture/stop-using-for-conditional-rendering-in-react-a0f7b96200f8)という記事を見かけた。

`&&`演算子を使った条件付きレンダーはちょうど昨日の動画教材で勉強したけど、この記事はそれを使わない方が良い理由を紹介したので、補充説明としてまた勉強になった。

`&&`演算子の注意点について、React 公式ドキュメントでも紹介があったけど、赤い文字とかで特別強調していないので、見逃されやすいかも。

## `&&`演算子を使った条件付きレンダー

React 公式ドキュメントでは下記の例を使っている。

未読メッセージが 0 件以上なら、ユーザーに提示するという条件付きレンダーがある。

```javascript
unreadMessages.length > 0 && (
  <h2>You have {unreadMessages.length} unread messages.</h2>
);
```

ここは論理演算子の短絡評価(short-circuit evaluation)という評価法を利用している。  
ruby でも似たよう使い方があるので、理解が早いと思う。

もし'&&'演算子の左の条件式が

- `true`の場合、 右の式が必ず評価される
- `false`の場合、右の式が評価されない

なぜなら、
`false && expression`の場合は、必ず`false`と評価して返すので、右の式を評価する必要がない。

React はそれの特性を利用して、もし条件部分が`true`の場合、`&&`後ろの要素が出力される。

もし条件部分が`false`なら、React は`&&`後ろの要素を無視して飛ばす。

## 注意点

ただ、`&&`演算子を使うとき、注意しなければならない点がある。

それは、もし`&&`左の条件部分の評価結果は boolean 型でない場合、問題が起こること。

- もし条件部分が`0`になった場合、`0`そのものが出力される。

例えば、下記の`count && <h1>Messages: {count}</h1>`について、

`count`が`0`になる場合、JavaScript では`falsy`な値に該当し、右の式が出力されないけど、`0`そのものが返されるため、最終的に`<div>0</div>`がレンダーされる。

```js
return <div>{count && <h1>Messages: {count}</h1>}</div>;
```

- もし条件部分が`undefined`になった場合、エラーが起こる

`Uncaught Error: Error(...): Nothing was returned from render. This usually means a return statement is missing. Or, to render nothing, return null.`

## 代替案：if 文か三項演算子を使う

予想外の問題を回避するため、`&&`を使うより、普通の if 文か三項演算子を使った方が良い。

```js
if (count) {
  <h1>Messages: {count}</h1>;
}

// or 三項演算子を使う

count ? <UnreadMessageコンポーネント /> : null;
```

[Stop Using “&&” for Conditional Rendering in React](https://medium.com/geekculture/stop-using-for-conditional-rendering-in-react-a0f7b96200f8)記事の結論としては、
`&&`より、常に三項演算子を使った方良いということ。

```
// Bad
条件式 && <条件付きコンポーネント />

// Good
条件式 ? <条件付きコンポーネント /> : null
```

個人的には、三項演算子を使う場合、式が少し複雑になると、全体的に読みにくくなるので、
場合によって、使い分けた方が良いと思う。

参照

[React 条件付きレンダー](https://ja.reactjs.org/docs/conditional-rendering.html)  
[Stop Using “&&” for Conditional Rendering in React](https://medium.com/geekculture/stop-using-for-conditional-rendering-in-react-a0f7b96200f8)
