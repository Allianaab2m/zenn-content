---
title: "herdr + hunk で GitHub の PR Diff を見れるプラグインを作った"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["herdr", "hunk"]
published: true
---

## TL:DR;

タイトルの通りです．

プラグイン本体は「`hunk diff` した時に今いるブランチで PR が出ていたら base との diff を herdr の pane か tab で開くプラグインを作って」とお願いしたら Claude Opus 4.8 が20分ぐらいで作ってくれました．

https://github.com/Allianaab2m/herdr-hunk-gh-diff

---

[Herdr](https://herdr.dev) がアツいです．AI エージェントのセッション管理には今まで cmux を使っていましたが，完全に Herdr に乗り換えてしまいました．

それと同じぐらい，[hunk](https://hunk.dev) もアツいです．こちらは TUI で diff が見れるツールですが，skill を入れることで diff 上で AI エージェントとのインタラクションが出来るようになる TUI ツールです．

これらを組み合わせて使うプラグインは探せばいくつか出てきますが，自分が今いるブランチで出している PR の base との diff を `hunk` で見るプラグインはなかったので作りました．

https://github.com/Allianaab2m/herdr-hunk-gh-diff

作ったというのは名ばかりで，中身は Claude Opus 4.8 が20分ぐらいで作ったものです．シェルスクリプトに毛が生えた程度の処理しかしていませんが，自分で書くには面倒だったので AI 様々といったところです．

↓ だいたいこんな設定を書けば使えます

```toml
[[keys.command]]
key = "prefix+p"
type = "plugin_action"
command = "hunk.gh-diff.pr-tab"
```

ちなみにエラーや，PR がない等で Diff が表示できない場合は，`herdr notifications show` コマンドを通じてエラーを通知するようにしています．今後 Herdr でプラグインを作る人はこれらで通知するようにしてあげると良さそう．

