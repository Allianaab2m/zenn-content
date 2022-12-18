---
title: "NextcloudにTailscale経由でアクセス出来ない時の対処法"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextcloud", "docker", "tailscale"]
published: false
---

最近[Tailscale](https://tailscale.com)というサービスを知りました。

特に複雑な設定をすることなく、自宅と外部ネットワーク間が繋がるサービスです。

また、自宅ではラズパイが稼動しており、NextcloudをDockerでホストしています。

Tailscaleにより割り当てられたアドレスにアクセスすると、Nextcloudのログイン画面が表示されるべき所、こんなエラーが出ました。

```
Access via an untrusted domain
Please contact your administrator. If you are administrator, edit the trusted_domains setting in config/config.php.
See example in config/config.sample.php.
```

要は、信頼してないドメイン(=Tailscaleのアドレス)からはアクセスできないよ、というエラーです。

## 対処法

```bash
docker exec --user www-data <Container Id> php occ config:system:get trusted_domains # 今のアドレス確認
docker exec --user www-data <Container Id> php occ config:system:set trusted_domains 2 --value=<Domain>
```

こうすることでアクセスできるようになります。

TailscaleにはMagicDNSという端末名だけでアクセスできる機能がありますが、その機能を使用する場合は端末名を直接指定してください。

例) 端末名がraspi4の場合 -> `--value=raspi4`

## 試したけどダメだった方法

ググるとdocker-compose.ymlの`environment`に`NEXTCLOUD_TRUSTED_DOMAINS`を追加する方法が出てきますが、これは上手くいきませんでした。

```yml
...
services:
    nextcloud:
        ...
        environment:
            - NEXTCLOUD_TRUSTED_DOMAINS=<Domain>
```

## 参考にしたサイト

[Nextcloud (in Docker) not accepting trusted domains - ℹ️ Support / 📦 Appliances (Docker, Snappy, VM, NCP, AIO) - Nextcloud community](https://help.nextcloud.com/t/nextcloud-in-docker-not-accepting-trusted-domains/64208/4)
