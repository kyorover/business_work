# ローカルでのアカウント追加・保存時に発生する Azure 認証エラー

## 事象

ローカルの foro Core Admin でアカウントを追加、または保存すると、API が次のエラーを返す。

```text
AADSTS7000215: Invalid client secret provided.
Ensure the secret being sent in the request is the client secret value,
not the client secret ID.
```

このエラーは、アカウントの入力値の検証エラーではなく、API が Azure AD B2C / Microsoft Graph のアクセストークンを取得するときの認証エラー。

## 仕組み

ローカル画面からの操作でも、アカウントの B2C 情報を扱う処理は Azure に接続する。

1. admin-web が Core API（通常 `http://localhost:8888`）へリクエストする。
2. ローカルの Core API が `apps/api/.env` の `B2C_TENANT_ID`、`B2C_CLIENT_ID`、`B2C_CLIENT_SECRET` を使い、Microsoft Graph のアクセストークンを取得する。
3. トークン取得後、Graph API を呼び出して B2C ユーザー情報を作成または更新する。

API の起動手順では `npm run dev -w apps/api` を使い、`apps/api/package.json` の `--env-file=.env` によって `apps/api/.env` が読み込まれる。したがって「ローカル環境には Azure 接続用の設定がない」というより、ローカル API に Azure 接続用の設定はあり、その認証情報が Azure に受け付けられていない状態と考えられる。

## 今回確認できたこと

- エラー応答は HTTP 500 で、内容は `AADSTS7000215`。
- `apps/api/.env` の値を表示せずに確認した結果、`B2C_CLIENT_ID`、`B2C_CLIENT_SECRET`、`B2C_TENANT_ID` は設定されていた。
- `B2C_CLIENT_ID` はエラーに記載されたアプリケーション ID と一致していた。
- シークレットの値自体が有効かどうかは、ローカルからの設定確認だけでは判定できていない。
- API はポート 8888 ですでに起動していた。二重起動を試みた際の `EADDRINUSE` はポート競合であり、今回の Azure 認証エラーとは別の問題。
- `source map` の警告や Node.js の非推奨警告も、`AADSTS7000215` の原因ではない。

## 考えられる原因

`B2C_CLIENT_SECRET` に設定した値が、エラーに記載されたアプリ登録の有効なシークレット値ではない可能性が高い。たとえば次のケースが考えられる。

- クライアントシークレットの「値」ではなく「シークレット ID」を設定している。
- シークレットが期限切れ、または失効している。
- 別のアプリ登録で発行されたシークレットを設定している。
- API 起動時の環境変数が `.env` の値を上書きしている。

## 対応手順

1. Azure のアプリ登録（エラーに記載されたアプリケーション ID）で、有効なクライアントシークレットを確認する。既存の値を再表示できない場合は、権限のある担当者に有効な値の再設定を依頼する。
2. シークレットの「値」を `apps/api/.env` の `B2C_CLIENT_SECRET` に設定する。シークレット ID は使用しない。
3. その値が `B2C_TENANT_ID`、`B2C_CLIENT_ID` と同じアプリ登録に対応していることを確認する。
4. 既存の API プロセスを停止し、8888 番ポートが空いていることを確認してから API を起動し直す。起動時に環境変数が読み込まれるため、`.env` を変更しただけでは既存プロセスに反映されない。
5. 管理画面から操作を再試行し、同じ `AADSTS7000215` が解消したか確認する。

クライアントシークレットは秘密情報のため、チャット、ログ、ソース管理に貼り付けない。

## 保存時の注意

アカウント保存では、実装上、ローカル DB のユーザー更新後に Azure 側のユーザー情報を更新する。このため Azure 更新で失敗した場合、API はエラーを返してもローカル DB の変更だけが完了している可能性がある。再試行前に画面を再読み込みし、現在のアカウント状態を確認する。

## 関連実装

- 環境変数から Azure 認証情報を使って Graph クライアントを作成: `apps/api/src/gateways/azure/client.ts`
- B2C ユーザーの作成・更新: `apps/api/src/gateways/db/repositories/auth/UserClaimRepositoryImp.ts`
- アカウント追加処理: `apps/api/src/modules/admin/applications/usecases/CreateAccountUseCase.ts`
- アカウント更新処理: `apps/api/src/modules/admin/applications/usecases/UpdateAccountUseCase.ts`
