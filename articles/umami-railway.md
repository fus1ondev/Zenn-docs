---
title: "オープンソースのアナリティクスサービス「umami」をRailwayで動かしてみる"
emoji: "👓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["umami","railway","seo"]
published: true
---

## はじめに

Googleアナリティクスなどの代替としてオープンソースで開発されている「umami」というサービスがあることを知りました。

Cookieを使用しないので、EUのGDPRで定められている「このサイトはCookieを使用します」のバナーを表示する必要がないそうです。

https://umami.is/

将来的にはクラウドプラン（？）も用意されるみたいですが、現状はセルフホストが前提となっています。

セルフホストの場合は、Node.jsの実行環境と、MySQLまたはPostgreSQLが必要です。

https://umami.is/docs/getting-started

ドキュメントには、NetlifyやVercel、DigitalOcean、Supabaseなど色々なホスティングサービスやデータベースへのデプロイ方法が書いてありますが、今回は「Railway」を使って無料で構築してみます。

こちらも~~最近何かと問題を起こしている~~Herokuの代替サービスとして注目を集めているみたいです。

https://railway.app/

Railwayへのデプロイの解説ページにはワンクリックで自動セットアップする方法と、自分でフォークする方法の2通り載っていますが、今後のアップデートに追従するために後者を試します。

https://umami.is/docs/running-on-railway

## Gitレポジトリをフォークする

以下のレポジトリを右上のボタンからフォークしてください。

https://github.com/umami-software/umami

## Railwayへデプロイする

Railwayのアカウントを作成します。GitHubのアカウントに紐づけるのが楽だと思います。

アカウントができたら、ダッシュボード画面の右上の「New Project」を押します。

![](https://storage.googleapis.com/zenn-user-upload/fe191acc5cd3-20220828.png)

このような画面が出てくるので、「Deploy from GitHub repo」を選びます。

（ランチャーアプリみたいにキーボードフレンドリー（？）でいいですね）

![](https://storage.googleapis.com/zenn-user-upload/d9b7ed656791-20220828.png)

先ほどフォークしたレポジトリを選択したら、「Deploy Now」を押してデプロイします。

![](https://storage.googleapis.com/zenn-user-upload/203b3fb0df69-20220828.png)

## エラーを直す

ドキュメント通りだとここで正常にビルドされるはずなのですが、自分の環境ではこのようなエラーが発生しました。

```
#16 4.864 $ node scripts/copy-db-files.js
 
#16 5.150 /app/scripts/copy-db-files.js:21
#16 5.150   throw new Error('Missing or invalid database');
#16 5.150   ^
#16 5.150
#16 5.150 Error: Missing or invalid database
#16 5.150     at Object.<anonymous> (/app/scripts/copy-db-files.js:21:9)
#16 5.150     at Module._compile (node:internal/modules/cjs/loader:1126:14)
#16 5.150     at Object.Module._extensions..js (node:internal/modules/cjs/loader:1180:10)
#16 5.150     at Module.load (node:internal/modules/cjs/loader:1004:32)
#16 5.150     at Function.Module._load (node:internal/modules/cjs/loader:839:12)
#16 5.150     at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
#16 5.150     at node:internal/main/run_main_module:17:47
#16 5.168 error Command failed with exit code 1.
#16 5.168 info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
#16 5.180 ERROR: "copy-db-files" exited with 1.
#16 5.198 error Command failed with exit code 1.
#16 5.198 info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
#16 5.210 ERROR: "build-db" exited with 1.
 
#16 5.229 error Command failed with exit code 1.
#16 5.229 info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
#16 ERROR: executor failed running [/bin/sh -c yarn build]: exit code: 1
 
-----
> [builder 5/5] RUN yarn build:
-----
executor failed running [/bin/sh -c yarn build]: exit code: 1
```

issuesを見てみると、同じエラーを報告している人がいました。
`DATABASE_TYPE`環境変数を追加することで直ったみたいです。

https://github.com/umami-software/umami/issues/1437

Variablesタブから、`DATABASE_TYPE`環境変数に`postgresql`とセットします。

![](https://storage.googleapis.com/zenn-user-upload/f67652a26c69-20220828.png)

Vercelとは違って環境変数を変更すると自動で再デプロイされるみたいです。
今度はビルドに成功しました。

![](https://storage.googleapis.com/zenn-user-upload/a70a48e751a6-20220828.png)

## データベース（PostgreSQL）を追加する

Railwayでは、プラグインという形で簡単にPostgreSQLを設定できます。

まず、`Cmd + K（Ctrl + K）`を押してランチャー画面を開き、`database`と入力すると出てくる「Database」をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/47d713d89875-20220828.png)

データベースの種類が選べるので、「Add PostgreSQL」を選択。

![](https://storage.googleapis.com/zenn-user-upload/9308f880db15-20220828.png)

## 環境変数の追加

PostgreSQLがセットアップされたら、先ほどのレポジトリの設定画面に戻り、「Variables」タブから環境変数を追加していきます。

![](https://storage.googleapis.com/zenn-user-upload/72a72f5dbc06-20220828.png)

まず、`HASH_SALT`環境変数にランダムな文字列をセットします。
セキリュティ的に、ID生成サイトなどで生成したものを使うといいと思います。

![](https://storage.googleapis.com/zenn-user-upload/d6ceaf6ef7cb-20220828.png)

次に、`PORT`環境変数に`3000`とセットします。

## データベースのテーブルを作成する

バックエンドの理解が浅いので合っているか分かりませんが、テーブルとはデータを格納する表のようなものです。

まず、最初に作成したGitレポジトリをローカルにクローンします。

次に、HomeBrewやnpmからRailway CLIをインストールします。
詳しくは以下のページをご参照ください。

https://docs.railway.app/develop/cli

CLIをインストールしたら`railway login`を実行してログインします。
ブラウザが開いて自動で認証されるはずです。

そして先ほどクローンしたレポジトリのディレクトリに移動し、`railway link`を実行してGitレポジトリとRailwayプロジェクトを紐づけます。

最後に、以下のコマンドを実行しテーブルを作成します。
`<PGHOST>`、`<PGUSER>`、`<PGDATABASE>`の部分は実際の値に置き換えてください。
環境変数の値は`railway variables`で確認できます。

```shell
railway run psql -h <PGHOST> -U <PGUSER> -d <PGDATABASE> -f sql/schema.postgresql.sql
```

実行すると「already exists」と色々エラーが出ましたが、特に問題はありませんでした（もしかしたらこのコマンドを実行する必要はないのかも？）

:::message
上記のコマンドが実行できない場合は、`psql`コマンドがインストールされていない可能性があります。

`postgresql`をインストールしてください。

```shell:Macの場合
brew install postgresql
```
:::

## ドメインを設定する

「Settings」タブの「Domains」という項目にある「Generate Domain」ボタンを押します。

独自ドメインを持っている場合は代わりにそれを使うことも可能です。

## パスワードの設定

生成されたドメインにアクセスすると、このようなログインページが出てきます。

![](https://storage.googleapis.com/zenn-user-upload/94d7813cb227-20220828.png)

デフォルトでは以下のように設定されているので、これを入力してログインします。

| Username | Password |
| - | - |
| `admin` | `umami` |

## 言語を日本語に設定

ログインできたら、右上の地球儀のアイコンを押して表示言語を日本語に変更できます。

![](https://storage.googleapis.com/zenn-user-upload/246aae62b572-20220828.png)

## パスワードの設定

初期パスワードのままだと第三者に乗っ取られる可能性があり危険なので、パスワードの変更を行います。

右上のアイコンをクリックし、「プロファイル」→「パスワードの変更」を押して新しいパスワードを入力してください。

![](https://storage.googleapis.com/zenn-user-upload/2259d7969241-20220828.png)

## ウェブサイトの追加

このままだとグラフには何も表示されないので、ウェブサイトを登録する必要があります。

設定画面から「Webサイトの追加」をクリックしてサイト名とURLを登録します。

![](https://storage.googleapis.com/zenn-user-upload/65c4febfb89a-20220828.png)

次に、リストの右側の4つ並んだアイコンの一番左を押し、出てきたHTMLタグをコピーして自分のサイトの`<head>`などに追加してください。

![](https://storage.googleapis.com/zenn-user-upload/edd1afa3010f-20220828.png)

## おわりに

以上で完了です。

試しに自分のサイトにアクセスしてみると、無事グラフが表示されました。

閲覧者のOS、デバイス、ブラウザ、国などが取得でき、現在の閲覧者のリアルタイム表示もできるのでGoogleアナリティクスなどと比べても遜色ないと思います。

導入は少し大変でしたが、UIもキレイで使いやすいのでオススメです。

![](https://storage.googleapis.com/zenn-user-upload/4e570dc25c48-20220828.png)