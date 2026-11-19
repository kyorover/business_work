# Referyでテナントユーザーとしてアカウントを発行する（ローカル）

Referyのローカル環境で、ゲストユーザーではなくテナントユーザーとしてログインし、テナント向けのアカウント管理画面からアカウントを発行する手順です。

Refery本体は別リポジトリ（`~/Desktop/work/refery`）にあります。この手順書は、そのリポジトリのローカルDBとAPIの`.env`を使ってログインユーザーを切り替える方法を説明します。

> **注意:** `.env`のID切り替えは非本番環境専用です。本番環境ではB2Cトークンのクレームからユーザーを特定するため、この方法は使えません。

## 1. ReferyのDBを起動する

Referyのリポジトリへ移動します。

~~~bash
pkill -f 'npm run dev'
~~~

```bash
cd ~/Desktop/work/refery
```

~~~bash
docker compose down
~~~

DBコンテナを起動します。

```bash
docker compose up -d
```

DBが未構築の場合は、APIディレクトリへ移動します。

```bash
cd ~/Desktop/work/refery/apps/api
```

マイグレーションを実行します。

```bash
npm run db:migrate:dev
```

## 2. テナントと管理者ユーザーを用意する

以下は初回セットアップ用です。すでに対象テナントや`refery-dev`ユーザーがある場合、既存データは変更されません。

`medup`テナントがない場合に作成します。

```bash
docker compose exec -T db psql -U postgres -d refery_development -c "INSERT INTO tenants (id, name, postal_code, prefecture, city, street, phone_number, days_before_instant_reservation, days_before_request_reservation, created_at, updated_at) VALUES (gen_random_uuid(), 'medup', '101-0061', '東京都', '千代田区', 'テスト住所', '00-0000-0000', 3, 3, now(), now()) ON CONFLICT (name) DO NOTHING;"
```

`refery-dev`管理者ユーザーがない場合に作成します。

```bash
docker compose exec -T db psql -U postgres -d refery_development -c "INSERT INTO users (id, display_name, tenant_id, email_address, user_name, role, created_at, updated_at) SELECT gen_random_uuid(), 'refery', id, NULL, 'refery-dev', 'admin', now(), now() FROM tenants WHERE name = 'medup' ON CONFLICT (user_name) DO NOTHING;"
```

## 3. テナントIDと管理者ユーザーIDを取得する

テナントとユーザーを結合して取得します。`LIMIT 1`で無関係なユーザーを選ぶのではなく、対象のテナントに属し、ゲストではない管理者を確認してください。

```bash
docker compose exec -T db psql -U postgres -d refery_development -c "SELECT t.id AS tenant_id, u.id AS tenant_user_id, u.user_name, u.role FROM tenants t JOIN users u ON u.tenant_id = t.id WHERE t.name = 'medup' AND u.user_name = 'refery-dev' AND u.guest_id IS NULL;"
```

結果が0件の場合は、テナント名とユーザー名を確認してください。必要に応じて、既存のテナント管理者一覧を確認します。

```bash
docker compose exec -T db psql -U postgres -d refery_development -c "SELECT t.name AS tenant_name, u.id AS tenant_user_id, u.user_name, u.role FROM tenants t JOIN users u ON u.tenant_id = t.id WHERE u.guest_id IS NULL ORDER BY t.name, u.user_name;"
```

## 4. APIの`.env`をテナントユーザー用に切り替える

APIディレクトリへ移動します。

```bash
cd ~/Desktop/work/refery/apps/api
```

`.env`をエディターで開きます。以下のコマンドはVS Codeを使う例です。

```bash
code .env
```

取得した値を使い、`.env`の有効な設定を次の状態にします。

- `TENANT_ID`に`tenant_id`を設定する。
- `TENANT_USER_ID`に`tenant_user_id`を設定する。
- ゲスト用に設定されている`GUEST_ID`と`GUEST_USER_ID`はコメントアウトする。
- テナント用とゲスト用のIDを同時に有効にしない。

アカウント発行では、DBのID設定に加え、B2Cユーザーの作成と招待メール送信に使う設定も必要です。`B2C_TENANT_ID`、`B2C_CLIENT_ID`、`B2C_CLIENT_SECRET`、`B2C_EXTENSION_APP_ID`、`B2C_ISSUER`、`REFERY_URL`およびメールアドレスで発行する場合の`SENDGRID_API_KEY`が、有効な開発環境の値になっていることを確認してください。シークレットはチームで管理されている値を使い、ログやソース管理に記載しないでください。

## 5. APIとWebを起動する

`.env`を変更した場合は、変更後の値を読み込ませるため、APIプロセスを必ず再起動してください。ファイルを編集しただけでは、すでに起動しているプロセスの環境変数は変わりません。

ポート3001を使用しているプロセスを確認します。

```bash
lsof -nP -iTCP:3001 -sTCP:LISTEN
```

表示されたPIDがRefery APIであることを確認し、そのPIDを指定して停止します。`12345`は確認したPIDに置き換えてください。

```bash
kill 12345
```

ポート3001が空いたことを確認します。

```bash
lsof -nP -iTCP:3001 -sTCP:LISTEN
```

APIを起動します。

```bash
cd ~/Desktop/work/refery/apps/api
```


```bash
nohup env NODE_ENV=development node --env-file=.env -r @swc-node/register -r tsconfig-paths/register src/server.ts > /tmp/svc_refery-api.log 2>&1 < /dev/null &
```

APIが起動したら、セッションAPIがテナントユーザーを返すことを確認します。

```bash
curl -fsS http://localhost:3001/common/session/me | python3 -c 'import json,sys; print(json.load(sys.stdin)["group"])'
```

`tenant`と表示されれば切り替え成功です。`guest`の場合は、`.env`の有効なIDと、起動したAPIプロセス・ポートが正しいか再確認してください。

Webディレクトリへ移動します。

```bash
cd ~/Desktop/work/refery/apps/web
```

Webを起動します。

```bash
nohup npm run dev > /tmp/svc_refery-web.log 2>&1 < /dev/null &
```

Web画面を開きます。

http://localhost:3003

APIを再起動した後、ブラウザーの画面も再読み込みしてセッション情報を更新してください。

## 6. アカウントを発行する

テナント側のアカウント管理画面を開き、管理者権限でアカウントを追加します。ローカルAPIは`.env`で指定したテナント管理者として処理するため、DB上の対象ユーザーが`admin`ロールであることを確認してください。

- メールアドレスで発行する場合、B2Cユーザー作成に加えて招待メールを送信します。B2CとSendGridの認証設定が必要です。
- ログインIDで発行する場合、画面に表示される認証URLと仮パスワードを利用者へ安全に共有します。

発行に失敗した場合は、再試行前に画面を再読み込みし、DBにアカウントが作成済みでないか確認してください。B2C側のユーザー作成に失敗しても、DB側のユーザーが残っている場合があります。

## ゲスト環境へ戻す

ゲストユーザーで動作確認する場合は、APIの`.env`で`TENANT_ID`と`TENANT_USER_ID`をコメントアウトし、`GUEST_ID`と`GUEST_USER_ID`を有効にします。設定変更後、APIを再起動してください。

どちらのIDを使っているか確認するには、Refery APIの`.env`で有効なキー名だけを表示します。

```bash
grep -E '^(TENANT_ID|TENANT_USER_ID|GUEST_ID|GUEST_USER_ID)=' ~/Desktop/work/refery/apps/api/.env
```
