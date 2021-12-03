---
title: "DiscordBotをTypeScriptで書くチュートリアル"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Discord", "TypeScript"]
published: false
---

## 前置き

DiscordのBotをTypeScriptで開発する記事が少なくて難儀したので，自分なりに記事を書いてみることにしました。

こういった記事を書くのは初めてなので，拙い文章ではありますが，誰かの助けになれば幸いです。

## 対象読者

- Discord.jsを使用したBot開発の経験がある人

今回の記事の主軸はTypeScriptを使ってBot開発をするという点にあるため，**Bot開発やNode.jsなどに関する基本的な説明はところどころ端折っています**。

予めご了承ください。

## 環境
|環境|バージョン|
| --- | --- |
| Node.js | v16.13.0 |
| TypeScript | v4.5.2 |
| Discord.js | v13.3.1 |
| Visual Studio Code | v1.62.3 |

エディタは好きなものを使っても構いませんが，Visual Studio CodeはTypeScript補完に関する機能が**非常に強力**[^1]なので，TypeScriptを書く際にはVisual Studio Codeをオススメします。

## 手順

1. 必要なパッケージをインストールする
2. 初期設定
3. 実際にコードを書く
4. null/undefinedに対処する

## 必要なパッケージをインストールする

適当なフォルダの中で，以下のコマンドを実行してください。

```bash
npm init -y
npm install --save-dev typescript ts-node eslint
npm install discord.js dotenv
```

バージョンが表示されれば，インストールは成功しています。

```bash
npx tsc --version
Version 4.5.2
```

## 初期設定

### TypeScript

以下のコマンドを実行してください。

```bash
npx tsc --init
```

`tsconfig.json`というファイルが新しく出来ているはずです。お好きなエディタで開いてみてください。

中身の殆どがコメントアウトされていますね。

以下の行を探し出し，コメントを外してから**書いてある通りに書き換えてください**。

```json
{
  // 中略
  "outdir": "./build"
  // 略
}
```

これはトランスコンパイルを行ったときに生成されたJavaScriptコードを出力するディレクトリを指定しています。

### ESLint

```bash
npx eslint --init
```

対話式のセットアップウィザードが始まります。矢印キーなどを使って以下のように指定してください。

|聞かれる内容|答える内容|
| --- | --- |
| How would you like to use ESLint? (ESLintをどのように使いますか？) | To check syntax, find problens, and enforce code style |
| What type of modules does your project use?(どのタイプのモジュールを使用しますか？) | JavaScript modules (import/export) |
| Which framework does your project use?(フレームワークを使用しますか？) | None of these |
| Does your project use TypeScript?(TypeScriptを使用しますか？) | Yes |
| Where does your code run?(コードをどの環境で実行しますか？) | Browser, Node(aキーを押すと両方選択される) |
| How would you like to define a style for your project?(コードのスタイルはどのように定義しますか？) | Use a popular style guide |
| Which style guide do you want to follow?(どのスタイルガイドに従いますか？) | Standard |
| What format do you want your config file to be in?(configファイルはどのよう形式で保存しますか？) | JavaScript |
| Would you like to install then now with npm?(npmを使用して今必要なパッケージをインストールしますか？) | Yes |

また，**Visual Stuido Codeの場合はESLint拡張機能をインストールしておきましょう**。

https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint

### package.json

package.jsonにnpmスクリプトを指定します。

```json
"scripts": {
	"test": "ts-node src/main.ts",
	"start": "node build/main.js",
	"compile": "tsc -p ."
}
```

最後に，現在のディレクトリの中に `src`と `build`フォルダ，そして  `.env`ファイルを新しく作ってください。以下のような状態になっているでしょうか。

```bash:現在のディレクトリの様子
📁 node_modules
📁 src
📁 build
📄 package-lock.json
📄 package.json
📄 tsconfig.json
📄 .eslintrc.js
📄 .env
```

## 実際にコードを書く

ではコードを書いていきます。

`src/main.ts` としてください。

```tsx:src/main.ts
import { Message, Client } from 'discord.js'
import dotenv from 'dotenv'

dotenv.config()

const client = new Client({
    intents: ['GUILDS', 'GUILD_MEMBERS', 'GUILD_MESSAGES'],
})

client.once('ready', () => {
    console.log('Ready!')
    console.log(client.user.tag)
})

client.on('messageCreate', async (message: Message) => {
    if (message.author.bot) return
    if (message.content.startsWith('!ping')) {
        message.channel.send('Pong!')
    }
})

client.login(process.env.TOKEN)
```

エラーが出ている行がありますが，ひとまずトランスコンパイルしましょう。

```bash
npm run compile
```

```bash:トランスコンパイルを実行した結果
src/main.ts:12:17 - error TS2531: Object is possibly 'null'.

12     console.log(client.user.tag)
                   ~~~~~~~~~~~

Found 1 error.
```

エラーの内容は，「 `client.user` がnullかもしれないよ」というエラーです。こういったエラーの対処法は手順4で解説するので，ひとまず置いておきましょう。

エラーが発生したものの，トランスコンパイル自体は実行されています。その証拠として `build/main.js` がありますね。

botを起動します。

`.env` ファイルに

```text:.env
TOKEN = 'Discord Developer Portalで取得したトークン'
```

を書き込み，

```bash
npm run start
```

を実行しましょう。

botがオンラインになったら，`!ping` と打ってみてください。返答があれば成功です。

![](https://storage.googleapis.com/zenn-user-upload/558fc665fba2-20211123.png)

## null/undefinedに対処する

TypeScriptはトランスコンパイラから見て，null/undefined(nullishと言います)になる値に対して**実行時ではなく，トランスコンパイル時にエラーを出すようになっています**。

先程のエラーは「 `client.user` がnullかもしれない」というエラーでした。

botにログインする前にreadyイベントが発火された場合，`client.user`は**nullを返します**。

もちろん，**null型はログイン中Botのユーザー名を示すtagプロパティを持たない**ので，このままでは実行時に**エラーになってしまいます**。

ということで，`client.user`がnullishで無いことを保証しなければなりません。

そのような場合は，以下のように対処します。

```tsx
client.once('ready', () => {
    console.log('Ready!')
    if (client.user) {
        console.log(client.user.tag)
    }
})

// or

client.once('ready', () => {
    console.log('Ready!')
    console.log(client.user?.tag)
})
```

上の例は理解しやすいと思います。

nullishでない値はTrueを返す性質を利用し，nullでないことを保証しています。

下の例は，オプショナルチェーンと呼ばれるもので，アクセス元がnullishであっても，エラーを出すことなく`undefined`を返すようになります。

https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Optional_chaining

どちらかのように対処すれば，赤波線が消えたはずです。

もう一度トランスコンパイルしてみましょう。何も表示されなければOKです。

## 参考にさせていただいた記事・サイト

https://qiita.com/hitori_yuu/items/02eae8b14dc6a9c91c0d

https://book.yyts.org/

---

[^1]: ちなみに，私は補完機能目当てでTypeScript使ってるまであります。~~paramとかわからん…~~
