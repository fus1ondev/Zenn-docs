---
title: "LICENSEファイルをコマンド1つで生成できるツール「genelic」を作った"
emoji: "📄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ライセンス","cli","typescript"]
published: true
---

## はじめに

GitHubにレポジトリを作成した時に最初にやることはライセンスファイル（`LICENSE`、`LICENSE.md`）の作成だと思います。

![](https://storage.googleapis.com/zenn-user-upload/8a42ecf901bc-20221010.png)

しかし毎回GitHubのページ上でボタンをポチポチ押して生成するのは手間がかかるのが気になっていました（僕が知らないだけでもっと楽な方法があるのかもしれませんが…）。

![](https://storage.googleapis.com/zenn-user-upload/c76f2a81dce4-20221010.png)

ググってみたら生成ツールがいくつか見つかりましたが、Homebrewからのインストールに対応していなかったり、レポジトリ内に直接ライセンスファイルが置かれているものが多く、どれも微妙な感じでした。

そこでライセンスファイルを簡単に生成できるツール「genelic」を作成しました。

https://github.com/Fus1onDev/genelic

npmかHomebrewからインストールできます。

名前の由来は「**gene**rate」+「**lic**ense」という安直な理由ですが、「generic（一般的な・包括的な）」という英単語ともかかっていて良い感じになったと思います(?)

## 使い方

![](https://storage.googleapis.com/zenn-user-upload/2f6bdf192d10-20221010.png)

`genelic MIT`のようにライセンスのIDを引数として渡すと、簡単に生成できます。

ちなみにリッチなコンソール出力には[`signale`](https://github.com/klaussinani/signale)を使っています。

![](https://storage.googleapis.com/zenn-user-upload/1c487ba7a773-20221010.png)

`select`コマンドを使うと、辞書順に並んだ全てのライセンスの中から選択できます。

ただ数が多すぎて選ぶのが大変なので、検索コマンドも実装したいです。

このピッカーの表示は[`prompts`](https://github.com/terkelg/prompts)を使用しています。

## 使用した技術

- TypeScript
- Webpack
- vercel/pkg
- yargs
- prompts
- signale

APIから取得したデータをタイプセーフに扱うためにTypeScriptを使用しました。それをWebpackでバンドルし、npmへ公開しています。

また、vercel/pkgでnode.jsランタイムを含んだバイナリを生成し、Homebrewからインストールできるようにしています。
しかしバンドルサイズが50MBを超えてしまっているので、将来的にはGoで書き直して軽量化したいと考えています。

### ライセンスのデータベース

Linux Foundation傘下のSPDXというプロジェクトがライセンスなどのフォーマットを規格化するという活動をしていて、SPDXのウェブサイトにライセンス一覧が公開されています。

![](https://storage.googleapis.com/zenn-user-upload/c16d2f5420ca-20221010.png)

一覧やライセンスごとのデータが取得できるJSON形式のAPIもあり、genelicではこのAPIを毎回叩くことによってデータを表示させています。

https://spdx.org/licenses/

GitHubレポジトリはこちらです。

https://github.com/spdx/license-list-data

## おわりに

気に入った方はぜひ🌟をお願いします！

https://github.com/Fus1onDev/genelic