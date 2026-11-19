# foro-crm-api 500エラー (resources is nil) 発生・解決のまとめ

## 経緯(前提)
- `medup-core-service` と `foro-crm-api` のローカル環境を構築中。
- マイグレーション、シードデータ投入、ローカルユーザー（ID: 4638）作成と各種環境変数の `USER_ID` 統一は完了済み。
- トラブルシューティングの過程で、手動でDB（`foro_dev`）のDROPおよび再作成を実施していた。

## 事象
- 画面から `foro-crm-api` のAPIへリクエストすると `500 Internal Server Error (resources is nil)` が発生し画面遷移できない。
- 一時、Railsコンテナ内でDBアクセス時に `permission denied for table tenants` エラーが確認された（※その後のSQL調査で、現在は権限が正常に付与されていることが判明）。

## 原因
- **不適切な起動ファイルの指定**。
- ローカル環境のCRMにおいて、環境変数による自動ログイン認証を正常に通過させるためには標準の `docker-compose.yml` を使用する必要があったが、`docker-compose.spa.yml` を使用して起動していたため、テナント情報等を引けず500エラーを引き起こしていた。

## 対処方法
DBのアクセス権限およびユーザー・テナント状態が正常であることをSQLで確認した上で、以下の手順で起動コマンドを修正した。

1. 誤った設定のコンテナを停止
   cd ~/Desktop/work/foro-crm-api
   docker compose down

2. 標準のymlファイルを指定して再起動
   docker compose -f docker-compose.yml up -d mt_web