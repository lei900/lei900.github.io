---
title:  No newline at end of fileエラーについて
category: "Ruby on Rails"
tags: [Ruby, Ruby on Rails, unix]
---

毎回Lintチェックで怒られたエラー..

なんで最後に新しい行が必要なの？と調べたら、理解が全然間違った。。


ここの`newline`は新しい行の'new line'ではなく、`newline`記号のことだ。
いわゆる行末記号(改行記号とも言われるらしい)

`No newline at end of file`は末尾に新しい行がないではなく、末尾に行末記号がないという理解の方がより正しい。

`newline`はline endingやend of line(EOL)とも言われるので、このエラーは`EOL記号がない`と理解しても良い。

---
参照:

[Why should text files end with a newline?](https://stackoverflow.com/questions/729692/why-should-text-files-end-with-a-newline)
