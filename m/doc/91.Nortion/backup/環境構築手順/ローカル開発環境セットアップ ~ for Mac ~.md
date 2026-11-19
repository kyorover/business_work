# ローカル開発環境セットアップ ~ for Mac ~

タグ: 全体
作成日: 2022年10月21日 22:27
最終更新日: 2026年9月24日 11:23

# GitHub から refery を clone

[https://github.com/MedUpInc/refery](https://github.com/MedUpInc/refery)

```bash
git clone https://github.com/MedUpInc/refery.git
```

※ ディレクトリは基本小文字、ファイルでindex以外を大文字と、小文字・大文字を明確に使い分けているので、git で大文字・小文字の区分ができるようにしておくことをおすすめします

```bash
$ git config core.ignorecase false
```

[git で大文字子文字を区別する方法](https://zenn.dev/msksgm/articles/20211122-clearify-upper-lower)

# ローカル開発環境の全体像を把握

refery は monorepo 構成になっており、api, web の両方が同じリポジトリ上で管理されています。

ざっくりとしてディレクトリ構成は以下。

```
├── apps // アプリケーション一覧
│   ├── api
│   └── web
├── config // ミドルウェアの設定ファイル群
├── packages // 各種アプリケーションで利用する独自ライブラリ(共通設定) ex) lint, tsconfig, ...etc
├── docker-compose.yml // ローカル用ミドルウェア群の管理はこのdocker-composeでやってます (DBとか)
├── generateApi.sh // api で生成した swagger から openapi-generator を利用して、HttpClient のコードを生成
├── ....
```

**◆ 全体図**

- ミドルウェアは、docker-compose で管理
- SPA, API はそれぞれ直接ローカル上で起動している

![スクリーンショット 2022-10-21 22.47.50.png](%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588_2022-10-21_22.47.50.png)

# Docker セットアップ

docker が手元の Mac に入っていない方はまずインストールしておいてください

### Docker 初回セットアップ

```bash
# ネットワークを生成
$ docker network create refery_default
```

# 各種アプリケーションのセットアップ

**◆ node, npm のバージョン**

- node v22.14.0 を使用 → node のバージョン管理は nvm を推奨
- npm 10.9.2 を使用

### npm packages のインストール

```bash
$ npm ci
```

**※※※ npm ci は必ずルートディレクトリで行ってください！ ※※※**

→ ちなみに… monorepo の管理として [**Turborepo**](https://turborepo.org/) を使用しています。

Turborepo の概要は馬場：個人Notionのまとめを御覧ください。

### 環境変数のセットアップ

**◆ api**

- /apps/api 配下に `.env` ファイルを作成し、以下を書き込んでください

```bash
DATABASE_URL="postgresql://postgres:refery@localhost:5432/refery_development?schema=public"
```

**◆ spa**

- /apps/web 直下に `.env` ファイルを作成し、以下を書き込んでください

```bash
NEXT_PUBLIC_B2C_CUSTOM_DOMAIN="referydevelopment.b2clogin.com"
NEXT_PUBLIC_B2C_TENANT_ID="2714c669-a815-4b88-91f7-6556d7d13de5"
NEXT_PUBLIC_B2C_CLIENT_ID="b1f3812f-1c50-4e89-a9c4-cbd2420d928d"
NEXT_PUBLIC_B2C_TENANT_NAME="referydevelopment.onmicrosoft.com"
NEXT_PUBLIC_API_HOST="http://localhost:3001"
NEXT_PUBLIC_SPA_URL="http://localhost:3003"
NEXT_PUBLIC_REFERY_APP_ENV="development"
NEXT_TELEMETRY_DISABLED=1
```

### データベース関連のセットアップ

**◆ マイグレーション**

※ マイグレーションの管理は [**prisma**](https://www.prisma.io/) を利用しています

(prisma には、データマッパーとしての役割もあり、マイグレートを実行した際に、prisma/client というDBクライアントのコードも自動生成します)

```bash
# ルートディレクトリにて、以下を実行 → DBコンテナの立ち上げ
$ docker-compose up

# /apps/api 配下にて、以下を実行
$ npm run db:migrate:dev
```

- **初期データ投入(アカウント)**

以下コマンドを実行して、DBにデータを入れておいてください

```bash

# tenant の作成
PGPASSWORD=refery psql -h localhost -p 5432 -d refery_development -U postgres -c "insert into tenants (id, name, postal_code, prefecture, city, street, phone_number, days_before_instant_reservation, days_before_request_reservation, created_at, updated_at) values (gen_random_uuid(), 'medup', '101-0061', '東京都', '千代田区', '神田三崎町3-2-14 GLORKS水道橋4F', '03-6820-2485', 3, 3, now(), now());"
# user の作成
PGPASSWORD=refery psql -h localhost -p 5432 -d refery_development -U postgres -c "insert into users (id, display_name, tenant_id, email_address, user_name, role, created_at, updated_at) values (gen_random_uuid(), 'refery', (select id from tenants limit 1), null, 'refery-dev', 'admin', now(), now());"
```

- **環境変数に以下を追記しましょう**

**◆ api**

- /apps/api 配下の .env ファイルに以下を追記してください

```bash
DATABASE_URL="postgresql://postgres:refery@localhost:5432/refery_development?schema=public"
GCS_BUCKET_NAME="refery-user-uploads-stg"

B2C_TENANT_ID="2714c669-a815-4b88-91f7-6556d7d13de5"
B2C_CLIENT_ID="f424d71a-4aea-444d-bd86-b2082fff0abc"
B2C_EXTENSION_APP_ID="dd91557d-b3e3-49d3-8202-59076362bcb5"
B2C_ISSUER="referydevelopment.onmicrosoft.com"
B2C_JWK_URI="https://referydevelopment.b2clogin.com/referydevelopment.onmicrosoft.com/b2c_1a_signup_signin/discovery/v2.0/keys"
B2C_JWK_ISSUER="https://referydevelopment.b2clogin.com/2714c669-a815-4b88-91f7-6556d7d13de5/v2.0/"

ADMIN_BASIC_AUTH_USERNAME="refery"
ADMIN_BASIC_AUTH_PASSWORD="refery"

REFERY_APP_ENV="development"

# TODO: GCSに上がっているrefery-stagingのenvを持ってきてSECRETとAPIKEYを埋める
B2C_CLIENT_SECRET="ここを埋める"
SENDGRID_API_KEY="ここを埋める"

TENANT_USER_ID="初期データ投入で作ったデータのIDを入れる"
TENANT_ID="初期データ投入で作ったデータのIDを入れる"

# GUEST_ID=""
# GUEST_USER_ID=""

# TENANT_ID,TENANT_USER_ID は以下コマンドで取得可能です
#PGPASSWORD=refery psql -h localhost -p 5432 -d refery_development -U postgres -c "select id from users limit 1"
#PGPASSWORD=refery psql -h localhost -p 5432 -d refery_development -U postgres -c "select id from tenants limit 1"
```

GUESTは作らないといけないので、TENANT側で立ち上げて 連携医療機関→医療機関の招待ボタンから連携医療機関を作成してからデータベースでidを取得してください。（メールが送信されるとその時点で作成されます。）

GUEST側で実際に予約等を試したいときは、TENANT_USER_ID, TENANT_ID をコメントアウトして、代わりに値を入れた GUEST_ID, GUEST_USER_ID をアンコメントすればそのIDでログインできます。

詳しくは [認証の仕組みとローカル環境での挙動](https://app.notion.com/p/336d5ef02ee84a13bb4cfbb4122646bb?pvs=21) 

## サービスアカウント鍵ファイル配置

[credentials.json](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=ioosoduc6swt754oqprlbftcwi&i=bvundmcqty4c5od7p74rjrj2pq&h=medupinc.1password.com)を1Passwordから取得し、 `apps/api`直下に配置します。

### 各種アプリケーションの立ち上げ

api 配下、web 配下で それぞれ `npm run dev` すると良いです。

それぞれの port は以下

- api → 3001
- spa → 3003

[http://localhost:3003](http://localhost:3003) からログインします。ログイン時はstaging環境の各自のアカウントを利用するため、事前にstaging環境で招待をもらいアカウントを作成しておいてください。

### Swagger

apiを立ち上げて以下のURLにアクセスすると確認できる

[http://localhost:3001/docs/static/index.html](http://localhost:3001/docs/static/index.html)

内容はfasity-swaggerによって自動生成される