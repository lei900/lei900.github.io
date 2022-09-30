---
title: VSCodeでReact JSX内のhtmlタグを自動補完する
category: "React"
tags: [JavaScript, React]
---

VSCodeでReactのJSXコードを書いている時、HTMLタグの自動補完機能が効かなくなっているので、
設定方法を調べてみた。

### 方法1：VSCodeのEmmet設定で`Include Lanuguages`を追加する

VSCodeの設定(setting)を開き、`Include Lanuguages`で検索したら、
設定画面が出てくる。

`Add Item`でItem枠に`javascript`、Value枠に`javascriptreact`と追加すればOK。

![設定](https://i.imgur.com/yDZx72Y.png)


(よく見たら、erbとrubyが先に自動追加されたようだ。
だからrailsでerbファイルを使っている時、HTMLタグが自動補完できて、不便を感じなかったかな。)

### 方法2：直接`settings.json`ファイルで追加設定する

設定内容自体は方法1と同じになる。

VSCodeの設定画面の右上アイコンから、`settings.json`ファイルを開き、
下記の設定コードを追加する。

```json
"emmet.triggerExpansionOnTab": true,

"emmet.includeLanguages": {
  "javascript": "javascriptreact"
}
  ```

### 方法3：右下の言語タブで言語モードを変更する

VSCode右下の言語タブを開き、Language Mode(言語モード)を`JavaScript`から`JavaScript React`に変更する。

![設定](https://i.imgur.com/FMhmDEK.png)


---
参照  
[JSX or HTML autocompletion in Visual Studio Code](https://stackoverflow.com/questions/39320393/jsx-or-html-autocompletion-in-visual-studio-code)
