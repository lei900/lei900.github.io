---
title:  provide と content_for、Streaming(viewとlayoutの評価順)について
category: "Coding"
tags: [Ruby, Ruby on Rails]
---

動的タイトルを出力するため、`content_for`を使うのがよく見かけるけど、Rails Tutorialの中に、`provide`メソッドを推奨しているそう。
>Railsでの開発経験者であれば、この時点でcontent_forを検討すると思いますが、残念ながらAsset Pipelineと併用すると正常に動作しないことがあります。provideメソッドはcontent_forの代替です。

Rails APIでの説明では、
>`provide(name, content = nil, &block)`
The same as `content_for` but when used with `streaming` flushes straight back to the layout. In other words, if you want to concatenate several times to the same buffer when rendering a given template, you should use `content_for`, if not, use `provide` to tell the layout to stop looking for more contents.

また`Streaming`については、
>By default, Rails renders views by first rendering the template and then the layout. The response is sent to the client after the whole template is rendered, all queries are made, and the layout is processed.

>`Streaming` inverts the rendering flow by rendering the layout first and streaming each part of the layout as they are processed. This allows the header of the HTML (which is usually in the layout) to be streamed back to client very quickly, allowing JavaScripts and stylesheets to be loaded earlier than usual.

なるほど。Railsでは、デフォルトでveiwのtemplateを先に評価してから、layoutを評価する。

そして全部の内容をユーザーにレンダリングする。

だた、もしDBクエリが多いアクションを含めた場合、templateを評価するのは時間がかかるため、先にlayoutをレンダリングして、ページタイトルなどの基本内容をユーザーに表示させる方法は`streaming.`

`content_for`は、同じtemplateに複数併用が可能なので、Railsはtemplateを上から下まで評価して、複数のcontent_forを評価して、内容を連結する

```erb
<%= content_for :title, "Hello" %>
    <h1>BOARD APP</h1>
  <%= content_for :title, "World" %>
```
{: file="top.html.erb" }

`content_for`の内容は連結されるので、表示されるタイトルはHelloWorld | OARD APPになる
つまり`<title>HelloWorld | RUNTEQ BOARD APP</title>`

一方で、`provide`の場合は、複数使うのが不可で、最初の`provide`のみが評価される
```
<% provide :title, "Hello" %>
    <h1>RUNTEQ BOARD APP</h1>
  <% provide :title, "World" %>
```
{: file="top.html.erb" }

上記では、最初の`provide`の内容のみが評価される
つまり`<title>Hello | BOARD APP</title>`

### 結論
`provide` と `content_for` は基本的には同じだけど、

もしDB操作が多くて、viewのtemplateが重たいなどの理由で、`streaming`を使いたい場合、`provide`を使うのが良い。

そうすると、template最初のprovide部分のみを評価して、残りの部分を置いといて、layoutのところに戻る。

### 残り疑問

「content_forはAsset Pipelineと併用すると正常に動作しないことがある」と言うのはどういう場合？

---
参照：
[Rails tutorial](https://railstutorial.jp/chapters/static_pages?version=5.1#cha-3_footnote-ref-14)
[Rails API](https://api.rubyonrails.org/v6.0.2.2/classes/ActionView/Helpers/CaptureHelper.html#method-i-provide)
[Streamingとは](https://api.rubyonrails.org/classes/ActionController/Streaming.html)
[AssetPipelineとは](https://railsguides.jp/asset_pipeline.html)
[railsのアセットパイプラインについて解説する](https://qiita.com/uma002/items/9a94ebc93c5f937502cd)
[Ruby on Rails: provide vs content_for](https://stackoverflow.com/questions/27814500/ruby-on-rails-provide-vs-content-for)
[Railsのviewとlayoutの評価順についてコードを読んで納得した](https://blog.freedom-man.com/rails-view-layout)