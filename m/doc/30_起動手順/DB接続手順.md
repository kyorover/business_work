# DB接続・切断手順まとめ（Staging / Production）

## 0. 事前準備 (GCP認証)

DBへ接続する前に、ローカルPCでGCPの認証を済ませておきます。ターミナルで以下のコマンドを実行し、ブラウザでメダップのGoogleアカウントでログインしてください。

```bash
# cloud-sql-proxy用 (foro_stg, refery全般)
gcloud auth application-default login

# SSHトンネル・アクセストークン用 (foro_prd)
gcloud auth login
```

## 1. foro 環境

### 1-1. Staging (`foro_stg`)

**① Port起動 (cloud-sql-proxy)**
ターミナルを開き、以下のコマンドでプロキシを起動します。

```bash
cloud-sql-proxy medup-foro:asia-northeast1:medup-foro-stg --port 13307 &
```

※ `Ready for new connections` と表示されれば準備完了です。

**② DBeaver 接続**

* **ホスト**: `localhost` (または `127.0.0.1`)
* **ポート**: `13307`
* **データベース**: `foro_stg`
* **ユーザー名**: `postgres`
* **パスワード**: 1Password等から取得したパスワード

**③ DBeaver 切断 / プロセス終了**

1. DBeaverで対象DBを右クリックし、「切断 (Disconnect)」。
2. ターミナルで `pkill -f cloud-sql-proxy` を実行。

### 1-2. Production (`foro_prd`)

#### 【初回設定（SSH鍵の準備）】
本番環境へ接続するには、踏み台サーバー（`alloy-vm`）に自身のSSH公開鍵が登録されている必要があります。未登録の場合は以下の手順で鍵を作成し、登録を依頼してください。

1. **SSH鍵の作成**
   ターミナルで以下を実行し、新しい鍵を作成します。
   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/foro-alloydb-vm -C "your.name@medup.jp"
   ```
2. **公開鍵の確認**
   以下のコマンドで公開鍵の中身を表示し、出力された文字列をコピーします。
   ```bash
   cat ~/.ssh/foro-alloydb-vm.pub
   ```
3. **登録依頼**
   コピーした文字列（`ssh-ed25519 ...`）を、Slackの `#foro_dev` 等でインフラ管理者へ共有し、「medup-foroプロジェクトの alloy-vm への公開鍵登録」を依頼してください。登録完了後に以下の接続手順へ進めます。

---

**① Port起動 (SSHトンネル)**
ターミナルで以下の変数を自身の設定に書き換えて実行します。

```bash
ALLOYVM_USER=your.name
ALLOYVM_SSHKEY=~/.ssh/foro-alloydb-vm

# バックグラウンドでSSHトンネルを起動
ssh -i $ALLOYVM_SSHKEY -L 5432:127.0.0.1:5432 -N ${ALLOYVM_USER}@35.200.113.56 -o ServerAliveInterval=60 &
```

**② DBeaver 接続**

* **ホスト**: `127.0.0.1`
* **ポート**: `5432`
* **データベース**: `foro_prd`
* **ユーザー名**: 自身のIAMユーザーメールアドレス（例: `your.name@medup.jp`）
* **パスワード**: ターミナルで `gcloud auth print-access-token` を実行して出力された文字列

**③ DBeaver 切断 / プロセス終了**

1. DBeaverで対象DBを右クリックし、「切断 (Disconnect)」。
2. ターミナルで `pkill -f "ssh -i.*-L 5432:127.0.0.1:5432"` を実行。

## 2. refery 環境

※ 競合を防ぐため、foroとは別のポート番号（例：13308, 13309）を指定しています。
※ `<インスタンス接続名>` の部分は、GCP上で確認した `プロジェクトID:リージョン:インスタンス名` に置き換えて実行してください。

### 2-1. Staging (`refery_staging`)

**① Port起動 (cloud-sql-proxy)**
ターミナルで以下のコマンドを実行します。

```bash
cloud-sql-proxy refery:asia-northeast1:refery-staging --port 13308 &
```

**② DBeaver 接続**

* **ホスト**: `localhost` (または `127.0.0.1`)
* **ポート**: `13308`
* **データベース**: `refery_staging`
* **ユーザー名**: `refery_ro`
* **パスワード**: 1Password等から取得したパスワード（設定されている場合）

**③ DBeaver 切断 / プロセス終了**

1. DBeaverで対象DBを右クリックし、「切断 (Disconnect)」。
2. ターミナルで `pkill -f cloud-sql-proxy` を実行。

### 2-2. Production (`refery_production`)

**① Port起動 (cloud-sql-proxy)**
ターミナルで以下のコマンドを実行します。

```bash
cloud-sql-proxy refery:asia-northeast1:refery-production --port 13309 &
```

**② DBeaver 接続**

* **ホスト**: `localhost` (または `127.0.0.1`)
* **ポート**: `13309`
* **データベース**: `refery_production`
* **ユーザー名**: `refery_ro`
* **パスワード**: 1Password等から取得したパスワード（設定されている場合）

**③ DBeaver 切断 / プロセス終了**

1. DBeaverで対象DBを右クリックし、「切断 (Disconnect)」。
2. ターミナルで `pkill -f cloud-sql-proxy` を実行。