# ローカル開発環境のDB・コンテナ・コード構成 対比表

## medup-core-service 作成分

以下のDB構成・テーブル一覧は `medup-core-service` の既存資料として維持する。`foro-crm-api` 固有のスキーマ照合は文末の追記節に記載する。

| Dockerコンテナ名 | DBeaverでの接続先<br>(データベース名) | スキーマ<br>(DBeaver上) | コード上の構成<br>(Prisma / アプリケーション) | 資料上の記載・システムの役割 |
| :--- | :--- | :--- | :--- | :--- |
| **`medup-foro-db`** | **`foro_dev`** | **`public`** | `packages/foro-prisma/prisma/foro/schema.prisma` | **foro データベース**<br>業務トランザクションデータ。メインDB。 |
| 〃 | **`mis_dev`** | **`public`** | `packages/foro-prisma/prisma/mis/schema.prisma` | **医療機関データベース (MIS)**<br>医療機関マスタ辞書。（初期構築時に `mis-seed-prd.sql` を投入） |
| 〃 | **`data_ops_dev`** | **`public`** | `packages/foro-prisma/prisma/data-ops/schema.prisma` | **Data Ops / データ運用**<br>データインポートや運用処理用のDB。 |
| **`refery-db`** | **`refery_development`** | **`public`** | `apps/api/prisma/schema.prisma` | **refery データベース**<br>医療機関向け予約・紹介管理サービス「refery」のトランザクションデータ。 |
| **`api`** など | *(上記 `foro_dev` 等へ接続)* | - | `medup-core-service` 配下の `apps/api` 等 | **コアサービス バックエンド**<br>Fastify/DDD構成でDBへ接続する共通APIサーバー群。 |


---


# 各スキーマ テーブル定義一覧

## 1. Data Ops スキーマ (`data_ops_dev`)

| テーブル名 (`@@map`) | PK (SEQ) | 外部キー (FK) 先テーブル | インデックス・ユニーク制約 カラム | 備考 |
|---|---|---|---|---|
| matching_logs | id (uuid) | - | - | `@updatedAt` あり |
| tenant_medical_institution_code_ids | id (uuid) | - | - | `@updatedAt` あり |
| own_department_mappings | id (uuid) | - | - | `@updatedAt` あり |
| medical_institution_mappings | id (uuid) | medical_institution_mapping_work_sheets | - | `@updatedAt` あり |
| medical_institution_mapping_work_sheets | id (uuid) | - | - | `@updatedAt` あり |
| medical_institution_mapping_work_sheet_sections | id (uuid) | medical_institution_mapping_work_sheets | - | `@updatedAt` あり |
| medical_institution_mapping_work_sheet_section_records | id (uuid) | medical_institution_mapping_work_sheet_sections | - | `@updatedAt` あり |
| medical_institution_mapping_work_sheet_reviews | id (uuid) | medical_institution_mapping_work_sheets | [work_sheet_id](Unique) | `@updatedAt` あり |
| medical_institution_mapping_work_sheet_review_records | id (uuid) | medical_institution_mapping_work_sheet_reviews<br>medical_institution_mapping_work_sheet_section_records(x2) | [tenantId, reviewId, receivedMedicalInstitutionName, receivedMedicalInstitutionAddress](Unique)<br>[sectionOneRecordId](Unique)<br>[sectionTwoRecordId](Unique) | `@updatedAt` あり |
| referral_masters | id (uuid) | referral_master_work_sheets | [tenantId, referralControlId](Unique) | `@updatedAt` あり |
| referral_master_work_sheets | id (uuid) | - | - | `@updatedAt` あり |
| referral_master_work_sheet_records | id (uuid) | referral_master_work_sheets | - | `@updatedAt` あり |
| reverse_referral_masters | id (uuid) | reverse_referral_master_work_sheets | [tenantId, reverseReferralControlId](Unique) | `@updatedAt` あり |
| reverse_referral_master_work_sheets | id (uuid) | - | - | `@updatedAt` あり |
| reverse_referral_master_work_sheet_records | id (uuid) | reverse_referral_master_work_sheets | - | `@updatedAt` あり |
| completion_notification_users | id (uuid) | - | [tenantId, email](Unique) | `@updatedAt` あり |
| tmp_ref_naviis | id (uuid) | - | [medicalInstitutionId](Unique) | - |
| tmp_ref_naviis_consultation_hours | id (uuid) | tmp_ref_naviis | - | - |
| navii_medical_institution_mappings | id (uuid) | - | [naviiName, naviiAddress](Unique) | - |

---

## 2. MIS スキーマ (`mis_dev`)

| テーブル名 (`@@map`) | PK (SEQ) | 外部キー (FK) 先テーブル（参照元カラム） | インデックス・ユニーク制約 カラム | 備考 |
|---|---|---|---|---|
| facility_type_groups | id (autoincrement) | - | - | - |
| facility_types | id (autoincrement) | facility_type_groups (facility_type_group_id) | [facility_type_group_id] | - |
| medical_institution_bulk_insert_job_statuses | id (autoincrement) | - | - | - |
| medical_institution_facility_types | id (autoincrement) | medical_institutions (medical_institution_id)<br>facility_types (facility_type_id) | [medical_institution_id, facility_type_id](Unique)<br>[facility_type_id] | - |
| medical_institution_specialties | id (autoincrement) | medical_institutions (medical_institution_id)<br>specialties (specialty_id) | [medical_institution_id, specialty_id](Unique)<br>[specialty_id]<br>[medical_institution_id] | - |
| medical_institutions | id (autoincrement) | - | [medical_institution_code]<br>[address]<br>[latitude, longitude]<br>[normalized_name(Gin ops)](Gin)<br>[normalized_address(Gin ops)](Gin) | - |
| specialties | id (autoincrement) | specialty_groups (specialty_group_id) | [specialty_group_id] | - |
| specialty_groups | id (autoincrement) | - | - | - |
| ref_naviis | id (uuid) | medical_institutions (medical_institution_id) | [medical_institution_id](Unique)<br>[medical_insurance_public_expense_ids](Gin)<br>[supported_foreign_language_ids](Gin)<br>[services_for_disabled_ids](Gin)<br>[considerations_for_wheelchair_user_ids](Gin)<br>[disease_treatment_detail_ids](Gin)<br>[vaccination_ids](Gin)<br>[medical_institution_id] | - |
| ref_naviis_consultation_hours | id (uuid) | ref_naviis (ref_navii_id) | [ref_navii_id] | - |
| m_ref_naviis_medical_insurance_public_expenses | id (uuid) | - | - | - |
| m_ref_naviis_supported_foreign_languages | id (uuid) | - | - | - |
| m_ref_naviis_services_for_disabled | id (uuid) | - | - | - |
| m_ref_naviis_considerations_for_wheelchair_users | id (uuid) | - | - | - |
| m_ref_naviis_specialist_medical_society_categories | id (uuid) | - | - | - |
| m_ref_naviis_specialist_medical_society_groups | id (uuid) | m_ref_naviis_specialist_medical_society_categories (category_id) | - | - |
| m_ref_naviis_specialist_medical_societies | id (uuid) | m_ref_naviis_specialist_medical_society_groups (group_id) | - | - |
| m_ref_naviis_disease_treatment_groups | id (uuid) | - | - | - |
| m_ref_naviis_disease_treatment_details | id (uuid) | m_ref_naviis_disease_treatment_groups (group_id) | - | - |
| m_ref_naviis_vaccinations | id (uuid) | - | - | - |
| m_ref_naviis_payment_methods | id (uuid) | - | - | - |
| municipalities | lg_code | - | - | - |
| mhlw_sync_works | id (uuid) | - | - | `@updatedAt` あり |
| mhlw_sync_work_monthly_report_import_logs | id (uuid) | mhlw_sync_works (work_id) | [work_id](Unique) | `@updatedAt` あり |
| mhlw_sync_work_insert_records | id (uuid) | mhlw_sync_works (work_id)<br>medical_institutions (medical_institution_id)<br>facility_types (facility_type_id) | [medical_institution_id](Unique) | `@updatedAt` あり |
| mhlw_sync_work_update_records | id (uuid) | mhlw_sync_works (work_id)<br>medical_institutions (medical_institution_id)<br>facility_types (facility_type_id) | - | `@updatedAt` あり |
| mhlw_sync_work_close_records | id (uuid) | mhlw_sync_works (work_id)<br>medical_institutions (medical_institution_id) | - | `@updatedAt` あり |
| mhlw_sync_work_overview_diff_records | id (uuid) | mhlw_sync_works (work_id)<br>medical_institutions (medical_institution_id)<br>facility_types (facility_type_id) | - | `@updatedAt` あり |
| mhlw_sync_work_overview_only_list_records | id (uuid) | mhlw_sync_works (work_id)<br>facility_types (facility_type_id)<br>medical_institutions (medical_institution_id) | - | `@updatedAt` あり |
| mhlw_sync_work_overview_only_mis_records | id (uuid) | mhlw_sync_works (work_id)<br>medical_institutions (medical_institution_id) | - | `@updatedAt` あり |
| mhlw_sync_work_overview_import_logs | id (uuid) | mhlw_sync_works (work_id) | [work_id](Unique) | `@updatedAt` あり |
| medical_institution_logs | id (uuid) | medical_institutions (medical_institution_id) | [medical_institution_id] | `@updatedAt` あり |
| department_mappings | id (uuid) | - | [raw_department_name](Unique) | `@updatedAt` あり |
| department_updated_logs | id (uuid) | - | - | - |

※ 現行 Prisma スキーマの全34モデルを掲載。インデックス欄はユニーク制約を `(Unique)`、Ginインデックスを `(Gin)` として表示。

---

## 3. foro スキーマ (`foro_dev`)

| テーブル名 (`@@map`) | PK (SEQ) | 外部キー (FK) 先テーブル（参照元カラム） | インデックス・ユニーク制約 カラム | 備考 |
|---|---|---|---|---|
| ar_internal_metadata | key | - | - | - |
| report_import_logs | id (uuid) | medup_foro_admins (generated_medup_foro_admin_id)<br>tenants (tenant_id) | - | `@updatedAt` あり |
| dpc_revisions | id (uuid) | - | - | - |
| dpc_mdcs | id (uuid) | dpc_revisions (dpc_revision_id) | - | - |
| dpc_diagnoses | id (uuid) | dpc_mdcs (dpc_mdc_id) | - | - |
| dpc_diagnosis_details | id (uuid) | dpc_diagnoses (dpc_diagnosis_id) | - | - |
| dpc_code_scores | id (uuid) | dpc_revisions (dpc_revision_id) | [dpc_revision_id, code](Unique) | - |
| kcode_scores | id (uuid) | dpc_revisions (dpc_revision_id) | [dpc_revision_id, practice_code](Unique) | - |
| hospital_admission_importing_logs | id (uuid) | medup_foro_admins (medup_foro_admin_id)<br>tenants (tenant_id) | - | - |
| hospital_admissions | id (uuid) | hospital_admission_importing_logs (hospital_admission_importing_log_id)<br>tenants (tenant_id) | [tenant_id, medical_institution_id]<br>[tenant_id, updated_at]<br>[hospital_admission_importing_log_id] | - |
| referral_hospital_admissions | id (uuid) | tenants (tenant_id) | [tenant_id, referral_id, hospital_admission_id](Unique)<br>[hospital_admission_id] | - |
| hospital_admission_referrals | id (uuid) | tenants (tenant_id) | [hospital_admission_id](Unique)<br>[tenant_id, referral_id, hospital_admission_id]<br>[referral_id] | - |
| referral_importing_logs | id (uuid) | tenants (tenant_id) | [imported_at]<br>[tenant_id] | - |
| referrals | id (uuid) | own_departments (specialty_id)<br>referral_importing_logs (referral_importing_log_id)<br>tenants (tenant_id) | [tenant_id, medical_institution_id_from]<br>[tenant_id, medical_institution_id_to]<br>[tenant_id, referral_date]<br>[tenant_id, specialty_id]<br>[tenant_id, updated_at]<br>[referral_importing_log_id] | - |
| reverse_referral_importing_logs | id (uuid) | tenants (tenant_id) | [imported_at]<br>[tenant_id] | - |
| reverse_referrals | id (uuid) | own_departments (specialty_id)<br>reverse_referral_importing_logs (reverse_referral_importing_log_id)<br>tenants (tenant_id) | [tenant_id, medical_institution_id_from]<br>[tenant_id, medical_institution_id_to]<br>[tenant_id, reverse_referral_date]<br>[tenant_id, specialty_id]<br>[reverse_referral_importing_log_id] | - |
| referral_replies | id (uuid) | tenants (tenant_id)<br>referrals (referral_id)<br>reverse_referrals (reverse_referral_id) | [referral_id, reverse_referral_id](Unique)<br>[referral_id](Unique)<br>[referral_id]<br>[reverse_referral_id] | - |
| schema_migrations | version | - | - | - |
| uploaded_referral_files | id (uuid) | tenants (tenant_id) | [user_id] | - |
| medup_foro_admins | id (uuid) | - | [email](Unique) | - |
| medup_foro_segmentation_settings | id (uuid) | tenants (tenant_id) | - | - |
| medup_foro_excluded_own_department_segmentation_settings | id (uuid) | medup_foro_segmentation_settings (segmentation_setting_id)<br>own_departments (own_department_id)<br>tenants (tenant_id) | - | `@updatedAt` あり |
| medup_foro_segments | id (uuid) | medup_foro_segmentation_settings (segmentation_setting_id)<br>tenants (tenant_id) | - | - |
| medup_foro_medical_institution_segments | id (uuid) | medup_foro_segments (segment_id)<br>tenants (tenant_id) | [tenant_id, segment_id, medical_institution_id, period](Unique) | - |
| medup_foro_outpatient_ef_importing_logs | id (uuid) | medup_foro_admins (medup_foro_admin_id)<br>tenants (tenant_id) | - | - |
| action_list_medical_institutions | id (autoincrement) | action_lists (action_list_id)<br>tenants (tenant_id) | [action_list_id, medical_institution_id](Unique)<br>[action_list_id] | - |
| action_lists | id (autoincrement) | tenants (tenant_id) | - | - |
| alert_conditions | id (autoincrement) | tenants (tenant_id) | - | - |
| alert_medical_institutions | id (autoincrement) | alerts (alert_id)<br>tenants (tenant_id) | [alert_id, medical_institution_id](Unique)<br>[alert_id] | - |
| alerts | id (autoincrement) | tenants (tenant_id) | - | - |
| area_report_definitions | id (autoincrement) | tenants (tenant_id)<br>area_report_definition_folders (folder_id) | - | - |
| area_report_definition_folders | id (autoincrement) | area_report_definition_folders (parent_folder_id)<br>tenants (tenant_id) | - | - |
| area_report_definition_display_items | id (autoincrement) | area_report_definitions (area_report_definition_id)<br>area_report_items (item_id)<br>tenants (tenant_id) | [item_id](Unique) | - |
| area_report_definition_filter_conditions | id (autoincrement) | area_report_definitions (area_report_definition_id)<br>area_report_items (item_id)<br>tenants (tenant_id) | - | - |
| area_report_definition_filter_matching_groups | id (autoincrement) | area_report_definition_filter_conditions (area_report_definition_filter_condition_id)<br>tenants (tenant_id) | - | - |
| area_report_definition_sort_conditions | id (autoincrement) | area_report_definitions (area_report_definition_id)<br>area_report_items (item_id)<br>tenants (tenant_id) | - | - |
| area_report_items | id (autoincrement) | area_report_definitions (area_report_definition_id)<br>tenants (tenant_id) | - | - |
| area_report_item_aggregation_conditions | id (autoincrement) | area_report_items (item_id)<br>tenants (tenant_id) | [item_id](Unique) | - |
| area_report_item_communication_label_conditions | id (autoincrement) | area_report_item_aggregation_conditions (aggregation_condition_id)<br>tenants (tenant_id) | - | - |
| area_report_item_formulas | id (autoincrement) | area_report_items (item_id)<br>area_report_items (left_item_id)<br>area_report_items (right_item_id)<br>tenants (tenant_id) | [item_id](Unique) | - |
| audits | id (autoincrement) | - | [associated_type, associated_id]<br>[auditable_type, auditable_id, version]<br>[associated_id]<br>[created_at]<br>[request_uuid]<br>[user_id, user_type] | - |
| communication_comments | id (autoincrement) | tenants (tenant_id)<br>communications (communication_id) | [communication_id]<br>[user_id] | - |
| communication_label_selections | id (autoincrement) | tenants (tenant_id)<br>communication_labels (label_id) | [label_id, order] | - |
| communication_label_value_selections | id (autoincrement) | tenants (tenant_id)<br>communication_label_values (value_id) | [value_id, selection_id](Unique) | - |
| communication_label_values | id (autoincrement) | tenants (tenant_id)<br>communications (communication_id)<br>communication_labels (label_id) | [communication_id, label_id](Unique)<br>[tenant_id, label_id]<br>[tenant_id, updated_at] | - |
| communication_labels | id (autoincrement) | tenants (tenant_id) | [tenant_id, communication_type, name](Unique) | - |
| communication_likes | id (autoincrement) | tenants (tenant_id)<br>communications (communication_id) | [communication_id, user_id](Unique)<br>[communication_id]<br>[user_id] | - |
| communications | id (autoincrement) | tenants (tenant_id) | [communication_type, acted_at]<br>[tenant_id, medical_institution_id]<br>[user_id] | - |
| communication_default_items | id (uuid) | tenants (tenant_id) | [tenant_id, communication_type, type](Unique) | `@updatedAt` あり |
| communication_classify_settings | id (uuid) | tenants (tenant_id) | [tenant_id](Unique) | - |
| crm_vocs | id (uuid) | tenants (tenant_id)<br>crm_voc_groups (voc_group_id) | [tenant_id, medical_institution_id] | - |
| crm_communication_vocs | id (uuid) | tenants (tenant_id)<br>communications (communication_id)<br>crm_vocs (voc_id) | [tenant_id, communication_id, voc_id](Unique) | - |
| crm_voc_groups | id (uuid) | tenants (tenant_id) | - | - |
| crm_medical_institution_special_notes | id (uuid) | tenants (tenant_id) | [tenant_id, medical_institution_id](Unique) | - |
| communication_report_definitions | id (autoincrement) | tenants (tenant_id)<br>communication_report_definition_folders (folder_id) | - | - |
| communication_report_definition_folders | id (autoincrement) | communication_report_definition_folders (parent_folder_id)<br>tenants (tenant_id) | - | - |
| communication_report_definition_display_items | id (autoincrement) | communication_report_definitions (communication_report_definition_id)<br>communication_report_items (item_id)<br>tenants (tenant_id) | [item_id](Unique) | - |
| communication_report_definition_filter_conditions | id (autoincrement) | communication_report_definitions (communication_report_definition_id)<br>communication_report_items (item_id)<br>tenants (tenant_id) | - | - |
| communication_report_definition_filter_matching_groups | id (autoincrement) | communication_report_definition_filter_conditions (communication_report_definition_filter_condition_id)<br>tenants (tenant_id) | - | - |
| communication_report_definition_sort_conditions | id (autoincrement) | communication_report_definitions (communication_report_definition_id)<br>communication_report_items (item_id)<br>tenants (tenant_id) | - | - |
| communication_report_items | id (autoincrement) | communication_report_definitions (communication_report_definition_id)<br>tenants (tenant_id) | - | - |
| communication_report_item_aggregation_conditions | id (autoincrement) | communication_report_items (item_id)<br>tenants (tenant_id) | [item_id](Unique) | - |
| communication_report_item_communication_label_conditions | id (autoincrement) | communication_report_item_aggregation_conditions (aggregation_condition_id)<br>tenants (tenant_id) | - | - |
| communication_report_item_formulas | id (autoincrement) | communication_report_items (item_id)<br>communication_report_items (left_item_id)<br>communication_report_items (right_item_id)<br>tenants (tenant_id) | [item_id](Unique) | - |
| communication_report_definition_aggregation_rows | id (autoincrement) | communication_report_definitions (communication_report_definition_id)<br>communication_report_items (item_id)<br>tenants (tenant_id) | - | - |
| communication_report_definition_aggregation_columns | id (autoincrement) | communication_report_definitions (communication_report_definition_id)<br>communication_report_items (item_id)<br>tenants (tenant_id) | - | - |
| communication_report_definition_aggregation_data | id (autoincrement) | communication_report_definitions (communication_report_definition_id)<br>communication_report_items (item_id)<br>tenants (tenant_id) | - | - |
| communications_user_importing_logs | id (autoincrement) | tenants (tenant_id) | [tenant_id, created_at] | - |
| customized_dpc_specialty_codes | id (autoincrement) | tenants (tenant_id) | [tenant_id, code](Unique) | - |
| dashboards | id (autoincrement) | tenants (tenant_id) | - | - |
| dashboard_contents | id (autoincrement) | dashboards (dashboard_id)<br>tenants (tenant_id) | - | - |
| doctors | id (autoincrement) | tenants (tenant_id) | [medical_institution_id] | - |
| dpc_specialty_codes | id (autoincrement) | - | [code](Unique) | - |
| email_attachments | id (autoincrement) | emails (email_id)<br>tenants (tenant_id) | [email_id]<br>[uploaded_file_id] | - |
| email_stats | id (autoincrement) | emails (email_id)<br>tenants (tenant_id) | [email_id]<br>[sg_message_id] | - |
| emails | id (autoincrement) | tenants (tenant_id) | - | - |
| crm_email_sender_allowed_addresses | id (autoincrement) | tenants (tenant_id) | - | - |
| group_medical_institutions | id (autoincrement) | tenants (tenant_id) | [tenant_id, medical_institution_id](Unique)<br>[medical_institution_id]<br>[tenant_id] | - |
| medical_institution_label_areas | id (autoincrement) | tenants (tenant_id)<br>medical_institution_labels (label_id) | [label_id, priority]<br>[label_id] | - |
| medical_institution_label_area_addresses | id (autoincrement) | tenants (tenant_id)<br>medical_institution_label_areas (area_id) | [area_id, order]<br>[area_id] | - |
| medical_institution_label_selections | id (autoincrement) | tenants (tenant_id)<br>medical_institution_labels (label_id) | [label_id, order]<br>[label_id] | - |
| medical_institution_label_value_selections | id (autoincrement) | tenants (tenant_id)<br>medical_institution_label_values (value_id)<br>medical_institution_label_selections (selection_id) | [value_id, selection_id](Unique) | - |
| medical_institution_label_values | id (autoincrement) | tenants (tenant_id)<br>medical_institution_labels (label_id) | [medical_institution_id, label_id, selection_id](Unique)<br>[medical_institution_id, label_id, area_id](Unique)<br>[tenant_id, label_id] | - |
| medical_institution_label_values_importing_logs | id (autoincrement) | tenants (tenant_id) | - | - |
| medical_institution_labels | id (autoincrement) | tenants (tenant_id) | - | - |
| medical_institution_memos | id (autoincrement) | tenants (tenant_id) | [tenant_id, medical_institution_id](Unique) | - |
| medical_institution_report_definition_display_items | id (autoincrement) | medical_institution_report_definitions (medical_institution_report_definition_id)<br>medical_institution_report_items (item_id)<br>tenants (tenant_id) | - | - |
| medical_institution_report_definition_filter_conditions | id (autoincrement) | medical_institution_report_definitions (medical_institution_report_definition_id)<br>tenants (tenant_id) | - | - |
| medical_institution_report_definition_filter_matching_groups | id (autoincrement) | medical_institution_report_definition_filter_conditions (medical_institution_report_definition_filter_condition_id)<br>tenants (tenant_id) | - | - |
| medical_institution_report_definition_sort_conditions | id (autoincrement) | medical_institution_report_definitions (medical_institution_report_definition_id)<br>tenants (tenant_id) | - | - |
| medical_institution_report_definitions | id (autoincrement) | tenants (tenant_id)<br>medical_institution_report_definition_folders (folder_id) | - | - |
| medical_institution_report_definition_folders | id (autoincrement) | medical_institution_report_definition_folders (parent_folder_id)<br>tenants (tenant_id) | - | - |
| report_favorites | id (autoincrement) | tenants (tenant_id) | [user_id, favoritable_type, favoritable_id](Unique)<br>[favoritable_type, favoritable_id, user_id] | - |
| medical_institution_report_item_aggregation_conditions | id (autoincrement) | medical_institution_report_items (item_id)<br>tenants (tenant_id) | - | - |
| medical_institution_report_item_communication_label_conditions | id (autoincrement) | medical_institution_report_item_aggregation_conditions (aggregation_condition_id)<br>tenants (tenant_id) | - | - |
| medical_institution_report_item_formulas | id (autoincrement) | medical_institution_report_items (item_id)<br>tenants (tenant_id) | - | - |
| medical_institution_report_items | id (autoincrement) | tenants (tenant_id) | - | - |
| medical_institution_report_alert_settings | id (autoincrement) | medical_institution_report_definitions (medical_institution_report_definition_id)<br>tenants (tenant_id) | [medical_institution_report_definition_id, user_id](Unique) | - |
| medical_institution_report_alert_events | id (autoincrement) | medical_institution_report_definitions (medical_institution_report_definition_id)<br>tenants (tenant_id) | [alert_date] | - |
| crm_phase_masters | id (uuid) | tenants (tenant_id) | [tenant_id, name](Unique) | - |
| crm_medical_institution_phases | id (uuid) | tenants (tenant_id)<br>crm_phase_masters (phase_id)<br>users (changed_user_id) | [tenant_id, medical_institution_id](Unique) | - |
| crm_medical_institution_phase_change_histories | id (uuid) | tenants (tenant_id)<br>users (changed_user_id) | [tenant_id, medical_institution_id] | - |
| own_department_report_definition_display_items | id (autoincrement) | own_department_report_definitions (own_department_report_definition_id)<br>tenants (tenant_id) | - | - |
| own_department_report_definition_filter_conditions | id (autoincrement) | own_department_report_definitions (own_department_report_definition_id)<br>tenants (tenant_id) | - | - |
| own_department_report_definition_filter_matching_groups | id (autoincrement) | own_department_report_definition_filter_conditions (own_department_report_definition_filter_condition_id)<br>tenants (tenant_id) | - | - |
| own_department_report_definition_sort_conditions | id (autoincrement) | own_department_report_definitions (own_department_report_definition_id)<br>tenants (tenant_id) | - | - |
| own_department_report_definitions | id (autoincrement) | tenants (tenant_id)<br>own_department_report_definition_folders (folder_id) | - | - |
| own_department_report_definition_folders | id (autoincrement) | own_department_report_definition_folders (parent_folder_id)<br>tenants (tenant_id) | - | - |
| own_department_report_item_aggregation_conditions | id (autoincrement) | own_department_report_items (item_id)<br>tenants (tenant_id) | - | - |
| own_department_report_item_communication_label_conditions | id (autoincrement) | own_department_report_item_aggregation_conditions (aggregation_condition_id)<br>tenants (tenant_id) | - | - |
| own_department_report_item_medical_institution_label_conditions | id (autoincrement) | own_department_report_item_aggregation_conditions (aggregation_condition_id)<br>tenants (tenant_id) | - | - |
| own_department_report_item_formulas | id (autoincrement) | own_department_report_items (item_id)<br>tenants (tenant_id) | - | - |
| own_department_report_items | id (autoincrement) | tenants (tenant_id) | - | - |
| own_department_tenants | id (autoincrement) | own_departments (own_department_id)<br>tenants (tenant_id) | [tenant_id, own_department_id](Unique) | - |
| own_departments | id (autoincrement) | - | - | - |
| own_department_groups | id (uuid) | tenants (tenant_id) | - | `@updatedAt` あり |
| own_department_group_own_departments | id (uuid) | tenants (tenant_id)<br>own_department_groups (own_department_group_id)<br>own_departments (own_department_id) | [own_department_group_id, own_department_id](Unique) | `@updatedAt` あり |
| referral_admission_not_counted_own_departments | id (autoincrement) | tenants (tenant_id)<br>own_departments (own_department_id) | - | - |
| referral_admission_segments | id (autoincrement) | tenants (tenant_id) | - | - |
| tag_groups | id (autoincrement) | tenants (tenant_id) | [tenant_id, name](Unique) | - |
| tag_medical_institutions | id (autoincrement) | tags (tag_id)<br>tenants (tenant_id) | [tag_id, medical_institution_id](Unique)<br>[tag_id] | - |
| tags | id (autoincrement) | tenants (tenant_id) | [name, tenant_id](Unique) | - |
| tasks | id (autoincrement) | tenants (tenant_id) | [taskable_id, taskable_type] | - |
| tenant_default_facility_types | id (autoincrement) | tenants (tenant_id) | - | - |
| crm_tenant_competitive_medical_institutions | id (uuid) | tenants (tenant_id) | [tenant_id, medical_institution_id](Unique) | `@updatedAt` あり |
| tenants | id (autoincrement) | - | - | - |
| uploaded_files | id (autoincrement) | tenants (tenant_id) | - | - |
| users | id (autoincrement) | tenants (tenant_id) | [reset_password_token](Unique)<br>[confirmation_token](Unique)<br>[email]<br>[login_name]<br>[tenant_id] | - |
| mc_map_tenant_default_facility_types | id (autoincrement) | tenants (tenant_id) | [tenant_id, facility_type_id](Unique) | - |
| data_importing_logs | id (uuid) | tenants (tenant_id) | [tenant_id, data_type, target_month](Unique)<br>[tenant_id]<br>[target_month] | - |
| tenant_data_availabilities | tenant_id | tenants (tenant_id) | - | - |
| op_optimizer_reverse_referral_candidate_notifications | id (uuid) | tenants (tenant_id)<br>op_optimizer_target_lists (target_list_id) | - | - |
| op_optimizer_departments | id (uuid) | tenants (tenant_id) | [tenant_id, name](Unique) | `@updatedAt` あり |
| op_optimizer_doctors | id (uuid) | tenants (tenant_id)<br>op_optimizer_departments (department_id) | [tenant_id, department_id, name](Unique) | `@updatedAt` あり |
| op_optimizer_doctor_contacts | id (uuid) | tenants (tenant_id)<br>op_optimizer_doctors (doctor_id) | [doctor_id](Unique) | `@updatedAt` あり |
| op_optimizer_staff_contacts | id (uuid) | tenants (tenant_id)<br>op_optimizer_doctors (doctor_id) | - | `@updatedAt` あり |
| op_optimizer_outpatient_reservations | id (uuid) | tenants (tenant_id)<br>op_optimizer_departments (department_id)<br>op_optimizer_doctors (doctor_id) | - | `@updatedAt` あり |
| op_optimizer_operation_reservations | id (uuid) | tenants (tenant_id) | - | `@updatedAt` あり |
| op_optimizer_target_patients | id (uuid) | tenants (tenant_id)<br>op_optimizer_target_lists (target_list_id) | [target_list_id, patient_number](Unique) | `@updatedAt` あり |
| op_optimizer_target_lists | id (uuid) | tenants (tenant_id) | - | `@updatedAt` あり |
| op_optimizer_target_patient_statuses | id (uuid) | tenants (tenant_id)<br>op_optimizer_target_lists (target_list_id) | [target_list_id, patient_number](Unique) | `@updatedAt` あり |
| op_optimizer_target_list_doctors | id (uuid) | tenants (tenant_id)<br>op_optimizer_target_lists (target_list_id)<br>op_optimizer_doctors (doctor_id) | [target_list_id, doctor_id](Unique) | `@updatedAt` あり |
| op_optimizer_allowed_ip_addresses | id (uuid) | tenants (tenant_id) | [tenant_id, allowed_ip_address](Unique) | `@updatedAt` あり |
| op_optimizer_reservation_importing_logs | id (uuid) | tenants (tenant_id) | - | `@updatedAt` あり |
| op_optimizer_outpatient_import_key_mappings | id (uuid) | tenants (tenant_id) | [tenant_id](Unique) | `@updatedAt` あり |
| op_optimizer_operation_import_key_mappings | id (uuid) | tenants (tenant_id) | [tenant_id](Unique) | `@updatedAt` あり |
| outpatient_allowed_ip_addresses | id (uuid) | tenants (tenant_id) | [tenant_id, allowed_ip_address](Unique) | `@updatedAt` あり |
| crm_allowed_ip_addresses | id (uuid) | tenants (tenant_id) | [tenant_id, allowed_ip_address](Unique) | `@updatedAt` あり |
| acute_cf_allowed_ip_addresses | id (uuid) | tenants (tenant_id) | [tenant_id, allowed_ip_address](Unique) | `@updatedAt` あり |
| medical_institution_attachment_files | id (uuid) | tenants (tenant_id) | [tenant_id, medical_institution_id] | `@updatedAt` あり |
| acute_cf_patients | id (uuid) | tenants (tenant_id)<br>users (created_by_user_id)<br>users (updated_by_user_id)<br>users (reception_staff_user_id)<br>acute_cf_ward_masters (ward_master_id)<br>acute_cf_disease_masters (disease_master_id)<br>acute_cf_referral_medical_institutions (referral_medical_institution_id) | [tenant_id, display_id](Unique)<br>[tenant_id, id](Unique)<br>[tenant_id, created_at]<br>[tenant_id, referral_medical_institution_id]<br>[tenant_id, disease_master_id]<br>[tenant_id, ward_master_id] | - |
| acute_cf_patient_status_histories | id (uuid) | tenants (tenant_id)<br>users (created_by_user_id)<br>acute_cf_patients (patient_id) | [tenant_id, patient_id, created_at] | - |
| acute_cf_patient_social_informations | id (uuid) | tenants (tenant_id)<br>users (created_by_user_id)<br>acute_cf_patients (patient_id) | [patient_id](Unique) | - |
| acute_cf_patient_details | id (uuid) | tenants (tenant_id)<br>acute_cf_patients (tenant_id, patient_id)<br>users (created_by_user_id)<br>users (updated_by_user_id) | [tenant_id, patient_id](Unique)<br>[patient_id](Unique) | `@updatedAt` あり |
| acute_cf_patient_sensitive_details | id (uuid) | tenants (tenant_id)<br>acute_cf_patients (tenant_id, patient_id)<br>users (created_by_user_id)<br>users (updated_by_user_id) | [tenant_id, patient_id](Unique)<br>[patient_id](Unique) | `@updatedAt` あり |
| acute_cf_ward_masters | id (uuid) | tenants (tenant_id) | [tenant_id, name](Unique)<br>[tenant_id, order] | - |
| acute_cf_disease_masters | id (uuid) | tenants (tenant_id) | [tenant_id, name](Unique)<br>[tenant_id, order] | - |
| acute_cf_patient_diseases | id (uuid) | tenants (tenant_id)<br>acute_cf_disease_masters (disease_master_id)<br>acute_cf_patients (patient_id) | [disease_master_id, patient_id](Unique)<br>[tenant_id]<br>[patient_id] | - |
| acute_cf_referral_medical_institutions | id (uuid) | tenants (tenant_id)<br>users (created_by_user_id) | [tenant_id, medical_institution_id](Unique)<br>[tenant_id, order] | - |
| acute_cf_attachment_files | id (uuid) | tenants (tenant_id)<br>acute_cf_patients (patient_id)<br>users (created_by_user_id) | [tenant_id, patient_id] | - |
| acute_cf_patient_status_settings | id (uuid) | tenants (tenant_id) | [tenant_id, disabled_status](Unique)<br>[tenant_id] | `@updatedAt` あり |

※ 現行 Prisma スキーマの全161モデルを掲載。インデックス欄はユニーク制約を `(Unique)`、Ginインデックスを `(Gin)` として表示。

---

## 4. refery スキーマ (`refery_development`)

| テーブル名 (`@@map`) | PK (SEQ) | 外部キー (FK) 先テーブル | インデックス・ユニーク制約 カラム | 備考 |
|---|---|---|---|---|
| tenants | id (uuid) | closed_days_files | [name](Unique)<br>[emailAddress](Unique)<br>[closedDaysFileId](Unique) | `@updatedAt` あり |
| users | id (uuid) | tenants<br>guests | [emailAddress](Unique)<br>[userName](Unique) | `@updatedAt` あり |
| departments | id (uuid) | tenants<br>uploaded_files(x2) | - | `@updatedAt` あり |
| department_referrer_restrictions | departmentId, referrerId | departments<br>referrers | - | `@updatedAt` あり |
| doctors | id (uuid) | departments<br>tenants | - | `@updatedAt` あり |
| working_terms | id (uuid) | tenants | - | `@updatedAt` あり |
| working_term_doctors | workingTermId, doctorId | working_terms<br>doctors | - | `@updatedAt` あり |
| work_shift_items | id (uuid) | doctors<br>working_terms<br>tenants | - | `@updatedAt` あり |
| reservation_slots | id (uuid) | doctors<br>tenants | - | `@updatedAt` あり |
| questions | id (uuid) | tenants<br>departments | - | `@updatedAt` あり |
| answer_options | id (uuid) | tenants<br>questions(x2)<br>assignments | - | `@updatedAt` あり |
| assignments | id (uuid) | tenants | - | `@updatedAt` あり |
| time_period_assignments | id (uuid) | assignments | [assignmentId, timePeriod](Unique) | `@updatedAt` あり |
| doctor_assignments | assignmentId, doctorId | assignments<br>doctors | - | `@updatedAt` あり |
| reservations | id (uuid) | tenants<br>reservation_slots<br>guests<br>users<br>departments | [COALESCE(cancellation_requested_at, last_modification_requested_at, reservation_requested_at)](式インデックス) | `@updatedAt` あり |
| requested_appointment_dates | id (uuid) | reservations<br>temporary_reservations | - | `@updatedAt` あり |
| requested_results | id (uuid) | reservations | - | `@updatedAt` あり |
| referrers | id (uuid) | tenants<br>guests | - | `@updatedAt` あり |
| guests | id (uuid) | - | - | `@updatedAt` あり |
| temporary_reservations | id (uuid) | tenants<br>guests<br>reservation_slots | [reservationSlotId](Unique) | `@updatedAt` あり |
| uploaded_files | id (uuid) | tenants | - | - |
| closed_days | id (uuid) | closed_days_files | - | `@updatedAt` あり |
| closed_days_files | id (uuid) | - | - | - |

※ `apps/api/prisma/schema.prisma` の全23モデルを掲載。`tenants.email_address` のユニークインデックスと、Prismaで表現されない予約受付日時の式インデックスは、それぞれマイグレーション `20250828013255_add_column_email_address_to_tenants`、`20250619062103_add_each_request_datetime_on_reservations` で作成される。

## foro-crm-api 追記（Rails DB）

### 対象と接続

この節だけを `foro-crm-api` の実装対象とする。スキーマ定義の正本は `db/schema.rb` で、Active Record を使用する Rails API は `foro` の業務DBへ接続する。`medical_institutions` などの MIS 実体はRailsの `db/schema.rb` にないため、外部サービス・外部DB参照として扱う。

| 実行基盤 | 接続先 | コード上の定義 | 役割 |
| :--- | :--- | :--- | :--- |
| `mt_web` | `foro_dev` 相当の `public` スキーマ | `app/models`、`db/schema.rb` | Rails REST API。業務データのCRUDと集計を実行する。 |
| Rails Worker | `mt_web` と同じ業務DB | `app/jobs`、`db/schema.rb` | CSV取込、アラート生成、通知、レポート通知を実行する。 |
| MIS連携 | MISデータベース | `app/services` 等の外部参照 | 医療機関・診療科・施設種別等を参照する。 |

### スキーマ照合結果

- `db/schema.rb` の `create_table` は160件。
- `_prisma_migrations` を含むため、上記のPrismaモデル件数とは単純比較しない。
- `ar_internal_metadata`、`schema_migrations` はRails管理テーブルとして存在する。
- PostgreSQL RLSにより、テナント対象テーブルはリクエスト時のテナント設定に従って参照・更新される。
- 上記の `foro` テーブル一覧にないRails固有の業務テーブルがある場合は、`db/schema.rb` を優先して更新する。

### Railsが直接利用する主なテーブル群

| 機能群 | 主なテーブル |
| :--- | :--- |
| 認証・テナント | `users`、`tenants`、`crm_allowed_ip_addresses`、`outpatient_allowed_ip_addresses`、`audits` |
| 医療機関CRM | `medical_institution_memos`、`medical_institution_labels`、`medical_institution_label_values`、`medical_institution_label_*`、`tags`、`tag_groups`、`tag_medical_institutions` |
| コミュニケーション | `communications`、`communication_comments`、`communication_likes`、`communication_labels`、`communication_label_values`、`communication_label_selections`、`communication_default_items` |
| VOC | `crm_vocs`、`crm_communication_vocs`、`crm_voc_groups` |
| 紹介・逆紹介分析 | `referrals`、`reverse_referrals`、`referral_replies`、`hospital_admissions`、`referral_hospital_admissions`、`hospital_admission_referrals` |
| タスク・アクション | `tasks`、`action_lists`、`action_list_medical_institutions` |
| レポート | `medical_institution_report_*`、`own_department_report_*`、`area_report_*`、`communication_report_*`、`report_favorites` |
| メール | `emails`、`email_attachments`、`email_stats`、`crm_email_sender_allowed_addresses` |
| アラート・通知 | `alert_conditions`、`alerts`、`alert_medical_institutions`、`medical_institution_report_alert_settings`、`medical_institution_report_alert_events` |
| セグメント・フェーズ | `medup_foro_segmentation_settings`、`medup_foro_segments`、`medup_foro_medical_institution_segments`、`crm_phase_masters`、`crm_medical_institution_phases`、履歴表 |
| 取込・ファイル | `uploaded_files`、`communications_user_importing_logs`、`medical_institution_label_values_importing_logs`、各取込ログ |
| 参照マスタ・集計 | `own_departments`、`dpc_revisions`、`dpc_mdcs`、`dpc_diagnoses`、`dpc_code_scores`、`kcode_scores`、`dpc_specialty_codes` |

`acute_cf_*`、`op_optimizer_*` 等の一部テーブルも同じ `foro` DB上にあるが、主な更新主体は別サービスである。RailsのCRUD一覧では当該Railsルート・ジョブから確認できる操作だけを対象とする。

## data-management-api 追記（FastAPI / SQLAlchemy）

### 対象と接続

この節だけを `data-management-api` の実装対象とする。テーブル定義は [`src/gateways/db/rdb/schemas`](../../src/gateways/db/rdb/schemas) のSQLAlchemyモデルで表現されるが、このリポジトリ自身はマイグレーション管理を行わず、既存の `foro`・`mis`・`data_ops` DBへ接続して利用する。

| 接続設定 | 接続先 | コード上の定義 | 主な役割 |
| :--- | :--- | :--- | :--- |
| `FORO_DATABASE_URL` | foro 業務DB | [`medup_foro`](../../src/gateways/db/rdb/schemas/medup_foro) | 入院・紹介データ取込、紹介入院・紹介回答の関連再生成 |
| `MIS_DATABASE_URL` | MIS DB | [`mis`](../../src/gateways/db/rdb/schemas/mis) | 医療機関、施設種別、厚生労働省同期作業レコードの参照・登録 |
| `DATA_OPS_DATABASE_URL` | Data Ops DB | [`data_ops`](../../src/gateways/db/rdb/schemas/data_ops) | マッピング作業、テナント医療機関コード、ナビィ一時データの参照・更新 |
| `GCP_DATA_MANAGEMENT_PROJECT_ID` / `BQ_DATASET_ID` | BigQuery | [`processed_medical_institution_repository_imp.py`](../../src/gateways/db/bigquery/repositories/processed_medical_institution_repository_imp.py) | `mis_facilities` の参照・upsert |

### 本APIが直接利用する永続テーブル

| DB | 主なテーブル | 利用内容 |
| :--- | :--- | :--- |
| foro | `hospital_admissions` | 入院取込データを一括登録し、紹介入院候補の照合に参照する。 |
| foro | `hospital_admission_importing_logs` | 入院取込ログを参照し、完了・件数・失敗理由へ更新する。 |
| foro | `medup_foro_outpatient_ef_importing_logs` | EF取込元ファイルのパスを参照し、完了・件数・失敗理由へ更新する。 |
| foro | `referrals`、`reverse_referrals` | 入院・紹介の関連付け、および紹介と逆紹介の回答関係を抽出する際に参照する。 |
| foro | `hospital_admission_referrals`、`referral_hospital_admissions` | テナント単位で既存行を削除し、入院・紹介の関連を再登録する。 |
| foro | `referral_replies` | テナント単位で既存行を削除し、抽出した紹介・逆紹介の回答関連を再登録する。 |
| MIS | `medical_institutions` | 同期、名称・住所マッチング、厚労省照合、廃止施設除外に参照する。 |
| MIS | `medical_institution_facility_types` | 医科施設の優先検索、および施設種別を絞った厚労省マッチングに参照する。 |
| MIS | `mhlw_sync_work_insert_records`、`mhlw_sync_work_update_records`、`mhlw_sync_work_close_records` | 厚生労働省の新設・廃止報告から追加・更新・廃止候補を登録する。既存更新候補は一覧照合にも利用する。 |
| MIS | `mhlw_sync_work_overview_diff_records`、`mhlw_sync_work_overview_only_list_records`、`mhlw_sync_work_overview_only_mis_records` | 厚生労働省一覧とMISの差分分類結果を登録する。 |
| MIS | `m_ref_naviis_*` | Refナビィ加工時の支払方法、保険医療費公費、外国語、障害者支援、車椅子配慮、専門医、疾患、予防接種等のマスタ参照に利用する。 |
| Data Ops | `tenant_medical_institution_code_ids` | テナントの医療機関等コードとMIS医療機関IDの対応を参照する。 |
| Data Ops | `medical_institution_mappings` | テナント別の名称・住所と医療機関IDの確定マッピングを参照する。 |
| Data Ops | `medical_institution_mapping_work_sheets`、`medical_institution_mapping_work_sheet_sections`、`medical_institution_mapping_work_sheet_section_records` | 作業シート構造を読み、推測マッチング後にセクションレコードの医療機関IDを更新する。 |
| Data Ops | `navii_medical_institution_mappings` | Refナビィの名称・住所と医療機関IDの対応を参照し、未確定データではupsertする。 |
| Data Ops | `tmp_ref_naviis`、`tmp_ref_naviis_consultation_hours` | Refナビィ更新結果の投入先。リセット時は親子データを一括削除し、県単位の投入に対応する。 |

### 分析基盤・一時データ

| 保管先 | 名称 | 利用内容 |
| :--- | :--- | :--- |
| BigQuery | `mis_facilities` | MIS同期バッチで処理対象レコードを削除・再挿入し、名称・住所マッチングの候補として参照する。 |
| PostgreSQL セッション | `temp_navii_seeds` | RefナビィのGCSシードを `COPY` で一時投入し、`navii_medical_institution_mappings` と結合する。接続後に残す永続テーブルではない。 |
| Cloud Storage | ナビィシード、入出力ファイル | JSONL gzipのRefナビィシード、FF1・Dファイル、E/F/EFファイル、マッチング入力・結果CSVを保持する。DBスキーマの対象外。 |

## foro-crm-devops 追記（分析基盤・運用データ）

このリポジトリは新しい業務DBスキーマのマイグレーション管理を行わない。既存の `foro`、`mis`、`data_ops` を参照・更新し、BigQuery、Cloud Storage、AlloyDBバックアップへデータを出力する。

### PostgreSQL への直接利用

| DB | テーブル・項目 | DevOps処理 | 更新・参照内容 |
| :--- | :--- | :--- | :--- |
| foro | `tenants` | レポート更新ディスパッチ | `enabled_medical_institution_report = true` の対象テナントIDを参照する。 |
| foro | 紹介、入院、コミュニケーション、ラベル、セグメント、VOC等の業務テーブル | テナント別レポート更新、seedダンプ | テナントIDをRLSへ設定して分析抽出する。seedではテナント依存表を指定テナントに絞り、非テナント表を併せてCSVへ出力する。 |
| foro | `users.aad_user_id` | Azure AD B2C利用者移行 | B2Cユーザー作成後、そのオブジェクトIDを対応する `users` 行へ更新する。 |
| MIS | `medical_institutions`、`medical_institution_specialties`、`medical_institution_facility_types`、`medical_institution_logs` | 共通レポート更新、MIS seedダンプ | 医療機関属性と厚生労働省指定履歴をBigQueryへ抽出する。seedでは `public` スキーマをデータのみでダンプする。 |
| MIS | `ref_naviis`、`ref_naviis_consultation_hours` | ナビィ本番反映 | 80%件数チェック後、既存行をトランザクション内で全削除し、Data Ops一時表からページ単位で再登録する。 |
| Data Ops | `tmp_ref_naviis`、`tmp_ref_naviis_consultation_hours` | ナビィマッチングと本番反映 | 外部マッチングAPIが県別に投入・リセットする一時データを参照し、MISへコピーする。 |

### BigQuery 分析テーブル

| データセット | テーブル | 更新方式・用途 |
| :--- | :--- | :--- |
| 共通レポート用 `<prefix>common` | `medical_institution_attributes` | MISの医療機関属性、診療科ID、施設種別ID等をGCS JSONL経由で `WRITE_TRUNCATE` する。 |
| 共通レポート用 `<prefix>common` | `mhlw_designation_histories` | `medical_institution_logs` の厚生労働省更新履歴を指定事由へ整形して置換する。 |
| 共通レポート用 `<prefix>common` | `medical_institution_navii_values` | `ref_naviis` の3タブJSONを `{ item_key: 値 }` へ正規化して置換する。空結果時は既存テーブルを消さない。 |
| テナント別 `<prefix><tenant_id>` | `medical_institution_attributes_tenant_extended`、`medical_institution_communications`、`medical_institution_label_values`、`medical_institution_referral_performance`、`medical_institution_reverse_referral_performance`、`referrals`、`hospital_admissions`、`own_department_attributes`、`area_attributes`、`medical_institution_segments`、`vocs` | テナント別RDBデータを抽出し、差分判定後にGCS JSONL経由でテーブル単位に置換する。データセット、テーブル、Webアプリ用SA権限がなければ作成する。 |

### Cloud Storage・バックアップ

| 保管先 | 内容 | 備考 |
| :--- | :--- | :--- |
| レポート用バケット | common・テナント別JSONL、更新時刻・カバレッジ統計 | BigQueryロード用の中間データ。更新スキップ判定にも利用する。 |
| SendGrid / Resend監査バケット | 1時間窓ごとのメールログJSONL gzip、`_SUCCESS` マーカー | アカウント・取得窓単位で冪等化する。監査ログは5年retentionを前提とし、メール業務DBへの取込は行わない。 |
| seed用バケット | foro seed tar.gz、MIS seed SQL | ローカル開発用。本番DBのダンプ成果物であり、DBスキーマ定義とは別管理。 |
| AlloyDB backup | クロスリージョンバックアップ | 7日保持を超えた同名形式のバックアップを運用ジョブが削除する。 |

メールログ取得、踏み台、URL Map、Cloud Scheduler設定更新は業務DBへ書き込まないため、テーブルCRUD一覧には対象永続先だけを記載する。

## outpatient-ops 追記（BigQuery 分析データ）

この節だけを `outpatient-ops` の実装対象とする。このリポジトリはPostgreSQLのスキーマや入力テーブルを作成・更新せず、BigQueryの既存データをSQLAlchemyで参照し、dbtで派生ビューを作成する。

### 接続設定

| 設定キー | 接続先 | 用途 |
| :--- | :--- | :--- |
| `GCP_PROJECT` | BigQueryプロジェクト（標準は `medup-foro`） | 全データセット共通のプロジェクト。 |
| `BQ_FORO_DATASET` | `foro_prd` | 紹介、入院、自部署、DPC診療科・病名・手術コードマスタ。 |
| `BQ_MIS_DATASET` | `mis_prd` | 医療機関、施設種別、診療科および関連マスタ。 |
| `BQ_DPC_PATIENTS_DATASET` | `dpc_patient_records_prd` | 外来EF、患者サマリー、医師コードマスタ。Python集計とdbt成果物の主対象。 |
| dbt `dev` / `prod` | `dpc_patient_records_stg` / `dpc_patient_records_prd` | intermediate・martsのビュー配備先。モデル設定により同名のデータセットへ展開する。 |

### BigQuery 入力テーブル・ビュー

| データセット | テーブル／ビュー | 利用内容 |
| :--- | :--- | :--- |
| `foro_prd` | `public_referrals` | テナント・患者単位の紹介履歴。直近紹介付与に利用する。 |
| `foro_prd` | `public_hospital_admissions` | 入院期間、DPC診療科、Kコード、DPCコードを参照し、入院フラグ・最新手術情報・病名を集計する。 |
| `foro_prd` | `public_hospital_admission_referrals`、`public_reverse_referrals` | 紹介入院関連と逆紹介イベントを時系列モデルへ含める。 |
| `foro_prd` | `public_own_departments` | 紹介の自院診療科名解決に利用する。 |
| `foro_prd` | `public_medical_institution_label_values`、`public_medical_institution_label_selections` | テナント135糖尿病科の逆紹介先候補施設IDをラベル・選択肢から取得する。dbt source定義ではなく、Pythonから直接参照する。 |
| `foro_prd` | `public_dpc_specialty_codes`、`public_customized_dpc_specialty_codes` | 基本診療科マスタとテナント別診療科コードを統合する。 |
| `foro_prd` | `public_dpc_code_scores`、`public_dpc_revisions`、`public_dpc_mdcs`、`public_kcode_scores` | DPC病名・手術・改定時期の参照。 |
| `mis_prd` | `public_medical_institutions`、`public_medical_institution_profiles` | 紹介元医療機関名・住所・緯度経度と候補施設の情報を参照する。 |
| `mis_prd` | `public_facility_types`、`public_medical_institution_facility_types`、`public_specialties`、`public_medical_institution_specialties` | 医療機関の属性候補。現行Pythonパイプラインでは医療機関本体と地理情報を主に利用する。 |
| `dpc_patient_records_prd` | `outpatient_ef` | 患者、受診日、診療科、医師、レセプトコード、行為、点数等の外来明細。患者サマリー、診療フラグ、コード該当判定の基礎。 |
| `dpc_patient_records_prd` | `doctor_code_master`、テナント固有 `*_doctor_master` | 医師コードから部門・医師名を解決する。 |
| `others` | `copy_dpc_code_scores` | 入退院イベントの病名・手術名補完に利用する。 |
| `dpc_patient_records_prd` | `view_tenant_<tenant_id>_patient_summary` | 旧年度ベースRepositoryが参照する動的なテナント別患者サマリービュー。v2期間指定集計は `outpatient_ef` 等を直接集計する。 |

### dbt 派生モデル

dbtのintermediate・martsは原則 `view` として物化し、入力テーブルを更新せずに参照式を置換する。`dbt_int_doctor_department` だけは明示的に `table` として物化する。

| 分類 | 主要モデル | 粒度・用途 |
| :--- | :--- | :--- |
| 外来集計 | `dbt_int_receipt_flags`、`dbt_int_outpatient_visits`、`dbt_int_outpatients` | 患者×日×診療科、費用・レセプトコード・行為明細。 |
| 入院・イベント | `dbt_int_admission_flag`、`dbt_int_surgical_admissions`、`dbt_int_referral_reverse_referral_admissions`、`dbt_int_chronological_order_data` | 入院有無、最新手術、紹介・逆紹介・入退院・外来の時系列。 |
| 患者サマリー | `dbt_int_patient_summary`、`dbt_int_patient_summary_overall` | 年度×患者×診療科、および年度×患者の全科サマリー。 |
| 医師・部門別 | `dbt_int_doctor_department`、`dbt_int_receipt_flags_by_doctor`、`dbt_int_outpatient_visits_by_department`、`dbt_int_patient_summary_by_department`、`dbt_int_patient_summary_overall_by_department`、`dbt_int_outpatients_by_department`、`dbt_int_chronological_order_data_by_department`、`dbt_int_outpatient_visit_classification_by_department` | 医師コードから解決した部門単位の外来・サマリー・時系列・受診分類。 |
| テナント別公開 | `dbt_tenant_11_*`、`dbt_tenant_204_*`、`dbt_tenant_274_*` | テナントおよび必要に応じて年度を絞ったmartsビュー。 |

### ローカル成果物・キャッシュ

| 保管先 | 内容 | 備考 |
| :--- | :--- | :--- |
| `data/` 配下 | テナントから受領した外来予約、手術予約、郵便番号、逆紹介先候補施設、職員等のCSV・Excel | パイプライン入力。列名はYAMLのマッピングで正規化する。 |
| `outputs/` 配下 | 逆紹介候補患者のCSVまたは医師別Excel | 実行日をファイル名・ディレクトリへ付与する。DBへの永続化はしない。 |
| `.cache/zipcode_latlon.json` | 郵便番号と緯度経度の対応 | ZipCloud・国土地理院APIの再利用キャッシュ。DBスキーマの対象外。 |