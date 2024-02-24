---
title: Chromeが第三者アプリからのリンクを開けない状況について
category: "Others"
tags: [Mac, Chrome]
---

## 問題背景

Mac でデフォルトブラウザを Google Chrome に設定している。  
Chrome 自体はいつも普通に使えるけど、最近は、Docker や Discord など第三者アプリ内のリンクをクリックしたら、chrome が反応しないことに気がついた。

デフォルトブラウザを Safari に変更したら、リンクは問題なく Safari から開けた。

##　原因

調べたら、Chrome が外部リンクを拒否するようになった原因は、新しいバージョンのアップデートが正常に完了していないことだそう。

Chrome は通常バックグランドで新しいバージョンを自動ダウンロードしておく。そしてブラウザが再起動する時、自動的にアップデートを完了する。

しかし、もしブラウザがいつも開けたままで、再起動にならない時は、自動アップデートの実行ができなくて、chrome がおかしくなり、外部リンクを拒否するようになる。

## 解決策

chrome の設定ページから最新のアップデートを完了させ、chrome を再起動したら、通常通りに動ける。

---

参照  
[Google Chrome won't open links anymore? Here the solution!](https://www.tutonaut.de/en/google-chrome-doesn%27t-open-any-more-links-here%27s-the-solution/)
