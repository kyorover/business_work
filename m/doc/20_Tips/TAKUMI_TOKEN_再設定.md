# TAKUMI_GUARD_TOKEN 再設定・npm ci 実行手順

## 1. トークンの再発行リクエスト
以下のコマンドを実行し、登録済みのメールアドレスを送信します。
```bash
curl -X POST https://npm.flatt.tech/api/v1/tokens \
  -H "Content-Type: application/json" \
  -d '{"email": "kyohei.yoshida@medup.jp", "language": "ja"}'
```

## 2. ワンタイムリセットコードの実行
届いたメールに記載されている「リセット用コマンド」をコピーしてターミナルで実行します。
（※ `code` にはメールに記載された1時間有効な一時コードが入ります）
```bash
curl --json '{"email": "ご自身のメールアドレス", "code": "XXXX"}' \
  https://npm.flatt.tech/api/v1/tokens/reset
```
実行が成功すると、ターミナルに新しいAPIキー（`tg_anon_` から始まる文字列）が表示されます。

## 3. 環境変数の設定
取得した新しいAPIキーを、現在のターミナルに環境変数として設定します。
```bash
export TAKUMI_GUARD_TOKEN="新しく取得した tg_anon_ から始まるAPIキー"
```

## 4. 設定の確認
正しく値が反映されたか確認します。
```bash
echo $TAKUMI_GUARD_TOKEN
```
※ターミナルに設定した新しいAPIキーが表示されれば完了です。

## 5. インストールの実行
ルート直下の .npmrc を直接開き TOKENの値を上書き

## 6. インストールの実行
環境変数が設定された同じターミナル上で、プロジェクトのルートディレクトリから再度インストールを実行します。
```bash
npm ci
```