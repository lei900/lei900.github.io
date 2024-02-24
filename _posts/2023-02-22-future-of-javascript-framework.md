---
title: JavaScriptフレームワークの過去、現在と未来
category: "Coding"
tags: [JavaScript, React, Next.js]
---

フロントエンドの流行り廃りが激しいとよく言われるけど、

なぜそんなに激しいのか、毎回の進化はどのような問題を解決しているのか、

この動画はその進化の歴史をわかりやすく説明した。

[The past, current state & future of JavaScript frameworks](https://www.youtube.com/watch?v=5EsLj3JOdE0)

(ちなみに、この講師の udemy コースはとてもわかりやすく、React や node.js など全部おすすめ！

## JavaScript フレームワークの進化史

この動画によると、JavaScript フレームワークの進化歴史を 7 つの段階に分けられる。

### Phase 1: 素の JavaScript 時代

90 年代から 2000 年代前半までは、レガシーな JavaScript コードを書いていた暗黒時代...

### Phase 2: jQuery & Ajax 時代

2006 年、JavaScript ライブラリ jQuery が誕生した。JS コードを書くのはより簡単になり、異なるブラウザの非互換性の問題も解決された。

特に Ajax のおかげで、データの取得が裏で行われ、毎回新しい HTML ファイルを取得する必要がないため、UX が大幅に改善された。

### Phase 3: JavaScript フレームワーク登場

2010 年から、Ember.js や Backbone.js、Angular JS など初期の JS フレームワークが続々と誕生した。

state 管理の概念が発明され、複雑な UI の実現がさらに簡単になった。

### Phase 4: React、Angular、Vue.js 2 御三家の登場

2013 年、React 誕生

2016 年、Angular、Vue.js 2 登場

この三つのフレームワークは今でも JS フレームワークの御三家と呼ばれている。

(最近のトレンドから見ると、React 一強になりつつある？)

### Phase 5: SPA や Client side rendering の流行

御三家以外にも、たくさんの JS フレームワークが誕生している。

この段階の主流思想は Client side rendering、つまり UIUX の実現はすべて JavaScript によってブラウザ側でコントロールする。

Single page application が流行ってきた。

### Phase 6: SPA よりは静的サイト生成（SSG）

SPA では、大量な JS コードを事前ダウンロードする必要があり、画面の読み込み速度に影響するほか、頻繁なデータ取得もサーバーに負担をかける。

(SEO が対応できない問題も...)

それらの問題を解決するため、全てをクライアントサイドで処理するより、一部のロジックをバックエンドに戻すという声が大きくなった。

Next.js を代表として、Client side rendering よりは、静的サイト生成（Static site generation) を推奨するフレームワークが流行ってきた。

### Phase 7: Web の未来: Server Side Rendering への復帰

直近では、React や Next.js をはじめ、画面表示のパフォーマンスを最大化するため、ブラウザ側の JS コードを極力削減する方向に進んでいる。

サーバー側での静的サイト生成よりも進んだのは、(古き良きの)サーバーサイドレンダリング...

昨年 3 月、React 18 がリリースされ、サーバーサイドレンダリングのサポートが宣言された。

その後の 10 月、Next.js 13 がリリースされ、(まだ beta 版の機能だけど)、デフォルトでのサーバーサイドレンダリングが推奨された。

(まだ SPA が流行りだと思っていたけど、Next.js 13 がリリースされた時はかなり衝撃を受けた。これって原点に戻ったのでは...?

先日、Node.js の後継と見られる Deno の開発チームより、下記の記事が発表された。

[The Future (and the Past) of the Web is Server Side Rendering](https://deno.com/blog/the-future-and-past-is-server-side-rendering)

デバイスの多様化や異なる通信速度に対応するため、Web の未来は(過去も）サーバーサイドレンダリングと宣言されている。

つまり、SPA はもう過去の流行りか....
