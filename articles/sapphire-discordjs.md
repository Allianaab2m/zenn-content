---
title: "Discord.jsのフレームワーク「Sapphire」を使おう"
emoji: "💎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Discord", "JavaScript", "TypeScript"]
published: false
---

## 想定読者
- Discord.jsや他のライブラリ(Discord.pyやJDAなど)を使用してDiscord用bot開発を行ったことがある人

---

Discord.js + TypeScriptでbotを書くのは非常に快適ですが，個人的に不満な点がいくつかあります。

- 初期設定がめんどくさい(Reactの`create-react-app`みたいなの欲しい！)
- ファイルはコマンドごとに切り分けたい(Discord.pyのext-commandsのようなイメージ)
- 切り分けたとしても，自前でコマンドハンドラ実装するのは結構難しい...

処理の切り分けに関しては，Discord.js v12時代に[commando](https://github.com/discordjs/Commando)という公式フレームワークがあったのですが，v13になりアーカイブされてしまいました。

そんな中，突如として[^1]現れたフレームワークが今回紹介する「Sapphire」です。

[^1]: 突如としてと書きましたが，Sapphire自体は結構前から開発されています。今月に入ってから，Docsが整備されたことでより開発が進めやすくなりました。

[公式サイト](https://www.sapphirejs.dev/)の記述を引用すると

>- 高度なプラグインのサポート(Advanced plugin support)
>- CommonJSとESMの両方をサポート(Supports both CommonJS and ESM)
>- コマンドやイベントハンドラの完全なモジュール化と拡張性(Completely modular and extendable)
>- TypeScriptのサポート(Designed with first class TypeScript support in mind)
>- あらゆるプロジェクトで利用可能なユーティリティを同梱(Includes optional utilities that you can use in any project)
>
> 引用:Welcome | Sapphire https://www.sapphirejs.dev/docs/General/Welcome#key-features

とのこと。という訳で早速Sapphireを使ってbot開発をしてみましょう。

## 筆者の環境

環境は以下の通りです。
yarnの代わりにnpmやpnpmを使用しても構いません。

|環境|バージョン|
|---|----|
|Node.js|v16.13.1|
|yarn|v1.22.17|
|Discord.js|v13.6.0|
|@sapphire/cli|v1.0.2|
|TypeScript|v4.5.4|

---

### 1.環境構築

SapphireのCLIツールを使えば，コマンド1つでbot開発環境を構築できます。

:::message alert
CLIツールを使用するにはv16.7以上が必要です。可能な限りv16(LTS)の最新版を使うようにしてください。
:::

`@sapphire/cli`をグローバルインストールします。

```bash
yarn global add @sapphire/cli
```

適宜ホームディレクトリなどに移動して，

```bash
sapphire new
```

を実行します。すると，対話式のウィザードが起動します。
ここでは以下のように選択します。
JavaScriptを使用する場合など，ご自身の環境に応じて変更してください。

```
✔ What's the name of your project? … プロジェクト名
✔ Choose a language for your project › TypeScript (Recommended)
✔ Choose a template for your project › Default template (Recommended)
✔ What format do you want your config file to be in? › YAML
✔ What package manager do you want to use? › Yarn (Recommended)
✔ Do you want to create a git repository for this project? … yes
```

しばらく待って，`Done!`が表示されればOKです。yarnもしくはnpmでSapphireに必要なパッケージがインストールされています。
プロジェクトディレクトリに移動しましょう。

CLIツールを使用して作成したbotのプロジェクトはこんな感じになっています。[^2]

[^2]: バージョンによって構成が変わる可能性があります。

一部ディレクトリは階層を省略しています。
```bash:プロジェクトディレクトリ
.
├── README.md
├── package.json
├── src
│   ├── commands # コマンドの処理を格納
│   │   └── General
│   │       ├── command-with-decorators.ts
│   │       ├── command-with-subcommands.ts
│   │       ├── eval.ts
│   │       ├── paginated-message.ts
│   │       └── ping.ts
│   ├── lib
│   │   └── constants.ts ...
│   ├── listeners # イベントリスナーの処理を格納
│   │   ├── commands
│   │   │   ├── commandDenied.ts
│   │   │   └── commandSuccessLogger.ts
│   │   ├── mentionPrefixOnly.ts
│   │   └── ready.ts
│   ├── preconditions # 前提条件を格納
│   │   └── OwnerOnly.ts
│   ├── routes
│   │   └── main.ts ...
│   └── index.ts
├── .env
├── tsconfig.json
└── yarn.lock

```

一度起動してみましょう。`.env`に以下の内容を記述します。
```txt:.env
DISCORD_TOKEN=[botのトークン]
OWNERS=[自分のユーザーID]
```
```bash:起動コマンド
yarn run watch:start
```
botがオンラインになっているのを確認して，適当なチャンネルで`dr!ping`と打って返答があればbotは正常に動作しています。

Sapphireは自動的にコマンドファイルなどの更新を行うので，コマンドファイルを変更するたびにbotを再起動する必要はありません。

### 2.簡単なコマンドを作る

環境構築と動作確認ができたので，botに追加する新たなコマンドを作っていきます。

コマンドの作成もCLIから行うことができます。(すごい)
```bash:新規コマンドを作成するCLIコマンド
sapphire generate command [コマンド名]
```
ここでは`test`というコマンドを追加します。
```bash
sapphire generate command test
```
`commands/`以下に`test.ts`が生成されました。中身はこんな感じです。

```ts:commands/test.ts
import { ApplyOptions } from '@sapphire/decorators';
import { SubCommandPluginCommand, SubCommandPluginCommandOptions } from '@sapphire/plugin-subcommands';
import type { Message } from 'discord.js';

@ApplyOptions<SubCommandPluginCommandOptions>({
	description: 'A basic command'
})
export class UserCommand extends SubCommandPluginCommand {
	public async messageRun(message: Message) {
        // コマンドを受け取ったときに実行される部分
		return message.channel.send('Hello world!');
	}
}
```

:::details なんかエラー出てるけど？
実は最初に生成されたコードにはちょっとしたバグがあります。(PRを送り，マージされたのでそのうち修正されるはずです)
3行目のインポートにtype接頭辞がついていないためにエラーが出ています。
```diff ts:修正
- import { Message } from 'discord.js';
+ import type { Message } from 'discord.js';
:::

`messageRun`関数を編集します。

```ts:commands/test.ts
...
public async messageRun(message: Message) {
    return message.channel.send('テストコマンドが実行されました')
}
```

`dr!test`と打って設定した通りに返答があれば成功です。

### 3.引数を受け取るコマンドを作る
// TODO: argsのとり方に応じてNumで取れたりstringで取れたりすることをアピール

### 4.コマンドにオプションを追加する
// TODO: ApplyOptionsに書ける内容を列挙する

### 5.前提条件を使用する
// TODO: preconditionという概念を簡単に説明できる例を提示する

### 6.