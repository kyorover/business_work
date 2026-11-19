# medup_ローカル環境構築_完全版

> 🎯 **この手順は、上から順に実行すれば途中で止まらずに最後まで通ります。** ★**2026-09-24 に、入社直後の人間が実際に最初から通して検証しました**（所要＝正味約40分。seed のダウンロード待ちを除く）。 ⚠️ **既存の「medup core service 編」「foro-crm-api 編」「foro-outpatient-web 編」「foro-op-optimizer編」を1本に統合し、詰まった箇所11件を修正・追記したものです。**
> 

---

## **📋 Step 0：始める前に揃えるもの（★ここで止まると後が全部止まります）**

| # | 要るもの | 確認方法 | 無いとき |
| --- | --- | --- | --- |
| **1** | **GitHub の招待**（MedUpInc） | `git clone` が通るか | **担当者へ依頼** |
| **2** | 🔴**Google グループ `dpc-coding-developers@medup.jp` への参加** | — | **AI DPC を触るなら必須。Slack で依頼** |
| **3** | **1Password の該当 Vault へのアクセス** | **`.env` が見えるか** | **@chihiro.naganuma へ** |
| **4** | 🔴**seed データ2本**（下記） | — | 🔴**Slack `#foro_crm_dev_seed`**（★旧名 `#foro_dev_seed`・ID は同じ） |
| **5** | 🔴**B2C（ステージング）のアカウント** | — | 🔴🔴**画面のログインに必要。先に依頼しておく**（発行に時間がかかる） |
| **6** | **Docker（OrbStack など）** | `docker info` | **インストール** |
| **7** | **Takumi Guard のトークン** | `echo $TAKUMI_GUARD_TOKEN` | **下記 Step 2** |

> 🔴🔴 **【いちばん大事】Step 0 は着手の前日にやってください。** ★**上の1・2・5は発行に時間がかかります**（実測 2026-09-24＝グループへの追加まで約50分／B2C アカウントは当日中に返事が来なかった）。 ⚠️ **最後に気づくと、そこで半日止まります。**
> 

### **🔴 seed データ2本（先に落としておく）**

| ファイル名（★この名前で保存） | 中身 | 目安 |
| --- | --- | --- |
| **`mis-seed-prd.sql`** | **医療機関マスタ** | 🔴**約 5GB。ダウンロードに時間がかかる** |
| **`foro_crm_seed_foro-crm-api_dump_foro-crm-api.tar.gz`** | **foro の業務データ（CSV 146本）** | **約 6MB** |

⚠️🔴 **`#foro_dev_seed` は `#foro_crm_dev_seed` にリネームされています**（★チャンネル ID `C0406G1EKT2` は同じ）。 ✅ **Slack に無ければ代替ルートが2つ**＝**①GCP Cloud Run ジョブ `dump-mis-seed-data` を手動実行（project: `medup-foro`）／②GCS `foro_development` バケットの `foro_crm_seed/foro-crm-api/dump_foro-crm-api.tar.gz`（project: `analytics-tool-206405`）。**

---

## **Step 1：コードを落とす**

```bash
cd ~
git clone https://github.com/MedUpInc/medup-core-service.git
cd medup-core-service
```

✅ **既定ブランチは `develop` です**（`main` ではありません）。

> 💡 **`git config` を `--global` で変えないでください。** **このリポジトリだけの設定にします。**
> 
> 
> ```bash
> git config user.name "<氏名>"
> git config user.email "<会社メール>"
> ```
> 

---

## **Step 2：Takumi Guard のトークンを通す**

**npm のパッケージは社内プロキシ（`npm.flatt.tech`）経由で取得します。**

1. **メール認証トークンを発行**（[手順](https://shisho.dev/docs/ja/t/guard/features/token-management/)）
2. **`~/.zshrc`（または `~/.bashrc`）に1行追加**

```bash
export TAKUMI_GUARD_TOKEN=tg_anon_xxxxxxxxxxxxxxxx
```

1. **ターミナルを開き直して確認**

```bash
echo ${TAKUMI_GUARD_TOKEN:+設定OK}
```

⚠️ **`401 Unauthorized` が出たら、トークンが違うか期限切れです。**

---

## **Step 3：node のバージョンを合わせる**

**`.node-version`（または `.nvmrc`）に書かれたバージョンを使います。**

```bash
cat .node-version 2>/dev/null || cat .nvmrc 2>/dev/null
```

### **nvm を使う場合**

```bash
nvm install && nvm use
```

### **🆕 fnm を使う場合（★推奨・速い）**

```bash
fnm install $(cat .node-version 2>/dev/null || cat .nvmrc)
fnm use
```

**確認：**

```bash
node -v    # 22.23.x
npm -v     # 10.9.x
```

> 🔴🔴 **【詰まりどころ①】バージョンが違うと `package-lock.json` が書き換わります。** ★**`package.json` の `engines` は `>=` なので、新しい node でも起動は通ります。でも `npm ci` すると lockfile のバージョンが上がり、チーム全体の lock を壊します。** ✅ **`npm ci` の後に必ず確認：**
> 
> 
> ```bash
> git status package-lock.json    # 何も出なければOK
> ```
> 
> **変更が出たら、その `npm ci` は捨てる：** `git checkout package-lock.json` → 正しい node で取り直す。
> 
> ⚠️ **スクリプトや CI から実行する場合、`fnm --use-on-cd` は効きません**（★対話シェル専用の仕組みのため）。**明示的に `fnm use` を打ってください。**
> 

---

## **Step 4：依存パッケージのインストール**

```bash
# 🔴 必ずリポジトリのルートで
cd ~/medup-core-service
npm ci
```

✅ **約1分・3,265 packages。** ⚠️ **`70 vulnerabilities` などの警告が出ますが、`npm audit fix` は勝手に実行しないでください**（★破壊的変更が入る可能性があります。気になる場合は開発へ相談）。

---

## **Step 5：`.env` と credentials を配置する**

**1Password から取得して、次の場所に置きます。**

### **`.env`（🆕9箇所）**

> 🆕🔴🔴 **【2026-09-25 訂正】ここには「5箇所」と書いてありました。実測は9箇所です。** ★ **足りなかった4本＝`foro-outpatient-web` / `foro-op-optimizer-web` / `acute-care-flow-web` / `admin-web`。**
> 

#### **A. キー名で見分けられるもの（6本）**

| 配置先 | 見分け方 |
| --- | --- |
| **`apps/api/.env`** | **`ADMIN_BATCH_BASIC_AUTH_USERNAME` がある（39行）** |
| **`apps/foro-command-service/.env`** | **`FORO_JOB_WORKER_QUEUE_PATH` がある（39行）** |
| **`apps/foro-query-service/.env`** | **`GEMINI_API_KEY` がある（19行）** |
| **`packages/foro-prisma/.env`** | **DB の URL だけ（3行）** |
| **`apps/foro-job-worker/.env`** | **`KMS_KEYRING_NAME` がある（12行）** |
| 🆕**`apps/admin-web/.env`** | 🔴**`AUTH_USER_NAME` と `AUTH_PASSWORD` がある（6行）。**★**B2C を使わず Basic 認証＋Google OAuth** |

#### **B. 🔴 キー名では見分けられないもの（3本）**

⚠️🔴🔴 **前線の3本はキーの構成が完全に同じです**（★どれも8行・`NEXT_PUBLIC_B2C_CUSTOM_DOMAIN` で始まる）。

| 配置先 | 画面 |
| --- | --- |
| **`apps/foro-outpatient-web/.env`** | **`:3004` 外来最適化** |
| **`apps/foro-op-optimizer-web/.env`** | **`:3005` 逆紹介管理** |
| 🆕**`apps/acute-care-flow-web/.env`** | **`:3006` 入退院調整** |

✅ **見分け方＝`NEXT_PUBLIC_B2C_CLIENT_ID` の値を、`apps/api/.env`（39行のもの）の中の次のキーと突き合わせます。**

| 一致する相手 | その `.env` の置き場所 |
| --- | --- |
| **`MC_MAP_B2C_CLIENT_ID`** | **`foro-outpatient-web`** |
| **`OP_OPTIMIZER_B2C_CLIENT_ID`** | **`foro-op-optimizer-web`** |
| **`ACUTE_CARE_FLOW_B2C_CLIENT_ID`** | **`acute-care-flow-web`** |

```bash
# 値を画面に出さずに照合する（★どれと一致したかだけ出ます）
cd ~/medup-core-service
python3 - <<'PYEOF'
import re
def kv(p):
    return {m.group(1): m.group(2).strip().strip('"').strip("'")
            for m in (re.match(r'^([A-Za-z_]\w*)=(.*)$', l.strip())
                      for l in open(p, encoding='utf-8', errors='ignore')) if m}
api = kv('apps/api/.env')
cid = kv('<ここに判定したい .env のパス>')['NEXT_PUBLIC_B2C_CLIENT_ID']
for k, v in api.items():
    if k.endswith('B2C_CLIENT_ID') and v and v == cid:
        print('一致:', k)
PYEOF
```

⚠️ **`apps/foro-crm-web/.env` はディレクトリです**（中に `development.js`）。★**1Password の `.env` を置く場所ではありません。**

### **credentials（JSON）**

| ファイル | 配置先 |
| --- | --- |
| **`credentials.json`** | **`apps/api/` と `apps/foro-command-service/` の両方** |
| **`medup-foro-service-account-key.json`** | **`apps/foro-command-service/` `apps/foro-job-worker/` `apps/foro-query-service/` の3箇所** |

> 💡 **どれがどれか分からなくなったら、中身のキーで見分けられます**（上の表）。**1Password からダウンロードすると `env (1)` `env (2)` … のような名前になるためです。**
> 

✅ **配置後の確認：**

```bash
git status     # 0件ならOK（.env も credentials も .gitignore 済み）
```

---

## **Step 6：Docker ネットワークを作る**

```bash
docker network create foro_common_link
docker network create medup_common_link
```

> 🔴🔴 **【詰まりどころ②】これを飛ばすと `docker compose up` が失敗します。** ★**`docker-compose.yml` が `external: true` で既にあることを前提にしているためです。** ⚠️ **既存手順では「foro-crm-api 編」の中にだけ書かれていましたが、medup-core-service でも必要です。** ⚠️ **`medup_common_link` はどの手順書にも書かれていませんでした**（`foro_common_link` のみ）。
> 

---

## **Step 7：データベースを起動する**

```bash
cd ~/medup-core-service
docker compose up -d
```

✅ **確認：**

```bash
docker ps                                           # medup-foro-db が Up
docker exec medup-foro-db pg_isready -U postgres    # accepting connections
```

---

## **Step 8：マイグレーション（テーブルを作る）**

```bash
cd packages/foro-prisma
npm run db:migrate:deploy
```

✅ **`mis_dev` / `foro_dev` / `foro_test` / `data_ops_dev` の4つができます。**

```bash
docker exec medup-foro-db psql -U postgres -tAc \
  "SELECT datname FROM pg_database WHERE datistemplate=false ORDER BY 1;"
```

> ⚠️ **【注意】手順書の末尾に `foro_prd`（本番DB）へのマイグレーションコマンドがあります。** 🔴 **ローカル環境構築では絶対に実行しないでください。** ★**ローカルで触るのは `mis_dev` / `foro_dev` / `foro_test` / `data_ops_dev` の4つだけです。**
> 

---

## **Step 9：seed の投入（医療機関データベース）**

```bash
cd ~/medup-core-service
psql "postgres://postgres:rootpassword@localhost:5432/mis_dev" \
  -c "SET session_replication_role = replica;" \
  -f mis-seed-prd.sql \
  -c "SET session_replication_role = DEFAULT;"
```

✅ **約1〜3分。`COPY 35件・195万行・約2.5GB` が入ります。** ★ **`session_replication_role = replica` は「外部キー制約を一時的に止める」指定です**（これが無いと一部のデータが入りません）。

> 🔴🔴 **【詰まりどころ③】`psql` が入っていない場合** **macOS には PostgreSQL クライアントが標準で入っていません。** ✅**Docker 経由で同じことができます：**
> 
> 
> ```bash
> {
>   echo "SET session_replication_role = replica;"
>   cat mis-seed-prd.sql
>   echo "SET session_replication_role = DEFAULT;"
> } | docker exec -i medup-foro-db psql -U postgres -d mis_dev
> ```
> 
> ⚠️ **以降の `psql` コマンドも同様に `docker exec -i medup-foro-db psql -U postgres -d <DB名>` で置き換えられます。**
> 

> 🔴🔴 **【詰まりどころ④】5GB のファイルが git に入りかけます** **`mis-seed-prd.sql` は `.gitignore` に入っていません。** ⚠️**`git add .` を1回打つとコミット対象になります。** ✅ **先に除外してください（★ローカルのみ。リポジトリは汚しません）：**
> 
> 
> ```bash
> cat >> .git/info/exclude <<'X'
> mis-seed-prd.sql
> foro_crm_seed_*.tar.gz
> out/
> X
> git status    # 0件になればOK
> ```
> 

---

## **Step 10：seed の投入（foro データベース）**

```bash
cd ~/medup-core-service
tar -zxf foro_crm_seed_foro-crm-api_dump_foro-crm-api.tar.gz

export PGPASSWORD=rootpassword
outputPath="./out/foro-crm-seed/"

for file in $outputPath/*.csv; do
    tableName=$(basename "$file" .csv)
    psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev \
      -c "SET session_replication_role = replica;" \
      -c "\\COPY $tableName FROM '$file' WITH CSV HEADER;" \
      -c "SET session_replication_role = DEFAULT;"
    psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev \
      -c "SELECT setval('${tableName}_id_seq', (SELECT max(id) FROM $tableName));"
done
```

✅ **CSV 146本が入ります。** 🔴 **`setval`（シーケンス更新）を飛ばさないでください。** ★**飛ばすと、新しいレコードを作るときに ID が重複してエラーになります。**

> 🔴 **`psql` が無い場合は、CSV をコンテナへ渡してから実行します：**
> 
> 
> ```bash
> docker exec medup-foro-db mkdir -p /tmp/seed
> docker cp out/foro-crm-seed/. medup-foro-db:/tmp/seed/
> for f in out/foro-crm-seed/*.csv; do
>   t=$(basename "$f" .csv)
>   docker exec medup-foro-db psql -U postgres -d foro_dev \
>     -c "SET session_replication_role = replica;" \
>     -c "\\COPY $t FROM '/tmp/seed/$t.csv' WITH CSV HEADER;" \
>     -c "SET session_replication_role = DEFAULT;"
>   docker exec medup-foro-db psql -U postgres -d foro_dev -tAc \
>     "SELECT setval('${t}_id_seq', COALESCE((SELECT max(id) FROM $t),1));" >/dev/null
> done
> ```
> 

---

## **Step 11：ログインユーザーを作る**

```bash
PGPASSWORD=rootpassword psql -U postgres -h 127.0.0.1 -p 5432 -d foro_dev \
  -c "insert into users (email, username, aad_user_id, tenant_id, encrypted_password, created_at, updated_at) values ('local-user@medup.jp', 'medup', '<1Password参照>', 36, '<1Password参照>', now(), now());"
```

🔴🔴 **作成された `id` を控えてください。後の Step 14 と Step 16 で使います。**

```bash
docker exec medup-foro-db psql -U postgres -d foro_dev -tAc \
  "SELECT id, email, tenant_id FROM users WHERE email='local-user@medup.jp';"
```

---

## **Step 12：rls_user の確認**

**初回の `docker compose up` で自動実行されます**（`script/` が `/docker-entrypoint-initdb.d` に入るため）。

✅ **確認だけしてください：**

```bash
docker exec medup-foro-db psql -U postgres -tAc \
  "SELECT rolname FROM pg_roles WHERE rolname='rls_user';"
docker exec medup-foro-db psql -U postgres -d foro_test -tAc \
  "SELECT has_database_privilege('rls_user','foro_test','CONNECT');"   # t ならOK
```

> ⚠️ **既存手順は「手動実行が必要」と書いていますが、`docker compose up` を先にやっていれば自動で済んでいます。** 🔴 **もし `rls_user` が無い場合だけ手動実行してください：**
> 
> 
> ```bash
> docker exec -i medup-foro-db psql -U postgres -d foro_dev  < script/01_create_rls_user.sql
> docker exec -i medup-foro-db psql -U postgres -d foro_test < script/02_grant_to_foro_test.sql
> ```
> 

---

## **Step 13：Prisma client を生成する**

```bash
cd packages/foro-prisma
npm run db:client:gen
```

> 🔴🔴 **【詰まりどころ⑤】既存手順ではこのコマンドが `$ c` とだけ書かれており、実行できませんでした。** ✅ **正しくは `npm run db:client:gen` です。**
> 

✅ **`prisma-mis` / `prisma-foro` / `prisma-data-ops` ＋ Kysely の型が生成されます。**

---

## **Step 14：🔴 `USER_ID` を自分の環境に合わせる（★既存手順に記載なし）**

**配布された `.env` の `USER_ID` は、別環境で作られたユーザーの ID です。そのままだと「存在しないユーザー」になります。**

```bash
cd ~/medup-core-service
# Step 11 で控えた id に置き換える
sed -i '' 's/^USER_ID=.*/USER_ID=<Step11のid>/' apps/foro-query-service/.env
sed -i '' 's/^USER_ID=.*/USER_ID=<Step11のid>/' apps/foro-command-service/.env
```

✅ **確認：**

```bash
grep '^USER_ID=' apps/foro-query-service/.env apps/foro-command-service/.env
```

> 🔴🔴 **【詰まりどころ⑥】これを飛ばすと、ログインは通っても「ユーザーが見つからない」状態になります。** ★ **確認方法：`docker exec medup-foro-db psql -U postgres -d foro_dev -tAc "SELECT count(*) FROM users WHERE id=<USER_IDの値>;"` が `1` になること。**
> 

---

## **Step 15：動作確認（API）**

```bash
cd apps/api
npm run dev
```

**別のターミナルで：**

```bash
curl localhost:8888/docs/yaml
```

✅ **次が返れば合格：**

```yaml
openapi: 3.0.3
info:
  title: medup-core-service api
  description: medup-core-service api schema
  version: 0.1.0
```

---

## **Step 16：画面とバックエンドを起動する**

> 🔴🔴 **【詰まりどころ⑦】画面（`:3004` `:3005`）は単体では動きません。** ★**`foro-query-service`（読む）と `foro-command-service`（書く）が必要です。**
> 

```bash
cd ~/medup-core-service
# 🔴 api（:8888）を先頭に置く＝:3001 が依存する（下の【詰まりどころ⑩】）
for s in api foro-query-service foro-command-service \
         foro-outpatient-web foro-op-optimizer-web \
         foro-crm-web acute-care-flow-web; do
  (cd "apps/$s" && nohup npm run dev > "/tmp/svc_$s.log" 2>&1 &)
  sleep 12      # 🔴 間を空ける
done
```

> 🆕🔴🔴 **【2026-09-25 訂正】ここには4本しか書いていませんでした。7本です。** ★ **足りなかった3本＝`api`（`:8888`）／`foro-crm-web`（`:3001`）／`acute-care-flow-web`（`:3006`）。**
> 

> 🆕🔴🔴 **【詰まりどころ⑩】`:3001`（CRM）で `/api/facility_type_groups` が 500 になります。**
> 
> 
> 🔴🔴 **【2026-09-28 訂正】原因は「`:8888` が落ちているから」ではありませんでした。** ★ **正＝`docker-compose.spa.yml` で CRM を起動すると、Rails が staging を見に行くからです。**（★指摘＝鈴木さん）
> 
> | 起動に使う compose | `MEDICAL_INSTITUTION_SERVICE_HOST` | Rails の向き先 | `:3001` |
> | --- | --- | --- | --- |
> | 🔴**`docker-compose.spa.yml`** | 🔴**`https://medup-core-service-api-staging-…run.app` / PORT=443**（★L18-19 に固定） | 🔴**staging** | 🔴**404 → 500** |
> | ✅**`docker-compose.yml`** | ✅**未設定** | ✅**`http://host.docker.internal:8888`**（★`base.rb` の既定） | ✅**表示される** |
> 
> 🔴 **つまり `spa.yml` だと、`:8888` を起動しても直りません。**
> 
> ```bash
> cd ~/foro-crm-api
> docker compose -f docker-compose.yml up -d      # 🔴 spa.yml ではない
> ```
> 
> ★ **経路**：
> 
> ```
> :3001 → :23322(Rails) /api/facility_type_groups
>      → host.docker.internal:8888 /api/facility-type-groups
> ```
> 
> ★ **`foro-crm-api/app/resources/medical_institution_service/base.rb` L6-7 が `MEDICAL_INSTITUTION_SERVICE_HOST`（既定 `http://host.docker.internal`）と `PORT`（既定 `8888`）を読みます。** ⚠️ **`facility_type_groups_controller.rb` は `rescue ActiveResource::BadRequest` しか書いていないので、接続先が違っても素通りして 500 になります。** ★ **Rails 側は `facility_type_groups`（`_`）ですが、`base.rb` が `dasherize` するので `:8888` 側は `facility-type-groups`（`-`）です。**
> 
> ✅ **検算（★2つとも見る）：**
> 
> ```bash
> # ① :8888 が生きているか
> curl -s -o /dev/null -w "%{http_code}\n" localhost:8888/api/facility-type-groups   # 200
> 
> # ② 🔴 Rails コンテナが「どこを向いているか」（★ここが本体）
> docker exec foro-crm-api-mt_web-1 sh -c 'echo "${MEDICAL_INSTITUTION_SERVICE_HOST:-未設定}"'
> #   → 「未設定」なら OK ／ run.app が出たら spa.yml で起動している
> 
> # ③ コンテナの中から届くか
> docker exec foro-crm-api-mt_web-1 sh -c \
>   'curl -s -o /dev/null -w "%{http_code}\n" http://host.docker.internal:8888/api/facility-type-groups'   # 200
> ```
> 
> 🎯 **型＝「ポートが上がっている」と「そこを見に行っている」は別。**★**`curl localhost:8888` が 200 でも、アプリが別のホストを見ていれば意味がありません。**
> 

> 🔴🔴 **【詰まりどころ⑧】`turbo run dev` で一括起動すると `foro-query-service` が `:8889` を LISTEN しません。** ★**単体起動では成功するので、同時起動の競合です。** ✅**12秒ずつ間を空ければ通ります。**
> 

✅ **確認：**

```bash
for p in 3004 3005 8889 8890; do
  echo ":$p → $(curl -s -o /dev/null -w '%{http_code}' localhost:$p)"
done
```

| ポート | 期待値 | 中身 |
| --- | --- | --- |
| **3004** | **200** | **外来最適化の画面** |
| **3005** | **200** | **手術枠最適化の画面** |
| **8889** | 🔴**404 でOK** | **読む専門の API**（★トップページを持たないため） |
| **8890** | 🔴**404 でOK** | **書く専門の API**（★同上） |

---

## **Step 17：foro-crm-api（別リポジトリ）**

```bash
cd ~
git clone https://github.com/MedUpInc/foro-crm-api.git
cd foro-crm-api
```

**1Password から配置：**

| ファイル | 配置先 |
| --- | --- |
| **`master.key`** | **`config/master.key`** |
| **`foro-crm-api-service-account-key-dev.json`** | **リポジトリ直下** |
| **`.env`**（★`SECRET_KEY_BASE` があるもの） | **リポジトリ直下** |

**`USER_ID` を Step 11 の id に書き換える：**

```bash
sed -i '' 's/USER_ID=[0-9]*/USER_ID=<Step11のid>/' docker-compose.yml docker-compose.spa.yml
```

**起動：**

```bash
# ⭐ 単体だけでよい場合（推奨・軽い）
docker compose -f docker-compose.spa.yml up -d

# 関連サービス（mhlw_api / foro-referral-api）も立てる場合
# docker compose up -d
```

⚠️ **`spa.yml` の場合、`mhlw_api` と `foro-referral-api` はステージングを見に行きます**（手順書の注記どおり）。

✅ **確認：**

```bash
curl -o /dev/null -w "CRM: %{http_code}\n"     localhost:23322
curl -o /dev/null -w "Swagger: %{http_code}\n" localhost:8080
```

★ **CRM は `USER_ID` で開発ログイン扱いになるので、ログイン画面は出ません**（`app/controllers/concerns/user_authenticatable.rb` の仕様）。

---

## **Step 18：🔴 画面へのログイン（B2C）**

**`localhost:3004` / `localhost:3005` を開くと、外部の認証画面へリダイレクトされます。**

```
id-outpatient.medup-foro-stg.com     （外来最適化）
id-op-optimizer.medup-foro-stg.com   （手術枠最適化）
```

> 🔴🔴 **【詰まりどころ⑨】会社のメールアドレスでは入れません。** ★**Azure AD B2C のローカルアカウント方式で、`@medup.jp` は自動では登録されていません。** ✅ **「メールアドレスが登録されていません」と出たら、アカウントの発行を依頼してください**（`#foro_dev` / `@all_dev`）。 🔴 **発行に時間がかかるので、Step 0 の時点で先に依頼しておくのが確実です。**
> 

★ **`localhost:23322`（CRM）と `localhost:8080`（Swagger UI）はログイン不要です。**

> 🆕🔴🔴 **【詰まりどころ⑫】`200` が返っても、画面が使える保証はありません（★2026-09-29 追記）。**
> 
> 
> ★ **`curl` が見ているのは「HTTP が返ったか」だけです。**🔴**中身が正しいか、ログインの先が開くか、画面のデータが揃っているかは見ていません。**
> 
> | 見え方 | `curl` の結果 | 実際 |
> | --- | --- | --- |
> | **`:23322`（Rails）が `200`** | ✅**200** | ⚠️**Rails は生きているが、`:8888` を向いていないと `:3001` の画面が 500 になる**（★【詰まりどころ⑩】） |
> | **`:3001` が `307`** | ✅**正常**（ログインへのリダイレクト） | ★**画面が見えるかはログインの先の話** |
> | **`:3005` / `:3006` が `200`** | ✅**200** | 🔴**ユーザーの権限フラグが `false` なら「アクセス権限が必要です」で止まる**（★【詰まりどころ⑪】） |
> | **`:8888` / `:8889` / `:8890` が `404`** | ⚠️**404** | ✅**正常**（トップページを持たない API。`/docs` は 200） |
> 
> 🎯 **型＝「ポートが上がっている」「HTTP が返る」「画面が使える」は3つ別のことです。** ✅ **だから起動確認は3段で見てください：**
> 
> ```bash
> # ① ポートが上がっているか
> curl -s -o /dev/null -w "%{http_code}\n" localhost:23322
> 
> # ② 依存先を向いているか（★:3001 の場合）
> docker exec foro-crm-api-mt_web-1 sh -c 'echo "${MEDICAL_INSTITUTION_SERVICE_HOST:-未設定}"'
> 
> # ③ 画面で確かめる（★これだけが「使える」の証拠）
> #    :3001 → 連携先一覧に件数が出るか
> #    :3005 → 逆紹介一覧の表のヘッダが出るか
> #    :3006 → 相談一覧が開くか
> ```
> 
> ⚠️ **①だけで「起動できた」と書かないでください。**★**②と③が通って初めて使える状態です。**
> 

> 🆕🔴🔴 **【詰まりどころ⑪】ログインは通るのに「アクセス権限が必要です」で止まります（`:3005` / `:3006`）。**
> 
> 
> ```
> ページを表示するには、アクセス権限が必要です。
> お手数ですが、貴院の○○管理者までお問い合わせください。
> ```
> 
> 🔴 **B2C ではなく、ローカル DB のユーザー単位のフラグです。既定が `false` なので、手順書どおりに作ったユーザーは必ずここで止まります。**
> 

| フラグ | 既定 | どこで使うか |
| --- | --- | --- |
| **`enabled_op_optimizer`** | **false** | **`:3005` 逆紹介管理** |
| **`enabled_op_optimizer_data_import`** | **false** | **`:3005` のデータ取り込み** |
| **`enabled_acute_care_flow`** | **false** | **`:3006` 入退院調整** |
| **`enabled_acute_care_flow_social_information`** | **false** | **`:3006` の社会的情報** |

★ **出所＝`packages/foro-prisma/prisma/foro/schema.prisma` L2426/L2427/L2651/L2653/L2654/L2656。** ★ **判定している場所＝`apps/foro-op-optimizer-web/src/features/DoctorContacts/index.tsx` L25 ／ `apps/acute-care-flow-web/src/features/TransferRequests/Index.tsx` L77。**

#### **⬜ 立て方（★2026-09-25 時点は暫定）**

> ⚠️🔴🔴 **正式な手順は定まっていません。** ★ **開発（業務委託の方）から「正しい手順というのは存在していない認識。管理画面からでも DB 直接更新でも良い」という個人的見解をいただきました。**🔴**決裁は取っていません。** ✅ **2026-10-01 までに追加の指示が無ければ、下を現時点の最適解として運用します。** 🆕✅ **2026-09-25 16:58 に開発から「ここに関してチームで決めて問題ないと思います」という返答をいただきました。**★**これで異論が出ない見込みは高いですが、決裁ではないので「暫定」の札は 10/01 まで外しません。**
> 

```bash
# Step 14 で調べた USER_ID を使う
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

⚠️🔴 **`docker exec` に `-i` が要ります**（★無いと SQL が渡らず、エラーも出ずに何も起きません。2026-09-25 に実測）。 ✅ **`UPDATE 1` と出ることを必ず確認してください。**

✅ **検算：**

```bash
docker exec medup-foro-db psql -U postgres -d foro_dev -c \
"select id, enabled_op_optimizer, enabled_op_optimizer_data_import, \
        enabled_acute_care_flow, enabled_acute_care_flow_social_information \
   from users where id = <あなたの USER_ID>;"
```

✅ **画面で確認＝`:3005/target-lists` に「逆紹介一覧」、`:3006/transfer-requests` に「相談一覧」が出れば通っています。** ★ **`:3006` が「データはありません」でも権限は通っています**（★表のヘッダと「新規登録」ボタンが出ていれば OK。seed にデータが無いだけ）。

🔴 **戻すとき：** 上の `UPDATE` の `true` を `false` に変えるだけです。

⬜ **本筋は管理ツール（`admin-web` / `:3002`）からの設定**とのことでした。★**どちらを正とするかが決まったら、この節を差し替えます。**

---

## **Step 19：単体テスト（任意）**

```bash
cd packages/foro-prisma && npm run db:migrate:test
cd ../../apps/api && npm run test
```

---

## **✅ 最終チェックリスト**

| # | 確認 | 期待 |
| --- | --- | --- |
| **1** | `git status`（medup-core-service） | **0件** |
| **2** | `docker ps` | **`medup-foro-db` ＋ foro-crm-api 系3本** |
| **3** | `curl localhost:8888/docs/yaml` | **`title: medup-core-service api`** |
| **4** | `curl localhost:3004` / `:3005` | **200** |
| **5** | `curl localhost:23322` | **200** |
| **6** | **`users` に自分の id があるか** | **1件** |
| **7** | **`mis_dev` の行数** | **約195万行** |
| **8** | **`.env` の `USER_ID`** | **Step 11 の id と一致** |

---

## **🔄 日々の起動と停止**

### **起動**

```bash
# ① Docker
cd ~/medup-core-service && docker compose up -d
cd ~/foro-crm-api && docker compose -f docker-compose.spa.yml up -d

# ② API と画面（★12秒ずつ）
cd ~/medup-core-service
for s in foro-query-service foro-command-service foro-outpatient-web foro-op-optimizer-web; do
  (cd "apps/$s" && nohup npm run dev > "/tmp/svc_$s.log" 2>&1 &); sleep 12
done
```

### **停止**

```bash
pkill -f 'npm run dev'
cd ~/medup-core-service && docker compose down
cd ~/foro-crm-api && docker compose -f docker-compose.spa.yml down
```

✅ **データは volume に残るので、seed の入れ直しは不要です。**

---

## **📎 付録：修正・追記した11件の一覧**

| # | 箇所 | 既存の記載 | 修正 |
| --- | --- | --- | --- |
| **1** | **Step 13** | 🔴**`$ c`（コマンドが欠落）** | **`npm run db:client:gen`** |
| **2** | **Step 9/10/12** | **`psql` を直接実行** | **`docker exec` での代替を追記**（macOS には標準で入っていない） |
| **3** | **Step 12** | **「手動実行が必要」** | **`docker compose up` 済みなら自動。確認方法を追記** |
| **4** | **Step 6** | **`foro_common_link` のみ**（★48 側に記載） | 🔴**`medup_common_link` も必要。47 側にも移動** |
| **5** | **Step 5** | **`.env` は4箇所** | 🔴**`apps/foro-job-worker/.env` を追加（5箇所）** |
| **6** | **Step 14** | **記載なし** | 🔴🔴**`USER_ID` の書き換えが必須（既存の値は別環境のもの）** |
| **7** | **Step 16** | **`npm run dev` で一括起動** | 🔴**`turbo` 同時起動では `:8889` が上がらない。12秒ずつ間を空ける** |
| **8** | **Step 16** | **画面だけ起動** | 🔴**バックエンド2本が必須であることを明記** |
| **9** | **Step 3** | **nvm 推奨** | **fnm の手順と、lockfile が壊れる注意を追記** |
| **10** | **Step 9** | **記載なし** | 🔴🔴**5GB の seed が `.gitignore` されていない。`.git/info/exclude` への追加を追記** |
| **11** | **Step 0/18** | **「ログインアカウントは共有」** | 🔴**B2C への個別登録が必要。先に依頼するよう Step 0 に前倒し** |

### **⚠️ 他の編で見つかったもの**

| 編 | 内容 |
| --- | --- |
| **foro-op-optimizer編** | 🔴**`npm run dev -w apps/op-optimzer-web` は誤字**（`i` が抜け）。**正しくは `foro-op-optimizer-web`** |
| **共通** | ⚠️**手順書末尾の `foro_prd`（本番DB）へのマイグレーションは、ローカル構築では実行しないことを明記** |

---

### **🆕 9/25 追加分（★実際に動かして出た4件）**

> 🔴 **上の11件と性質が違います。** ★ **11件＝作業前に手順書と実際を読み比べて出たもの（★机上で分かる）。** ★ **この4件＝作業中に、実際に動かして初めて出たもの（★読んでも分からない）。** 🎯 **混ぜると、次に同じことをする人が「どこまでが机上で分かるか」を見失います。**（★指摘＝Web顧問）
> 

| # | どこ | 何がずれていたか | どう直したか |
| --- | --- | --- | --- |
| **12** | **Step 5** | 🔴**`.env` は「5箇所」ではなく9箇所。しかも前線3本（`foro-outpatient-web` / `foro-op-optimizer-web` / `acute-care-flow-web`）はキー構成が完全に同じで、キー名では見分けられない** | **表を A（キー名で分かる6本）と B（分からない3本）に分け、B は `NEXT_PUBLIC_B2C_CLIENT_ID` を `apps/api/.env` と突き合わせる方法を追記** |
| **13** | **Step 16** | 🔴**起動が4本しか書かれていない。7本が要る**（★足りない＝`api`(:8888) / `foro-crm-web`(:3001) / `acute-care-flow-web`(:3006)） | **ループを7本に。【詰まりどころ⑩】として `:3001` → `:23322` → `:8888` の依存と検算コマンドを追記** |
| **14** | **Step 18** | 🔴**ログインが通っても「アクセス権限が必要です」で止まる**（`:3005` / `:3006`）。**ローカル DB のフラグ4つが既定 `false`** | **【詰まりどころ⑪】として `UPDATE` 文・`docker exec -i` の注意・検算・戻し方を追記。**⚠️**正式手順が未定のため「暫定＋2026-10-01」と明示** |
| **15** | **Step 5** | **`acute-care-flow-web` と `admin-web` の `.env` が配られていなかった**（★`.env.example` も無い） | **1Password にあった。見分け方を Step 5 の表に追記** |

⚠️🔴 **12〜15 は、手順書を読むだけでは出ません。** ★**実際に9アプリを起動し、画面を開いて初めて出たものです。** ✅ **だから「通し検証」は読み合わせでは代用できない**——**これがこの節を分けた理由です。**

---

## **📌 出所**

> ✅ **この手順書の主張が、いつ・何で裏づけられたか。**★**2026-09-29 に Notion 側から手元へ取り込んだ**（★Notion にあって手元に無かった）。
> 

| 中身 | 出所 |
| --- | --- |
| **`:23322` が 200 でも `:3001` が 500** | **2026-09-28 の実測**（★`docker-compose.spa.yml` が staging を向いていた。鈴木さんの指摘） |
| **`:3005` / `:3006` の権限フラグ** | **2026-09-25 の実測**（★`users` の4列が既定 `false`。【詰まりどころ⑪】に詳細） |
| **`:8888` の 404 が正常** | **2026-09-24 の実測**（★トップページを持たない API） |
| **`:3001` の 307** | **同上**（★ログインへのリダイレクト） |
| 🆕**`200` が返っても画面が使えるとは限らない** | **2026-09-29 の実測**（★MSAL の設定エラー。【詰まりどころ⑫】に詳細） |

---

## **🔴🔴 実データを触る日に読むこと（★2026-09-30 追記）**

> 🔴🔴 **この手順書はテスト用の合成データで通せますが、実データを触る日が来ます。**★**その日はこれを先に読んでください。**
> 

### **🔴 社内の規程に、こう書かれています（★実物6箇所・原本を引用）**

| 出所 | 明文 |
| --- | --- |
| **`production-runbook.md` L58** | **「message 本文、object path、CSV 値、PHI を log、ticket、shell output に残しません」** |
| **同 L143** | **「payload 全体は表示せず、既知の非 PHI key だけを抽出します」** |
| 🔴**同 L180** | 🔴**「`normalized_object` は basename に患者、施設、日付が含まれ得るため表示しません」** |
| **同 L259** | **「証跡に残しません」** |
| **`input-contract.md` L194** | **「response や log には CSV セル値を含めません」** |
| **同 L223** | **「operator は response、Cloud Run log、DLQ のいずれにも CSV セル値や PHI を出力しません」** |

★ **つまり「ファイル名すら出さない」水準です。**🎯**理由＝ファイル名に患者・施設・日付が入り得るから。**

### **✅ だから、実データの日にやること3つ**

| # | やること |
| --- | --- |
| **1** | 🔴**画面に出た値を、そのままどこにも貼らない**（★Slack・チケット・メモ・生成AI のすべて） |
| **2** | 🔴**エラーを共有するときは「行番号と件数だけ」**（★「3行目で落ちた」は可。**中身は貼らない**） |
| **3** | 🔴🔴**ファイル名も出さない**（★`normalized_object` の basename に患者名が入り得る。**「1件目のファイル」と言う**） |

⚠️🔴🔴 **そして AI DPC の出力にも診療情報が含まれます。**★**結果の Excel・CSV を開いたら、それも同じ扱いです。** 🔴 **生成AI には入れません**（★社内の AI 利用ルールの明文を確認するまで）。

---

## **✍️ 詰まったところを書き足してください**

> 🎯 **この手順は「1人が1回通した」だけです。** ★**2人目・3人目が通すたびに強くなります。** ✅ **詰まったら、下に1行足してください**（★直さなくて構いません。「ここで止まった」だけで十分です）。
> 

| 日付 | 誰 | どの Step | 何が起きたか | どう抜けたか |
| --- | --- | --- | --- | --- |
| **2026-09-24** | **小山** | **（初回の通し検証）** | **16件の詰まりを修正・追記** | **本文に反映済み** |
|  |  |  |  |  |
|  |  |  |  |  |