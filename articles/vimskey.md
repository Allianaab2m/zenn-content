---
title: "VimでMisskeyが見れるプラグインを作ってる話"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Vim", "Neovim", "Misskey", "TypeScript"]
published: false
---

:::
この記事はVim Advent Calendar 2022(2)の記事です。
https://qiita.com/advent-calendar/2022/vim
:::

アドベントカレンダー初参加です。よろしくお願いします。

## 前置き

突然ですが，皆さんはMisskeyというSNSをご存知でしょうか。
MisskeyはOSSで開発されている分散型SNSです。

Mastodonなどで採用されているActivityPubプロトコルを実装したインスタンス[^1]とは相互にやり取りができる一方，Discord, SlackライクなリアクションなどのMisskey独自の機能が実装されています。

[^1]: インスタンスとは，FedibirdやPawoo，Misskey.ioなどのサーバー単位のことを指します。分散型SNSでは，所属するサーバーごとに提供される機能や規約，サーバーの存在目的などが若干異なります。

(Placeholder)

昨今のTwitter情勢が不安定であることから移行している人が多く，かくいう私も移民の一人で，一ヶ月ほど前に引っ越してきたばかりです。

## 本題

さて，そんなMisskeyにはノート[^2]の送信やリノートなどが出来るWebAPI，タイムラインや通知をリアルタイムに受け取れるStreamAPIがあり，更にはTypeScriptで書かれた公式SDKも用意されています。

[^2]: MisskeyではTwitterのツイートに当たる投稿単位をノートと表現します。

となれば，VimでMisskeyが見れるプラグインが作れ，流れるタイムラインを作業のお供に出来るのではないか？ということで制作を始めました。

https://github.com/Allianaab2m/vimskey

機能は今のところタイムライン表示と通知，コマンドラインからのノートと，かなり限定されていますが，徐々に形になってきたと思います。

Denopsを採用しているので，Vim/Neovim両対応です。

少し躓いたのは，公式SDKにNode依存のコードが一部あり，そのままではDenoで使うことができませんでした。

しかし，npmサポートが安定版になったDeno v1.28がリリースされたので，npm互換モードで今のところ問題なく利用できています。

## まとめ

Vimのプラグインを作るにはVimScriptの知識がかなり求められると思っていましたが，実際にやってみるとそうでもありませんでした。

Denopsを利用することで極力VimScriptを書くことなく，型安全で補完が効く環境でプラグイン開発が進められるのは快適ですね。

今後も開発を進めて機能を増やしていこうと思っているので，是非使ってみてください。

また，この記事がきっかけでMisskeyにも興味を持っていただければ幸いです。

↓良かったらフォローお願いします

https://misskey.io/@Alliana_ab2m
