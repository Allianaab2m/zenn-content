---
title: "Vitestが`crypto is not defined`と言って落ちるときは"
emoji: "💥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vitest", "vscode", "typescript"]
published: true
---

# TL:DR;

Vitest拡張機能(vitest.explorer)の`"vitest.nodeExecutable"`の項目を確認しましょう．

## 何があったの

[最近入ったプロジェクト](https://github.com/pulsate-dev/pulsate)でVitestとVSCodeを使うことになりました．

ワイ「VSCode使うん久しぶりやな...せや！拡張機能でVitestのランナー入れたるで！ Vitest...っと」

https://marketplace.visualstudio.com/items?itemName=vitest.explorer

ワ「お！すぐ見つかったわ！ VSCodeはこれがあるからええな」

---

ワ「ほなテストするで～」

Vitest「`crypto` is not defined」

![](/images/why-vitest-fail/image01.png)

ワ「テスト落ちてまんがな！こらきっとテストケースが」

Github Actions「いや 動いとるで」

ワ「なんでや...てか落ちとるん最初のケースだけやんけ」

ワ「`pnpm run test`はどないや」

ターミナル「全部Greenや」

ワ「てことは，Vitest拡張機能，お前か...？」

Vitest「`"vitest.nodeExecutable"`になんも書いてなかったらVSCode内蔵のNode.js使うで」

![](/images/why-vitest-fail/image02.png)

ワ「VSCode内蔵のNode.jsってなんや」

VSCode「これや(v18.18.2)」

![](/images/why-vitest-fail/image03.png)

ワ「これやんけ」

Node.js v18ではWeb crypto APIはまだunstableなので，うまく動かなかったということです．[^1] おしまい．

## 教訓

- 早いところ`pnpm run test`して原因の特定をすればよかった
  - というかNodeのバージョン違いという発想が頭の中に無かった...
  - ~~VSCodeの内蔵Node.jsってなんやねん~~
- 拡張機能の設定項目にはある程度目を通そう

GitHub ActionsでCI環境が整っていたことが原因特定の一助になりました．ありがたい限りです．

## 宣伝

Pulsateでは開発メンバーを募集しています．興味がございましたら，ぜひご参加ください．

[メンバー募集記事](https://blog.m1sk9.dev/posts/2024/about-pulsate-2024-03)

https://twitter.com/m1s2r8/status/1771822948564807935?s=20

[^1]: 依然1回目だけ律儀に落ちてた原因は不明のままです．解決したのでこれ以上調査する気も起きませんでした．