# 機能別 CRUD 一覧

## medup-core-service 作成分

以下の内容は `medup-core-service` の既存資料として維持する。`foro-crm-api` 固有のCRUDは文末の追記節に記載する。

## 読み方

- `C`：作成、取込、登録、アップロード。`R`：一覧・詳細・検索・集計。`U`：更新、置換、状態変更、upsert。`D`：削除、アーカイブ。
- `○` は、このリポジトリ内のルートまたはユースケースから確認できる操作。`-` は未確認または非提供。
- 画面に機能名があっても、処理本体が別リポジトリにあるものは除外している。
- 署名 URL 発行はファイル処理を伴うため C、AI 要約・文字起こし等の非永続処理は注記のみとする。
- 「関連テーブル」は、実装から確認できる CRUD 対象または直接参照先を物理名で記載する。複雑な結合の全テーブルではなく、機能に直接関与する範囲に限定する。
- テーブルの詳細は [DB定義](./40_DB定義/DB定義.md)、画面・処理の概要は [機能一覧](./機能一覧.md) を参照。

## 管理 REST API

実装：[apps/api/src/apis/routes/admin](../apps/api/src/apis/routes/admin)

| 機能                             |  C  |  R  |  U  |  D  | 関連テーブル                                                                                                                                                           | 補足                                         |
| :------------------------------- | :-: | :-: | :-: | :-: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------- |
| テナント                         |  ○  |  ○  |  ○  |  ○  | `tenants`、`medical_institutions`、`medical_institution_labels`                                                                                                        | 一覧・詳細、作成、更新、削除                 |
| テナントアカウント               |  ○  |  ○  |  ○  |  -  | `users`、`tenants`                                                                                                                                                     | 一覧・詳細、作成、更新、パスワード再設定     |
| 認証中管理者                     |  -  |  ○  |  -  |  -  | `medup_foro_admins`                                                                                                                                                    |                                              |
| 許可 IP アドレス                 |  ○  |  -  |  -  |  ○  | `crm_allowed_ip_addresses`、`op_optimizer_allowed_ip_addresses`、`outpatient_allowed_ip_addresses`                                                                     |                                              |
| 自部署                           |  ○  |  ○  |  ○  |  -  | `own_departments`、`own_department_tenants`                                                                                                                            |                                              |
| 自部署グループ                   |  ○  |  ○  |  ○  |  ○  | `own_department_groups`、`own_department_group_own_departments`                                                                                                        | 一括更新を含む                               |
| 自部署マッピング                 |  -  |  ○  |  ○  |  -  | `own_department_mappings`、`own_departments`                                                                                                                           | 一括置換                                     |
| 診療科                           |  ○  |  ○  |  ○  |  -  | `specialties`、`specialty_groups`                                                                                                                                      |                                              |
| 生診療科・診療科マッピング       |  ○  |  ○  |  ○  |  ○  | `department_mappings`、`department_updated_logs`、`medical_institutions`                                                                                               | 未マッピング取得、一括更新、反映、アーカイブ |
| 施設種別グループ                 |  -  |  ○  |  -  |  -  | `facility_type_groups`、`facility_types`                                                                                                                               |                                              |
| テナント標準施設種別             |  -  |  -  |  ○  |  -  | `tenant_default_facility_types`                                                                                                                                        | 置換                                         |
| 医療機関                         |  ○  |  ○  |  ○  |  -  | `medical_institutions`、`medical_institution_bulk_insert_job_statuses`、`medical_institution_logs`                                                                     | 詳細・検索、一括登録、登録状態               |
| 競合医療機関                     |  -  |  ○  |  ○  |  -  | `crm_tenant_competitive_medical_institutions`、`medical_institutions`                                                                                                  |                                              |
| 医療機関マッピング               |  -  |  ○  |  ○  |  -  | `medical_institution_mappings`、`medical_institutions`                                                                                                                 |                                              |
| 医療機関マッピング作業シート     |  ○  |  ○  |  -  |  ○  | `medical_institution_mapping_work_sheets`、`medical_institution_mapping_work_sheet_sections`、`medical_institution_mapping_work_sheet_reviews`                         | 完了処理を含む                               |
| 医療機関マッピング作業セクション |  -  |  ○  |  ○  |  -  | `medical_institution_mapping_work_sheet_sections`、`medical_institution_mapping_work_sheet_section_records`                                                            | レビュー内容更新・完了                       |
| 医療機関マッピングレビュー       |  ○  |  ○  |  ○  |  -  | `medical_institution_mapping_work_sheet_reviews`、`medical_institution_mapping_work_sheet_review_records`                                                              | レビュー作成・完了                           |
| セグメント設定                   |  -  |  ○  |  ○  |  -  | `medup_foro_segmentation_settings`、`medup_foro_excluded_own_department_segmentation_settings`、`medup_foro_segments`                                                  |                                              |
| 入退院調整設定                   |  -  |  ○  |  ○  |  -  | `acute_cf_patient_status_settings`                                                                                                                                     | 有効ステータス                               |
| データ取込ログ                   |  ○  |  ○  |  ○  |  -  | `data_importing_logs`                                                                                                                                                  | 受領ログ作成、取込済み状態更新               |
| 病院入院データ                   |  ○  |  ○  |  -  |  ○  | `hospital_admission_importing_logs`、`hospital_admissions`、`tenant_data_availabilities`                                                                               | 取込履歴・可用性、取込、削除                 |
| 外来 EF データ                   |  ○  |  ○  |  -  |  ○  | `medup_foro_outpatient_ef_importing_logs`、`tenant_data_availabilities`                                                                                                | 取込履歴・可用性、取込、削除                 |
| 紹介データ                       |  -  |  ○  |  -  |  ○  | `referral_importing_logs`、`referrals`                                                                                                                                 | 取込一覧、取込データ削除                     |
| 逆紹介データ                     |  -  |  ○  |  -  |  ○  | `reverse_referral_importing_logs`、`reverse_referrals`                                                                                                                 | 取込一覧、取込データ削除                     |
| 紹介マスタ                       |  -  |  ○  |  -  |  -  | `referral_masters`、`medical_institutions`、`own_departments`                                                                                                          |                                              |
| 逆紹介マスタ                     |  -  |  ○  |  -  |  -  | `reverse_referral_masters`、`medical_institutions`、`own_departments`                                                                                                  |                                              |
| 紹介マスタ作業シート             |  ○  |  ○  |  ○  |  ○  | `referral_master_work_sheets`、`referral_master_work_sheet_records`、`referral_masters`                                                                                | ID 付与、一括更新、反映                      |
| 逆紹介マスタ作業シート           |  ○  |  ○  |  ○  |  ○  | `reverse_referral_master_work_sheets`、`reverse_referral_master_work_sheet_records`、`reverse_referral_masters`                                                        | ID 付与、一括更新、反映                      |
| マッチング                       |  ○  |  -  |  -  |  -  | `matching_logs`                                                                                                                                                        | ファイルアップロード・実行依頼               |
| マッチングログ                   |  -  |  ○  |  -  |  -  | `matching_logs`、`tenants`                                                                                                                                             | 実行履歴、ログ URL                           |
| 厚生労働省データ同期作業         |  ○  |  ○  |  ○  |  -  | `mhlw_sync_works`、`mhlw_sync_work_insert_records`、`mhlw_sync_work_update_records`、`mhlw_sync_work_close_records`、`mhlw_sync_work_monthly_report_import_logs`       | 作成、CSV 取込、候補更新、照合・分類         |
| 厚生労働省一覧差分               |  ○  |  ○  |  ○  |  -  | `mhlw_sync_work_overview_diff_records`、`mhlw_sync_work_overview_only_list_records`、`mhlw_sync_work_overview_only_mis_records`、`mhlw_sync_work_overview_import_logs` | 差分取込、一次保存、分類・確定               |
| レポート                         |  ○  |  -  |  -  |  -  | `report_import_logs`                                                                                                                                                   | テンプレートから生成                         |
| レポート取込ログ                 |  -  |  ○  |  -  |  -  | `report_import_logs`                                                                                                                                                   |                                              |
| レポートテンプレート             |  -  |  ○  |  -  |  -  | なし（外部ストレージ）                                                                                                                                                 | 一覧・画像 URL                               |
| 完了通知先                       |  ○  |  ○  |  ○  |  -  | `completion_notification_users`                                                                                                                                        |                                              |
| テナント医療機関コード           |  ○  |  ○  |  -  |  -  | `tenant_medical_institution_code_ids`                                                                                                                                  |                                              |

## 入退院調整 tRPC

実装：[command](../apps/foro-command-service/src/modules/acute_care_flow)、[query](../apps/foro-query-service/src/modules/acute_care_flow)

| 機能               |  C  |  R  |  U  |  D  | 関連テーブル                                                                                                      | 補足                                 |
| :----------------- | :-: | :-: | :-: | :-: | :---------------------------------------------------------------------------------------------------------------- | :----------------------------------- |
| 患者               |  ○  |  ○  |  ○  |  ○  | `acute_cf_patients`、`acute_cf_patient_status_histories`、`acute_cf_patient_diseases`                             | 一覧・詳細、登録、更新、削除         |
| 患者社会情報       |  ○  |  ○  |  ○  |  -  | `acute_cf_patient_social_informations`                                                                            | upsert                               |
| 患者詳細・機微情報 |  -  |  ○  |  -  |  -  | `acute_cf_patient_details`、`acute_cf_patient_sensitive_details`                                                  | 権限制御付き参照                     |
| 患者ステータス件数 |  -  |  ○  |  -  |  -  | `acute_cf_patient_status_histories`                                                                               | 現在・検索条件別                     |
| 患者添付ファイル   |  -  |  ○  |  -  |  ○  | `acute_cf_attachment_files`                                                                                       | 一覧・参照 URL・削除                 |
| 入院実績           |  -  |  ○  |  -  |  -  | `acute_cf_patients`、`acute_cf_disease_masters`、`acute_cf_referral_medical_institutions`、`medical_institutions` | 集計                                 |
| アカウント         |  ○  |  ○  |  ○  |  ○  | `users`、`tenants`                                                                                                |                                      |
| 疾患マスタ         |  -  |  ○  |  ○  |  -  | `acute_cf_disease_masters`                                                                                        |                                      |
| 病棟マスタ         |  -  |  ○  |  ○  |  -  | `acute_cf_ward_masters`                                                                                           |                                      |
| 紹介医療機関       |  ○  |  ○  |  ○  |  -  | `acute_cf_referral_medical_institutions`、`medical_institutions`                                                  | 一括作成・一括保存を含む             |
| 医療機関           |  -  |  ○  |  -  |  -  | `medical_institutions`                                                                                            | 選択候補・一覧                       |
| 利用者・MFA        |  -  |  ○  |  ○  |  -  | `users`、`tenants`、`acute_cf_patient_status_settings`                                                            | MFA 無効化は Azure 側、設定参照は DB |

## CRM tRPC

実装：[command](../apps/foro-command-service/src/modules/crm)、[query](../apps/foro-query-service/src/modules/crm)

| 機能                       |  C  |  R  |  U  |  D  | 関連テーブル                                                                                                                                                                                                                                          | 補足                                        |
| :------------------------- | :-: | :-: | :-: | :-: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------ |
| CRM 医療機関               |  -  |  ○  |  -  |  -  | `medical_institutions`、`medical_institution_specialties`、`medical_institution_facility_types`、`medical_institution_labels`、`medical_institution_label_values`、`tags`、`tag_medical_institutions`、`crm_medical_institution_phases`、`ref_naviis` | 検索・詳細                                  |
| 医療機関特記事項           |  -  |  ○  |  -  |  -  | `crm_medical_institution_special_notes`                                                                                                                                                                                                               |                                             |
| 医療機関添付ファイル       |  ○  |  ○  |  -  |  ○  | `medical_institution_attachment_files`                                                                                                                                                                                                                | 署名 URL、登録、プレビュー URL、削除        |
| コミュニケーションラベル   |  -  |  ○  |  ○  |  -  | `communication_labels`、`communication_default_items`                                                                                                                                                                                                 | 表示順更新                                  |
| コミュニケーション音声     |  ○  |  -  |  -  |  ○  | なし（外部ストレージ）                                                                                                                                                                                                                                | 署名 URL 発行、アップロード済みファイル削除 |
| VOC                        |  -  |  ○  |  -  |  -  | `crm_vocs`、`crm_communication_vocs`、`crm_voc_groups`、`communications`、`communication_label_values`                                                                                                                                                | 検索・個別・集計                            |
| VOC グループ               |  -  |  ○  |  -  |  -  | `crm_voc_groups`                                                                                                                                                                                                                                      |                                             |
| フェーズ                   |  -  |  ○  |  -  |  -  | `crm_phase_masters`                                                                                                                                                                                                                                   |                                             |
| 自部署グループ             |  -  |  ○  |  -  |  -  | `own_department_groups`、`own_department_group_own_departments`                                                                                                                                                                                       |                                             |
| データ同期状況             |  -  |  ○  |  -  |  -  | `data_importing_logs`、`tenants`                                                                                                                                                                                                                      |                                             |
| 利用者・MFA                |  -  |  ○  |  ○  |  -  | `users`、`tenants`                                                                                                                                                                                                                                    | MFA 無効化は Azure 側で、DB は利用者参照    |
| コミュニケーション要約     |  -  |  -  |  -  |  -  | `communications`                                                                                                                                                                                                                                      | AI 計算の入力として参照                     |
| 音声文字起こし             |  -  |  -  |  -  |  -  | なし（外部ストレージ・AI）                                                                                                                                                                                                                            | 永続 CRUD ではない                          |
| 医療機関向け AI アドバイス |  -  |  -  |  -  |  -  | `tenants`、`communications`、`referrals`、`reverse_referrals`、`medical_institutions`                                                                                                                                                                 | 推論用データを参照                          |

## 連携マップ tRPC

実装：[command](../apps/foro-command-service/src/modules/outpatient)、[query](../apps/foro-query-service/src/modules/outpatient)

| 機能               |  C  |  R  |  U  |  D  | 関連テーブル                                                                                                                                                                 | 補足                                   |
| :----------------- | :-: | :-: | :-: | :-: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------- |
| 医療機関検索・地図 |  -  |  ○  |  -  |  -  | `medical_institutions`、`medical_institution_specialties`、`medical_institution_facility_types`、`medical_institution_label_values`、`ref_naviis`、`municipalities`          | 名称・自治体・紹介元・件数・フィルター |
| 医療機関詳細       |  -  |  ○  |  -  |  -  | `medical_institutions`、`ref_naviis`、`ref_naviis_consultation_hours`、`m_ref_naviis_payment_methods`、`m_ref_naviis_disease_treatment_details`、`m_ref_naviis_vaccinations` |                                        |
| 自治体             |  -  |  ○  |  -  |  -  | `municipalities`                                                                                                                                                             |                                        |
| 紹介情報           |  -  |  ○  |  -  |  -  | `referrals`                                                                                                                                                                  | 患者 ID 単位                           |
| 紹介タスク         |  ○  |  -  |  -  |  -  | `tasks`                                                                                                                                                                      | 患者 ID に紐づく作成                   |
| 利用者・MFA        |  -  |  ○  |  ○  |  -  | `users`、`tenants`、`medical_institution_labels`、`mc_map_tenant_default_facility_types`                                                                                     | MFA 無効化は Azure 側                  |

## 逆照会管理 tRPC

実装：[command](../apps/foro-command-service/src/modules/op_optimizer)、[query](../apps/foro-query-service/src/modules/op_optimizer)

| 機能             |  C  |  R  |  U  |  D  | 関連テーブル                                                                                                                                          | 補足                             |
| :--------------- | :-: | :-: | :-: | :-: | :---------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------- |
| 対象リスト       |  ○  |  ○  |  ○  |  ○  | `op_optimizer_target_lists`、`op_optimizer_target_list_doctors`                                                                                       | アーカイブ状態更新を含む         |
| 対象患者         |  -  |  ○  |  ○  |  -  | `op_optimizer_target_patients`、`op_optimizer_target_patient_statuses`、`op_optimizer_outpatient_reservations`、`op_optimizer_operation_reservations` | 一覧・患者番号検索、対応状態更新 |
| 患者詳細         |  -  |  ○  |  -  |  -  | `op_optimizer_target_patients`、`op_optimizer_outpatient_reservations`、`op_optimizer_operation_reservations`                                         |                                  |
| 診療科マスタ     |  -  |  ○  |  -  |  -  | `op_optimizer_departments`、`op_optimizer_doctors`                                                                                                    |                                  |
| 医師連絡先       |  ○  |  ○  |  ○  |  -  | `op_optimizer_doctor_contacts`、`op_optimizer_staff_contacts`、`op_optimizer_doctors`                                                                 | 保存は upsert                    |
| 取込項目対応付け |  ○  |  ○  |  ○  |  -  | `op_optimizer_outpatient_import_key_mappings`、`op_optimizer_operation_import_key_mappings`                                                           | 外来・手術の保存は upsert        |
| 予約取込ログ     |  ○  |  ○  |  -  |  -  | `op_optimizer_reservation_importing_logs`                                                                                                             | 取込依頼により登録               |
| 外来予約ファイル |  ○  |  -  |  -  |  -  | `op_optimizer_reservation_importing_logs`、外部ストレージ                                                                                             | 署名 URL、取込依頼               |
| 手術予約ファイル |  ○  |  -  |  -  |  -  | `op_optimizer_reservation_importing_logs`、外部ストレージ                                                                                             | 署名 URL、取込依頼               |
| 利用者・MFA      |  -  |  ○  |  ○  |  -  | `users`、`tenants`                                                                                                                                    | MFA 無効化は Azure 側            |

## バッチ・非同期処理

実装：[apps/foro-job-worker/src/modules](../apps/foro-job-worker/src/modules)

| 機能                     |  C  |  R  |  U  |  D  | 関連テーブル                                                                                                                                   | 補足                           |
| :----------------------- | :-: | :-: | :-: | :-: | :--------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------- |
| コミュニケーション分類   |  -  |  -  |  ○  |  -  | `communications`、`communication_label_values`、`communication_classify_settings`                                                              | 分類結果を更新                 |
| エリアラベル値更新       |  ○  |  -  |  ○  |  -  | `medical_institution_labels`、`medical_institution_label_values`、`medical_institutions`                                                       | 更新ジョブ                     |
| 医療機関セグメント更新   |  ○  |  -  |  ○  |  -  | `medup_foro_segmentation_settings`、`medup_foro_segments`、`medup_foro_medical_institution_segments`                                           | 更新ジョブ                     |
| 逆照会用データセット作成 |  ○  |  -  |  -  |  -  | `op_optimizer_departments`、`op_optimizer_doctors`、`op_optimizer_target_lists`、`op_optimizer_target_patients`                                | BigQuery データセット作成      |
| 外来予約ログ取込         |  ○  |  -  |  ○  |  -  | `op_optimizer_outpatient_reservations`、`op_optimizer_departments`、`op_optimizer_doctors`、`op_optimizer_reservation_importing_logs`          | 予約・マスタ・ログを作成／更新 |
| 手術予約ログ取込         |  ○  |  -  |  ○  |  -  | `op_optimizer_operation_reservations`、`op_optimizer_departments`、`op_optimizer_doctors`、`op_optimizer_reservation_importing_logs`           | 予約・マスタ・ログを作成／更新 |
| 医療・疾患記録更新       |  -  |  -  |  ○  |  -  | `referrals`、`reverse_referrals`、`hospital_admissions`、`dpc_revisions`、`dpc_mdcs`、`dpc_code_scores`、`kcode_scores`、`dpc_specialty_codes` | BigQuery 側データ更新          |
| 逆紹介候補通知           |  ○  |  -  |  -  |  -  | `op_optimizer_reverse_referral_candidate_notifications`、`op_optimizer_target_lists`、`tenants`                                                | 通知レコード・メール作成       |

## 対象外

- `foro-crm-web` のうち、別リポジトリの `foro-crm-api` 等だけに実装される CRM・レポート・メール・ラベル等の CRUD。
- 画面ルートや Prisma モデルだけでは更新経路を確認できない機能。
- `package.json` にスクリプト名だけ存在し、実体スクリプトを確認できない BigQuery 同期・CSV 取込。

## foro-crm-api 追記（Rails REST API）

### 判定ルール

- `C` は作成・取込・送信、`R` は一覧・詳細・検索・集計、`U` は更新・置換・状態変更、`D` は削除・アーカイブを表す。
- 検索・集計で `POST` を使用する場合も、永続更新がなければ `R` と判定する。
- `○` は `config/routes.rb`、`config/routes/api.rb` と対応コントローラーまたはジョブから確認できる操作、`-` は未確認または非提供を表す。
- 「実装コード」はAPI入口から処理本体までを示す。ルートだけでコントローラーまたはアクションが確認できない操作は `-` とする。
- 外部 MIS に実体がある医療機関・診療科等は、本Railsの対象テーブルとは分けて扱う。

### 利用者・基本 CRM

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| Azure 利用者識別・サインイン | ○ | - | - | - | [`identify_controller.rb`](../../app/controllers/azure/users/identify_controller.rb)、[`sign_in_controller.rb`](../../app/controllers/azure/users/sign_in_controller.rb) | `users`、外部 Azure 認証 | 認証情報の受理 |
| 現在利用者・プロフィール | - | ○ | ○ | - | [`current_users_controller.rb`](../../app/controllers/api/current_users_controller.rb)、[`users/me/profiles_controller.rb`](../../app/controllers/api/users/me/profiles_controller.rb) | `users`、`tenants` | 本人情報の参照・更新 |
| アカウント | ○ | ○ | ○ | ○ | [`accounts_controller.rb`](../../app/controllers/api/accounts_controller.rb)、[`account.rb`](../../app/domains/services/account.rb) | `users`、`tenants` | 削除相当の停止・対象制御を含む |
| スタッフ・担当者候補 | - | ○ | - | - | [`staffs_controller.rb`](../../app/controllers/api/staffs_controller.rb) | `users` | CRM または連携マップの権限がある利用者 |
| 医療機関CRUD | - | - | - | - | [`config/routes/api.rb`](../../config/routes/api.rb) の `resources :medical_institutions` | `medical_institutions`（外部 MIS） | ルートのみ。対応コントローラー未確認のためCRUDを確定しない |
| 医療機関検索・営業マップ | - | ○ | - | - | [`search_for_sales_map_results_controller.rb`](../../app/controllers/api/medical_institutions/search_for_sales_map_results_controller.rb)、[`search_for_labels_controller.rb`](../../app/controllers/api/medical_institutions/search_for_labels_controller.rb)、[`communications/search_results_controller.rb`](../../app/controllers/api/communications/search_results_controller.rb) | 外部 MIS、ラベル・施設種別・紹介 | POST検索もR |
| 医療機関プロフィール | - | ○ | ○ | - | [`medical_institutions/summaries_controller.rb`](../../app/controllers/api/medical_institutions/summaries_controller.rb)、[`memos_controller.rb`](../../app/controllers/api/medical_institutions/memos_controller.rb)、[`special_notes_controller.rb`](../../app/controllers/api/medical_institutions/special_notes_controller.rb)、[`phases_controller.rb`](../../app/controllers/api/medical_institutions/phases_controller.rb)、[`segments_controller.rb`](../../app/controllers/api/medical_institutions/segments_controller.rb) | `medical_institution_memos`、`crm_medical_institution_special_notes`、`crm_medical_institution_phases` | `profile` のshowだけはルートのみ |
| 診療科・施設種別・Kコード・MDC | - | ○ | - | - | [`departments_controller.rb`](../../app/controllers/api/departments_controller.rb)、[`facility_type_groups_controller.rb`](../../app/controllers/api/facility_type_groups_controller.rb)、[`kcodes_controller.rb`](../../app/controllers/api/kcodes_controller.rb)、[`mdcs_controller.rb`](../../app/controllers/api/mdcs_controller.rb) | `own_departments`、外部 MIS、`kcode_scores`、`dpc_mdcs` | 選択肢・参照 |
| 担当医 | ○ | ○ | ○ | ○ | [`medical_institutions/doctors_controller.rb`](../../app/controllers/api/medical_institutions/doctors_controller.rb)、[`doctors/email_destinations_controller.rb`](../../app/controllers/api/doctors/email_destinations_controller.rb) | `doctors` | 医療機関配下で管理 |
| タグ・タググループ | ○ | ○ | ○ | ○ | [`tags_controller.rb`](../../app/controllers/api/tags_controller.rb)、[`tag_groups_controller.rb`](../../app/controllers/api/tag_groups_controller.rb)、[`medical_institutions/tags_controller.rb`](../../app/controllers/api/medical_institutions/tags_controller.rb) | `tags`、`tag_groups`、`tag_medical_institutions` | 医療機関との対応を含む |
| アクションリスト | ○ | ○ | ○ | ○ | [`action_lists_controller.rb`](../../app/controllers/api/action_lists_controller.rb)、[`action_lists/`](../../app/controllers/api/action_lists/) | `action_lists`、`action_list_medical_institutions` | 一括追加・一括削除、状態変更を含む |
| セグメント | - | ○ | - | - | [`segmentation_settings_controller.rb`](../../app/controllers/api/segmentation_settings_controller.rb)、[`medical_institutions/segments_controller.rb`](../../app/controllers/api/medical_institutions/segments_controller.rb) | `medup_foro_segmentation_settings`、`medup_foro_medical_institution_segments` | 設定・医療機関別参照 |

### コミュニケーション・タスク

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| コミュニケーション | ○ | ○ | ○ | ○ | [`communications_controller.rb`](../../app/controllers/api/communications_controller.rb)、[`medical_institutions/communications_controller.rb`](../../app/controllers/api/medical_institutions/communications_controller.rb)、[`communications/search_results_controller.rb`](../../app/controllers/api/communications/search_results_controller.rb) | `communications` | POST検索はR、医療機関配下POSTはC |
| コミュニケーションコメント | ○ | - | ○ | ○ | [`communications/comments_controller.rb`](../../app/controllers/api/communications/comments_controller.rb) | `communication_comments` | |
| いいね | ○ | - | - | ○ | [`communications/likes_controller.rb`](../../app/controllers/api/communications/likes_controller.rb) | `communication_likes` | 更新メソッドでも追加操作のためCと判定 |
| VOC | ○ | - | ○ | ○ | [`vocs_controller.rb`](../../app/controllers/api/vocs_controller.rb)、[`communications/vocs_controller.rb`](../../app/controllers/api/communications/vocs_controller.rb) | `crm_vocs`、`crm_communication_vocs` | コミュニケーション配下の更新を含む |
| VOCグループ | - | - | ○ | - | [`voc_groups_controller.rb`](../../app/controllers/api/voc_groups_controller.rb)、[`voc_groups.rb`](../../app/domains/services/voc_groups.rb) | `crm_voc_groups`、`communications` | |
| タスク | ○ | ○ | ○ | ○ | [`tasks_controller.rb`](../../app/controllers/api/tasks_controller.rb)、[`tasks/search_result.rb`](../../app/domains/services/tasks/search_result.rb) | `tasks` | |
| コミュニケーションCSV取込 | ○ | ○ | ○ | ○ | [`imports/communications_controller.rb`](../../app/controllers/api/imports/communications_controller.rb)、[`communications_import_job.rb`](../../app/jobs/communications_import_job.rb)、[`communications_update_import_job.rb`](../../app/jobs/communications_update_import_job.rb) | `communications_user_importing_logs`、`communications` | 更新取込・再処理・取込ログ削除を含む |
| 医療機関ラベルCSV取込 | ○ | ○ | - | - | [`imports/medical_institution_label_values_controller.rb`](../../app/controllers/api/imports/medical_institution_label_values_controller.rb)、[`medical_institution_label_values_import_job.rb`](../../app/jobs/medical_institution_label_values_import_job.rb) | `medical_institution_label_values_importing_logs`、`medical_institution_label_values` | Active Jobで一括取込 |
| アップロードファイル登録 | ○ | - | - | - | [`uploaded_files_controller.rb`](../../app/controllers/api/uploaded_files_controller.rb)、[`storage_client_service.rb`](../../app/services/storage_client_service.rb) | `uploaded_files` | |

### 分析・レポート

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| 紹介・逆紹介分析 | - | ○ | - | - | [`charts/`](../../app/controllers/api/charts/)、[`referrals/`](../../app/controllers/api/referrals/)、[`reverse_referrals/`](../../app/controllers/api/reverse_referrals/) | `referrals`、`reverse_referrals`、`own_departments` | POSTで実行してもR |
| 訪問・ランキング分析 | - | ○ | - | - | [`charts/visits/`](../../app/controllers/api/charts/visits/)、[`data_analysis/`](../../app/controllers/api/data_analysis/) | `communications`、`referrals`、`reverse_referrals` | POSTで実行してもR |
| 医療機関レポート定義 | ○ | ○ | ○ | ○ | [`medical_institution_reports/`](../../app/controllers/api/medical_institution_reports/)、[`medical_institution_reports/definition.rb`](../../app/domains/services/medical_institution_reports/definition.rb) | `medical_institution_report_definitions`、関連項目・フィルター・集計表 | フォルダー・表示順・お気に入りを含む |
| 医療機関レポート結果 | - | ○ | - | - | [`medical_institution_reports/results_controller.rb`](../../app/controllers/api/medical_institution_reports/results_controller.rb)、[`aggregation_results_controller.rb`](../../app/controllers/api/medical_institution_reports/aggregation_results_controller.rb) | `referrals`、`communications`、ラベル値 | POST集計・結果取得もR |
| 自部署レポート | ○ | ○ | ○ | ○ | [`own_department_reports/`](../../app/controllers/api/own_department_reports/)、[`own_department_reports/`](../../app/domains/services/own_department_reports/) | `own_department_report_definitions`、関連項目・フォルダー | 結果取得はR |
| エリアレポート | ○ | ○ | ○ | ○ | [`area_reports/`](../../app/controllers/api/area_reports/)、[`area_reports/`](../../app/domains/services/area_reports/) | `area_report_definitions`、関連項目・フォルダー | 結果取得はR |
| コミュニケーションレポート | ○ | ○ | ○ | ○ | [`communication_reports/`](../../app/controllers/api/communication_reports/)、[`communication_reports/`](../../app/domains/services/communication_reports/) | `communication_report_definitions`、関連項目・フォルダー | 結果・集計取得はR |
| 紹介レポート | - | ○ | - | - | [`referral_reports/aggregation_results_controller.rb`](../../app/controllers/api/referral_reports/aggregation_results_controller.rb)、[`referral_reports/results.rb`](../../app/domains/services/referral_reports/results.rb) | `referrals`、`reverse_referrals` | 集計POSTはR |
| ダッシュボード | ○ | ○ | ○ | ○ | [`dashboards_controller.rb`](../../app/controllers/api/dashboards_controller.rb) | `dashboards`、`dashboard_contents` | |

### メール・ラベル・アラート

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| メール | ○ | ○ | ○ | ○ | [`emails_controller.rb`](../../app/controllers/api/emails_controller.rb)、[`emails/`](../../app/controllers/api/emails/) | `emails`、`email_attachments` | コピー・テスト送信・本送信はC |
| メール配信結果 | - | ○ | - | - | [`emails/stats_controller.rb`](../../app/controllers/api/emails/stats_controller.rb)、[`emails/reports_controller.rb`](../../app/controllers/api/emails/reports_controller.rb)、[`emails_statistics_controller.rb`](../../app/controllers/emails_statistics_controller.rb) | `email_stats`、`emails` | SendGrid等の統計コールバック解析を含む |
| 医療機関ラベル | ○ | ○ | ○ | ○ | [`medical_institution_labels_controller.rb`](../../app/controllers/api/medical_institution_labels_controller.rb)、[`medical_institution_labels/orders_controller.rb`](../../app/controllers/api/medical_institution_labels/orders_controller.rb) | `medical_institution_labels`、選択肢・エリア表 | 表示順更新を含む |
| 医療機関ラベル値 | ○ | ○ | - | - | [`medical_institutions/label_values_controller.rb`](../../app/controllers/api/medical_institutions/label_values_controller.rb) | `medical_institution_label_values`、選択肢・エリア | |
| コミュニケーションラベル | ○ | ○ | ○ | ○ | [`communication_labels_controller.rb`](../../app/controllers/api/communication_labels_controller.rb)、[`communication_labels/orders_controller.rb`](../../app/controllers/api/communication_labels/orders_controller.rb) | `communication_labels`、選択肢 | 表示順更新を含む |
| アラート条件 | ○ | ○ | - | ○ | [`alert_conditions_controller.rb`](../../app/controllers/api/alert_conditions_controller.rb) | `alert_conditions` | |
| アラート | ○ | ○ | - | ○ | [`alerts_controller.rb`](../../app/controllers/api/alerts_controller.rb)、[`alerts/`](../../app/controllers/api/alerts/)、[`generate_alert_job.rb`](../../app/jobs/generate_alert_job.rb) | `alerts`、`alert_medical_institutions` | ジョブ生成・アーカイブを含む |

### 非同期・運用

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| コミュニケーション取込ジョブ | ○ | - | ○ | - | [`communications_import_job.rb`](../../app/jobs/communications_import_job.rb)、[`communications_update_import_job.rb`](../../app/jobs/communications_update_import_job.rb) | `communications`、取込ログ、ラベル値 | 新規取込・更新取込 |
| 医療機関ラベル取込ジョブ | ○ | - | ○ | - | [`medical_institution_label_values_import_job.rb`](../../app/jobs/medical_institution_label_values_import_job.rb) | `medical_institution_label_values`、取込ログ | 一括作成・更新 |
| アラート生成ジョブ | ○ | - | - | - | [`generate_alert_job.rb`](../../app/jobs/generate_alert_job.rb)、[`domains/services/alerts/`](../../app/domains/services/alerts/) | `alerts`、`referrals`、`communications` | Worker入口から実行 |
| 通知ジョブ | ○ | - | ○ | - | [`alerts_summary_notification_job.rb`](../../app/jobs/alerts_summary_notification_job.rb)、[`communication_reactions_summary_notification_job.rb`](../../app/jobs/communication_reactions_summary_notification_job.rb)、[`medical_institution_report_alert_event_notification_job.rb`](../../app/jobs/medical_institution_report_alert_event_notification_job.rb) | `alerts`、レポートアラートイベント等 | 送信・通知状態更新 |
| Worker HTTP 入口 | ○ | - | - | - | [`job_execution_controller.rb`](../../app/controllers/job_execution_controller.rb) | ジョブ | Worker環境の実行要求 |
| ヘルスチェック | - | ○ | - | - | [`gae_healthcheck_controller.rb`](../../app/controllers/gae_healthcheck_controller.rb) | なし | liveness・readiness |

## data-management-api 追記（FastAPI データ処理API）

### 判定ルール

- この節は [`src/presentations/routes`](../../src/presentations/routes) と、呼び出し先のコントローラー・リポジトリから確認できる操作を対象とする。
- `POST` でもファイル取込や再生成の実体を見てCRUDを判定する。`○` はDBまたは外部ストレージへの作成・参照・更新・削除が確認できることを表す。
- 既存行を削除して再登録する置換処理は、`C`・`U`・`D` を○とする。BigQueryのdelete後insertによるupsertも同様に扱う。
- Cloud Tasksへの投入は永続CRUDとはみなさず、補足に記載する。

### 取込・関連再生成

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| 入院データ取込 | ○ | ○ | ○ | - | [`hospital_admissions.py`](../../src/presentations/routes/hospital_admissions.py)、[`controllers/hospital_admissions.py`](../../src/presentations/controllers/hospital_admissions.py) | `hospital_admissions`、`hospital_admission_importing_logs`、`tenant_medical_institution_code_ids`、FF1・Dファイル | 取込ログの完了・失敗を更新し、入院データを一括作成。関連再生成も実行 |
| 外来EF取込 | ○ | ○ | ○ | - | [`outpatient_efs.py`](../../src/presentations/routes/outpatient_efs.py)、[`controllers/outpatient_efs.py`](../../src/presentations/controllers/outpatient_efs.py) | `medup_foro_outpatient_ef_importing_logs`、Cloud StorageのE/F/EFファイル | E/Fを参照してEFを新規保存し、取込ログを完了・失敗へ更新 |
| 紹介入院関連再生成 | ○ | ○ | ○ | ○ | [`referral_hospital_admissions.py`](../../src/presentations/routes/referral_hospital_admissions.py)、[`hospital_admission_referral_repository_imp.py`](../../src/gateways/db/rdb/repositories/hospital_admission_referral_repository_imp.py) | `hospital_admissions`、`referrals`、`hospital_admission_referrals`、`referral_hospital_admissions` | テナント対象を削除後に再登録し、`ANALYZE`を実行 |
| 紹介・逆紹介回答関連再生成 | ○ | ○ | ○ | ○ | [`referral_replies.py`](../../src/presentations/routes/referral_replies.py)、[`extract_referral_replies.py`](../../src/gateways/db/rdb/services/extract_referral_replies.py) | `referrals`、`reverse_referrals`、`referral_replies` | 患者番号・施設関係・365日以内の日付差で抽出し、テナント対象を置換 |

### 医療機関マッピング・厚労省同期

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| マッピング結果ファイル作成 | ○ | ○ | - | - | [`matching_records.py`](../../src/presentations/routes/matching_records.py)、[`controllers/matching_records.py`](../../src/presentations/controllers/matching_records.py) | `medical_institution_mappings`、`medical_institutions`、BigQuery `mis_facilities`、Cloud Storage | 入力CSVと既存マッピング・正規化候補を参照し、結果CSVを保存 |
| 作業シート推測マッチング | - | ○ | ○ | - | [`controllers/matching_records.py`](../../src/presentations/controllers/matching_records.py)、[`medical_institution_mapping_work_sheet_repository_imp.py`](../../src/gateways/db/rdb/repositories/medical_institution_mapping_work_sheet_repository_imp.py) | `medical_institution_mapping_work_sheets`、`medical_institution_mapping_work_sheet_sections`、`medical_institution_mapping_work_sheet_section_records`、BigQuery `mis_facilities` | 両セクションの `medical_institution_id` を一括更新 |
| 厚労省新設・廃止報告取込 | ○ | ○ | - | - | [`mhlw_sync_work.py`](../../src/presentations/routes/mhlw_sync_work.py)、[`controllers/mhlw_sync_work.py`](../../src/presentations/controllers/mhlw_sync_work.py) | `medical_institutions`、`mhlw_sync_work_insert_records`、`mhlw_sync_work_update_records`、`mhlw_sync_work_close_records` | CSVと既存MISを照合し、各候補を作成 |
| 厚労省一覧照合 | ○ | ○ | - | - | [`controllers/mhlw_sync_work.py`](../../src/presentations/controllers/mhlw_sync_work.py)、[`mhlw_sync_work_overview_report_record_group_repository_imp.py`](../../src/gateways/db/rdb/repositories/mhlw_sync_work_overview_report_record_group_repository_imp.py) | `mhlw_sync_work_update_records`、`mhlw_sync_work_overview_diff_records`、`mhlw_sync_work_overview_only_list_records`、`mhlw_sync_work_overview_only_mis_records` | 差分分類結果を一括作成 |

### マスタ同期・外部保存

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| MIS施設情報同期 | ○ | ○ | ○ | ○ | [`mis_facilities.py`](../../src/presentations/routes/admin/batch/mis_facilities.py)、[`processed_medical_institution_repository_imp.py`](../../src/gateways/db/bigquery/repositories/processed_medical_institution_repository_imp.py) | `medical_institutions`、BigQuery `mis_facilities` | 前日更新分を読み、対象IDの既存BigQuery行を削除して再挿入 |
| Refナビィ更新 | ○ | ○ | ○ | ○ | [`ref_naviis.py`](../../src/presentations/routes/admin/batch/ref_naviis.py)、[`controllers/admin/batch/ref_naviis.py`](../../src/presentations/controllers/admin/batch/ref_naviis.py) | `navii_medical_institution_mappings`、`tmp_ref_naviis`、`tmp_ref_naviis_consultation_hours`、`m_ref_naviis_*`、GCSナビィシード | リセット時に一時テーブルをtruncateし、県単位で投入。未対応マッピングはupsert |
| 医療記録再計算依頼 | - | - | - | - | [`foro_job_worker_client.py`](../../src/gateways/cloudtasks/foro_job_worker_client.py) | `foro-job-worker`、Cloud Tasks | 入院・外来EF取込後に非同期実行要求を作成。永続DB CRUDではない |

## foro-crm-devops 追記（運用処理）

### 判定ルール

- Cはファイル・外部ストレージ・BigQueryテーブルへの作成、RはDBや外部APIからの参照、Uは既存データの更新、Dは削除・ローテーションを表す。
- `WRITE_TRUNCATE`、または既存行を削除して再登録する処理は、結果を置換するため C・U・D を○とする。
- Pub/SubメッセージやCloud Tasks要求だけは永続CRUDとせず、補足に記載する。

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| レポート更新ディスパッチ | - | ○ | - | - | [`dispatch_update_tenants_report/src/index.ts`](../../medical_institution_report_updater/dispatch_update_tenants_report/src/index.ts) | `tenants`、Cloud Tasks、Pub/Sub | 有効テナントと未完了タスクを参照し、更新要求を発行。メッセージ発行は永続CRUD外 |
| 共通医療機関データ更新 | ○ | ○ | ○ | ○ | [`update_common_dataset/src/index.ts`](../../medical_institution_report_updater/update_common_dataset/src/index.ts) | MIS医療機関関連表、GCS、BigQuery common | `medical_institution_attributes` と `mhlw_designation_histories` を置換 |
| 共通ナビィ値更新 | ○ | ○ | ○ | ○ | [`update_common_navii_values/src/index.ts`](../../medical_institution_report_updater/update_common_navii_values/src/index.ts) | `ref_naviis`、GCS、BigQuery `medical_institution_navii_values` | 正規化JSONLを `WRITE_TRUNCATE`。空生成物では更新を止める |
| テナント別レポート更新 | ○ | ○ | ○ | ○ | [`update_single_tenant_dataset/src/index.ts`](../../medical_institution_report_updater/update_single_tenant_dataset/src/index.ts) | foro業務表、GCS、BigQuery tenant dataset | データセット・テーブルを必要時作成し、11分析テーブルをテーブル単位で置換 |
| ナビィ本番反映 | ○ | ○ | - | ○ | [`scrape_navii/src/importer.ts`](../../scrape_navii/src/importer.ts) | `tmp_ref_naviis`、`tmp_ref_naviis_consultation_hours`、`ref_naviis`、`ref_naviis_consultation_hours` | 80%チェック後に親子表を削除して一時表から再登録 |
| SendGridメールログ保管 | ○ | ○ | - | - | [`sendgrid_email_logs_fetcher/src/index.ts`](../../sendgrid_email_logs_fetcher/src/index.ts) | SendGrid Logs API、GCS、`_SUCCESS` マーカー | 1時間窓を取得しJSONL gzipを作成。業務DBは更新しない |
| Resendメールログ保管 | ○ | ○ | - | - | [`resend_email_logs_fetcher/src/index.ts`](../../resend_email_logs_fetcher/src/index.ts) | Resend List Emails API、GCS、`_SUCCESS` マーカー | カーソルを遡って対象窓を取得しGCSへ保存 |
| Azure AD B2C利用者移行 | ○ | ○ | ○ | ○ | [`aadb2c_users_migration/src/migrate.ts`](../../aadb2c_users_migration/src/migrate.ts)、[`rollback.ts`](../../aadb2c_users_migration/src/rollback.ts) | `users`、Azure AD B2C | B2C作成と `aad_user_id` 更新、rollback時のB2C利用者削除 |
| B2Cメールアドレス変更 | - | ○ | ○ | - | [`aadb2c_users_migration/src/updateUser.ts`](../../aadb2c_users_migration/src/updateUser.ts) | Azure AD B2C | CSVの変更リストを参照してB2C利用者を更新。DB更新SQL生成は別処理 |
| Cloud Scheduler設定更新 | - | ○ | ○ | - | [`job_scheduler/src/main.ts`](../../job_scheduler/src/main.ts) | Cloud Scheduler、`schedule.yaml` | 既存ジョブのscheduleとtimeZoneを更新。作成・削除は未対応 |
| クロスリージョンDBバックアップ | ○ | - | - | ○ | [`backup_database_cross_region/run.sh`](../../backup_database_cross_region/run.sh) | AlloyDB backups | 別リージョンへ作成し、7日を超えた対象バックアップを削除 |
| foro seedダンプ | ○ | ○ | - | - | [`dump_foro_seed_data/dump_job/src/foro-crm-api/main.ts`](../../dump_foro_seed_data/dump_job/src/foro-crm-api/main.ts) | foro業務DB、GCS | 情報スキーマを参照し、対象テーブルをCSV化してGCSへ保存 |
| MIS seedダンプ | ○ | ○ | - | - | [`dump_mis_seed_data/dump_job/dump.sh`](../../dump_mis_seed_data/dump_job/dump.sh) | MIS DB、GCS | `public` スキーマのデータのみSQLを作成してGCSへ保存 |
| メンテナンスURL Map切替 | - | ○ | ○ | - | [`loadbalancer_url_maps/README.md`](../../loadbalancer_url_maps/README.md) | GCP URL Map、`foro-crm-spa`、メンテナンス用バケット | 通常・メンテナンス定義をURL Mapへ適用 |
| 踏み台プロキシ設定 | - | - | - | - | [`medup-foro-bastion/setup-bastion-proxy.command`](../../medup-foro-bastion/setup-bastion-proxy.command) | GCP IAP、ローカルSOCKS・PAC | 永続DB CRUDではなく、接続経路を一時設定 |

## refery 追記（Fastify API）

### 判定ルール

- この節は refery の [`apps/api/src/apis/routes`](../../apps/api/src/apis/routes) と呼び出し先から確認できる永続操作を対象とする。
- 予約の取消など、行を物理削除せず状態を変更する操作も、利用者から見た削除・取消に該当する場合は `D` とする。
- Azure AD B2C、メール、Cloud Storageの署名URL発行など、外部サービスだけで完結する処理はDB CRUDに含めず補足する。

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| テナント設定 | - | ○ | ○ | - | [`Tenant.ts`](../../apps/api/src/apis/routes/Tenant.ts) | `tenants`、`closed_days_files` | 設定参照・更新 |
| アカウント | ○ | ○ | ○ | ○ | [`Account.ts`](../../apps/api/src/apis/routes/Account.ts) | `users`、`tenants`、`guests` | 医療機関側利用者の登録・参照・権限／情報更新・削除 |
| 紹介元・紹介元アカウント | ○ | ○ | ○ | ○ | [`Referrer.ts`](../../apps/api/src/apis/routes/Referrer.ts)、[`guest/ReferralHospital.ts`](../../apps/api/src/apis/routes/guest/ReferralHospital.ts) | `referrers`、`guests`、`users`、`department_referrer_restrictions` | 招待、招待受諾、更新・削除を含む |
| 診療科・診療科別紹介元制限 | ○ | ○ | ○ | ○ | [`Department.ts`](../../apps/api/src/apis/routes/Department.ts)、[`Referrer.ts`](../../apps/api/src/apis/routes/Referrer.ts) | `departments`、`department_referrer_restrictions`、`tenants` | 診療科のCRUDと紹介元の利用制限 |
| 医師 | ○ | ○ | ○ | ○ | [`Doctor.ts`](../../apps/api/src/apis/routes/Doctor.ts) | `doctors`、`departments`、`tenants` | 登録・一覧／詳細・更新・削除 |
| 予約ルール・質問・回答選択肢 | ○ | ○ | ○ | ○ | [`ReservationRule.ts`](../../apps/api/src/apis/routes/ReservationRule.ts) | `questions`、`answer_options`、`assignments`、`time_period_assignments`、`doctor_assignments` | 診療科別ルールの取得・下書き保存・反映。保存時に関連設定を削除して再作成する |
| 稼働期間・勤務シフト | ○ | ○ | ○ | ○ | [`WorkingTerm.ts`](../../apps/api/src/apis/routes/WorkingTerm.ts)、[`WorkShift.ts`](../../apps/api/src/apis/routes/WorkShift.ts)、[`admin/batch/WorkShift.ts`](../../apps/api/src/apis/routes/admin/batch/WorkShift.ts) | `working_terms`、`working_term_doctors`、`work_shift_items`、`doctors` | 稼働期間はCRUD、勤務シフトは生成・参照・状態更新。勤務シフト行の削除ユースケースはAPIルートに未接続 |
| 予約枠 | ○ | ○ | - | ○ | [`admin/batch/WorkShift.ts`](../../apps/api/src/apis/routes/admin/batch/WorkShift.ts)、[`admin/batch/ReservationSlot.ts`](../../apps/api/src/apis/routes/admin/batch/ReservationSlot.ts)、[`guest/AvailableAppointment.ts`](../../apps/api/src/apis/routes/guest/AvailableAppointment.ts) | `reservation_slots`、`work_shift_items`、`doctors`、`tenants` | シフト等から生成、予約可能枠参照、過去枠削除 |
| 即時予約の医療機関側管理 | - | ○ | ○ | - | [`InstantReservation.ts`](../../apps/api/src/apis/routes/InstantReservation.ts) | `reservations`、`reservation_slots`、`requested_results`、`users` | 一覧・詳細、予約状態・予約枠・担当者・紹介状状況の更新 |
| リクエスト予約の医療機関側管理 | - | ○ | ○ | - | [`RequestReservation.ts`](../../apps/api/src/apis/routes/RequestReservation.ts) | `reservations`、`requested_appointment_dates`、`requested_results`、`users` | 一覧・詳細、予約結果・状態・担当者・紹介状状況の更新 |
| 紹介元の予約申請・変更・取消 | ○ | ○ | ○ | ○ | [`guest/Reservation.ts`](../../apps/api/src/apis/routes/guest/Reservation.ts) | `reservations`、`requested_appointment_dates`、`requested_results`、`reservation_slots`、`guests`、`departments` | 予約作成、一覧・詳細、変更、取消（状態変更） |
| 紹介元の仮予約 | ○ | ○ | ○ | ○ | [`guest/TemporaryReservation.ts`](../../apps/api/src/apis/routes/guest/TemporaryReservation.ts)、[`guest/Reservation.ts`](../../apps/api/src/apis/routes/guest/Reservation.ts) | `temporary_reservations`、`requested_appointment_dates`、`reservation_slots`、`guests` | 仮予約作成・参照・更新・削除 |
| ゲスト情報・紹介先情報 | - | ○ | ○ | - | [`guest/Guest.ts`](../../apps/api/src/apis/routes/guest/Guest.ts)、[`guest/ReferralHospital.ts`](../../apps/api/src/apis/routes/guest/ReferralHospital.ts) | `guests`、`referrers`、`users` | ログイン中の紹介元に紐づく情報の参照・更新 |
| 休診日CSV | ○ | ○ | - | - | [`ClosedDays.ts`](../../apps/api/src/apis/routes/ClosedDays.ts) | `closed_days`、`closed_days_files`、外部ファイルストレージ | CSV取込、既存ファイルのダウンロード |
| 紹介状・添付ファイル | ○ | ○ | - | - | [`DepartmentReferralTicket.ts`](../../apps/api/src/apis/routes/DepartmentReferralTicket.ts)、[`UploadedFile.ts`](../../apps/api/src/apis/routes/UploadedFile.ts)、[`guest/Reservation.ts`](../../apps/api/src/apis/routes/guest/Reservation.ts) | `departments`、`reservations`、`uploaded_files`、Cloud Storage | 署名URL発行・ファイル参照。バイナリ本体は外部ストレージ |
| 認証・メール送信 | - | - | - | - | [`admin/b2c`](../../apps/api/src/apis/routes/admin/b2c)、[`email` use cases](../../apps/api/src/applications/usecases/email) | Azure AD B2C、メール配信サービス | アカウント招待・本人確認・MFA・パスワード再設定。業務DB CRUDではない |

## outpatient-ops 追記（外来適正化分析）

### 判定ルール

- PythonパイプラインとSQLAlchemy Repositoryは、BigQueryの入力テーブル・ビューを参照するのみで、元レコードを作成・更新・削除しない。
- dbt実行はBigQuery上の派生ビューを作成または定義置換し、`dbt_int_doctor_department` はテーブルを更新する。基礎データのCRUDではないため、C・Uを「分析オブジェクトの作成・更新」として扱う。
- ローカルCSV・Excelおよび郵便番号キャッシュの作成・更新もC・Uに含めるが、業務DBの永続CRUDとは分ける。

### 分析パイプライン

| 機能 | C | R | U | D | 実装コード | 主な関連テーブル・外部参照 | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| 期間別患者サマリー集計 | - | ○ | - | - | [`patient_summary_by_period_repo.py`](../../src/bigquery/repository/patient_summary_by_period_repo.py) | `outpatient_ef`、`hospital_admissions`、DPC・診療科マスタ | 患者×診療科の受診、費用、診療フラグ、入院・手術情報を任意期間で集計する。 |
| 外来レセプトコード判定 | - | ○ | - | - | [`outpatient_ef_repo.py`](../../src/bigquery/repository/outpatient_ef_repo.py) | `outpatient_ef`、DPC診療科マスタ | 指定コードの実績、患者×来院の診療フラグ、医師別対象患者を参照する。 |
| 直近紹介付与 | - | ○ | - | - | [`referral_repo.py`](../../src/bigquery/repository/referral_repo.py) | `public_referrals`、`public_own_departments`、`public_medical_institutions` | 患者ごとの直近1件を紹介日降順で取得する。 |
| 候補施設・医師マスタ参照 | - | ○ | - | - | [`facility_repo.py`](../../src/bigquery/repository/facility_repo.py)、[`doctor_master_repo.py`](../../src/bigquery/repository/doctor_master_repo.py) | MIS医療機関、医師コードマスタ | 施設IDによる名称・住所・座標、医師名・部門を取得する。 |
| 郵便番号座標変換 | ○ | ○ | ○ | - | [`geo.py`](../../src/geo.py) | ZipCloud、国土地理院API、`.cache/zipcode_latlon.json` | APIを参照し、座標キャッシュを作成・更新する。 |
| 最寄り施設計算 | - | ○ | - | - | [`geo.py`](../../src/geo.py) | 患者座標、BigQuery医療機関座標 | Haversine距離で候補施設を近接順に並べ、成果物用列を付与する。 |
| 逆紹介候補患者リスト生成 | ○ | ○ | - | - | [`scripts/run_tenant_pipelines.py`](../../scripts/run_tenant_pipelines.py)、[`src/pipeline/tenants`](../../src/pipeline/tenants) | BigQuery分析データ、予約・手術・郵便番号・候補施設CSV | テナント・診療科別に抽出・除外・結合し、ローカルCSVまたはExcelを作成する。既存の日付付き出力は再実行時に上書きされる場合がある。 |

### dbt 分析オブジェクト

| 機能 | C | R | U | D | 実装コード | 主な関連オブジェクト | 補足 |
| :--- | :-: | :-: | :-: | :-: | :--- | :--- | :--- |
| 外来・イベントintermediateビュー生成 | ○ | ○ | ○ | - | [`dbt/models/intermediate`](../../dbt/models/intermediate) | `foro_prd`、`mis_prd`、`dpc_patient_records_prd`、`others` | レセプトフラグ、診療科、外来集計、入院・手術、紹介・逆紹介・時系列、医師部門、受診分類ビューを作成または置換する。 |
| 患者サマリーintermediateビュー生成 | ○ | ○ | ○ | - | [`dbt_int_patient_summary.sql`](../../dbt/models/intermediate/dbt_int_patient_summary.sql)、[`dbt_int_patient_summary_overall.sql`](../../dbt/models/intermediate/dbt_int_patient_summary_overall.sql) | 外来・入院・手術・DPCマスタのintermediateビュー | 年度×診療科×患者、年度×患者のサマリーを作成または置換する。 |
| テナント別martsビュー生成 | ○ | ○ | ○ | - | [`dbt/models/marts`](../../dbt/models/marts) | intermediateモデル | テナント11・204・274向けの年度別、全科別、部門別、時系列、受診分類ビューを作成または置換する。 |
| dbt品質テスト | - | ○ | - | - | [`_int_models.yml`](../../dbt/models/intermediate/_int_models.yml)、[`_mart_models.yml`](../../dbt/models/marts/_mart_models.yml) | `dbt` intermediate・martsビュー | not null、一意キー、許容値を検証し、データ自体は更新しない。 |
