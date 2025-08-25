---
title: "NestJSでOxlintを使う"
emoji: "⚓️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nestjs", "oxlint"]
published: true
publication_name: emobi_tech
---

NestJS はフォーマッタとしてデフォルトで ESLint + Prettier が設定されています。

フォーマットはファイルの保存時やコミット時といったタイミングで頻繁に実行されることから、実行速度が遅いと実行を待つ時間がちょっとしたストレスになります。

そこでフォーマッタを Oxlint に置き換えることでフォーマットにかかる時間を短縮しました。

Oxlint を採用した理由は以下の通りです。

- 高速である
- ESLint と共存できる
- v1 安定版がリリースされた

NestJS や Next.js のようなフレームワークでは、今でも ESLint や Prettier が広く使われています。フレームワークによっては、そのフレームワーク専用の ESLint ルールがあらかじめ設定されていることもあります。

これらのツールを別のツールに置き換えてしまうと、以下のような問題が起こる可能性があります。

- 独自ルールの移行が大変：フレームワーク用のカスタムルールを移行する必要がある
- エラーの見逃し：これまで ESLint が見つけてくれていた種類のエラーを見つけられなくなる

Oxlint は、既存の ESLint, Prettier と共存させて使うことができます。

全てのルールを Oxlint に任せるのではなく、Oxlint がまだ対応していない型情報が必要な複雑なルール、カスタムルールなどは、今まで通り ESLint や`typescript-eslint`を使ってチェックできます。

Biome も候補として挙がりましたが、カスタムルールの移行やフレームワークのサポートがまだ不十分であることから、採用を見送りました。

## 導入

すでに NestJS のプロジェクトが作られている前提で進めます。

```sh
pnpm add -D oxlint eslint-plugin-oxlint
```

`eslint.config.mjs`を編集して Oxlint で適用されているルールを ESLint 側で無効化する Plugin を導入します。

```diff js:eslint.config.mjs
  // @ts-check
  import eslint from '@eslint/js';
+ import oxlint from "eslint-plugin-oxlint";
  import eslintPluginPrettierRecommended from 'eslint-plugin-prettier/recommended';
  import globals from 'globals';
  import tseslint from 'typescript-eslint';

  export default tseslint.config(
    {
      ignores: ['eslint.config.mjs'],
    },
    eslint.configs.recommended,
    ...tseslint.configs.recommendedTypeChecked,
    eslintPluginPrettierRecommended,
    {
      languageOptions: {
        globals: {
          ...globals.node,
          ...globals.jest,
        },
        sourceType: 'commonjs',
        parserOptions: {
          projectService: true,
          tsconfigRootDir: import.meta.dirname,
        },
      },
    },
    {
      rules: {
        '@typescript-eslint/no-explicit-any': 'off',
        '@typescript-eslint/no-floating-promises': 'warn',
        '@typescript-eslint/no-unsafe-argument': 'warn'
      },
    },
+   ...oxlint.configs['flat/recommended']
  );
```

あとは`pnpm lint`時に実行されるコマンドに`oxlint`を追加すれば完了です。

```diff json:package.json
...
- "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
+ "lint": "oxlint \"{src,apps,libs,test}/**/*.ts\" --fix && eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
...
```

## 感想

まだ始まりたてのプロジェクトでコードベースが大きくなっていないことから、導入前後でそこまでの速度の差は感じませんでしたが、コードベースが大きくなるにつれて高速化の恩恵を感じられそうです。

一週間ほど前に oxc から Type-Aware Linting のプレビューリリースがされており、こちらも高速で`typescript-eslint`を置き換える候補になりそうです。

https://oxc.rs/blog/2025-08-17-oxlint-type-aware.html

今後も Oxlint の動向はチェックしていきたいと思います。
