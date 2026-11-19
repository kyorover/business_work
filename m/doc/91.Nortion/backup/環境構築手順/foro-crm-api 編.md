# foro-crm-api 編

環境構築の流れ

- DBセットアップ
- DBマイグレーション
- コンテナ起動

## Dockerセットアップ

※初回のみ

ネットワークを事前に作っておくこと

```bash
docker network create foro_common_link
docker network ls #で↑がリストにあればOK
```

## DBセットアップ

データベースは medup-core-service のデータベースに統合しているので以下を参照して構築すること。

[medup core service 編](https://app.notion.com/p/medup-core-service-f33ab51be0d044e689adcfc008107b51?pvs=21) 

## コンテナ起動

- foro-crm-apiリポジトリをcloneする
- docker-compose.ymlとdocker-compose.spa.yml内の環境変数`USER_ID`をローカルでログインしたいユーザーのIDに書き換える

```bash
git clone https://github.com/MedUpInc/foro-crm-api.git
```

各種credentials を1Passwordから取得して配置する

| 項目 | 配置パス | 1PasswordのURL |
| --- | --- | --- |
| master.key | foro-crm-api/config/master.key | [https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=fguepyelzznxkbh56gtoek47la&i=7iiibqv5vq667mpy3xuvohcda4&h=medupinc.1password.com](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=fguepyelzznxkbh56gtoek47la&i=7iiibqv5vq667mpy3xuvohcda4&h=medupinc.1password.com) |
| [foro-crm-api-service-account-key-dev.json](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=fguepyelzznxkbh56gtoek47la&i=275akpmfs6bb46cmcjik3hnhie&h=medupinc.1password.com) | foro-crm-api/foro-crm-api-service-account-key-dev.json | [https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=fguepyelzznxkbh56gtoek47la&i=275akpmfs6bb46cmcjik3hnhie&h=medupinc.1password.com](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=fguepyelzznxkbh56gtoek47la&i=275akpmfs6bb46cmcjik3hnhie&h=medupinc.1password.com) |
| .env(環境変数)

↑（2026/1/8 shimboが追加しました） | foro-crm-api/.env | [https://medupinc.1password.com/app#/ZQUJLE6YSBH2PHXHDFLDCLV6N4/Vault/ZQUJLE6YSBH2PHXHDFLDCLV6N4:3xxkfqkns3uljc6gxtiy72hwla:6q7tzhes2melskppjslhgwgez4?itemListId=ZQUJLE6YSBH2PHXHDFLDCLV6N4%3A3xxkfqkns3uljc6gxtiy72hwla](https://medupinc.1password.com/app#/ZQUJLE6YSBH2PHXHDFLDCLV6N4/Vault/ZQUJLE6YSBH2PHXHDFLDCLV6N4:3xxkfqkns3uljc6gxtiy72hwla:6q7tzhes2melskppjslhgwgez4?itemListId=ZQUJLE6YSBH2PHXHDFLDCLV6N4%3A3xxkfqkns3uljc6gxtiy72hwla) |

※ファイルへのアクセス権限がない場合は @chihiro.naganuma  までお知らせください。

指定のパスに配置したうえで、コンテナを立ち上げます

他のサービス(`mhlw_api`,`foro-referral-api`)もローカルで立ち上げる場合

```bash
docker compose up
```

foro-crm-apiのみローカルで立ち上げたい場合(spa開発時など)

```bash
docker compose -f docker-compose.spa.yml up
```

※この場合、`mhlw_api`,`foro-referral-api`は共にstagingのものを見に行くようになるので注意すること

port listenが始まれば成功です

ステージング・プロダクション環境へのDBマイグレーション手順は以下を参照

[データベースマイグレーション手順&接続方法](https://app.notion.com/p/6aaf357996ee491aa7fee0e645671921?pvs=21) 

AZURE_MS_GRAPH_API_CLIENT_ID