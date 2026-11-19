# Git開発フロー（develop最新化 〜 featureブランチPushまで）
※ `git clone` 完了済みの前提手順です。

## 1. developブランチの最新化
* **現在のブランチ確認**（`develop` 以外にいる場合は `git switch develop` で移動）
  ```bash
  git branch
  ```
* **リモートの最新変更を取り込む**
  ```bash
  git pull origin develop
  ```

## 2. 作業用（feature）ブランチの作成
* **新規ブランチを作成して切り替える**
  ```bash
  git switch -c feature/XXXXXX
  # 例: git switch -c feature/add-2fa-notice-to-invite-mail
  ```

*(※ ここでコードの実装・編集作業を行います)*

## 3. 変更内容の確認とステージング
* **変更・追加されたファイルの一覧を確認する**
  ```bash
  git status
  ```
* **コミット対象としてステージングする**（単一 / 複数指定）
  ```bash
  git add <対象ファイルのパス>
  git add <対象ファイルのパス1> <対象ファイルのパス2>
  # 例: git add apps/api/.../InviteEmailSender.ts apps/foro-command-seraddvice/.../InviteEmailSender.ts
  ```
* **ステージングされたか再確認する**
  ```bash
  git status
  ```

## 4. コミットとリモートへのプッシュ
* **変更内容をメッセージ付きでコミットする**
  ```bash
  git commit -m "XXXXXXXX"
  # 例: git commit -m "feat: アカウント招待メールに2要素認証を促す一文を追加"
  ```
* **作成したブランチをリモートへプッシュする**
  ```bash
  git push origin feature/XXXXXX
  # 例: git push origin feature/add-2fa-notice-to-invite-mail
  ```
