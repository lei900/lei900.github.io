---
title: 'Clean Agile 感想: アジャイルは速く進むことではない'
category: "Career"
tags: [Agile, "Reading Notes", "Software Development", "Robert C. Martin"]
---

Twitterで見かけて、[Clean Agile 基本に立ち戻れ](https://www.amazon.co.jp/dp/4048930745)を読んでみた。

この本では、アジャイル開発の具体的な実践方法より、アジャイルの歴史や考え方を説明している。特に近年アジャイルがよく誤解されている状況について、作者はアジャイルの第一人者として、自分の考えから**アジャイルの本質**を語っている。

私は、いままでアジャイルは、チーム開発を迅速に推進するための方法論だと理解していた。

この本を読んだら、全くそうではないことがわかった。

どちらというと、アジャイルは、納期を死守させるためではなく、遅れそうなプロジェクトを早期に発見し、適切な対応をするための方法論に近いほう。


# なぜいつも納期に間に合わないのか

> 考えている暇はない。納期は迫っているし、ステークホルダーが怒っている。プレッシャーが高まる。残業は急増する。退職者は続出する。誰もやる気を失っていた。こんなプロジェクトなど二度とやるかと誓った。

この本の第一章を読むと、まさに本に書かれている状況通り、私は痛いほどこの現実を経験している。

当然、スケジュール見積自体の問題やプロジェクト管理面、私自身のスキル不足など、様々な要因があると思うが、本に書かれているこの失敗光景にすごい共感を持っている。

> スケジュールが遅れて、人員追加して、また遅れて、さらに追加して、結局これだけリソース投下しているのに、なぜうまくいかないのか？

私の職場では、この本に書かれているように、みんなこの疑問を持っている。

そして、この本では、以下で述べている。

> 人数が２倍になれば、２倍の速度で進むことを誰もが知っている。

> だが、実際はその逆になる。「遅れているソフトウェアプロジェクトへの要員追加は、プロジェクトをさらに遅らせる」と、ブルックスの法則が指摘している。(「人月の神話」)

なぜなら、２倍の人数が増えると、それ以上のコミュニケーションコストが増えるからだ。

> 新しいメンバーは未熟であり、既存のメンバーの活力を奪うため、数週間は生産性が落ちるはず。

実際でも、毎回新しく入ってきたインターンに業務ロジックやコードの説明などで、かなりの時間を取られた。一日中、サポートに追われ、自分の業務が進まない状況が良くあった。

# 鉄十字のトレードオフ

> 「品質」、「速度」、「費用」、「完成」のうち、好きな三つを選べる。四つは選べない。例えば、高品質で、高速で、安価なプロジェクトは、完成することはない。安価で、高速で、完成するプロジェクトは、高品質ではない。

この鉄十字のトレードオフは、プロジェクトの進行において、常に意識しておきながら、バランスを取ることが重要だと。

この法則を理解したうで、アジャイルはどのように支援してくれるのか。それが、開発の速度データを提供すること。

チームのベロシティと残りのストーリーポイントを使って、プロジェクトの進行状況を予測することができる。

ここで注意すべきなのは、開発中に新しい要求や課題が発見された場合は、残作業を再見積もりすることが重要だと。

# 開発を確実に見積もることはできない

> 我々プログラマーは、作業にかかる時間を把握できない。それは、我々が無能だとか怠惰だとか、そういうことではない。実際に作業に取り掛かり、それが終わるまで、どれだけ複雑になるかを知る方法がないからだ。

この結論は私にとってとても新鮮だった。見積もれないのは、単純に経験不足や事前の確認不足だを思った。

でもよく考えると、確かにそうかも。完璧に見積もれるは、すべてのコードを脳内でシミュレーションできること。それは不可能ではないか。

この現実を受け入れて、イテレーションを繰り返し、見積もりを修正しながら、誤差範囲を狭めることが重要だと。

# アジャイルは速く進むことではない


> このように、イテレーションを進めていくと、元の終了日には希望がなかったことが判明するまで、誤差範囲が狭まるはずだ。

アジャイルは、希望を破壊する。

> 希望を持つとチームがマネージャーに進捗を誤解させてしまう。希望はプロジェクトをマネジメントするのに最悪の方法だ。アジャイルは早い段階から希望を殺し、継続的に冷たくて厳しくい現実を提供する。

私のような、アジャイルは速く進むことだと思っている人もいるだろう。

> だが、そうではない。これまでそうだったこともない。

> アジャイルとは、どれだけうまくいっていないかをできるだけ早く知ることだ。そうすれば、状況をマネジメントできるからだ。

マネージャーのやることは、データを収集し、そのデータをもとに最善の意思決定をすること。

アジャイルは、**プロジェクトをマネジメントするためのデータを生成する。**

このデータを使って、プロジェクトの成果を可能な限り最大化する。


# 鉄十字のマネジメント

それでは、プロジェクトがうまくいってないことがわかったら、どうすればいいのか。

前述した「品質」、「速度」、「費用」、「完成」の鉄十字をどう選択取捨するのか。

結論というと、一番現実的なのは、「完成」、つまりスコープを変更すること。

なぜなら、

- ビジネス上、スケジュール、つまり納期の調整が高い確率で無理。
- 人員追加は、コストもかかるし、生産スピードが改善できるまで、十分な時間が必要。
- 品質を落とすと、さらに遅くなってしまう。**汚いものはすべて遅いのだ**

唯一調整可能なのは、機能のスコープだ。

本当に、この期限までに、すべての機能が必要なのか。一部後回しできないのか。

この調整が最後の手段だ。

# その他の感想

以上はあくまでこの本の第一章の内容だけ。でも十分な啓発を受けたと思う。

アジャイルの実践方法について、この作者が書いた別の本を読む予定だが、この本では、ほかに新鮮と思ったところがまだたくさんある。

## ソフトウエアは安価に変更できるもの

> 「ソフト」とは、「変更しやすい」である。よって、「ソフトウェア」とは「変更しやすい製品」である。

元々ソフトウエアが発明されたのは、マシンの動作をすばやく簡単に変更する方法が求められたから。動作の変更が難しいものこそ、「ハードウエア」と呼ばれる。

確かに私でもそうだし、要件の変更に不満を持つ人は多いかも。でも、この本を読んで、そもそもソフトウエアは変更しやすいものだということを再認識した。

もし「そんな変更したらアーキテクチャがダメになる」ことだったら、それはアーキテクチャがダメなのでは。

> 我々開発者は、変更を喜ぶべきだ。そのために我々は雇われている。我々の仕事は、変更を受け入れてエンジニアリングする能力と、そうした変更を比較的に安価にできるかどうかにかかっている。

この言葉は、私の心に響いた。エンジニアの仕事は、変更を受け入れてエンジニアリングする能力だと。

自分の仕事について、いま一度考え直したい。

## できない時は、「ノー」と言うべき

> 自分が雇われている理由は、コードを書く能力よりも「ノー」と言える能力のためだと自覚すべきだ。

自分にしか、できるかどうかの判断をできないのだ。どれだけ納期のプレッシャーを感じていても、どれだけ多くのマネージャーたちが結果を求めていても、答えが本当に「ノー」であるなら、それを言うべきだと。

いま振り返ってみたら、プロジェクト最初の段階で、このスケジュールでは無理だと思うことをはっきり伝えるべきだった。

少なくとも、二回目の延期となったタイミングでも、もう一度「ノー」と言うべきだった。

自分がスケジュール通りに行ってないことで怒られることを怯えて、実際テストなしでも、タスク完了したと報告したこともあった。

そのあと、バグが多発し、結局、やり直す時間が「想定外時間」としてかかってしまった。

連続二か月で土日残業深夜残業までをして、コードを書きたくないぐらい疲れた。

それで本当に良かったのか。

## 残業はダメ、持続可能なペースを

作者はアジャイルの第一人者として、アジャイルの実践単位を「スプリント」と呼ばれることに反対している。

なぜなら、スプリントという言葉は、短距離走のように、短期間で全力を出し切ることを意味する。

だが、ソフトウエア開発は、短距離走の連続ではなく、**マラソン**である。

長期にわたって、持続可能なペースで走り続けることが重要だと。

> マネージャーから速く走れと言われることもあるだろうが、絶対に従わないでほしい。

それは本当。最後まで持ちこたえられるように、自分のリソースをうまく節約するのは自分の責任だ。

残業しても、プロジェクトが早く終わるわけではない。むしろ、疲れて、集中力が落ちて、結局時間がかかることが多い。

また、残業をすることを誇りに思うひともいるかもしれないが、それは間違いだと。

> あなたが計画できない人であり、合意すべきではない納期に合意した人であり、守れない約束をしたひとである、専門家ではなく、従順な労働者であることを示すだけだ。

残業によって、スケジュールが短縮されるよりも、**残業にかかるコストのほうが大きくなりやすいこと**を意識するべきだ。


## 顧客と開発者の権利章典

> 顧客には、考えを途中で変えて機能を差し替えたり、プライオリティを変更したりする権利がある。その際に特別に多額の出費は必要ない。

こちら、前述した「ソフトウエアは安価に変更できるもの」の考え方と一致するので、改めて意識しておきたい。

> 開発者には、常に質の高い仕事をする権利がある。

これは、エンジニアとして、常に意識しておきたい。

> 開発者には、自ら見積りを行い、またそれを更新する権利がある。

自分のタスクを見積もれる人は、自分以外にはいないこと。

すでにタスクを見積もったあとでも、新たな要素が明らかになった場合には、いつでも見積りを変えられる。

見積りはあくまで推測だ。見積りはけっして約束ではない。

上記にすごく感謝な気持ちだけど、これを実践できるだろう。


## テスト駆動開発と複式簿記

一つ面白い例えがあった。テスト駆動開発は、複式簿記に似ていると。

複式簿記とは、帳簿に記入する取引は、必ず２回記入すること。一回は、貸方、もう一回は借方。

開発に対しても、「必要となる振る舞いは、必ず２回入力する。」ということ。

> 一回目はテスト、二回目はそのテストをパスさせるコードだ。

> 両者を一緒に実行すると、結果はゼロになる。つまり、失敗したテストがゼロということ。

テスト駆動開発について、面白い視点だと思った。

一旦、この本の感想は以上。

次は、アジャイル実践方法について、読んでみたいと思う。