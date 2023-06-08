---
title: "何もしてないのにNerdFontの表示が壊れたので直した"
emoji: "💦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ "nerdfont", "neovim" ]
published: true
---

何もしてないのにある日突然TypeScriptロゴが表示されなくなりました．

![TypeScript logo not displayed](/images/nerdfontv3-tofu/image01.png)

TypeScriptロゴ以外にも，かなり表示されていないNerdFontロゴがあります．
しかし，ステータスラインの区切り文字やファイルロゴは正しく表示されています．なぜ急にこんなことが起こったのでしょうか．

## 原因

Nerd Fontがv3になり，Material Design Iconsのコードポイントに対して破壊的変更が行われたことが原因でした．

https://github.com/ryanoasis/nerd-fonts/releases/tag/v3.0.0

というのも，v3以前は漢字などが割り当てられているコードポイントに誤ってMaterial Design Iconsが割り当てられてしまっていた[^1]ようです．

そこで，v3からは漢字と被らないコードポイントにMaterial Design Iconsを移動させるということになりました．

## プラグインを直す

私はNeovimでアイコンを表示するプラグインとして[nvim-material-icon](https://github.com/DaikyXendo/nvim-material-icon)を利用しています．

このプラグインをフォークして手作業で修正していきます．
以前のコードポイントから名前を割り出し，新しいコードポイントに置き換えていく作業です．

意外にも数はそこまで多くなかったので，すぐに終わりました．

https://github.com/Allianaab2m/nvim-material-icon-v3

このプラグインを導入すると，nvim-material-iconが以前の正しい表示に戻ります．一時的な対処としてご利用ください．

[^1]: 議論が行われていたissue:
https://github.com/ryanoasis/nerd-fonts/issues/365
起票から修正までなんと3年以上かかっています．関係者の皆さんには足を向けて寝られません．
