# ローカル環境 起動・デバッグ手順（foroシリーズ）

本ドキュメントは、foroシリーズのDBが空の状態からの「初回セットアップ」と、構築完了後の「日々の起動・デバッグ手順」をまとめたものです。用途に応じて該当するセクションを参照してください。

---

## Ⅰ. 既存foro環境の停止

foroシリーズ内の別サービスから切り替える場合、起動中のNodeプロセスとコアDBコンテナを停止します。

バックグラウンドで起動しているNodeプロセス（`npm run dev`等）を停止します。

~~~bash
pkill -f 'npm run dev'
~~~

コアサービスのリポジトリルートでDockerコンテナを停止します。

~~~bash
cd ~/Desktop/work/medup-core-service
~~~

~~~bash
docker compose down
~~~

---

## Ⅱ. 【メインシステム】 foro シリーズ

### Ⅱ-1. 初回セットアップ（DB構築・データ投入）

DBが空の状態から構築する場合のみ実行します。既存DBでforo 管理ツールのテナント一覧が `TenantDataAvailabilityがありません` で失敗する場合は「Ⅱ-2」を実行し、それ以外は「Ⅱ-3」へ進んでください。

#### 1. DB起動とマイグレーション

リポジトリルートへ移動します。

~~~bash
cd ~/Desktop/work/medup-core-service
~~~

コアサービスのデータベースコンテナを起動します。

~~~bash
docker compose up -d
~~~

Prismaのディレクトリへ移動します。

~~~bash
cd packages/foro-prisma
~~~

テーブル構造を作成します。

~~~bash
npm run db:migrate:deploy
~~~

Prisma Clientを生成します。

~~~bash
npm run db:client:gen
~~~

#### 2. seedデータの投入

リポジトリルートへ戻ります。

~~~bash
cd ~/Desktop/work/medup-core-service
~~~

医療機関マスタ（約5GB）をDBへ流し込みます。

~~~bash
psql "postgres://postgres:rootpassword@localhost:5432/mis_dev" -c "SET session_replication_role = replica;" -f mis-seed-prd.sql -c "SET session_replication_role = DEFAULT;"
~~~

foro業務データのCSV群を解凍します。

~~~bash
tar -zxf foro_crm_seed_foro-crm-api_dump_foro-crm-api.tar.gz
~~~

DB接続用のパスワードを環境変数にセットします。

~~~bash
export PGPASSWORD=rootpassword
~~~

CSVフォルダのパスを変数にセットします。

~~~bash
outputPath="./out/foro-crm-seed/"
~~~

各CSVファイルをDBへインポートし、シーケンスを更新します（※ここは1コマンドとして実行します）。

~~~bash
for file in $outputPath/*.csv; do
    tableName=$(basename "$file" .csv)
    psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "SET session_replication_role = replica;" -c "\COPY $tableName FROM '$file' WITH CSV HEADER;" -c "SET session_replication_role = DEFAULT;"
    psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "SELECT setval('${tableName}_id_seq', (SELECT max(id) FROM $tableName));"
done
~~~

foro用CSVには `tenant_data_availabilities.csv` も含まれますが、一部テナントの行がありません。CSV投入後、「Ⅱ-2. foro 管理ツール のテナントデータ補正」を実行してください。

#### 3. ログインユーザーの作成と権限付与

DBにログイン用のユーザーを新規作成し、IDを取得します。`<1Password参照>` の部分は実際の値に置き換えて実行し、出力された **ID（数字）** を控えてください。

~~~bash
PGPASSWORD=rootpassword psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "insert into users (email, username, aad_user_id, tenant_id, encrypted_password, created_at, updated_at) values ('local-user@medup.jp', 'medup', '<1Password参照>', 36, '<1Password参照>', now(), now()) RETURNING id;"
~~~

取得したIDに対して、各システムの権限フラグを付与します（`<取得したID>`を先ほどの数字に書き換えてください）。

~~~bash
PGPASSWORD=rootpassword psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "UPDATE users SET enabled_op_optimizer = true, enabled_op_optimizer_data_import = true, enabled_acute_care_flow = true, enabled_acute_care_flow_social_information = true WHERE id = <取得したID>;"
~~~

#### 4. USER_ID の書き換え

各サービスの `.env` および `docker-compose` の `USER_ID` を、先ほど取得したIDに一括で書き換えます。`<取得したID>`を数字に書き換えて順に実行してください。

~~~bash
sed -i '' 's/^USER_ID=.*/USER_ID=<取得したID>/' apps/api/.env
~~~

~~~bash
sed -i '' 's/^USER_ID=.*/USER_ID=<取得したID>/' apps/foro-query-service/.env
~~~

~~~bash
sed -i '' 's/^USER_ID=.*/USER_ID=<取得したID>/' apps/foro-command-service/.env
~~~

~~~bash
cd ~/Desktop/work/foro-crm-api
~~~

~~~bash
sed -i '' 's/USER_ID=[0-9]*/USER_ID=<取得したID>/' docker-compose.yml docker-compose.spa.yml .env
~~~

#### 5. admin-web（foro 管理ツール）の初回設定

foro 管理ツールを利用する場合のみ、初回に `apps/admin-web/.env` を設定し、ログインに使うGoogleアカウントを `foro_dev.medup_foro_admins` に登録します。OAuthの値はチームで管理している開発用のものを使用してください。

~~~dotenv
AUTH_USER_NAME=ローカルBasic認証ユーザー名
AUTH_PASSWORD=ローカルBasic認証パスワード
NEXT_PUBLIC_GOOGLE_CLIENT_ID=開発用Google OAuthクライアントID
GOOGLE_CLIENT_SECRET=開発用Google OAuthクライアントシークレット
NEXT_PUBLIC_API_HOST=http://localhost:8888
~~~

ログインに使うGoogleアカウントが `medup_foro_admins` に未登録の場合、リポジトリルートでメールアドレスを置き換えて実行します。

~~~bash
PGPASSWORD=rootpassword psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev -c "INSERT INTO medup_foro_admins (id, username, email, enabled_data_ops, enabled_mis, enabled_foro_crm, created_at, updated_at) VALUES (gen_random_uuid(), 'local-admin', '<Googleアカウントのメールアドレス>', true, true, true, now(), now()) RETURNING id, email;"
~~~

`.env` はGit管理対象外です。設定を変更した場合はadmin-webを再起動してください。

---

### Ⅱ-2. foro 管理ツール のテナントデータ補正

`foro_dev` にシードデータを投入した後、または既存のローカルDBで foro 管理ツール のテナント一覧が `TenantDataAvailabilityがありません` で失敗した場合に実行します。`tenant_data_availabilities` に存在しないテナント分だけ、各データ種別を未設定（`false`）として登録します。既存レコードは変更しないため、再実行しても安全です。本番DBでは実行しないでください。

~~~
PGPASSWORD=rootpassword psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev <<'SQL'
INSERT INTO tenant_data_availabilities (
    tenant_id,
    referral,
    reverse_referral,
    ff1_file,
    d_file,
    hospital_admission_ef_file,
    outpatient_ef_file,
    created_at,
    updated_at
)
SELECT
    t.id,
    false,
    false,
    false,
    false,
    false,
    false,
    CURRENT_TIMESTAMP,
    CURRENT_TIMESTAMP
FROM tenants AS t
ON CONFLICT (tenant_id) DO NOTHING;
SQL
~~~

### Ⅱ-3. 日常の起動手順

初回セットアップと環境設定が完了している環境で、必要なサービスを起動します。

#### 1. 共通DB・APIコンテナの起動

コアサービスのリポジトリルートへ移動します。

~~~bash
cd ~/Desktop/work/medup-core-service
~~~

コアDBコンテナを起動します。

~~~bash
docker compose up -d
~~~

CRM APIのディレクトリへ移動します。

~~~bash
cd ~/Desktop/work/foro-crm-api
~~~

500エラーを回避するため、CRM APIは標準のymlで起動します。

~~~bash
docker compose -f docker-compose.yml up -d mt_web
~~~

#### 2. コアAPI群のバックグラウンド起動

コアサービスのリポジトリルートへ戻ります。

~~~bash
cd ~/Desktop/work/medup-core-service
~~~

CRM APIコンテナから呼ばれるCore API（8888番）をバックグラウンド起動します。

~~~bash
nohup npm run dev -w apps/api > /tmp/svc_api.log 2>&1 < /dev/null &
~~~

起動ログに `Server listening at ...:8888` が出ることを確認します（10秒ほど待ちます）。

~~~bash
tail -n 40 /tmp/svc_api.log
~~~

8888番で待ち受けていることを確認します。

~~~bash
lsof -nP -iTCP:8888 -sTCP:LISTEN
~~~

CRM APIコンテナからCore APIへ接続できることを確認します。

~~~bash
docker exec foro-crm-api-mt_web-1 curl --fail --silent --show-error --output /dev/null --write-out 'HTTP %{http_code}\n' http://host.docker.internal:8888/docs
~~~

foro-query-service（8889番）を起動します。

~~~bash
nohup npm run dev -w apps/foro-query-service > /tmp/svc_foro-query-service.log 2>&1 < /dev/null &
~~~

foro-command-service（8890番）を起動します。

~~~bash
nohup npm run dev -w apps/foro-command-service > /tmp/svc_foro-command-service.log 2>&1 < /dev/null &
~~~

#### 3. 各Web画面の起動

利用する画面だけ起動します。admin-webとforo CRMはVS Codeの **Run and Debug** からF5で起動し、その他はリポジトリルートでコマンドを実行します。

| 画面 | 起動方法 | URL | Project |
|---|---|---|---|
| foro CRM | `Next.js: Server (バックエンド/SSR起動)` を選択してF5 | http://localhost:3001 | foro-crm-web foro-crm-api |
| foro 管理ツール | `Next.js: admin-web` を選択してF5。Basic認証後にGoogleログイン | http://localhost:3002 | admin-web |
| foro 連携マップ | 下記コマンドを実行 | http://localhost:3004 | foro-outpatient-web |
| foro 逆照会管理 | 下記コマンドを実行 | http://localhost:3005 | foro-op-optimizer-web |
| foro 入退院調整 | 下記コマンドを実行 | http://localhost:3006 | acute-care-flow-web |

~~~bash
nohup npm run dev -w apps/foro-outpatient-web > /tmp/svc_foro-outpatient-web.log 2>&1 < /dev/null &
~~~

~~~bash
nohup npm run dev -w apps/foro-op-optimizer-web > /tmp/svc_foro-op-optimizer-web.log 2>&1 < /dev/null &
~~~

~~~bash
nohup npm run dev -w apps/acute-care-flow-web > /tmp/svc_acute-care-flow-web.log 2>&1 < /dev/null &
~~~

---

## Ⅲ. ログ確認とプロセスの停止

**ログの確認**
各サービスの起動ログは `/tmp/svc_<サービス名>.log` に保存されています。

~~~bash
tail -n 80 /tmp/svc_api.log
~~~

**プロセスの一括停止**
Web画面などのNode開発プロセスを一括で終了します。

~~~bash
pkill -f 'npm run dev'
~~~

