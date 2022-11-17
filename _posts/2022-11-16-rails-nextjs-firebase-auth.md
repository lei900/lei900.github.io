---
title: Rails + Next.js + Firebase V9 Authentication で認証機能を実装する
category: "Rails"
tags: [React, Next.js, Rails, API, Firebase, Typescript]
---

# 背景

最近、個人 PF で使う認証方式を検討している。最初は Auth0 を使おうと思っていたけど、実装で躓いて、代替案を探していたとき、Firebase Authentication が目に留まった。

電話認証以外は基本的に無料で使えるし、利用できる認証方法も非常に豊富で、特に Auth0 が対応していない匿名認証もできる点に惹かれた。

> 特徴的なのは匿名認証で、一時的にユニークな ID を付与したユーザーとして扱いますが、その後他の認証方式に昇格することができます。

参照：[ちょっとでもセキュリティに自信がないなら、 Firebase Authentication を検討しよう](https://mizchi.dev/202008172159-firebase-authentication)

<details> 
<summary>実装でハマった話</summary>

Firebase Authentication を実装したところ、Auth0 の時と同じ、実装の仕組みやコードの意図が理解できず、一時的ハマった。

ちょっと焦っていたところ、この動画チュートリアルに救われた。

[NextJS Firebase Auth Tutorial • How to Authenticate Users for Your App](https://www.youtube.com/watch?v=BQrE98bP6m4)

今ままで参考にした記事はちょっと古いので、公式の記述と一致しないコードもあり、この動画で使用している Firebase JavaScript SDK は最新の V9 バージョンで助かった。

また、JWT や認証の流れもわかりやすく説明されており、おかげでなんとなく仕組みがわかるようになった。

今回も、Auth0 の時も、コードが理解できなかったのは、そもそも JWT や認証の仕組みが全然わかっていないからだとわかった。(React Context API についての理解不足も原因の一つ..)

その後、検索キーワードを変えて、他の記事も見つけて、ようやく Rails 側のコードも理解できるようになった。

コードの全体流れが理解できたので、参照したソースコードを自分なりに少し書き換えてみた。自分の理解はメモとして整理したいと思う。

</details>

# Firebase Authentication の初期設定

Firebase のアカウント取得やプロジェクト作成などの初期設定はこちらの記事を参照できる。

参照：[Firebase の初期設定](https://reffect.co.jp/react/react-firebase-auth#Firebase)

# Next.js フロントエンド側の実装

# Rails API バックエンド側の実装

### 参考になったソース

- [Ruby で Firebase の id トークンを認証に使ってみる](https://qiita.com/otakky/items/b7582202f5cde8f2dd21)

こちらは説明は簡単だけど、ソースコードはすごく勉強になった。

[Rails + Next.js + Firebase Authentication で認証付きアプリを作成する](https://qiita.com/kmgk0215/items/a174c6eef0a940ad03ea#%E3%83%90%E3%83%83%E3%82%AF%E3%82%A8%E3%83%B3%E3%83%89rails%E3%81%AE%E5%AE%9F%E8%A3%85)

認証全体の流れはパッとわかるようになったのは、こちらの issue 文の説明のおかげだ。詳細説明記事とソースコードも大変参考になった。

- [Proper way to verify Firebase id tokens](https://github.com/jwt/ruby-jwt/issues/216)
- [How to validate Firebase ID token in Ruby](https://medium.com/@igorkhomenko/how-to-validate-firebase-id-token-in-ruby-23f4f54c89ab)

こちらは ボランティアたちが作った Ruby 用 Firebase-admin-sdk だけど、まだ alpha 版で、プロダクションでの使用はまだ推奨されていない。

> This gem is currently in alpha and not recommended for production use (yet).

ソースコード自体は参考になった。

- [firebase-admin-sdk-ruby](https://github.com/cheddar-me/firebase-admin-sdk-ruby)

こちらは Firebase 公式の python 版 admin-sdk ソースコード。
コード内のコメント説明も大変参考になった。

- [firebase-admin-python](https://github.com/firebase/firebase-admin-python/blob/b9e95e8248eb1473ca5a13bf64e8a33b79dc9db3/firebase_admin/_token_gen.py#L292)

参照

[Firebase: Verify ID Tokens](https://firebase.google.com/docs/auth/admin/verify-id-tokens#verify_id_tokens_using_a_third-party_jwt_library)

[react-firebase-hooks](https://github.com/csfrequency/react-firebase-hooks/tree/593f47965709d4abf23029c1160bb834c9f825bc/auth#useidtoken)

[How to Sign and Validate JSON Web Tokens – JWT Tutorial](https://www.freecodecamp.org/news/how-to-sign-and-validate-json-web-tokens/)
[ruby-jwt/lib/jwt.rb](https://github.com/jwt/ruby-jwt/blob/main/lib/jwt.rb)
[Google Certificate url](https://www.googleapis.com/robot/v1/metadata/x509/securetoken@system.gserviceaccount.com)
