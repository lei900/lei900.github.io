---
title: '今日の反省: 本番反映前に絶対DBをバックアップしておくこと'
category: "other"
tags: [docker, PostgreSQL]
---

# やってしまったこと

今日は、エンジニアになってから最悪のミスを犯してしまった...

まず、上司の指示を誤解して、自分で勝手にクライアントの本番サーバーでバージョンアップ作業を行った。

当初の指示はバージョンアップ作業の手順をまとめることだけ。なぜか、私は本番反映もやると理解してしまった。

クライアント側では、まだステージング環境での確認がまだ終わっていない。そもそも本番反映はサーバーを一度シャットダウンする必要があって、絶対に事前にクライアント側に確認すべきだった。

そして、バージョンアップ作業する前に、現時点のデータベースのバックアップを怠ってしまった。

リストア用データベースを作っておいたけど、使っていたデータは今朝cronでバックアップした分のみで、現在の最新データを使っていなかった。

もしこの後、何か不具合が発生する場合、たとえデーターをリストアするとしても、データーの損失が発生する可能性がある。

あるいは、新規マイグレーションをmigration downして、元のmigration状態に戻すこと。2年間分のマイグレーションファイルを一個一個rollbackすることは、リスクも非常に高い...

結局、バージョンアップ後の不具合に気付き、バージョンを復元することになった。

幸い、cronから作業時点までにまだ最新の更新がなかったので、直接dumpデーター使って直接リストアできた。

# 今回の反省

- 上司の指示に対して、自分の理解に間違いがないか、必ず確認する。特に本番サーバーでの作業は、自分ひとりでやらないこと。本番反映は必ず二人以上で行うべきだった。

- バージョンアップやデプロイする前に、必ず現在データーをバックアップすること。