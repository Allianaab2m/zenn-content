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

昨年8月頃にDiscord.pyが開発停止になってからというもの，もっぱらDiscord.jsとTypeScriptでbotを書いていたのですが，個人的に気に入らないポイントがいくつかあります。

- 初期設定がめんどくさい(Reactの`create-react-app`みたいなの欲しい！)
- ファイルはコマンドごとに切り分けたい(Discord.pyのext-commandsのようなイメージ)
- 切り分けても，自前でコマンドハンドラ実装するのは結構難しい...

これらを一気に解決できるフレームワークがあります。**そう，Sapphireならね。**

ということで，今回はDiscord.js v13のフレームワーク「Sapphire」でのbot開発を行っていきます。
Sapphireは今月に入ってからDocsが整備され，非常に使用しやすくなりました。ありがたい...！

[公式サイト](https://www.sapphirejs.dev/)の記述を引用すると

>- 高度なプラグインのサポート(Advanced plugin support)
>- CommonJSとESMの両方をサポート(Supports both CommonJS and ESM)
>- コマンドやイベントハンドラの完全なモジュール化と拡張性(Completely modular and extendable)
>- TypeScriptのサポートを前提に設計(Designed with first class TypeScript support in mind)
>- あらゆるプロジェクトで利用可能なユーティリティを同梱(Includes optional utilities that you can use in any project)
>
> 引用:Welcome | Sapphire https://www.sapphirejs.dev/docs/General/Welcome#key-features

とのこと。という訳で早速進めていきましょう。

## 筆者の環境

環境は以下の通りです。
yarnの代わりにnpmやpnpmを使用しても構いません。

|環境|バージョン|
|---|----|
|Node.js|v16.13.1|
|yarn|v1.22.17|
|@sapphire/cli|v1.0.2|
|Discord.js|v13.6.0|
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
プロジェクト名のディレクトリに移動しましょう。

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

`watch:start`で起動すると，コマンドファイルの変更を監視しているので，コマンドファイルを変更するたびにbotを再起動する必要はありません。

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
型として使用する場合は，明示的に型としてインポートする必要があるようです。
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
次は引数を受け取るコマンドを作ってみましょう。
新しく`cmdArg`コマンドを作成します。
```
sapphire generate commands cmdArg
```

以下のように自動生成されたコードを編集します。
```ts:commands/cmdArg.ts
import { ApplyOptions } from '@sapphire/decorators';
import type { Args } from '@sapphire/framework';
import { SubCommandPluginCommand, SubCommandPluginCommandOptions } from '@sapphire/plugin-subcommands';
import type { Message } from 'discord.js';

@ApplyOptions<SubCommandPluginCommandOptions>({
	description: 'A basic command'
})
export class UserCommand extends SubCommandPluginCommand {
	public async messageRun(message: Message, args: Args) {
        const arg = await args.pick('string')
		return message.channel.send(arg);
	}
}
```

引数を受け取るには，Argsクラスのpickメソッドを使用します。
pickメソッドの返り値は，コマンドの引数を渡された引数の型に変換したものです。
引数が変換できなかったり，そもそも無かったりするとエラーが発生します。

```ts:pickメソッドでの引数の受け取り方
const arg = await args.pick('型名')
```

渡すことのできる型はプリミティブ型の他に
- member(@[ユーザー名]orID)→[GuildMember](https://discord.js.org/#/docs/discord.js/stable/class/GuildMember)
- guildChannel(#[チャンネル名]orID)→[GuildChannel](https://discord.js.org/#/docs/discord.js/stable/class/GuildChannel) or [ThreadChannel](https://discord.js.org/#/docs/discord.js/stable/class/ThreadChannel)

などを渡すことができます。渡すことのできる型の一覧は[こちら](https://www.sapphirejs.dev/docs/Guide/arguments/built-in-arguments)です。

複数の引数を受け取りたい場合は，受け取りたい引数の数だけpickメソッドを使用します。
```ts:複数の引数を受け取る
const arg1st = await args.pick('string')
const arg2nd = await args.pick('string')
return message.channel.send(`1st:${arg1st}\n2nd:${arg2nd}`)
```
![](/images/sapphire-discordjs/cmdArg2.png)

オプション引数やデフォルト値を設定したい場合は，このように処理します。
```ts:オプション引数として引数を受け取る場合
const optionalArg = await args.pick('string').catch(() => undefined)
if (optionalArg){
    return message.channel.send(`引数:${optionalArg}`)
} else {
    return message.channel.send('引数がありません！')
}
```
```ts:デフォルト値を設定する場合
const defaultArg = await args.pick('string').catch(() => 'default')
```

#### 複数の引数を配列として受け取る
pickメソッドでは，引数の数が一定でないコマンドを作る場合に不都合です。

そこで，複数の引数を配列として受け取るArgsクラスのrepeatメソッドを使用します。
```ts
const argArray = await args.repeat('型名')
```
repeatメソッドの返り値はコマンドの引数をすべて渡された型に変換し，その配列として返します。

注意すべき点として，渡されたコマンドの引数の中に指定した型へ変換できない引数があった場合，repeatメソッドはその時点で処理が止まります。

渡された引数の最大値を返すmaxコマンドを作ったとしましょう。
引数をすべてNumber型に変換し，その最大値を返す処理です。
```ts:maxコマンドの処理
public async messageRun(message: Message, args: Args) {
    const argArray: number[] = await args.repeat('number')
    return message.channel.send(`最大値:${Math.max(...argArray)}`)
}
```
引数の途中にnumber型に変換できないものを渡すと，その後ろの引数を無視し，次の処理に進みます。
以下の例では，hogeの後ろの引数が無視され，最大値が60ではなく，42になっています。

![](/images/sapphire-discordjs/cmdArg3.png)

この例で引数をすべて取得するには，一度stringの配列で取得した上でnumber型に変換するなどの工夫が必要です。

```ts:工夫した例
public async messageRun(message: Message, args: Args) {
    const stringArray: string[] = await args.repeat('string')
    const numberArray: number[] = stringArray.map(str => parseInt(str, 10)).filter(v => v)
    return message.channel.send(`${Math.max(...numberArray)}`)
}
```

### 4.コマンドにオプションを追加する
`@ApplyOptions`にオプションを追記することで，コマンドの省略形(エイリアス)や後述のPreconditionなどを指定することができます。
指定できるオプションは以下のページにリストアップされています。
https://www.sapphirejs.dev/docs/Documentation/api-framework/interfaces/CommandOptions

ここではエイリアス，サブコマンド，コマンドのクールタイムについて触れます。
#### エイリアス
エイリアスはこのように指定します。
他のコマンド名やエイリアスと重複しないようにしてください。
```ts
@ApplyOptions<SubCommandPluginCommandOptions>({
    aliases: ['hoge', 'fuga']
})
```
#### サブコマンド
// TODO: サブコマンドの注意 関数名とサブコマンド名を一致させるなど

#### クールタイム
単位はミリ秒です。同一人物によるコマンド使用を指定した秒数受け付けません。
```ts
@ApplyOptions<SubCommandPluginCommandOptions>({
    cooldownDelay: 10000
})
```
### 5.Preconditionコンポーネントを使用する
高機能なbotを作る上で，権限の有無や指定したチャンネルでのみ返答するなど，条件に応じてコマンドの動作を切り替えたいことはよくあります。

そういった処理を共通化し，どのコマンドでも使用出来るようにできるのがPreconditionというコンポーネントです。

正確な日本語訳がないので，この記事ではPreconditionを「コマンドの前提条件」と表記します。

### ということで

