---
title: "GitHub Actionsからsshしてデプロイする(Tailscale使用)"
emoji: "🚇"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Tailscale", "GithubActions"]
published: true
---

mainブランチに更新があった時，デプロイ先の自宅サーバで`git pull`をして所定の更新スクリプトを走らせる作業を自動化したかったのでActionsでやってみました．
Tailscaleを使うことでGithub Actionsから比較的安全にデプロイ先へsshすることができます．

## 手順

OAuthクライアントを通じてTailscaleに接続する場合は，タグをACLに書き込む必要があります．
ACLに以下のように追記します．

https://login.tailscale.com/admin/acls/file

```json
{
    "tagOwners": {
        "tag:ci": ["<Your_Tailscale_ID>"] // 追記
    },
    "acls": [
        {"action": "accept", "src": ["*"], "dst": ["*:*"]},
        {"action": "accept", "src": ["tag:ci"], "dst": ["tag:ci:*"]}, // 追記
    ],
}
```

`tagOwners`のIDは記述しなくてもいいようですが，今回は記述しておきました．

次にActionsからTailscaleのネットワークに接続するためのOAuthクライアントを作成します．

https://login.tailscale.com/admin/settings/oauth にアクセスし，DevicesのReadとWriteをscopeに指定します．
Writeにチェックを入れると，先程作成したタグを選択するよう求められるので，選択します．
Generate Clientを押し，Client IDとClient Secretを控えておきます．

次にWorkflowを作成します．今回は`main`ブランチへpushされた場合にActionを実行するようにしています．scriptなどの内容は適宜読み替えてください．

```yml:.github/workflows/action.yml
on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Auto Deploy via ssh
    runs-on: ubuntu-latest
    steps:
      - name: Setup Tailscale
        uses: tailscale/github-action@v2
        with:
          oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
          oauth-secret: ${{ secrets.TS_OAUTH_SECRET }}
          tags: tag:ci
      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          post: ${{ secrets.SSH_PORT }}
          script: |
            cd /path/to/repository
            git pull origin main
            pnpm build
```

レポジトリのSettings>Secret and Variables>Actionsに移り，環境変数を設定します．

| 変数名  | 記述する内容   |
|-------------- | -------------- |
| TS_OAUTH_CLIENT_ID | 先程控えたClient ID     |
| TS_OAUTH_CLIENT_SECRET | 先程控えたClient Secret     |
| SSH_HOST | デプロイ先のTailscale IP |
| SSH_USERNAME | デプロイ先のユーザーネーム |
| SSH_PORT | デプロイ先のポート番号 |
| SSH_PRIVATE_KEY | SSH接続用の秘密鍵を生成 |

接続用の鍵は`ssh-keygen`を実行し，新しく鍵を生成した後，公開鍵をデプロイ先の`authorized_keys`に追記してください．

当初設定した条件で問題なく動作していれば完了です．お疲れ様でした．

## 注意事項

なぜか一部パスが通っていないようで，絶対パスを使わなければエラーとなってしまうことがありました．具体的にはpnpm globalでインストールしたpm2などが利用できませんでした．

## 参考文献

https://softwarenote.info/p3789/
https://tailscale.com/kb/1276/tailscale-github-action/
https://tailscale.com/kb/1068/acl-tags/
