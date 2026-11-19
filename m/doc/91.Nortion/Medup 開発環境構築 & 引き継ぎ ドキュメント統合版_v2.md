# Medup 開発環境構築 & 引き継ぎ ドキュメント統合版

本ドキュメントは、各サービス（foroシリーズ・refery）のローカル環境構築手順の「完全版/追記版」をベースに、過去の個別手順書および開発時の問合せメモ（トラブルシューティング）の情報を一切損なうことなく、重複のみを整理して統合したものです。

---

## Ⅰ. medup-core-service / foro シリーズ ローカル開発環境構築

> 🎯 **この手順は、上から順に実行すれば途中で止まらずに最後まで通ります。**
> （※ `medup_ローカル環境構築_完全版` をベースに、各個別ドキュメントの 1Password/Notion/Figma リンクや、問合せメモの内容を統合しています）

### Step 0：始める前に揃えるもの（★ここで止まると後が全部止まります）
1. **GitHub の招待**（MedUpInc）: `git clone` が通るか確認。まだの場合は担当者へご一報ください。
   ※ パスワード認証は廃止されているため、`gh auth token | pbcopy` でトークンをコピーしてパスワードの代わりに入力することを推奨します。
2. 🔴 **Google グループ `dpc-coding-developers@medup.jp` への参加**: AI DPC を触るなら必須。Slack で依頼。
3. **1Password の該当 Vault へのアクセス**: `.env` 等の取得用。見えなければ @chihiro.naganuma へ。
4. 🔴 **seed データ2本**: Slack `#foro_crm_dev_seed`（旧名 `#foro_dev_seed`・ID は `C0406G1EKT2`）より取得。
    - `mis-seed-prd.sql`（医療機関マスタ: 約5GB。ダウンロードに時間がかかる）
    - `foro_crm_seed_foro-crm-api_dump_foro-crm-api.tar.gz`（foroの業務データ CSV 146本: 約6MB）
    - ※Slack に無ければ代替ルート: ①GCP Cloud Run ジョブ `dump-mis-seed-data` を手動実行（project: `medup-foro`）／②GCS `foro_development` バケットの `foro_crm_seed/foro-crm-api/dump_foro-crm-api.tar.gz`（project: `analytics-tool-206405`）
5. 🔴 **B2C（ステージング）のアカウント**: 画面のログインに必要。発行に時間がかかるため先に依頼しておく。
6. **Docker（OrbStack など）**: インストール済か確認。
    - ※ OrbStack起動時に `failed to start` (Permission denied) が出る場合、移行アシスタントの影響の可能性があります。`sudo chown -R $USER ~/Library/Group\ Containers/HUAQ24HBR6.dev.orbstack/data` を実行して再起動してください。
7. **Takumi Guard のトークン**: npmパッケージインストール用に必要。

### Step 1：コードを落とす
```bash
cd ~
git clone https://github.com/MedUpInc/medup-core-service.git
cd medup-core-service
```
✅ **既定ブランチは `develop` です**。
💡 **`git config` を `--global` で変えないでください。このリポジトリだけの設定にします。**
```bash
git config user.name "<氏名>"
git config user.email "<会社メール>"
```

### Step 2：Takumi Guard のトークンを通す
npm のパッケージは社内プロキシ（`npm.flatt.tech`）経由で取得します。
1. **メール認証トークンを発行**（手順: https://shisho.dev/docs/ja/t/guard/features/token-management/ ）
2. **`~/.zshrc`（または `~/.bashrc`）に1行追加**
```bash
export TAKUMI_GUARD_TOKEN=tg_anon_xxxxxxxxxxxxxxxx
```
3. **ターミナルを開き直して確認**
```bash
echo ${TAKUMI_GUARD_TOKEN:+設定OK}
```
⚠️ `401 Unauthorized` が出たら、トークンが違うか期限切れです。

### Step 3：node のバージョンを合わせる
`.node-version`（または `.nvmrc`）に書かれたバージョン（Node 22.22.x/22.23.x, npm 10.9.x）を使います。fnmを推奨します。
```bash
fnm install $(cat .node-version 2>/dev/null || cat .nvmrc)
fnm use
```
🔴 **【詰まりどころ】バージョンが違うと `package-lock.json` が書き換わります。**
`npm ci` の後に必ず確認: `git status package-lock.json` (何も出なければOK)。

### Step 4：依存パッケージのインストール
```bash
# 🔴 必ずリポジトリのルートで
cd ~/medup-core-service
npm ci
```
※ monorepo の管理として **Turborepo** (https://turborepo.org/) を使用しています。

### Step 5：`.env` と credentials を配置する
1Password から取得して、以下の場所に置きます。

#### A. 各種 `.env` の配置 (1Passwordリンクあり)
- **`apps/api/.env`**: [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=mmcfknezamzrlukhu62y5x3wuu&h=medupinc.1password.com)
- **`apps/foro-command-service/.env`**: [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=3c2hdla2lf5yspqfw7v7o2uv5y&h=medupinc.1password.com)
- **`apps/foro-query-service/.env`**: [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=5hc5iwjxbtdlpaw2z3a3s6iqgm&h=medupinc.1password.com)
- **`packages/foro-prisma/.env`**: [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=4p3bpocjnpduyk5ctjfbsrqd74&h=medupinc.1password.com)
- **`apps/foro-outpatient-web/.env`**: [1Password Link](https://my.1password.com/vaults/all/tags/lrl7iwhu56zycoke4xunetyshp/rxbgur66rl65rjlxbjpn3hvo4u) (見つからなければ `foro-outpatient-web` で検索)
- **`apps/foro-op-optimizer-web/.env`**: [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=fguepyelzznxkbh56gtoek47la&i=y6el7oewq7uoczknd7gpb34fui&h=medupinc.1password.com)
- **`apps/foro-job-worker/.env`**: `KMS_KEYRING_NAME` があるもの
- **`apps/admin-web/.env`**: `AUTH_USER_NAME` と `AUTH_PASSWORD` があるもの
- **`apps/acute-care-flow-web/.env`**: `NEXT_PUBLIC_B2C_CLIENT_ID` を `apps/api/.env` の `ACUTE_CARE_FLOW_B2C_CLIENT_ID` と突き合わせて確認。

#### B. credentials（JSON）の配置
- **`credentials.json`**: [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=i5vap3bmtwwo4x734wdmhw7pdu&h=medupinc.1password.com)
  → `apps/api/` と `apps/foro-command-service/` の両方に配置
- **`medup-foro-service-account-key.json`**: [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=3xxkfqkns3uljc6gxtiy72hwla&i=simkrow2ekkd37yvqadjer6x64&h=medupinc.1password.com)
  → `apps/foro-command-service/`、`apps/foro-job-worker/`、`apps/foro-query-service/` の3箇所に配置

※ 配置後、`git status` で0件（`.gitignore` 済み）であることを確認してください。

### Step 6：Docker ネットワークを作る
```bash
docker network create foro_common_link
docker network create medup_common_link
```
※ これを飛ばすと `docker compose up` が失敗します。

### Step 7：データベースを起動する
```bash
cd ~/medup-core-service
docker compose up -d
```

### Step 8：マイグレーション（テーブルを作る）
```bash
cd packages/foro-prisma
npm run db:migrate:deploy
```
※ `mis_dev` / `foro_dev` / `foro_test` / `data_ops_dev` が作成されます。本番DB(`foro_prd`)へのマイグレーションコマンドは絶対に実行しないでください。

### Step 9：seed の投入（医療機関データベース）
🔴 **5GB のファイルが git に入らないよう先に除外してください：**
```bash
cat >> .git/info/exclude <<'X'
mis-seed-prd.sql
foro_crm_seed_*.tar.gz
out/
X
```
**投入コマンド:**
```bash
cd ~/medup-core-service
psql "postgres://postgres:rootpassword@localhost:5432/mis_dev" \
  -c "SET session_replication_role = replica;" \
  -f mis-seed-prd.sql \
  -c "SET session_replication_role = DEFAULT;"
```

### Step 10：seed の投入（foro データベース）
```bash
cd ~/medup-core-service
tar -zxf foro_crm_seed_foro-crm-api_dump_foro-crm-api.tar.gz

export PGPASSWORD=rootpassword
outputPath="./out/foro-crm-seed/"

for file in $outputPath/*.csv; do
    tableName=$(basename "$file" .csv)
    psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "SET session_replication_role = replica;" -c "\COPY $tableName FROM '$file' WITH CSV HEADER;" -c "SET session_replication_role = DEFAULT;"
    psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "SELECT setval('${tableName}_id_seq', (SELECT max(id) FROM $tableName));"
done
```

### Step 11：ログインユーザーを作る
認証情報は1passwordに登録されています。
```bash
PGPASSWORD=rootpassword psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev \
  -c "insert into users (email, username, aad_user_id, tenant_id, encrypted_password, created_at, updated_at) values ('local-user@medup.jp', 'medup', '<1Password参照>', 36, '<1Password参照>', now(), now());"
```
🔴 **作成された `id` を必ず控えてください。**
```bash
docker exec medup-foro-db psql -U postgres -d foro_dev -tAc "SELECT id, email, tenant_id FROM users WHERE email='local-user@medup.jp';"
```
※ `rls_user` の作成・権限追加のスクリプトは初回 docker compose 起動時に自動で実行されています。

### Step 12：Prisma client を生成する
```bash
cd packages/foro-prisma
npm run db:client:gen
```

### Step 13：🔴 `USER_ID` を自分の環境に合わせる（必須）
配布された `.env` の `USER_ID` を、Step 11 で控えた `id` に置き換えます。
```bash
cd ~/medup-core-service
sed -i '' 's/^USER_ID=.*/USER_ID=<Step11のid>/' apps/foro-query-service/.env
sed -i '' 's/^USER_ID=.*/USER_ID=<Step11のid>/' apps/foro-command-service/.env
```
※ 【問合せメモ】`foro-outpatient-web` でログイン時にリダイレクトループになる原因は、`foro-query-service` の `.env` で設定している `USER_ID` がローカル DB に存在しないため（ログイン後のユーザー取得が 401 になるため）です。必ず自身のローカルDBのIDと合わせてください。

### Step 14：動作確認（API）と画面・バックエンドの起動
```bash
cd ~/medup-core-service
# 🔴 api（:8888）を先頭に置く＝:3001 が依存するため。同時起動の競合を避けるためsleepを入れる。
for s in api foro-query-service foro-command-service \
         foro-outpatient-web foro-op-optimizer-web \
         foro-crm-web acute-care-flow-web; do
  (cd "apps/$s" && nohup npm run dev > "/tmp/svc_$s.log" 2>&1 &)
  sleep 12
done
```
※ API仕様書の確認: `curl localhost:8888/docs/yaml`
※ Turborepoによる一括起動 (`npm run dev` または `turbo run dev`) も可能ですが、上記のように順次起動しないと `foro-query-service` が LISTEN しない競合リスクがあります。

### Step 15：foro-crm-api（別リポジトリ）のセットアップ
```bash
cd ~
git clone https://github.com/MedUpInc/foro-crm-api.git
cd foro-crm-api
```
**1Passwordから以下を取得して配置：**
- **`master.key`**: [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=fguepyelzznxkbh56gtoek47la&i=7iiibqv5vq667mpy3xuvohcda4&h=medupinc.1password.com) → `config/master.key`
- **`foro-crm-api-service-account-key-dev.json`**: [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=fguepyelzznxkbh56gtoek47la&i=275akpmfs6bb46cmcjik3hnhie&h=medupinc.1password.com) → リポジトリ直下
- **`.env`**: [1Password Link](https://medupinc.1password.com/app#/ZQUJLE6YSBH2PHXHDFLDCLV6N4/Vault/ZQUJLE6YSBH2PHXHDFLDCLV6N4:3xxkfqkns3uljc6gxtiy72hwla:6q7tzhes2melskppjslhgwgez4?itemListId=ZQUJLE6YSBH2PHXHDFLDCLV6N4%3A3xxkfqkns3uljc6gxtiy72hwla) → リポジトリ直下

**🔴 `USER_ID` を書き換える:**
`docker-compose.yml` と `docker-compose.spa.yml` 内の `USER_ID` を書き換えます。
【問合せメモより】既存ユーザーのIDを使うか、新規作成するか選べます（既定の 2448 はシードに無いのでログインできません）。
```bash
# 既存ユーザー確認
docker compose exec mt_web rails db -p
SELECT id, username, email FROM users WHERE status_id = 1 AND enabled_crm ORDER BY id;

# 新規作成コマンド例（作成された数字をUSER_IDへ）
docker compose exec mt_web bin/rails runner 'u = User.create!(tenant_id: 36, username: "ローカル", email: "local-yourname@medup.jp"); puts u.id'

# 書き換え実行
sed -i '' 's/USER_ID=[0-9]*/USER_ID=<確認したid>/' docker-compose.yml docker-compose.spa.yml
```

**起動:**
```bash
# 単体でよい場合（推奨・軽い。mhlw_api / foro-referral-api はステージングを見に行く）
docker compose -f docker-compose.spa.yml up -d
```
Swagger UI は `http://localhost:8080` (ログイン不要) です。

### Step 16：画面へのログイン（B2C）と権限付与
`localhost:3004` (外来最適化) / `localhost:3005` (手術枠最適化) にアクセスし、Azure AD B2Cのステージングアカウントでログインします。

🔴 **ログインは通るのに「アクセス権限が必要です」で止まる場合（:3005 / :3006）**
ローカル DB の権限フラグが既定で `false` なためです。以下の SQL でフラグを立ててください。
```bash
docker exec -i medup-foro-db psql -U postgres -d foro_dev <<'SQL'
BEGIN;
UPDATE users SET
  enabled_op_optimizer = true,
  enabled_op_optimizer_data_import = true,
  enabled_acute_care_flow = true,
  enabled_acute_care_flow_social_information = true
WHERE id = <あなたの USER_ID>;
COMMIT;
SQL
```

### 参考資料リンク集
- **開発 Tips・構成・認証**: 
  - tRPCを採用しているため、Swagger等のスキーマファイル生成は不要です。
  - リポジトリ構成 (Figma): [https://www.figma.com/file/RqQ6pyy0o0l5DY7m7N3SSX/%...](https://www.figma.com/file/RqQ6pyy0o0l5DY7m7N3SSX/%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA%E6%A7%8B%E6%88%90?type=whiteboard&node-id=0-1&t=i1YWDbqxPpdTuPkU-0)
  - 認証周り (Figma): [https://www.figma.com/file/KeW8bsofBhMpuPu0lOz3JD/%...](https://www.figma.com/file/KeW8bsofBhMpuPu0lOz3JD/%E8%AA%8D%E8%A8%BC%E5%91%A8%E3%82%8A%E3%81%AE%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF?type=whiteboard&node-id=0-1&t=vbUw3uaKPrd644nM-0)
- **マイグレーションのお作法**: [マイグレーションファイルの作り方](https://app.notion.com/p/092e5c051dd14c39a816abef96d6b6a2?pvs=21) / [データベースマイグレーション手順&接続方法](https://app.notion.com/p/6aaf357996ee491aa7fee0e645671921?pvs=21)

---

## Ⅱ. refery ローカル開発環境構築

refery は monorepo 構成になっており、api, web の両方が同じリポジトリ上で管理されています。
- `apps/` : アプリケーション一覧 (api, web)
- `config/` : ミドルウェアの設定ファイル群
- `packages/` : 各種アプリケーションで利用する独自ライブラリ

### Step 1：コードの取得と git 設定
```bash
# 🔴 パスワード認証は廃止されているため、gh コマンドでトークンをコピーして利用します。
gh auth token | pbcopy 

cd ~
git clone https://github.com/MedUpInc/refery.git
```
※ Password を聞かれたら ⌘V で貼り付けます。
※ `gh auth setup-git` はグローバルのgitconfigを書き換えてしまうため使用しないでください。

```bash
cd refery
# 🔴 大文字・小文字を区別させる (必ずリポジトリローカルで実行)
git config core.ignorecase false
# 🔴 このリポジトリ用の名前とメールを設定
git config user.name "氏名"
git config user.email "会社メールアドレス"
```

### Step 2：node バージョンの設定とインストール
refery の `.node-version` は `22.23.1` です。**fnm** を使用します。
他のプロジェクト（例: v25系）とバージョンが競合しないよう、`~/.zshrc` に `recursive` 設定を追加します。

```bash
# fnm を有効化（~/.zshrc への追記例）
# eval "$(fnm env --use-on-cd --version-file-strategy recursive --shell zsh)"

cd ~/refery
fnm install 22.23.1
npm ci
```
※ `npm ci` は必ずリポジトリルートで実行。

### Step 3：Docker (PostgreSQL) の起動
```bash
cd ~/refery
docker compose up -d
```
🔴 **ポートの競合に注意:** refery-db は `5432` を使用します。medup-core-service と重なるため、**同時に起動できません。** 先に `cd ~/medup-core-service && docker compose down` を実行して停止してください。
※ OrbStack利用時、`failed to start` のエラーが出た場合は移行アシスタントの権限エラーです。`sudo chown -R $USER ~/Library/Group\ Containers/HUAQ24HBR6.dev.orbstack/data` で所有者を修正してください。

### Step 4：環境変数と credentials の配置
`apps/api/.env` と `apps/web/.env` を作成します。

1. **api側のシークレット取得 (D-2 相当)**: 
   GCS の `refery-staging/api/.env` から `B2C_CLIENT_SECRET` と `SENDGRID_API_KEY` の値だけを取得し、ローカルの `.env` に追記します。
   ⚠️ **GCSにある `web/.env` はステージングを向いているため、ローカル構築では絶対に使用しないでください。**
2. **credentials の配置**:
   `credentials.json` は 1Password（`refery Dev.` → `credentials.json:refery-api-stg`）からダウンロードし、`apps/api/credentials.json` に配置します。配置後はダウンロードフォルダから削除してください。
   [1Password Link](https://start.1password.com/open/i?a=ZQUJLE6YSBH2PHXHDFLDCLV6N4&v=ioosoduc6swt754oqprlbftcwi&i=bvundmcqty4c5od7p74rjrj2pq&h=medupinc.1password.com)

### Step 5：データベースの作成と初期データ投入
ローカルに `psql` コマンドが無い場合を考慮し、コンテナ（db）経由で実行します。

```bash
cd ~/refery/apps/api
npm run db:migrate:dev

# tenant と user の初期データ投入
cd ~/refery
docker compose exec -T db psql -U postgres -d refery_development -c \
"insert into tenants (id, name, postal_code, prefecture, city, street, phone_number, days_before_instant_reservation, days_before_request_reservation, created_at, updated_at) values (gen_random_uuid(), 'medup', '101-0061', '東京都', '千代田区', 'テスト住所', '00-0000-0000', 3, 3, now(), now());"

docker compose exec -T db psql -U postgres -d refery_development -c \
"insert into users (id, display_name, tenant_id, email_address, user_name, role, created_at, updated_at) values (gen_random_uuid(), 'refery', (select id from tenants limit 1), null, 'refery-dev', 'admin', now(), now());"
```

生成されたUUIDを確認し、`apps/api/.env` の `TENANT_ID` と `TENANT_USER_ID` に書き戻します。
```bash
docker compose exec -T db psql -U postgres -d refery_development -c "select id from tenants limit 1"
docker compose exec -T db psql -U postgres -d refery_development -c "select id from users limit 1"
```

#### 🔴 ゲストアカウントの作成について
GUEST側で予約等を試したい場合は、GUESTアカウントが必要です。
以下のSQLをコンテナ経由で実行し、出力された値を `GUEST_ID` と `GUEST_USER_ID` に設定します。
```bash
docker compose exec -T db psql -U postgres -d refery_development <<'SQL'
WITH g AS (
  INSERT INTO guests (id, name, created_at, updated_at)
  VALUES (gen_random_uuid(), 'テスト連携先クリニック', now(), now())
  RETURNING id
), u AS (
  INSERT INTO users (id, display_name, guest_id, email_address, user_name, created_at, updated_at)
  SELECT gen_random_uuid(), 'ゲスト太郎', id, 'guest-user@example.com', 'guest-dev', now(), now() FROM g
  RETURNING id, guest_id
), r AS (
  INSERT INTO referrers (id, tenant_id, guest_id, name, departments, created_at, updated_at)
  SELECT gen_random_uuid(), (SELECT id FROM tenants LIMIT 1), id, 'テスト連携先クリニック', '{}', now(), now() FROM g
)
SELECT u.guest_id AS "GUEST_ID", u.id AS "GUEST_USER_ID" FROM u;
SQL
```

### Step 6：起動と確認
🔴 **【重要】api の `npm run dev` は、起動のたびに `docs/swagger.yaml` が書き換わり再起動ループに陥ります。以下のコマンドで watch を無効にして起動してください。**

```bash
# ターミナル1（API）
cd ~/refery/apps/api
NODE_ENV=development node --env-file=.env -r @swc-node/register -r tsconfig-paths/register src/server.ts

# ターミナル2（WEB）
cd ~/refery/apps/web
npm run dev
```

- **画面**: `http://localhost:3003` (B2C ステージング環境のアカウントでログイン)
- **API仕様書**: `http://localhost:3001/docs/` 
  ⚠️ `/docs/static/index.html` ではありません。Swaggerの画面が真っ白な場合は、⌘Rでリロードすると表示されます。

### トラブルシューティング (refery)
- **`npm ci` が `401 Unauthorized`**: `TAKUMI_GUARD_TOKEN` が未設定です。メール認証で取得し `.zshrc` に設定してください。
- **Swagger が再起動ループする**: `npm run dev` ではなく `node` コマンドで直接 `src/server.ts` を起動してください（Step 6 参照）。
- **画面が staging を見ている**: GCSの `web/.env` を誤って使用しています。ローカル用の `NEXT_PUBLIC_API_HOST="http://localhost:3001"` などを向くように再作成してください。
- **別のディレクトリで node のバージョンが戻らない**: `fnm` の設定で `--version-file-strategy recursive` が不足している可能性があります。

---

## Ⅲ. 吉田(享平)さん（業務委託）への引き継ぎ実施

**ステータス:** 有効（周知済み）
**担当:** Jun.Yamada

### 背景
2~5月の4ヶ月で採用予定の業務委託メンバー2名が、メダップで運用しているSaaSサービス全体の全体像を掴み、サービスの安定運用に向けて稼働できる状態にする。AI-DPCは当面深く関与することは無い予定だが、保守業務で関与することは考えられるため概要理解までは進めておく。

### ToBe
- 2026/11：新保さんがai-dpcへ軸足を移し始める
- 2026/12末時点：馬場、黒川の契約終了
- 2026/12末時点：自社クラウドSaaSサービスの保守・メンテナンスを2名でできる（機能改善、定期実施作業、手作業対応業務、セキュリティチェックシート対応）

### カリキュラム（週次）と関連ドキュメントリンク群

#### 1. 11/2週：開発環境セットアップ
- **タスク**:
  - [foroCRMチーム参画初回キックオフ：吉田(享)さん](https://app.notion.com/p/foroCRM-3fb1da5a7f3b80e88c98f9852bf7b126?pvs=21)
  - [環境構築 ~ for MacOS ~](https://app.notion.com/p/for-MacOS-703c86d7614a461582f0fe67150b42ee?pvs=21)（foro）
  - [ローカル開発環境セットアップ ~ for Mac ~](https://app.notion.com/p/for-Mac-3ab7572708fa4143a2ceeeda20dcf89f?pvs=21)（refery）
- **座学**:
  - [202504_foro CRMの基本的な使い方まとめ_v1.0](https://docs.google.com/presentation/d/1oMkJAUGEOHT0-brEdh-EaORNK84EK83gmzifkNR3n58/edit?slide=id.g33e7074b878_0_2461#slide=id.g33e7074b878_0_2461)
  - [フロントエンド開発のお約束](https://app.notion.com/p/de5fbf8d96f6489e9fdbc5ab931ab168?pvs=21)
  - [foro CRM 構成管理](https://app.notion.com/p/foro-CRM-4b6f572bca5549c9880dbefa130a2665?pvs=21)
  - [foro バックエンドAPI実装のインターフェース](https://app.notion.com/p/foro-API-731fc5011389435fbfb4cb629a4c1583?pvs=21)
  - 各種レポートの仕組み・データ構造: [各種レポートの仕組み、運用](https://app.notion.com/p/78b7bb9468064144b4ae352a53c59887?pvs=21)
  - ラベルの仕組み: [ラベルの仕組み](https://app.notion.com/p/f3565b81bae949138b5b371110d8af8f?pvs=21)
  - [wiki](https://app.notion.com/p/wiki-fbf9466ee4f041f4b1664135200e5cd0?pvs=21)
  - [🚧 メダップ流?! オニオン Functional DDD 🚧](https://app.notion.com/p/Functional-DDD-5ef678ec18cb4f95b52dfebd54f57321?pvs=21)
  - [将来的なアーキテクチャに向けた方針決め](https://app.notion.com/p/252bb2c3cdbb411d9a6cd1307898cec2?pvs=21)
  - [対応環境、ブラウザ](https://app.notion.com/p/1aa24b23027846ec9d9b4b08e9e54a93?pvs=21)
  - [foroシリーズ開発の歴史的経緯](https://app.notion.com/p/foro-2ca1da5a7f3b809f89c5eeec1b82ced8?pvs=21)
- **環境URL一覧**:
  - foro CRM 本番: https://foro-crm.com / staging: https://stg-foro-crm.com
  - foro 管理ツール 本番: https://admin-web.medup-core-service.com/login / staging: https://admin-web.medup-core-service-staging.com/
  - foro 連携マップ 本番: https://outpatient.medup-foro.com/ / staging: https://outpatient.medup-foro-stg.com/
  - refery 本番: https://refery-app.com/ / staging: https://refery-app-dev.com/
- **接続スクリプト (cloud-sql-proxy / AlloyDB SSH トンネル)**: `吉田(享平)さん（業務委託）への引き継ぎ実施.md` 原文内のスクリプトを使用のこと。

#### 2. 11/9週：foro CRM(1)
- **タスク**: [レポートに項目「特記事項」を追加する](https://app.notion.com/p/33d1da5a7f3b8094b339ee7270e6f14e?pvs=21)

#### 3. 11/16週：foro 連携マップ・foro逆紹介・foro 管理ツール
- **座学**:
  - [foro 連携マップ構成管理](https://app.notion.com/p/foro-1ad1da5a7f3b80b1b68ddca44d1b8b2d?pvs=21)
  - [foro 逆紹介管理（外来患者分析）](https://app.notion.com/p/foro-1d81da5a7f3b8072991df87981fd21fe?pvs=21)
  - [foro 逆紹介管理構成管理](https://app.notion.com/p/foro-2031da5a7f3b80c2893bcdff56bc45e6?pvs=21)
  - [※ ユーザー側で機能が利用不可の場合、障害対応フローに乗せましょう！](https://app.notion.com/p/31e4041e411748d5b9f772cce850ed8a?pvs=21)

#### 4. 11/23週：data-ops運用について/ナビイスクレイパー
- **タスク**:
  - [RailsバージョンUP](https://app.notion.com/p/Rails-EOL-40c1da5a7f3b804c87d0f49fc40435b5?pvs=21)
  - 外来適正化データ抽出 / MISマスタ月次更新
- **座学**:
  - 様式1: https://www.mhlw.go.jp/file/05-Shingikai-12404000-Hokenkyoku-Iryouka/0000027946.pdf
  - [MISの更新フロー〜厚生局の医科歯科編〜](https://app.notion.com/p/MIS-1821da5a7f3b805fba57efa6396f362a?pvs=21)
  - [【厚生局の届出情報追加】新規開院・移転/移設、交代/継承](https://app.notion.com/p/17c1da5a7f3b806c8fe7e66029878e8e?pvs=21)
  - [ナビイデータスクレイピングについて](https://app.notion.com/p/2ce1da5a7f3b80b2807cdfaf7576d3a2?pvs=21)
  - [CloudTasks CloudSchedulerによる非同期処理構成について](https://app.notion.com/p/CloudTasks-CloudScheduler-d357b534e8404d42b89b4b322812db1e?pvs=21)
  - [各レポートでのデータ構造・実装の違い（202601時点）](https://app.notion.com/p/202601-2f81da5a7f3b8007acabeba23f3c367b?pvs=21)

#### 5. 11/30週：refery
- **座学**:
  - GitHub: https://github.com/MedUpInc/refery
  - [refery 構成管理](https://app.notion.com/p/refery-00a1a567d58f474a8bced938fb23cefb?pvs=21)
  - [テナントデータ削除手順](https://app.notion.com/p/20a1da5a7f3b80b29a3dd3c387ceebfb?pvs=21)
  - [20250527_referyリクエスト予約ステータス整理](https://app.notion.com/p/20250527_refery-2001da5a7f3b80dea630fb6dada575db?pvs=21)
  - [refery非同期バッチジョブ一覧](https://app.notion.com/p/refery-e45eeeb10541473a996eda1c4f17d716?pvs=21)
  - [04.refery](https://app.notion.com/p/04-refery-3affae5e901e46d0b07fabb55c64ed97?pvs=21)

#### 6. 12/6週：foro CRM(2)
- **マイルストーン**: ama対応の一次受け対応おまかせ開始
- **座学**:
  - [foroシリーズ非同期バッチジョブ一覧](https://app.notion.com/p/foro-971bdb41f6b9446e8e8746e2946d6dcd?pvs=21)
  - [hotfix適用フロー](https://app.notion.com/p/hotfix-2fb1da5a7f3b8026a561ed396f189d76?pvs=21)
  - [カスタムダッシュボードにforoのデータを反映する仕組み（Datastream@medup-foro）](https://app.notion.com/p/foro-Datastream-medup-foro-252c8735ee724013a46a9dbbf6dde720?pvs=21)

#### 7. 12/13週：data-management-api
- **座学**:
  - [Data Management API（データの前処理API）](https://app.notion.com/p/Data-Management-API-API-7f5fba9e858d48508f3439e4135b8099?pvs=21)
  - [データマネジメント・管理ツール構成管理](https://app.notion.com/p/24e1da5a7f3b80f28590c8ad53b5d53d?pvs=21)
  - [IPアドレス制限をかける必要がある場合の対応について](https://app.notion.com/p/IP-25b1da5a7f3b8049ba05fda9e7b876eb?pvs=21)

### 大項目・業務内容
- **自社クラウドサービス**: foro CRM, 連携マップ, 外来患者分析, 管理ツール, 入退院調整, data-management-api, data-integration, refery, AI DPCコーディング, ナビイスクレイパー
- **定期業務**: AADB2Cシークレット更新, ドメイン更新, ユーザー環境アカウント棚卸, [DPCコードマスタをDBに取り込む手順](https://app.notion.com/p/DPC-DB-b0314118beeb4b94886f680930d7ad1a?pvs=21), システム復旧テスト実施

### 12/1 時点で吉田さんが担えていること（一部山田、新保フォロー）
- **開発系**: 簡単な改修、QA対応(ama回答)、障害調査、セキュリティ対応(dependabot)、コードレビュー、リリース
- **Data Ops系**: MISマスタ月次更新、外来適正化月次データ抽出、MISマスタ重複対応
- **AI-DPC**: 概要理解