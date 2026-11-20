# Refery 2FA必須化 ステージング不具合対応 統合レポート（MSAL認証制御・B2C認証画面ヘッダー修正）

**全体ステータス**
* **対応①（2FA有効化後の画面遷移・トークン取得不具合）**：修正実施・動作確認完了
* **対応②（B2Cメール認証画面のヘッダーロゴバー非表示）**：コード修正実施済み（元コミット `54b8a51` / 旧PR #2492）・Azure Blob反映待ち

本PRでは、2FA必須化対応（PR #2488 相当・再読み込み修正含む）のステージング検証で判明した以下2件の不具合修正を一括で反映する。

| 対応区分 | 対象領域 | 変更対象ファイル | 反映経路 | ステータス |
| :--- | :--- | :--- | :--- | :--- |
| **対応①** | フロントエンド MSAL 制御・ユニットテスト | `apps/web/src/middlewares/Auth/config.ts`<br>`apps/web/src/middlewares/Auth/__tests__/config.test.ts` | PRマージによる自動デプロイ（Cloud Build / Cloud Run） | 修正実施・動作確認完了 |
| **対応②** | Azure AD B2C 静的HTML/CSSテンプレート | `apps/adb2c/src/templates/activate_email_mfa.html.ejs`<br>`apps/adb2c/src/templates/css/activate_email_mfa.css` | **ローカルビルド後、Azure Blob Storage へ手動アップロードが必要** | コード修正実施済み・Azure Blob反映待ち |

---

## 1. 事象サマリー

### 1-1. 対応①：2FA有効化後の画面遷移・APIトークン取得不具合

2FA必須化対応（PR #2488 相当・再読み込み修正含む）をステージング環境に反映後、以下の2パターンの不具合が発生していた。

| パターン | 操作フロー | 発生事象 |
| :--- | :--- | :--- |
| **パターン1** | パスワード登録 → 二要素認証を設定する → 認証コード送信 → 認証 → 二要素認証を有効化 | ダッシュボードが一瞬表示された後、画面遷移して **「指定されたページは表示できません」** となる |
| **パターン2** | 同上 | ダッシュボードが一瞬表示された後、画面遷移して **「二要素認証を設定」画面（`/auth/mfa-setup`）に戻される** |

**挙動の前提整理**

* **「ダッシュボードが一瞬表示される」** ことから、B2C 有効化ポリシー（`B2C_1A_activate_email_mfa`）でのメールOTP検証および `extension_requireEmailMfa = true` の書き込み、`/common/session/me` の取得、初回ガード通過までは完了していた。
* 不具合は、**ダッシュボードがマウントされて業務API（`/tnt/...` や `/gst/...`）用のアクセストークン取得（`acquireAccessToken`）が走ったタイミング**、および **リダイレクト復帰時の初期化順序・イベント連鎖** によって発生していた。

### 1-2. 対応②：B2Cメール認証画面でのヘッダーロゴバー非表示

2FA必須化対応のステージング検証において、二要素認証の初期設定画面からメール認証画面へ遷移した際、以下の表示不具合が発生した。

| 対象画面 | 操作フロー | 発生事象 |
| :--- | :--- | :--- |
| **メールアドレスの認証（認証コード入力）画面** | ログイン → 二要素認証の設定（`/auth/mfa-setup`）→「二要素認証を設定する」ボタン押下 | 遷移先の B2C ホスト画面（`id.refery-app-dev.com/.../b2c_1a_activate_email_mfa/...`）で **ヘッダーのロゴバーが表示されず、「戻る」リンクのみが表示される** |

**挙動の前提整理**

* 遷移元の「二要素認証の設定」画面（`/auth/mfa-setup`）は Next.js（`apps/web/src/pages/auth/mfa-setup.tsx`）で描画されており、Refery のロゴバーが表示される。
* 一方、「二要素認証を設定する」押下後の画面は **Azure AD B2C のカスタムポリシー（`B2C_1A_activate_email_mfa`）が `LoadUri` 経由で Azure Blob Storage 上の静的 HTML / CSS を読み込んで描画** している。
* そのため、Web アプリ（`refery-app-dev.com`）とは配信元が異なり、Cloud Build の個別トリガー（`refery-web-staging` / `refery-api-staging`）や `develop` への PR マージを行っても、Azure Blob 上のテンプレートが更新されない限り画面表示は変化しない。

---

## 2. 根本原因

### 2-1. 対応①の根本原因（MSALアカウント競合・初期化順序・トークン取得制御）

Azure AD B2C ではポリシーごとに別アカウント・別キャッシュが生成されるため、2FA有効化直後の `sessionStorage` には以下2つのアカウントが共存する。

1. **ログイン時アカウント**：`B2C_1A_signup_signin`（`extension_requireEmailMfa` が `false` または未設定のキャッシュ済みトークンを保持）
2. **2FA有効化時アカウント**：`B2C_1A_activate_email_mfa`（`extension_requireEmailMfa: true`）

この状態で、修正前の `apps/web/src/middlewares/Auth/config.ts` に以下の問題があり、2パターンの不具合を引き起こしていた。

| # | 原因箇所（修正前） | 問題内容 | 発生していた不具合との対応 |
| :--- | :--- | :--- | :--- |
| 1 | `ACQUIRE_TOKEN_SUCCESS` コールバック | アクセストークン取得成功のたびに **`msalInstance.setActiveAccount(account)` を無条件実行** していた。未設定側アカウントで解決されたイベントが飛ぶと、有効化済みの Active Account が未設定アカウントへ上書き（巻き戻り）され、`AuthGuard` が `/auth/mfa-setup` へリダイレクトしていた。 | **パターン2（主因）** |
| 2 | `acquireAccessToken()` | `acquireTokenSilent` に `authority` 指定や初回リフレッシュ制御がなく、キャッシュされた未設定側トークンがそのまま業務APIへ送信される／不整合エラーとなる経路があった。また、並列API呼び出し時にトークン取得が競合していた。 | **パターン1（主因）** |
| 3 | `setupMsalInstance()` の初期化順序 | **`handleRedirectPromise()` の実行前** に `getAllAccounts()` / `selectAccount()` / `setActiveAccount()` を行っていたため、リダイレクト復帰で追加された2FA有効化済みアカウントが初回選択に反映されなかった。また、`LOGIN_SUCCESS` 内でリダイレクト処理完了を待たずにセッション取得を走らせていた。 | **パターン1・2（複合要因）** |
| 4 | `ACQUIRE_TOKEN_SUCCESS` のセッション再取得 | トークン取得成功のたびに `callback.acquireTokenSuccess()`（＝`sessionApi.getMe()`）が発火しており、不要なセッション再取得とイベント連鎖が起きていた。 | **競合の誘発** |
| 5 | `acquireTokenRedirect` フォールバック | `login_required` / `interaction_required` 時に `account: activeAccount` をそのまま渡してリダイレクトしていた。 | **再認証時の不整合リスク** |

### 2-2. 対応②の根本原因（B2Cテンプレートのマークアップ不足・Blob未反映）

本事象は「テンプレート自体の実装不足」と「デプロイ経路の相違（未反映）」の2点に起因する。

| # | 原因箇所 | 問題内容 | 影響・補足 |
| :--- | :--- | :--- | :--- |
| 1 | `apps/adb2c/src/templates/activate_email_mfa.html.ejs`<br>`apps/adb2c/src/templates/css/activate_email_mfa.css` | 従来の2FA有効化画面はアプリ内の個人設定画面から開く前提だったため、テンプレート側に「戻る」リンク（`.header`）しかなく、**ロゴバー（`.logoBar`）のマークアップおよびスタイルが存在しなかった**。 | 2FA必須化に伴い初期設定画面（ロゴバーあり）から直接遷移する導線になったことで、遷移時にヘッダーバーが消えたように見える状態となっていた。 |
| 2 | ステージング配信元（Azure Blob Storage） | 配信元である `referydevelopmentstorage` 上の `root/activate_email_mfa.html` および `root/css/activate_email_mfa.css` が修正前のままであり、CI/CD（GitHub Actions・Cloud Build）にも `apps/adb2c` のビルドおよび Blob アップロード処理が存在しない。 | コード修正をブランチへ反映・デプロイしても、手動で Blob へ上書きアップロードするまでステージング・本番ともに見た目が変わらない。 |

---

## 3. 実施した修正内容（差分まとめ）

いずれの対応においても、既存の B2C カスタムポリシー XML（`Signin.xml.ejs` / `ActivateEmailMfa.xml.ejs` 等）は変更せず、フロントエンドの MSAL 制御・テストおよび B2C 静的テンプレートの計4ファイルのみを修正した。

### 3-1. 対応①：`apps/web/src/middlewares/Auth/config.ts`

1. **初期化順序の是正（`handleRedirectPromise()` 完了後にアカウント選択・セッション取得）**
   * `setupMsalInstance()` 内で `handleRedirectPromise()` の完了を待ってから `getAllAccounts()` → `selectAccount()` → `setActiveAccount()` を実行するように変更。
   * `LOGIN_SUCCESS` イベントでは `setActiveAccount(account)`、`refreshedEmailMfaAccountKey = null`、フラグ `loginSucceeded = true` の設定のみにとどめ、リダイレクト処理完了後に `loginSucceeded` に応じて `callback.loginSuccess()` または `callback.acquireTokenSuccess()` を1回だけ `await` する形に変更。

2. **`ACQUIRE_TOKEN_SUCCESS` による Active Account 上書き・セッション再取得の廃止**
   * `ACQUIRE_TOKEN_SUCCESS` コールバックから `msalInstance.setActiveAccount(account)` と `callback.acquireTokenSuccess()` の呼び出しを削除し、Sentry の `traceUser`（`payload.account && payload.idTokenClaims` ガード付き）のみを実行するように変更。
   * これにより、業務API用のアクセストークン取得によって Active Account が未設定アカウントへ巻き戻る経路と、不要なセッション再取得の連鎖を完全に遮断。

3. **`acquireAccessToken()` のトークン取得制御（初回 `forceRefresh`・`authority` 明示・並列呼び出しの共有）**
   * `authority: msalConfig.auth.authority` を明示指定。
   * 2FA有効化済みアカウント（`isEmailMfaActivated(activeAccount)`）について、アカウント識別キー（`getAccountKey(activeAccount)`）単位でページロード初回のみ `forceRefresh: true` を指定して `acquireTokenSilent` を実行し、完了後に `refreshedEmailMfaAccountKey` へ記録（2回目以降は `forceRefresh: false` でキャッシュを利用）。
     * これにより、2FA有効化直後であってもキャッシュ済みの旧トークン（未設定クレーム）を掴まず、最新の `extension_requireEmailMfa: true` を持つアクセストークンを確実に取得できる。
   * モジュールスコープの `accessTokenPromise` を導入し、ダッシュボード初期描画時などに複数APIから並列で `acquireAccessToken()` が呼ばれた場合でも、進行中のトークン取得 Promise を共有して多重リクエスト・多重リフレッシュを防止。

4. **再認証フォールバックの安全化**
   * `acquireAccessToken()` の `catch` 節（`login_required` / `interaction_required`）でのリダイレクトを、`account: activeAccount` 指定から `msalInstance.acquireTokenRedirect(loginRequest)` へ変更。

### 3-2. 対応①：`apps/web/src/middlewares/Auth/__tests__/config.test.ts`

以下の9ケースのユニットテストを整備し、今回修正した動作と回帰防止を担保した。

| # | テストケース | 検証内容 |
| :--- | :--- | :--- |
| 1 | `fetches the initial session for a cached account` | キャッシュ済み単一アカウントで初期セッションを取得すること |
| 2 | `selects the activated account and uses it to acquire an API token` | 未設定・設定済みアカウント共存時に設定済みを選択し、`forceRefresh: true` でAPIトークンを取得すること |
| 3 | `keeps the existing active account when multiple accounts are unactivated` | 未設定アカウントが複数ある場合に既存の Active Account を維持すること |
| 4 | `prompts for an account instead of silently choosing an ambiguous account` | 曖昧な複数アカウント時に黙って先頭を選ばず `prompt: 'select_account'` で誘導し、トークン取得もエラーにすること |
| 5 | `does not fetch the session when no account is cached` | キャッシュアカウントがない場合にセッション取得・リダイレクトを行わないこと |
| 6 | `selects an activated account added by redirect processing` | `handleRedirectPromise()` 実行中に追加された2FA有効化済みアカウントを選択すること |
| 7 | `does not replace the active account on access token acquisition` | `ACQUIRE_TOKEN_SUCCESS` で未設定アカウントが来ても Active Account が上書きされず、`acquireTokenSuccess` も再発火しないこと |
| 8 | `force refreshes an activated account only once per page load` | 2FA有効化済みアカウントの `forceRefresh: true` は1回目のみで、2回目以降は `forceRefresh: false` になること |
| 9 | `waits for the redirect response before fetching the session` | `handleRedirectPromise()` の完了を待ってから `loginSuccess` が1回だけ実行され、`acquireTokenSuccess` と二重実行されないこと |

### 3-3. 対応②：`apps/adb2c/src/templates/activate_email_mfa.html.ejs`

* `<div class="base">` 直下、`<header class="header">` の前にロゴバー領域（`.logoBar`）を追加し、既存のロゴ画像（`<%= assetUrl %>/assets/logo-refery.svg`）を表示するように変更（元コミット: `54b8a51` / 旧PR #2492）。

~~~html
  <body>
    <div class="base">
      <div class="logoBar">
        <img
          alt="logo"
          src="<%= assetUrl %>/assets/logo-refery.svg"
          width="102"
          height="33"
        />
      </div>
      <header class="header">
        <img src="<%= assetUrl %>/assets/chevron_left.svg" alt="<" />
        <a href="javascript:history.back();">戻る</a>
~~~

### 3-4. 対応②：`apps/adb2c/src/templates/css/activate_email_mfa.css`

* 追加した `.logoBar` のレイアウトスタイル（高さ `56px`、余白 `12px 24px`、白背景、`flex-shrink: 0`）を定義。
* ロゴバー追加に伴い、直下の `.header` の上余白が過剰にならないよう `padding` を `24px 40px 0` から `0 40px` へ調整。

~~~css
.logoBar {
  width: 100%;
  height: 56px;
  padding: 12px 24px;
  background-color: #fff;
  flex-shrink: 0;
}

.header {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 0 40px;
  gap: 8px;
  width: 100%;
~~~

---

## 4. デプロイ・反映手順および動作確認状況

### 4-1. 対応①（MSAL制御）の動作確認結果

* 修正を適用後、不具合が発生していたフロー（パスワード登録 → 二要素認証を設定する → 認証コード送信 → 認証 → 二要素認証を有効化）において、パターン1（エラー画面遷移）・パターン2（設定画面への差し戻し）のいずれも解消し、正常にダッシュボードが表示・動作することを確認済み。

### 4-2. 対応②（B2Cテンプレート）のデプロイ・反映手順および確認状況

現在のリポジトリ構成には `apps/adb2c` の自動デプロイパイプラインが存在しないため、本PRのマージとは別に、ローカル環境でビルドした静的ファイル（`.html` / `.css`）を Azure Portal から対象ストレージアカウントへ手動で上書きアップロードする必要がある。

#### 事前準備（初回のみ）

1. **Microsoft 365 アカウントの作成**：Azure Portal ログイン用アカウント（未所持の場合、corporate へ申請）。
2. **Azure ストレージ書き込み権限の付与**：Azure Portal の **`medup.jp` ディレクトリ**配下にある以下ストレージアカウントへの Blob 操作権限を取得。
   * ステージング用：`referydevelopmentstorage`
   * 本番用：`referyproductionstorage`
   * ※Blob の上書きのみであれば B2C テナント（`refery development`）側の操作は不要だが、今後の運用を見据えて併せて招待・権限確認を行う。

#### ビルドおよび Azure Blob へのアップロード手順

1. **対象ブランチでビルドを実行**（`.ejs` の `<%= assetUrl %>` を展開した静的ファイルを生成）
   ~~~bash
   cd apps/adb2c
   npm run build
   ~~~
   * ステージング検証時：ローカルの `feature` ブランチでビルドした成果物を使用可。
   * 正式反映・本番リリース時：マージ後の最新 `develop` / `main` を `git pull` してからビルドする。

2. **Azure Portal から対象 Blob を上書きアップロード**（ステージングの場合）
   * [Azure Portal](https://portal.azure.com/) にログインし、右上のディレクトリが **`medup.jp`** であることを確認。
   * ストレージアカウント **`referydevelopmentstorage`** ＞「コンテナー」＞ **`root`** を開く。
   * 切り戻し用に既存の `activate_email_mfa.html` と `css/activate_email_mfa.css` をダウンロードして控える。
   * 以下の2ファイルを上書きアップロードする（カスタムポリシー XML の再アップロードは不要）。

   | ローカルビルド成果物（STG） | アップロード先（`referydevelopmentstorage`） |
   | :--- | :--- |
   | `out/templates/staging/activate_email_mfa.html` | `root/activate_email_mfa.html` |
   | `out/templates/staging/css/activate_email_mfa.css` | `root/css/activate_email_mfa.css` |

3. **反映確認**
   * 以下の Blob URL を直接ブラウザで開き、`.logoBar` が含まれていることを確認する。
     * `https://referydevelopmentstorage.blob.core.windows.net/root/activate_email_mfa.html`
     * `https://referydevelopmentstorage.blob.core.windows.net/root/css/activate_email_mfa.css`
   * 参照先のロゴ画像（`https://referydevelopmentstorage.blob.core.windows.net/root/assets/logo-refery.svg`）が 404 にならず表示されることを確認する。
   * ステージング（`refery-app-dev.com`）の 2FA 設定フローでヘッダーのロゴバーが表示されることを確認する（反映されない場合はシークレットウィンドウやキャッシュ状態を確認）。

#### ローカル環境での事前確認について

* ローカル環境（`localhost:3003`）の認証はステージングの B2C テナントおよび Blob（`referydevelopmentstorage`）を間借りしているため、**ローカルビルドのみでは実認証フローを通した画面反映の確認はできない**。
* アップロード前に見た目だけ簡易確認したい場合は、`out/templates/staging/activate_email_mfa.html` を直接ブラウザで開くか、Chrome DevTools の「Overrides（オーバーライド）」機能を用いて手元ブラウザ上のみで差し替え確認を行う。

---

## 5. 設計・運用上のポイント（セルフレビュー・申し送り）

### 5-1. 実装・設計面（対応①）

* **B2Cポリシーを変更せず既存構成を維持**：`ActivateEmailMfa.xml.ejs` には `RedeemRefreshToken` が定義されていないが、業務API向けトークン取得は `msalConfig.auth.authority`（`B2C_1A_signup_signin`）に対して初回 `forceRefresh: true` で更新取得する方式としたため、B2Cポリシー改修なしで要件を満たし、かつ1時間後のリフレッシュ更新も通常どおり機能する。
* **並列API呼び出し耐性**：ダッシュボード遷移直後に複数APIが一斉に `acquireAccessToken()` を呼んでも、`accessTokenPromise` による in-flight 共有により `forceRefresh: true` の二重実行やレースコンディションが起きない設計となっている。
* **既存ドキュメントとの整合**：`Refery 2要素認証（2FA）必須化 実装・検証.md` の `config.ts` / `config.test.ts` 欄にも、今回の「`ACQUIRE_TOKEN_SUCCESS` での Active Account 上書き廃止」「初回 `forceRefresh` と並列取得の共有」の要点を追記しておくと、後続保守時の仕様と実装の乖離を防げる。

### 5-2. デプロイ・運用面（対応②）

* **PR マージでは自動反映されない点に注意**：GitHub Actions および Cloud Build（`cloudbuild.stg.yml` / `cloudbuild.prd.yml`）は Cloud Run 上の Web / API のみを対象としている。**本番リリース時にも、`main` ブランチ最新からビルドした `out/templates/production/` 配下の2ファイルを `referyproductionstorage` の `root/`・`root/css/` へ手動アップロードする必要がある**（staging 用の成果物を誤って本番へ上げないよう注意）。
* **共有環境への即時反映と切り戻し**：Azure Blob への上書きはブランチ単位の分離ができず、ステージング環境全体へ即時反映される。作業前に必ず既存ファイルのバックアップを取得しておくこと。
* **キャッシュによる反映遅延の可能性**：構成上、B2C 向けに Azure Front Door / CDN（`medup-refery-b2c-development` / `medup-refery-b2c-frontdoor-production`）が存在する。Blob URL 上では更新されているのに実画面へ反映されない場合は、ブラウザキャッシュに加えて Front Door 側のキャッシュ影響も確認する。
* **リリース手順書への追記推奨**：チーム内に B2C テンプレート更新の明文化された手順が見当たらなかったため、今後の保守・リリース漏れ防止として、リリース手順書へ STG / PRD それぞれの Blob 反映作業を追記しておくことを推奨する。