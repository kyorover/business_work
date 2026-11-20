# Refery 2FA必須化 ステージング不具合対応レポート

2FA必須化対応（PR #2488 相当・再読み込み修正含む）のステージング検証で判明した2件の不具合（事象・原因・対応内容）をまとめたものです。

## 概要

| 区分 | 事象 | 変更対象ファイル | 状況 |
| :--- | :--- | :--- | :--- |
| **対応①** | 2FA有効化直後にエラー画面へ遷移、または `/auth/mfa-setup` へ差し戻される | `apps/web/src/middlewares/Auth/config.ts`<br>`apps/web/src/middlewares/Auth/__tests__/config.test.ts` | 修正・動作確認完了 |
| **対応②** | 2FA設定時のB2Cメール認証画面でヘッダーのロゴバーが表示されない | `apps/adb2c/src/templates/activate_email_mfa.html.ejs`<br>`apps/adb2c/src/templates/css/activate_email_mfa.css` | 修正・動作確認完了 |

---

## 対応①：2FA有効化後の画面遷移・APIトークン取得不具合

### 事象
「パスワード登録 → 二要素認証を設定する → 認証コード送信 → 認証 → 二要素認証を有効化」の操作後、ダッシュボードが一瞬表示された直後に以下のいずれかが発生する。

* **パターン1**：画面遷移して「指定されたページは表示できません」となる
* **パターン2**：画面遷移して「二要素認証を設定」画面（`/auth/mfa-setup`）に戻される

※ ダッシュボードが一瞬表示されることから、B2CでのOTP検証・`extension_requireEmailMfa = true` の書き込み・初回ガード通過までは完了しており、後続の業務API用トークン取得およびリダイレクト復帰時の初期化順序で不具合が発生していた。

### 原因
Azure AD B2C はポリシーごとに別アカウントを生成するため、2FA有効化直後の `sessionStorage` には **未設定アカウント（`B2C_1A_signup_signin`）** と **有効化済みアカウント（`B2C_1A_activate_email_mfa`）** が共存する。この状態で `apps/web/src/middlewares/Auth/config.ts` に以下の問題があった。

1. **`ACQUIRE_TOKEN_SUCCESS` での Active Account 無条件上書き（パターン2主因）**
   トークン取得成功のたびに `setActiveAccount(account)` を実行していたため、未設定アカウントのイベントで Active Account が巻き戻り、`AuthGuard` が `/auth/mfa-setup` へリダイレクトしていた。また同コールバック内で `callback.acquireTokenSuccess()`（セッション再取得）も毎回発火し、イベント連鎖を誘発していた。
2. **`acquireAccessToken()` のトークン取得制御不足（パターン1主因）**
   `acquireTokenSilent` に `authority` 指定や初回リフレッシュ制御がなく、未設定クレームのままのキャッシュ済みトークンが業務APIへ送信される／不整合エラーとなる経路があった。また並列API呼び出し時にトークン取得が競合していた。
3. **`setupMsalInstance()` の初期化順序の不備（パターン1・2複合要因）**
   `handleRedirectPromise()` の完了前にアカウント選択を行っていたため、リダイレクト復帰で追加された有効化済みアカウントが初回選択に反映されず、`LOGIN_SUCCESS` 内でもリダイレクト完了を待たずにセッション取得が走っていた。
4. **再認証フォールバックの指定**
   `login_required` / `interaction_required` 時の `acquireTokenRedirect` に `account: activeAccount` をそのまま渡しており、再認証時の不整合リスクがあった。

### 対応内容

#### 1. `apps/web/src/middlewares/Auth/config.ts`
* **初期化順序の是正**：`handleRedirectPromise()` 完了後に `getAllAccounts()` → `selectAccount()` → `setActiveAccount()` を実行。`LOGIN_SUCCESS` では Active Account 設定・リフレッシュ状態リセット・フラグ記録のみ行い、リダイレクト処理完了後にセッション取得を1回だけ `await` する形に変更。
* **`ACQUIRE_TOKEN_SUCCESS` の整理**：`setActiveAccount(account)` と `callback.acquireTokenSuccess()` の呼び出しを削除し、Sentry の `traceUser`（`payload.account && payload.idTokenClaims` ガード付き）のみ実行。Active Account の巻き戻りと不要なセッション再取得を遮断。
* **`acquireAccessToken()` の制御強化**：
  * `authority: msalConfig.auth.authority` を明示指定。
  * 2FA有効化済みアカウントはアカウント識別キー単位でページロード初回のみ `forceRefresh: true` で取得し、最新の `extension_requireEmailMfa: true` クレーム付きトークンを確実に取得（2回目以降は `forceRefresh: false`）。
  * モジュールスコープの `accessTokenPromise` で進行中のトークン取得を共有し、初期描画時の並列API呼び出しによる多重リクエスト・競合を防止。
* **再認証フォールバックの安全化**：`acquireTokenRedirect` の引数を `account` 指定から `loginRequest` へ変更。

#### 2. `apps/web/src/middlewares/Auth/__tests__/config.test.ts`
回帰防止のため以下の9ケースのユニットテストを整備。
* 単一キャッシュアカウントでの初期セッション取得、アカウントなし時の非取得
* 未設定・設定済み共存時の設定済みアカウント選択とAPIトークン取得（初回のみ `forceRefresh: true`、2回目以降は `false`）
* 未設定アカウント複数時の既存 Active Account 維持、曖昧な複数アカウント時の `prompt: 'select_account'` 誘導およびトークン取得エラー化
* `handleRedirectPromise()` 実行中に追加された有効化済みアカウントの選択、および完了待ち後の `loginSuccess` 1回実行
* `ACQUIRE_TOKEN_SUCCESS` 発火時に Active Account が上書きされず、`acquireTokenSuccess` も再発火しないこと

### 動作確認結果
不具合が発生していた一連の2FA有効化フローにおいて、パターン1・パターン2ともに解消し、ダッシュボードが正常に表示・動作することを確認済み。

---

## 対応②：B2Cメール認証画面のヘッダーロゴバー非表示

### 事象
「ログイン → 二要素認証の設定（`/auth/mfa-setup`）→『二要素認証を設定する』」で遷移する B2C メール認証画面（`b2c_1a_activate_email_mfa`）にて、遷移元画面にあったヘッダーのロゴバーが表示されず、「戻る」リンクのみが表示される。

### 原因
従来の2FA有効化画面はアプリ内の個人設定画面から開く前提だったため、B2C テンプレート（`activate_email_mfa.html.ejs` / `activate_email_mfa.css`）にロゴバー（`.logoBar`）のマークアップおよびスタイルが実装されていなかった。2FA必須化によりロゴバー付きの初期設定画面から直接遷移する導線になったことで、遷移時にヘッダーが消えたように見える状態となっていた。

### 対応内容

#### 1. `apps/adb2c/src/templates/activate_email_mfa.html.ejs`
* `<div class="base">` 直下、`<header class="header">` の前にロゴバー領域（`.logoBar`）を追加し、既存のロゴ画像（`<%= assetUrl %>/assets/logo-refery.svg`）を表示するように修正。

#### 2. `apps/adb2c/src/templates/css/activate_email_mfa.css`
* 追加した `.logoBar` のスタイル（高さ `56px`、余白 `12px 24px`、白背景、`flex-shrink: 0`）を定義。
* ロゴバー追加に伴い、直下の `.header` の上余白が過剰にならないよう `padding` を `24px 40px 0` から `0 40px` へ調整。

※ なお、本画面は Azure Blob Storage 上の静的ファイルから配信される構成のため、画面への反映にはコードマージとは別にテンプレート反映が必要です（反映手順は本資料の対象外）。