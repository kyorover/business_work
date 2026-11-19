# Refery 2FA設定直後の再読み込み調査・修正タスク

## 目的

Referyの2FA必須化PRで指摘された「2FA設定直後に再読み込みすると、未設定扱いに戻って設定ページへ誘導される可能性」を解消する。画面の2FA判定とAPIアクセストークン取得で、同じ2FA有効化済みアカウントを利用することを保証する。

## 経緯

- ReferyのPR [#2488](https://github.com/MedUpInc/refery/pull/2488)「feat: refery 2要素認証必須化対応」に対し、2026-11-19に次のレビュー懸念が出た。
  - 2FA設定後のMSALキャッシュに、ログイン時の未設定アカウントと有効化時の設定済みアカウントが共存する。
  - 再読み込み時に未設定アカウントが選ばれると、設定画面へ戻る、または業務API向けのトークンも未設定アカウントから取得される可能性がある。
  - ログイン時のトークン有効期間中はこれが繰り返されるのではないか、という懸念。
- 調査の結果、レビュー指摘は妥当と判断した。アカウントの共存は実機で確認済みで、アカウント選択とトークン取得の実装に、指摘につながる経路がある（詳細は「調査結果」）。
- 本文書はReferyプロジェクトに引き継ぐための調査・実装タスクであり、Referyのソースコードはこの作業では変更していない。

## 前提・スコープ

- 2FA必須化の導線・他画面をロックする方針は、[Refery 2要素認証（2FA）必須化 実装・検証.md](./Refery%202要素認証（2FA）必須化%20実装・検証.md)に従う。
- 対象ブランチは `feature/mandatory-email-mfa`。実装開始前にPRの最新差分とブランチ状態を再確認する。
- 既存アカウントの移行や2FA登録フローの変更は今回のスコープ外。アカウント選択、トークン取得、およびその回帰テストを対象とする。
- 実装・検証.md の「アカウント選択方式自体は今回変更しない」は、本タスクの内容と矛盾する。本タスクの実装時に、同資料の該当記述と `config.ts` の対応欄を更新する。
- トークン期限を前提とした修正はしない。

## 調査結果

### 確認できた実装

1. `apps/web/src/middlewares/Auth/config.ts`
   - `setupMsalInstance` は初期化直後に `getAllAccounts()` を取得し、アカウントがあれば無条件で `accounts[0]` をactive accountに設定する。
   - この処理は、キャッシュに未設定・設定済みのアカウントが複数ある場合にも先頭を選ぶ。配列順を2FA状態に基づいて決める処理はない。
   - `acquireAccessToken` はactive accountを優先するが、それがなければ再び `getAllAccounts()[0]` にフォールバックする。
   - 上記2点はPR以前からの既存実装。PRで追加されたのは、`ACQUIRE_TOKEN_SUCCESS` で `setActiveAccount(account)` する処理と、2FA状態による画面ガード・API拒否である。
2. `apps/web/src/hooks/useIsEmailMfaActivated.ts`
   - MFA状態は `getActiveAccount()` の `idTokenClaims.extension_requireEmailMfa` から判定する。
3. `apps/web/src/middlewares/Auth/index.tsx`
   - MFA状態が `required` の場合、2FA設定ページ以外から設定ページへリダイレクトする。
4. `apps/api/src/apis/middlewares/Auth.ts`
   - MFAクレームが `true` でないトークンは、必須化対象APIで `EmailMfaRequired` として拒否される。
5. `apps/web/src/middlewares/Auth/__tests__/config.test.ts`
   - 現状のキャッシュアカウントテストは1アカウントのケースであり、複数アカウントの選択とAPIトークン取得時のアカウントを検証していない。

### 実機確認（2026-11-19、PR反映前のステージング）

- 2FA未設定のテストアカウントでログインし、既存の2FA有効化画面で有効化した。同じタブで、`sessionStorage` 内の `homeAccountId` を確認した。
  - 設定前: 1件（`…-b2c_1a_signup_signin.<tenant>`）。
  - 設定後: 2件（上記に加え `…-b2c_1a_activate_email_mfa.<tenant>`）。
- B2Cのアカウントはポリシー単位で別になり、有効化のたびに署名ポリシー側とは別のアカウントが増えることを確認した。

### MSAL・B2Cポリシーの仕様（コードとドキュメントで確認）

- `getAllAccounts()` は順序が保証されない（MSAL公式FAQ）。`accounts[0]` は呼び出しごとに変わり得る。
- active accountは `cacheLocation`（`sessionStorage`）に保存され、再読み込み後も保持される（`@azure/msal-browser` 5.17.1 の `BrowserCacheManager`）。初期化時の `setActiveAccount(accounts[0])` は、保持されたactive accountを上書きする。
- キャッシュ済みトークンは、有効期限内は更新されずに返る。署名ポリシー側アカウントのキャッシュ済みトークンは、更新されるまで2FA未設定のclaimのままとなる。
- 署名ポリシー（`Signin.xml.ejs`）は `Token` endpointに `RedeemRefreshToken` を持ち、更新時に `extension_requireEmailMfa` をディレクトリから再読込する。有効化ポリシー（`ActivateEmailMfa.xml.ejs`）は `Token` endpointを持たない。
- `JwtIssuer` にトークン有効期間の指定はなく、B2Cの既定値が適用される。

### 評価

- 2FA有効化後は、署名側（未設定claim）と有効化側（設定済みclaim）の2アカウントが必ず共存する。
- `accounts[0]` が未設定側になると、再読み込み後に未設定側がactive accountとなり、画面の2FA判定は設定ページへ戻る。
- その状態でAPIトークンを取得すると、同じ未設定側のアカウントのキャッシュ済みトークンが使われ、APIに拒否される。
- 「約1時間続く」は、上記のキャッシュ済みトークンの挙動と整合する。ただし実測はしていない。
- よって、アカウント一覧の先頭に依存する動作を解消する修正と回帰テストが必要。

### 未確認事項

- 再読み込みで設定ページへ戻る、またはAPIが403になる再現。ガードとAPI拒否がPRの内容でステージングに未反映のため、developへの反映後に確認する。
- 「約1時間続く」の実測。
- 有効化側アカウントのトークンが期限切れになったとき、署名ポリシー側で更新できるか。更新できない場合の挙動（再ログインやOTPの要否）は確認していない。
- 「毎回メールOTPが必要」。確認しておらず、確定事実として扱わない。

## 実装タスク

### 1. MSALアカウント選択を一貫させる

対象: `apps/web/src/middlewares/Auth/config.ts`

- 初期化時に、キャッシュ一覧の先頭を無条件にactive accountへ設定する動作を改める。
- 2FA有効化済みアカウントがキャッシュにある場合は、それを選ぶ。MFA判定とAPIトークン取得の双方でその同一アカウントを使う。
- 複数アカウントがある場合の選択基準をコード上で明示する。単に一覧順や偶然のキャッシュ順に依存させない。
- 有効化済みアカウントが存在しない場合の挙動も決める。未設定アカウントが1件だけなら従来どおり設定へ誘導できる。一意に選べない複数アカウントを黙って先頭から選ばない。
- `acquireAccessToken` のフォールバックも初期化時の選択規則と矛盾させない。active accountが欠けたときに未設定アカウントを無条件で使わない。

### 2. ユニットテストを追加する

対象: `apps/web/src/middlewares/Auth/__tests__/config.test.ts`

- MSALモックでactive accountを保持・返却できるようにし、`acquireTokenSilent` が受け取った `account` も検証可能にする。
- 少なくとも次の回帰ケースを追加する。
  1. キャッシュ先頭が2FA未設定、後続に設定済みアカウントがある場合、設定済み側を選択する。
  2. 初期化後、APIトークン取得に設定済み側のアカウントを渡す。
  3. 設定済みアカウントがない／アカウントが複数で一意に決められない場合の選択動作を、採用した方針どおりに確認する。
- 既存の単一アカウント・アカウントなし・リダイレクト応答待ちのテストを維持する。
- 選択ロジックを純粋関数に切り出す場合は、その関数に対するテストでもよい。ただしテストが実際の初期化とAPIトークン選択を保証すること。

### 3. 関連テストを確認する

- `useIsEmailMfaActivated` の既存テストとAuthGuardの既存テストを実行し、active account選択の変更が画面判定・リダイレクトに反映されることを確認する。
- 必要に応じて、複数アカウントのactive accountを使ったAuthGuardテストを追加する。

## ローカル検証

Referyリポジトリのルートで、まず対象テストを実行する。

```bash
npm run test:unit -w apps/web -- src/middlewares/Auth/__tests__/config.test.ts
```

関連するガード・フックのテストも必要に応じて実行する。

```bash
npm run test:unit -w apps/web -- src/middlewares/Auth/__tests__/AuthGuard.test.ts src/hooks/__tests__/useIsEmailMfaActivated.test.ts
```

テストスクリプトが複数のパス指定を受け付けない場合は、各ファイルを個別に指定して実行する。コード変更後は `npm run typecheck -w apps/web` も実行する。

## 完了条件

- 2FA有効化済みと未設定のアカウントが共存し、未設定アカウントが配列先頭にあるケースの回帰テストが成功する。
- MFA判定とAPIアクセストークン取得が同じ有効化済みアカウントに基づく。
- アカウントがない場合や一意に選べない場合の挙動が明示され、暗黙の先頭選択に戻らない。
- 既存のAuthGuard・MFA状態テストと対象アプリの型チェックが成功する。
- PR上のレビューコメントに、修正内容と検証結果を回答する。
