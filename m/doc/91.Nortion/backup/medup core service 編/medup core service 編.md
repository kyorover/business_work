# medup core service 編

## ローカルにコードを落とす

[https://github.com/MedUpInc/medup-core-service](https://github.com/MedUpInc/medup-core-service)

まだ、GitHubの招待が済んでいない場合は、担当者にご一報ください

```bash
$ git clone https://github.com/MedUpInc/medup-core-service.git
```

## データベースのスキーマ管理について

データベースのスキーマ管理は `packages/foro-prisma` に集約しています。

以降の環境構築手順にてデータベースのマイグレーション関連のコマンドは packages/foro-prisma 配下で実行します。

## node のセットアップ

- node 22.22.x を使用 → node のバージョン管理は [**nvm**](https://github.com/nvm-sh/nvm) を推奨
- npm 10.9.x を使用

※TAKUMI_GUARD_TOKENの設定

[https://shisho.dev/docs/ja/t/guard/features/token-management/](https://shisho.dev/docs/ja/t/guard/features/token-management/)

- メール認証トークンを実施
    - メールアドレス登録を行うと、登録したメールアドレスにAPIキーが発行される
    - 通知メールにある「利用開始方法」を実行することでnpmのインストールがTAKUMI＿GUARD経由で実行可能になる
        
        

### npm packages のインストール

```bash
# ルート配下で実施
$ npm ci
```

**※※※ npm ci は必ずルートディレクトリで行ってください！ ※※※**

→ ちなみに… monorepo の管理として [**Turborepo**](https://turborepo.org/) を使用しています。

Turborepo の概要は馬場：個人Notionのまとめを御覧ください。

## 環境変数・credentialsのセットアップ

各サービス配下に 1pass から .env を取得し配置する

- [/apps/api](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=mmcfknezamzrlukhu62y5x3wuu&h=medupinc.1password.com)
- [/apps/foro-command-service](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=3c2hdla2lf5yspqfw7v7o2uv5y&h=medupinc.1password.com)
- [/apps/foro-query-service](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=5hc5iwjxbtdlpaw2z3a3s6iqgm&h=medupinc.1password.com)

- [/packages/foro-prisma](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=4p3bpocjnpduyk5ctjfbsrqd74&h=medupinc.1password.com)

同様に1passからcredentialsを取得し配置する

- [/apps/api](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=i5vap3bmtwwo4x734wdmhw7pdu&h=medupinc.1password.com)
- [/apps/foro-command-service](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=simkrow2ekkd37yvqadjer6x64&h=medupinc.1password.com)

## Docker (DB) 起動

```bash
# ルート配下で実施
$ docker compose up
```

## データのセットアップ

データベースの作成

以下のコマンドで医療機関データベース、foroデータベースが作成されます

```bash
# packages/foro-prisma 配下で実施
$ npm run db:migrate:deploy
```

**医療機関データベースへ seed 投入**

seedデータを [#foro_dev_seed](https://medup.slack.com/archives/C0406G1EKT2) から取得、ルート直下に `mis-seed-prd.sql` として配置して以下を実行

- (最新データがほしい時) medup-foro/dump-mis-seed-data cloud run job を手動実行させる方法
    - [https://console.cloud.google.com/run/jobs?authuser=0&invt=AbucWg&project=medup-foro](https://console.cloud.google.com/run/jobs?authuser=0&invt=AbucWg&project=medup-foro) cloud run job に移動
    - dump-mis-seed-data の詳細に遷移し、GUI から実行をクリック

```bash
# ルート配下で実施 (外部キー制約が効いていると一部のデータが入らないので一時的に無効化する
$ psql "postgres://postgres:rootpassword@localhost:5432/mis_dev" -c "SET session_replication_role = replica;" -f mis-seed-prd.sql -c "SET session_replication_role = DEFAULT;"
```

**foro用データベースへ seed 投入**

seedデータを [#foro_dev_seed](https://medup.slack.com/archives/C0406G1EKT2) から取得、ルート直下に `foro_crm_seed_foro-crm-api_dump_foro-crm-api.tar.gz` として配置して以下のスクリプトを実行

[](https://console.cloud.google.com/storage/browser/_details/foro_development/foro_crm_seed/foro-crm-api/dump_foro-crm-api.tar.gz;tab=live_object?authuser=1&project=analytics-tool-206405)

```bash
tar -zxf foro_crm_seed_foro-crm-api_dump_foro-crm-api.tar.gz

export PGPASSWORD=rootpassword
outputPath="./out/foro-crm-seed/"

for file in $outputPath/*.csv; do
    tableName=$(basename "$file" .csv)

    # CSVからテーブルにデータをコピー
    psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "SET session_replication_role = replica;" -c "\\COPY $tableName FROM '$file' WITH CSV HEADER;" -c "SET session_replication_role = DEFAULT;"
    
		# シーケンスを更新
    sequenceName="${tableName}_id_seq"
    psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "SELECT setval('$sequenceName', (SELECT max(id) FROM $tableName));"
done
```

**※ まだアカウントを作成していない方向け　アカウントを作成してください！**

```bash
# ルート配下で実施
$ PGPASSWORD=rootpassword psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "insert into users (email, username, aad_user_id, tenant_id, encrypted_password, created_at, updated_at) values ('local-user@medup.jp', 'medup', '37a38072-acd9-4590-a71f-233bcdf9af86', 36, '$2a$11$t9IW0YZAjdDajCcl3L.JdeVhCcT/.sYWt21niPia/cX5wsqSnICaO', now(), now());"
```

認証情報は、1passwordに登録されています！

### **rls_user の作成・権限追加のスクリプトを流す**

(初回 docker compose 起動時に自動で実行されるが既にDBやテーブルが存在する場合は手動実行が必要)

ルートディレクトリで以下を実行する

```bash
# ルート配下で実施
$ psql -U postgres -h 0.0.0.0 -p 5432 -d foro_dev < script/01_create_rls_user.sql
```

テストデータベース用

```bash
# ルート配下で実施
$ psql -U postgres -h 0.0.0.0 -p 5432 -d foro_test < script/02_grant_to_foro_test.sql
```

# アプリケーションのセットアップ

**◆ prisma client (kysely) のコードを生成する**

```bash
# packages/foro-prisma 配下で実施
$ c
```

↑ こちらのコードを実行することで、schema.prisma の情報をみて、ローカルで prisma client (kysely) のコードが自動生成されます！

### 動作確認

```bash
# apps/api 配下で実施
$ npm run dev

# 別ターミナルから次のコマンドを実行して以下のようなレスポンスが返ってきたらOK
$ curl localhost:8888/docs/yaml
openapi: 3.0.3
info:
title: medup-core-service api
  description: medup-core-service api schema
  version: 0.1.0
...f
```

### 一括起動

turborepo の機能で全サービスを一気に起動することもできる

```bash
# ルート配下で実施
$ npm run dev
```

### 単体テスト環境の構築

**◆ 単体テスト用のデータベースを作成する**

```bash
# packages/foro-prisma 配下で実施
$ npm run db:migrate:test
```

### 単体テストの実行

- 以下のコマンドで全ての単体テストが実行されます (ファイル指定も可能)

```bash
# apps/api 配下で実施
$ npm run test
```

---

# (付録) マイグレーションのお作法

### スキーマを変更する場合

[マイグレーションファイルの作り方](https://app.notion.com/p/092e5c051dd14c39a816abef96d6b6a2?pvs=21) 

### マイグレーションを適用する場合

既に作成されたマイグレーションファイルを適用する場合は以下を実施する。

```bash
# packages/foro-prisma 配下で実施
$ npm run db:migrate:deploy:foro
```

## テスト環境

DBの変更後にspecや jest を走らせる必要がある場合に明示的にマイグレーションを実行する

```bash
# packages/foro-prisma 配下で実施
$ npm run db:migrate:test
```

### 本番環境へのマイグレーション

**◆** 医療機関データベースへのマイグレーション

```bash
# packages/foro-prisma 配下で実施
$ MIS_DATABASE_URL=postgresql://postgres:[password]@localhost:5432/foro_prd?shcema=public npm run db:migrate:deploy:mis
```

**◆ foro**データベースへのマイグレーション

[データベースマイグレーション手順&接続方法](https://app.notion.com/p/6aaf357996ee491aa7fee0e645671921?pvs=21) 

## マイグレーションに失敗したら？

`_prisma_migrations`テーブルにマイグレーション失敗の記録が残っており、そのままだとマイグレーションを実行できないため、失敗したマイグレーションをrollback状態にマークする必要がある。

`.env`を対象の環境に向けた設定にしてから以下を実行することでrollback状態にマークできる。完了後は改めて修正したマイグレーションを実行することが可能。

```sql
# .env 実行する環境に向けた状態にしておく

$ cd packages/foro-prisma

$ npx prisma migrate resolve \
		--rolled-back マイグレーション名 \
		--schema prisma/foro/schema.prisma
```