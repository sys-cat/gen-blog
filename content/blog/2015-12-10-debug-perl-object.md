+++
date = "2015-12-10T18:00:00+09:00"
title = "Perlでデバッグしたくて苦労した話"
draft = false

+++

Perlのお勉強のお話。
<!--more-->
入社して1周間チョット経って絶賛タスクを消化しまくる日々を過ごしています。  
なんせPerl力があまりに無いのでコーディングして慣れろって話ですね。わかります。

そんな訳でタスクを消化しているのですがPerlという言語は面白いですね。  
PHPやRubyに慣れた人間としてはPHPにも似てるしRubyにも似てるところあるなぁという認識が強いです。  
対象のシステムは非常に長く運用されたシステムで規模も数百万単位のユーザを管理するレベルでして、かれこれ10年近く運用され続けたものです。  
そんなシステムに知識0、モジュール周りの知識も皆無な僕が中身を見ると意味不明なコードがずらっと並んでるなという感覚すら出てきます。

とりあえず `sub` はメソッド定義時に使う。そんなとこからのスタートだし仕方ないなぁとは思いつつ勉強の毎日。

---

さて、そんな大規模な開発で**ログを出力したいということは**まぁよくある話かなぁと思います。
PHPやRubyだとWAFに `Logger` クラスがあったりするのでそれ使えば良いのだけど
Perlだとそれっぽいクラスが見当たらず…  
プロジェクト内で `info`, `warn` なんかを自作されてる部分があったのでそれを使えたのだけど
配列な変数の中身を出力出来ず…どうすっかなと思っていろいろとあたってみたところ…

**Dumper** 使えば良いとの事。

```perl
use strict;
use warning;
use Data::Dumper;

.....
warn Dumper($hoge);
```
こんなモジュールがあるんだなぁって。。。