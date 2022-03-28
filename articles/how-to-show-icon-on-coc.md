---
title: "Neovim(Coc.nvim)の補完ウィンドウにアイコンを表示する"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Neovim"]
published: true
---

日本語の情報が無かったので，メモ書きレベルですが記事にします。もっといい方法を知っている人がいれば教えてください。

Coc.nvimを使用するとLSP経由で補完を表示してくるのですが，デフォルトの状態では「m」とか「k」と表示されるだけでは，どことなく味気なさを感じていました。
![デフォルトの補完表示](/images/coc_icon/default.png)

## 発端(?)
https://twitter.com/mattn_jp/status/1499208542481686530?s=20&t=gsd__JTxTCu3mhTiTUMLcA

**なんということでしょう。** Emacs上の補完ウィンドウ上に，メソッドやキーワードであることを示すVSCodeライクなアイコンが表示されているではありませんか。
EmacsにできてVimにできないはずがない，そんな気持ちで色々と調べていたところ，Coc.nvimにドンピシャな設定がありました。

## 方法
```
:CocConfig
```
でCocのConfigファイルである`coc-settings.json`を開きます。
そこに以下の内容を記述します。
```json
"suggest.completionItemKindLabels": {
    "function": "\uf794",
    "method":"\uf6a6",
    "variable": "\uf71b",
    "constant": "\uf8ff",
    "struct": "\ufb44",
    "class": "\uf0e8",
    "interface": "\ufa52",
    "text": "\ue612",
    "enum": "\uf435",
    "enumMember": "\uf02b",
    "color": "\ue22b",
    "property": "\ufab6",
    "field": "\uf93d",
    "unit": "\uf475",
    "file": "\uf471",
    "value": "\uf8a3",
    "event": "\ufacd",
    "folder": "\uf115",
    "keyword": "\uf893",
    "snippet": "\uf64d",
    "operator": "\uf915",
    "reference": "\uf87a",
    "typeParameter": "\uf278",
    "default": "\uf29c"
  },
```
これだけです。
NerdFontが表示できる環境であれば，補完ウィンドウ上にアイコンが並びます。
![After](/images/coc_icon/after.png)

## 参考
[How to use colored icons for autocomplete menu nvim.coc?](https://vi.stackexchange.com/questions/28561/how-to-use-colored-icons-for-autocomplete-menu-nvim-coc)
