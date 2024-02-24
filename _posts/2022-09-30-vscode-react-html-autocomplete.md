---
title: VSCodeでReact JSX内のhtmlタグを自動補完する
category: "Coding"
tags: [JavaScript, React]
---

VSCode で React の JSX コードを書いている時、HTML タグの自動補完機能が効かなくなっているので、
設定方法を調べてみた。

### 方法 1：VSCode の Emmet 設定で`Include Lanuguages`を追加する

VSCode の設定(setting)を開き、`Include Lanuguages`で検索したら、
設定画面が出てくる。

`Add Item`で Item 枠に`javascript`、Value 枠に`javascriptreact`と追加すれば OK。

![設定](https://i.imgur.com/yDZx72Y.png)

(よく見たら、erb と ruby が先に自動追加されたようだ。
だから rails で erb ファイルを使っている時、HTML タグが自動補完できて、不便を感じなかったかな。)

### 方法 2：直接`settings.json`ファイルで追加設定する

設定内容自体は方法 1 と同じになる。

VSCode の設定画面の右上アイコンから、`settings.json`ファイルを開き、
下記の設定コードを追加する。

```json
"emmet.triggerExpansionOnTab": true,

"emmet.includeLanguages": {
  "javascript": "javascriptreact"
}
```

### 方法 3：右下の言語タブで言語モードを変更する

VSCode 右下の言語タブを開き、Language Mode(言語モード)を`JavaScript`から`JavaScript React`に変更する。

![設定](https://i.imgur.com/FMhmDEK.png)

---

参照  
[JSX or HTML autocompletion in Visual Studio Code](https://stackoverflow.com/questions/39320393/jsx-or-html-autocompletion-in-visual-studio-code)
