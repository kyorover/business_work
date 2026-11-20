## Refery B2Cテンプレート（2FA画面）デプロイ手順（概要）

1. **事前準備**: Microsoft 365 アカウントと、Azure（`medup.jp` ディレクトリ）の対象ストレージへの Blob 操作権限を取得する。
2. **ビルド**: 対象ブランチで `cd apps/adb2c && npm run build` を実行し、静的ファイルを生成する。
3. **アップロード**: Azure Portal からストレージ（STG: `referydevelopmentstorage` / PRD: `referyproductionstorage`）の `root` コンテナーを開き、既存ファイルをバックアップした上で、ビルド後の `activate_email_mfa.html`（`root/`）と `activate_email_mfa.css`（`root/css/`）を上書きアップロードする（カスタムポリシー XML の再アップロードは不要）。
4. **確認**: 対象 Blob URL で `.logoBar` の反映を確認し、実画面でロゴバーが表示されることを確認する。