# DataOps 定例作業・MIS運用 統合リファレンス辞書

## 1. 収録案件・テナント一覧

### 1-1. 収録案件一覧
| 業務カテゴリ | 案件名 | 主な対象DB・システム | 実施・基準時期 |
| --- | --- | --- | --- |
| 月次入院データ投入 | がん研有明病院（tenant `204`）：匿名化患者番号の復号・洗替・月次投入 | foro Core Admin, Pythonスクリプト | 2026/09洗替（2606・2607）、2026/10月次（2608・2609） |
| 月次入院データ投入 | 静岡済生会総合病院（tenant `136`）：産婦人科（様式1なし）DB直接投入・紐付けAPI | `foro_prd`, `data-customer-ops`, 紐付けAPI | 2026/09（5〜7月経緯）、2026/10（2608・2609） |
| 初期設定 | 入院データ投入の初期設定（川崎市立多摩・広尾・群馬県立心臓血管センター） | `data_ops_prd` (`tenant_medical_institution_code_ids`), MIS管理画面 | 2026/08〜09（2026/10/08画面化） |
| エラー・重複削除 | 入院投入データエラー対応（済生会今治・JCHO大阪）／紹介データ重複削除（座間） | foro Core Admin, `referrals`, `referral_masters` | 2026/09/09 |
| 逆紹介候補抽出 | 外来適正化 月次データ抽出（済生会熊本・新東京・鳥取県立中央・がん研有明） | BigQuery (`outpatient_ef`), `outpatient-ops` | 2026/08〜09 |
| 逆紹介候補抽出 | 鳥取県立中央病院（tenant `274`）：3診療科抽出条件確定・BQ SQL作成 | BigQuery (`outpatient_ef`, `foro_prd`) | 2025/08〜2026/07（2026/08〜09作業） |
| 逆紹介候補抽出 | 新東京病院グループ（tenant `135`）：自己注射除外条件追加・未受領EF調査 | BigQuery (`outpatient_ef`), GitHub | 2025年度〜2026/06（2026/07〜08作業） |
| MISマスタ月次更新 | MIS月次作業（厚生局マスタ更新・エラー対処） | `mis_prd`, MIS管理ツール, `data-customer-ops` | 2026/08〜09 |
| MIS重複医療機関統合 | MISのデータ統合（仕様・全DBテーブル一覧・実務手順・11施設確認ログ） | `foro_prd`, `data_ops_prd`, `mis_prd`, Redash | 2026/07/08, 2026/08/04 |
| MIS不整合修正・改修 | アーカイブ医療機関マッピング修正／MIS診療科更新／referral紐付け修正 | Data Mgmt Admin, MIS Admin, `mis_prd`, `data_ops`, `foro_prd` | 2026/07〜10 |
| 通知先マスタ更新 | データ反映完了メール宛先変更（SOP・実績） | `data_ops_prd` (`public.completion_notification_users`) | 2026/10/01 |

### 1-2. 主な対象テナント・施設コード一覧
| テナントID | 施設名 | 施設コード / 施設ID / 医療機関ID | 主な関連業務 |
| --- | --- | --- | --- |
| `11` | 済生会熊本病院 | — | 外来適正化月次抽出（泌尿器科・呼吸器外科） |
| `13` | 海老名座間（グループ病院） | — | MIS統合時レポート系 `tenant_medical_institution_ids` 要確認 |
| `21` | JCHO大阪 | — | 入院投入データエラー対応 |
| `23` | （テナント23） | 四谷内科・内視鏡クリニック：`23197` → `229401` | referral紐付け直接修正 |
| `61` | 済生会今治 | — | 入院投入データエラー対応 |
| `135` | 新東京病院グループ（3施設統合） | — | 外来適正化月次抽出（心臓内科・糖尿病内科）／アーカイブマッピング修正 |
| `136` | 静岡済生会総合病院 | 施設ID: `48518` | 産婦人科（様式1なし）DB直接投入・紐付けAPI |
| `165` | 音羽／音羽リハ／音羽記念（グループ病院） | — | MIS統合時レポート系 `tenant_medical_institution_ids` 要確認 |
| `204` | がん研有明病院 | 施設コード: `130870923` | 入院データ復号投入・洗替／外来適正化（※2026/08終了） |
| `274` | 鳥取県立中央病院 | 鳥取大紹介元ID: `75462` | 3診療科逆紹介候補抽出（整形・脳外・泌尿） |
| `516` | 広尾病院 | 医療機関ID: `25786` / DPCコード: `136070007` | 入院データ投入初期設定 |
| `517` | 川崎市立多摩病院 | 医療機関ID: `40156` / DPCコード: `145402565` | 入院データ投入初期設定 |
| `551` | 群馬県立心臓血管センター | 医療機関ID: `14133` / DPCコード: `105010018` | 入院データ投入初期設定 |

---

## 2. 共通リファレンス（用語・基盤・実装ルール・コード辞書）

### 2-1. 医療・DPCデータ用語
- **様式1（FF1）**：DPC調査の診療録要約ファイル（入院日、退院日、傷病名ICD-10、手術Kコード、DPCコード等）。
- **Dファイル（Dn）**：DPC包括評価部分の明細ファイル。通常は様式1と月次セットで投入。
- **外来EFファイル**：出来高実績。Eファイル（行為単位：`action_detail_number = 0`）とFファイル（行為明細・加算等：`action_detail_number > 0`）。
- **DPCコード**：14桁の診断群分類番号。上6桁が疾患分類（例：`010010` 脳腫瘍、`010020` くも膜下出血、`110080` 前立腺悪性腫瘍）。
- **Kコード**：手術の診療報酬区分番号（例：`K044`、`K061`、`K178`、`K609-2`、`K843`）。
- **レセプト電算コード**：厚労省「診療報酬情報提供サービス」診療行為マスター（令和6年度版 `s_20260501.csv` 等）の9桁コード。参照URL：`https://shinryohoshu.mhlw.go.jp/shinryohoshu/downloadMenu/`
- **ICD-10**：国際疾病分類第10版（例：`O809` 自然分娩、`C64` 腎悪性新生物）。

### 2-2. データ基盤・テーブル定義

#### BigQuery
| テーブル名 | 主なカラム・用途 | 注意点 |
| --- | --- | --- |
| `medup-foro.dpc_patient_records_prd.outpatient_ef` | `tenant_id`, `patient_id`, `doctor_code`, `specialty_code`, `outpatient_date.value`, `receipt_code`, `action_name`, `action_fee`, `data_type`, `action_detail_number` | ・`data_type != 'SY'` で傷病名除外。<br>・通常は `action_detail_number = 0`（Eレコードのみ。Fを含めると件数2倍）。<br>・**例外**：造影剤使用加算等は明細行（`> 0`）のため `= 0` で絞らない。 |
| `medup-foro.dpc_patient_records_prd.dbt_int_doctor_department` | `tenant_id`, `doctor_code`, `department_name` | `doctor_code_master` はSheets外部表でDrive権限が必要なため実体テーブルである本表を使用。`SAFE_CAST(tenant_id AS INT64)` で突合。 |
| `medup-foro.foro_prd.public_hospital_admissions` | `tenant_id`, `patient_number`, `dpc_code`, `dpc_specialty_code`, `kcode_operation_1`, `admission_started_on`, `admission_ended_on` | ・患者ID列名は `patient_number`。<br>・手術Kコードは `kcode_operation_1` のみ（`_2` 以降はスキーマに無し）。<br>・遡及範囲は概ね2019年4月以降。 |
| `medup-foro.foro_prd.public_dpc_specialty_codes` | `code`, `name` | DPC診療科マスタ。 |
| `medup-foro.foro_prd.public_referrals` | `tenant_id`, `patient_id`, `specialty_id`, `medical_institution_id_from` | 紹介データ。医療機関名確認は `tenant_29_view_medical_institutions` 等を利用。 |
| `medup-foro.foro_prd.public_reverse_referrals` | `tenant_id`, `patient_id`, `specialty_id`, `reverse_referral_date` | 逆紹介データ（foro CRM流用）。 |
| `medup-foro.foro_prd.public_own_departments` / `public_own_department_tenants` | `id`, `name`, `own_department_id`, `tenant_id` | `public_own_departments` 自体に `tenant_id` は無く、`public_own_department_tenants` で紐付ける。 |
| `medup-foro.foro_prd.public_op_optimizer_outpatient_reservations` / `public_op_optimizer_doctors` | `tenant_id`, `patient_number`, `doctor_id`, `reserved_at`, `name` | `op_optimizer_prd` は別リージョンの可能性あり。Datastream連携済みの `foro_prd` を使用。 |

#### PostgreSQL（`foro_prd` / `data_ops_prd` / `mis_prd`）
- **`foro_prd`**：`hospital_admission_importing_logs`（取込ログ）、`hospital_admissions`（入院本体）、`hospital_admission_referrals` / `referral_hospital_admissions`（紹介・入院紐付け）、`referrals`、`reverse_referrals` 等。
  - **RLS設定（必須）**：クエリ実行・DBeaver閲覧前に `SET session.current_tenant_id = '<tenant_id>';` を実行。
- **`data_ops_prd`**：`tenant_medical_institution_code_ids`（入院バリデーション初期設定）、`completion_notification_users`（完了通知メール宛先）、`referral_masters`、`medical_institution_mappings` 等。
- **`mis_prd`**：`medical_institutions`、`department_mappings`、`mhlw_sync_work_*` 等。

### 2-3. 共通実装ルール
1. **患者ID正規化**：EFはゼロ埋め文字列（例：`0001948881`）、逆紹介・入院は非ゼロ埋めの場合があるため、`CAST(SAFE_CAST(x AS INT64) AS STRING)` で統一。
2. **最大行為カテゴリ（`medical_service_category`）優先順位**：
   1. `1.入院`（期間内対象診療科の入院あり）
   2. `2.初診`（`receipt_code` 先頭 `111`）
   3. `3.処置/手術/注射`（再診 `112` かつ `140` / `150` / `13`）
   4. `4.CT/MRI`（再診 `112` かつ `170033410`, `170012070`, `170011810`, `170035010`, `170020110`, `170020470`）
   5. `5.検査（US・XP含む）`（再診 `112` かつ `170` / `160`）
   6. `6.投薬`（再診 `112` かつ `12`）
   7. `7.リハ`（再診 `112` かつ `180756010`, `180024710`）
   8. `8.再診`（再診 `112` のみ）
   9. `9.その他`
3. **逆紹介リストA出力方針**：対象全員を行として出力し除外で行を消さない。除外条件ごとに `0/1` フラグ列（日本語＋条件コード）を付け、全除外フラグ `0` を `最終候補 = 1` とする。

### 2-4. 診療報酬・DPC・レセプト電算コード辞書

#### DPC・ICD-10コード
| 種別 | コード | 名称 | 用途・備考 |
| --- | --- | --- | --- |
| DPC | `010010` | 脳腫瘍 | 鳥取（脳外）旧NX3検討。MDC01。悪性・良性・性状不詳が混在するため不採用 |
| DPC | `010020` | くも膜下出血、破裂脳動脈瘤 | 鳥取（脳外）NX7 |
| DPC | `110080` | 前立腺の悪性腫瘍 | 鳥取（泌尿）UX4。手術区分 `01`（例：`110080xx01xxxx`）が前立腺悪性腫瘍手術等に対応 |
| ICD-10 | `O809` | 自然分娩 | 静岡済生会（産婦人科）固定傷病名 |
| ICD-10 | `C64` | 腎悪性新生物 | 鳥取（泌尿）腎腫瘍監視患者の検討メモ |

#### CT・MRI・PET系（初期混在一覧および本番区分）
| レセプト電算コード | 名称 | 区分・備考 |
| --- | --- | --- |
| `170011710` | CT撮影（イ、ロ又はハ以外） | CT（NX2本番採用） |
| `170011810` | CT撮影（16列以上64列未満マルチスライス型機器） | CT（NX2本番採用） |
| `170012070` | 造影剤使用加算（CT） | CT（NX2本番採用） |
| `170027310` | ポジトロン・CT複合撮影（18FDG使用） | PET/CT（NX2本番採用） |
| `170028610` | CT撮影（4列以上16列未満マルチスライス型機器） | CT（NX2本番採用） |
| `170033410` | CT撮影（64列以上マルチスライス型機器）（その他） | CT（NX2本番採用） |
| `170034910` | CT撮影（64列以上マルチスライス型機器）共同利用施設 | CT（NX2本番採用） |
| `170038810` | CT撮影（64列以上マルチスライス型機器）（その他）頭部外傷 | CT（NX2本番採用） |
| `170038910` | CT撮影（16列以上64列未満マルチスライス型機器）頭部外傷 | CT（NX2本番採用） |
| `170040210` | CT撮影（64列以上）共同利用施設（画診共同） | CT（NX2本番採用） |
| `170040250` | （名称未確定・マスタ要確認） | CT（NX2本番採用） |
| `170040410` | CT撮影（64列以上）（その他）（画診共同） | CT（NX2本番採用） |
| `170040610` | CT撮影（16列以上64列未満）（画診共同） | CT（NX2本番採用） |
| `170704810` | ポジトロン・CT複合撮影（アミロイドPET）（イ以外） | PET/CT（NX2本番採用） |
| `170704910` | ポジトロン・CT複合撮影（アミロイドPET）（イ以外・画診共同） | PET/CT（NX2本番採用） |
| `170901710` | CT撮影（128列以上マルチスライス型機器）（その他） | CT（NX2本番採用） |
| `170020110` | MRI撮影（1.5テスラ以上3テスラ未満の機器） | MRI（鳥取実データ出現分・N3/UX2採用） |
| `170033510` | MRI撮影（3テスラ以上の機器）（その他） | MRI（鳥取実データ出現分・N3/UX2採用） |
| `170020470` | 造影剤使用加算（MRI） | MRI加算（NX5採用。明細行 `action_detail_number > 0`） |
| `170015210` | MRI撮影（1又は2以外） | MRI（旧CT一覧混在分・NX2から除外） |
| `170035010` | MRI撮影（3テスラ以上の機器）共同利用施設 | MRI（旧CT一覧混在分・NX2から除外） |
| `170041410` | MRI撮影（3テスラ以上）共同利用施設（画診共同） | MRI（旧CT一覧混在分・NX2から除外） |
| `170041610` | MRI撮影（1.5テスラ以上3テスラ未満）（画診共同） | MRI（旧CT一覧混在分・NX2から除外） |

#### 整形外科：非観血的整復術（鳥取OX2）
- **骨折非観血的整復術（B / K044系等）**：`150016510`（肩甲骨）、`150016610`（上腕）、`150016710`（大腿）、`150016810`（前腕）、`150016910`（下腿）、`150017010`（鎖骨）、`150017110`（膝蓋骨）、`150017210`（手）、`150017310`（足その他）、`150060410`（骨盤骨折）、`150114610`（下顎骨折）、`150115010`（上顎骨折）
- **関節脱臼非観血的整復術（B / K061系等）**：`150033810`（肩）、`150033910`（股）、`150034010`（膝）、`150034110`（胸鎖）、`150034210`（肘）、`150034310`（手）、`150034410`（足）、`150034510`（肩鎖）、`150034610`（指）、`150034710`（小児肘内障）、`150035050`（先天性股関節脱臼・両側・リーメンビューゲル法）、`150035110`（先天性股関節脱臼・両側・その他）、`150114810`（顎関節脱臼）

#### 脳神経外科：血管内手術（鳥取NX4）
| K区分 | 名称 | レセプト電算コード | 採否 |
| --- | --- | --- | --- |
| K178 | 脳血管内手術（1箇所） | `150254910` | 採用 |
| K178 | 脳血管内手術（2箇所以上） | `150344410` | 採用 |
| K178 | 脳血管内手術（脳血管内ステント） | `150355410` | 採用 |
| K178-2 | 経皮的脳血管形成術 | `150273510` | 採用 |
| K178-4 | 経皮的脳血栓回収術 | `150372510` | 採用 |
| K178-5 | 経皮的脳血管ステント留置術 | `150380850` | 採用 |
| K609-2 | 経皮的頸動脈ステント留置術 | `150337110` | 採用 |
| K609-2 | 経皮的頸動脈ステント留置術（薬事承認又は認証医療機器） | `150453450` | 採用 |
| K178-3 | 経皮的選択的脳血栓・塞栓溶解術（頭蓋内脳血管） | `150301110` | 除外 |
| K178-3 | 経皮的選択的脳血栓・塞栓溶解術（頸部脳血管）（内頸、椎骨動脈） | `150301210` | 除外 |

#### 泌尿器科：前立腺悪性腫瘍手術（鳥取UX4）
- `K843`（前立腺悪性腫瘍手術）、`K843-2`（腹腔鏡下）、`K843-3`（腹腔鏡下小切開）、`K843-4`（腹腔鏡下・内視鏡手術用支援機器／ロボット）

#### その他検査・指導管理料
| カテゴリ | レセプト電算コード | 名称 | 用途 |
| --- | --- | --- | --- |
| 超音波検査 | `160165010` | 超音波検査（断層撮影法）（その他） | 鳥取（脳外N4_2：頸動脈狭窄エコー） |
| 血小板凝集能（D006） | `160238710` / `160238610` | 血小板凝集能（その他）／（鑑別診断の補助に用いるもの） | 鳥取（脳外NX4代理指標） |
| てんかん指導料（B001の6） | `113002850` / `113029610` | てんかん指導料／（情報通信機器） | 鳥取（脳外NX6） |
| 悪性腫瘍特異物質治療管理料 | `113001210` / `113001310` / `113002110` | （尿中BTA）／（その他・1項目※PSA主）／（その他・2項目以上） | 鳥取（泌尿UX5検討・不採用） |
| 前立腺針生検法（D413） | `160231810` / `160098110` | （MRI撮影・超音波検査融合画像）／（その他） | 鳥取（泌尿UX5検討・不採用） |
| 骨シンチグラフィ | `170024710` | シンチグラム（全身）（一連につき） | 鳥取（泌尿UX5検討・不採用） |
| 在宅自己注射指導管理料 | `114021010`, `114028410`, `114040610`, `114062910`, `114063010`, `114063110` | 在宅自己注射指導管理料（加算除く本体） | 新東京（糖尿病内科除外） |

---

## 3. 月次データ投入・初期設定・エラー対応

### 3-1. 入院データ投入の初期設定
- **概要**：新規顧客導入時や医療機関ID変更時に実施する、入院データバリデーション用レコードの登録作業。
- **対象テーブル**：`data_ops_prd.tenant_medical_institution_code_ids`
- **画面化対応**：**2026/10/08 にMIS管理画面へ入力機能を追加済み**（Notion：`https://app.notion.com/p/MIS-3971da5a7f3b80fbaac0f4e3dc149b9f?pvs=21`）。以前はDB直接INSERTが必須だった。
- **作業手順ドキュメント**：`https://docs.google.com/document/d/1yFoaNe805x8HGfMLY_xw08FFO52CYEi15dqKzbYxsPU/edit?tab=t.0#heading=h.s4vxgqxpgng`（添付PDF：`入院データ投入初期設定ガイド.pdf`）

#### DB直接投入実績（画面化以前）
1. **川崎市立多摩病院（`tenant_id = 517`）**
   - Asana：`https://app.asana.com/1/1199355644723937/project/1207948470706947/task/1217759063928968`
   - Slack：`https://medup.slack.com/archives/C06HQ2TLJKF/p1787099997328429`
   - 医療機関URL：`https://admin-web.medup-core-service.com/medical-institutions/40156`
   - パラメータ：`tenant_id = 517`, `medical_institution_id = 40156`, `dpc_medical_institution_code = '145402565'`
2. **広尾病院（`tenant_id = 516`、2026/08/26 追加依頼）**
   - Asana：`https://app.asana.com/1/1199355644723937/project/1207948470706947/task/1216481688446991`
   - Slack：`https://medup.slack.com/archives/C06HQ2TLJKF/p1787723562324839?thread_ts=1784006506.291029&cid=C06HQ2TLJKF`
   - パラメータ：`tenant_id = 516`, `medical_institution_id = 25786`, `dpc_medical_institution_code = '136070007'`
3. **群馬県立心臓血管センター（`tenant_id = 551`、2026/09/07 実績）**
   - Slack：`https://medup.slack.com/archives/C06HQ2TLJKF/p1788739204209739`
   - 医療機関URL：`https://admin-web.medup-core-service.com/medical-institutions/14133`
   - パラメータ：`tenant_id = 551`, `medical_institution_id = 14133`, `dpc_medical_institution_code = '105010018'`

```sql
-- 事前確認（該当テナントのデータが存在しないこと）
SELECT id, tenant_id, dpc_medical_institution_code, medical_institution_id, created_at, updated_at
FROM public.tenant_medical_institution_code_ids
WHERE tenant_id = 551;

-- 登録SQL（例：川崎市立多摩病院 / 広尾病院 / 群馬県立心臓血管センター）
INSERT INTO tenant_medical_institution_code_ids (id, tenant_id, dpc_medical_institution_code, medical_institution_id, created_at, updated_at)
VALUES (gen_random_uuid(), '517', '145402565', 40156, now(), now());

INSERT INTO tenant_medical_institution_code_ids (id, tenant_id, dpc_medical_institution_code, medical_institution_id, created_at, updated_at)
VALUES (gen_random_uuid(), '516', '136070007', 25786, now(), now());

INSERT INTO tenant_medical_institution_code_ids (id, tenant_id, dpc_medical_institution_code, medical_institution_id, created_at, updated_at)
VALUES (gen_random_uuid(), '551', '105010018', 14133, now(), now());
```

---

### 3-2. がん研有明病院（tenant_id: 204）：匿名化復号・洗替・月次投入
- **注意**：有明はDPCを厚生局提出用に匿名化したデータしか出せない。復号せずに管理画面へ上げると紹介データの診療系IDと紐づかない。必ず患者番号を復号してから様式1・Dファイルを月ごとにセットで投入する。
- **施設コード**：`130870923`（ファイル名例：`FF1_130870923_2608`、`Dn_130870923_2608`）
- **変換ルール**：
  - IT推進課の匿名化：診療系IDの先頭 `0` を削り、末尾に `0` を足す（`0007925233` → `0079252330`）。
  - 復号：末尾の `0` を除き、元の桁数になるよう先頭を0埋め（`0079252330` → `0007925233`）。様式1・Dファイル両方の2列目（施設コードの次列）に適用。出力ファイル名は `*_patient_number_fixed`。
- **共通リンク**：
  - 管理画面：[foro Core Admin 入院タブ（テナント204）](https://admin-web.medup-core-service.com/foro-core/tenants/204?tab=hospitalAdmission)
  - 受領一覧：[データ受領/投入状況一覧](https://admin-web.medup-core-service.com/foro-core/data-importing-logs)
  - 関連Notion：[テナント個別](https://app.notion.com/p/2c01da5a7f3b80e0bcbdd497bda02a96?pvs=21) / [通常投入マニュアル](https://app.notion.com/p/2651da5a7f3b80c99cf8e94756160a4a?pvs=21) / [前回洗替記録（6月・7月）](https://app.notion.com/p/3d61da5a7f3b803c9dffcfb31f355ed6?pvs=21)

#### 作業手順
1. Driveから様式1とDファイルを月ごとに取得し `input/` に配置。片方しかない月は投入せずCSへ連絡。
2. `convert_patient_number.py --dry-run` で件数・サンプルを確認。
3. 問題なければ本実行し `output/*_patient_number_fixed*` を作成。
4. 変換結果確認（`0079252330` → `0007925233`、**変換0件は停止**。ヘッダ行の2列目が識別番号でなければヘッダはそのまま残る）。
5. 入院タブで対象月が未登録であることを確認（洗替時・登録済みの場合は該当月を削除してから入れ直し）。
6. 変換後の様式1とDファイルを月ごとにインポート。
7. 投入後チェック：レコード数・入院紹介数・DPCコード数が直近月と比べ**差分20%以内**か確認。エラー時は黒川氏へ連絡。
8. 完了処理：データ受領/投入状況一覧に様式1・Dファイルの該当月を作成し受領済み化 → Asana `04_データ投入ops` を `Imported` にして完了 → Slack依頼スレッドへ完了報告。

#### 実施実績
- **2026/09 洗替（2606・2607）**：6月の診療報酬改定の影響で様式1に重複があったため再提出データで洗替。
  - Slack：`https://medup.slack.com/archives/C036KMJT06L/p1788918416671009?thread_ts=1788860270.129589&cid=C036KMJT06L`
  - 元ファイルDrive：`https://drive.google.com/drive/folders/1cw2yBCxrEygfIPpC6l5f5KGcoVCG4Pc-?usp=drive_link`
  - 変換後データDrive：`https://drive.google.com/drive/folders/1lHZfzN5o_155WObgBum9NvMW_cfnJtam`
- **2026/10 月次投入（2608・2609）**：洗替ではなく月次投入（期限：2026年10月25日 14:17）。
  - Slack：`https://medup.slack.com/archives/C036KMJT06L/p1792734843101159?cid=C036KMJT06L&thread_ts=1792473430.862739`
  - 2026年8月分Drive：`https://drive.google.com/drive/folders/1vaw_kEU0pvxmjm3eQzvEQtY4Ij8zY2Zs`
  - 2026年9月分Drive：`https://drive.google.com/drive/folders/1LQInrsKXrvkffVVLA6sJnvSCy3zYL-Mi`

| 月 | 洗替前レコード | 洗替前入院紹介 | 洗替前DPCコード | 投入日 |
| --- | --- | --- | --- | --- |
| `2606` | `1530` | `1119` | `1383` | `2026/07/29` |
| `2607` | `1577` | `1140` | `1396` | `2026/08/28` |

#### 復号スクリプト（`convert_patient_number.py`）
```bash
python3 convert_patient_number.py --dry-run
python3 convert_patient_number.py
```
```python
#!/usr/bin/env python3
"""がん研有明 DPC データ識別番号の復号（末尾0除去 → 先頭0埋め）。

匿名化: 診療系ID 0007925233 → DPC 0079252330（先頭0削除 + 末尾0追加）
復号:   DPC 0079252330 → 診療系ID 0007925233（末尾0除去 + 元桁数まで0埋め）

様式1 (FF1) / Dファイルとも、施設コードの次列（2列目）がデータ識別番号。
"""

from __future__ import annotations

import argparse
import csv
import sys
from pathlib import Path

ENCODINGS = ("cp932", "utf-8-sig", "utf-8")

def decode_dpc_patient_id(value: str) -> tuple[str, bool]:
    s = value.strip().strip('"')
    if not s:
        return s, False
    if s.endswith("0") and len(s) >= 2:
        return s[:-1].zfill(len(s)), True
    return s, False

def detect_encoding(path: Path) -> str:
    raw = path.read_bytes()
    for enc in ENCODINGS:
        try:
            raw.decode(enc)
            return enc
        except UnicodeDecodeError:
            continue
    return "cp932"

def detect_dialect(sample: str) -> csv.Dialect:
    try:
        return csv.Sniffer().sniff(sample, delimiters=",\t|")
    except csv.Error:
        dialect = csv.excel
        dialect.delimiter = ","
        return dialect

def convert_file(src: Path, dest: Path, id_col: int, dry_run: bool) -> dict:
    encoding = detect_encoding(src)
    text = src.read_text(encoding=encoding, errors="replace")
    lines = text.splitlines(keepends=True)
    if not lines:
        raise ValueError(f"empty file: {src}")

    sample = "".join(lines[:20])
    dialect = detect_dialect(sample)
    reader = csv.reader(lines, dialect)
    rows = list(reader)
    col = id_col - 1

    converted = 0
    skipped = 0
    samples: list[tuple[str, str]] = []
    out_rows: list[list[str]] = []

    for row in rows:
        if len(row) <= col:
            out_rows.append(row)
            skipped += 1
            continue
        new_id, changed = decode_dpc_patient_id(row[col])
        if changed:
            converted += 1
            if len(samples) < 5:
                samples.append((row[col], new_id))
            row = list(row)
            row[col] = new_id
        else:
            skipped += 1
        out_rows.append(row)

    if not dry_run:
        dest.parent.mkdir(parents=True, exist_ok=True)
        with dest.open("w", encoding=encoding, newline="") as f:
            writer = csv.writer(f, dialect)
            writer.writerows(out_rows)

    return {
        "src": str(src),
        "dest": str(dest),
        "encoding": encoding,
        "delimiter": dialect.delimiter,
        "rows": len(rows),
        "converted": converted,
        "skipped": skipped,
        "samples": samples,
    }

def default_dest_name(src: Path) -> str:
    stem = src.stem
    if "patient_number_fixed" in stem:
        return src.name
    return f"{stem}_patient_number_fixed{src.suffix}"

def main() -> int:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--input-dir", type=Path, default=Path(__file__).resolve().parent / "input")
    parser.add_argument("--output-dir", type=Path, default=Path(__file__).resolve().parent / "output")
    parser.add_argument("--id-col", type=int, default=2, help="1-based column of データ識別番号")
    parser.add_argument("--dry-run", action="store_true")
    args = parser.parse_args()

    if not args.input_dir.exists():
        args.input_dir.mkdir(parents=True, exist_ok=True)
        print(f"created empty input dir: {args.input_dir}", file=sys.stderr)

    files = sorted(
        p
        for p in args.input_dir.iterdir()
        if p.is_file() and p.suffix.lower() in {".txt", ".csv"} and not p.name.startswith(".")
    )
    if not files:
        print(f"no .txt/.csv in {args.input_dir}", file=sys.stderr)
        return 1

    rc = 0
    for src in files:
        dest = args.output_dir / default_dest_name(src)
        try:
            stats = convert_file(src, dest, args.id_col, args.dry_run)
        except Exception as exc:
            print(f"ERROR {src.name}: {exc}", file=sys.stderr)
            rc = 1
            continue
        mode = "DRY-RUN" if args.dry_run else "WROTE"
        print(f"[{mode}] {src.name}")
        print(f"  encoding={stats['encoding']} delimiter={repr(stats['delimiter'])} rows={stats['rows']}")
        print(f"  converted={stats['converted']} skipped/unchanged={stats['skipped']}")
        if not args.dry_run:
            print(f"  -> {dest.name}")
        for before, after in stats["samples"]:
            print(f"  sample {before} -> {after}")
        if stats["converted"] == 0:
            print("  WARN: 変換0件。列位置や既変換ファイルの可能性あり")
            rc = 1
    return rc

if __name__ == "__main__":
    raise SystemExit(main())
```

---

### 3-3. 静岡済生会総合病院（tenant_id: 136）：産婦人科（様式1なし）DB直接投入
- **注意**：出産は保険・DPC対象外のため、産婦人科は様式1・Dファイルの形式で受領されない。管理画面インポートは使わず、病院本体のFF1/Dnとは別ファイルとして `hospital_admissions` 登録用CSVへ直接変換しDBへ入れる。患者ファイル・生成CSV・生成SQLはGitに入れない。
- **参照リンク**：
  - 依頼Slack（10月）：`https://medup.slack.com/archives/C06HQ2TLJKF/p1793067454459149` / 経緯Slack（9月・5〜7月分）：`https://medup.slack.com/archives/C06HQ2TLJKF/p1787903708542919`
  - 変換スクリプト：`https://github.com/MedUpInc/data-customer-ops/blob/main/preprocess_user_data/docs/tenant_136_not_ff1_admissions.md`
  - 管理画面：[136 入院タブ](https://admin-web.medup-core-service.com/foro-core/tenants/136?tab=hospitalAdmission)
  - 関連Notion：[施設ページ](https://app.notion.com/p/25b1da5a7f3b800cb265d68487dee24e?pvs=21) / [旧手順（パス古）](https://app.notion.com/p/1c31da5a7f3b801aa186e2a3005a0679?pvs=21) / [経緯](https://app.notion.com/p/3d41da5a7f3b80dea049e764269fd0a7?pvs=21)
- **変換ルール・出力**：
  - 入力：1行目ヘッダー、以降4列（入院日、退院日、診療科コード、データ識別番号。日付は `YYYYMMDD`）。1回につき1ファイル処理。
  - 固定値：`tenant_id = 136`、施設ID = `48518`、Dファイル名 = 空、傷病名 = `O809`（自然分娩）、`dpc_code` = 全件 `NULL`。
  - 出力（上書きなし）：`tenant_136_hospital_admissions_日時.csv`、`tenant_136_hospital_admission_importing_log_日時.csv`

#### 作業手順
1. 入院タブで対象ファイル名が未登録であることを確認。
2. `--output-dir` を事前作成し、変換コマンドで1ファイルずつCSV化。
3. 入院日がファイル名の月（例：2608なら `2026-07-27`〜`2026-08-25` の前月末〜当月25日）に収まっているか確認。外れていたら投入しない。
4. **CSVヘッダー誤字修正**：スクリプト出力の `icd10_most_resourse_used_disease` をDB列名の **`icd10_most_resource_used_disease`**（`name` 列も同じ）へ修正。値は `O809` / `自然分娩` のまま。
5. `foro_prd` へ接続し `SET session.current_tenant_id = '136';` を実行（DBeaver閲覧時も同様）。
6. 同一トランザクションで **`hospital_admission_importing_logs` を先、`hospital_admissions` を後**に投入（空文字は `NULL`）。同一 `ff1_file_name` がテナント136にあれば中止。
7. 紐付けAPI前に `hospital_admission_referrals`、`referral_hospital_admissions` の件数と対象ファイルの入院紹介数（投入直後は `0`）を控える。
8. 紐付けAPIを1回呼ぶ（テナント136の紐付けを全削除し再作成。成功は HTTP `204`）。
9. 対象ファイルの入院紹介数が付いたこと、既存FF1の入院紹介数が変わっていないことを確認し、過去実績と比較。

#### コマンド集
```bash
# 1. 変換（data-customer-ops/preprocess_user_data で実行）
uv run --python 3.12 --with pydantic --with pandas python src/tenant_136_not_ff1_admissions.py /path/to/産婦人科様式1なし_YYMM.txt --output-dir /path/to/output

# 2. DB接続（SSHトンネル 127.0.0.1:5434 開通前提）
export PGPASSWORD="$(gcloud auth print-access-token)"
psql "host=127.0.0.1 port=5434 dbname=foro_prd user=jun.yamada@medup.jp sslmode=disable"
```
```sql
SET session.current_tenant_id = '136';
```
```bash
# 3. 紐付けAPI（Basic認証はcore serviceクライアントと同じ。成功は204）
curl -sS -w "http_status=%{http_code}\n" -X PUT \
  -u 'data-management:data-management' \
  -H 'Content-Type: application/json' \
  -d '{"tenant_id":"136"}' \
  --max-time 3500 \
  'https://data-management-api-production-ozkea72soq-an.a.run.app/referral_hospital_admissions/refresh'
```

#### 実績・検証ログ
- 過去実績：2605（30件中25）、2606（25件中22）、2607（20件中17）、2608（38件中34）
- 2026/10/27 作業結果（受領日：2026/10/21。受領済み記録はデータ受領一覧に既存）：
  - `産婦人科様式1なし_2608.txt`：38件（入院日 `2026-07-27`〜`2026-08-25`）。1件+38件を投入し紐付けAPI実行（204）。実行前：`hospital_admission_referrals` 49,032件 / `referral_hospital_admissions` 38,934件 / 対象ファイル 0件 → 実行後：49,066件(+34) / 38,964件(+30) / 対象ファイル 34件（38件中）。既存FF1変化なし。
  - `産婦人科様式1なし_2609.txt`：16件変換したが、入院日が `2025-12-27`〜`2026-01-24` のため先方ミスとして投入中止。

---

### 3-4. 入院投入データエラー対応（2026/09/09）
- **対象タスク**：`04_データ投入ops`（様式1 + D）／Slack：`https://medup.slack.com/archives/C06HQ2TLJKF/p1787903708542919`
- **対象エラー案件**：
  - テナント `61`：済生会今治 7月分（8/19エラー）：`https://admin-web.medup-core-service.com/foro-core/tenants/61?tab=hospitalAdmission`
  - テナント `21`：JCHO大阪 7月分（8/28エラー）：`https://admin-web.medup-core-service.com/foro-core/tenants/21?tab=hospitalAdmission`
- **一般エラー対応手順**：
  1. ログからエラー内容を確認。
  2. 受領ファイルを直接確認してエラー詳細を特定。
  3. エラー内容に応じて2パターンで対応：
     - **ユーザーへ再提出依頼**：ファイルの中身が全く別物である場合など。
     - **自社側でデータ加工して取り込み**：項目の一部破損、列が1列多いなどの軽微なレイアウト不備。

---

### 3-5. 紹介データ重複削除（2026/09/09 座間）
- **依頼Slack**：`https://medup.slack.com/archives/C036KMJT06L/p1788752098064459?thread_ts=1788499315.547009&cid=C036KMJT06L`
- **事象**：座間にてファイル単位での紹介データ重複が発生。2024年秋以前のデータのため、管理ツール上の削除機能が非対応であり手動対応が必要。
- **対象テーブルと削除仕様**：
  - `referrals`：MIS管理画面からの削除で消える。
  - `referral_masters`：2024年秋以前（削除機能搭載前）の古いデータは管理画面で消しても消えないため手動削除が必要（※2024/10以降分は Data Mgmt Admin で削除可能）。

---

## 4. 外来適正化・逆紹介候補抽出

### 4-1. 月次抽出の全体フロー・事前確認・済生会熊本（tenant 11）実行手順
- **目的**：翌月予約患者の逆紹介候補リストを抽出する月次作業（概ね月末に実施）。抽出した臨床データと予約データを突合し、逆紹介先医療機関とのマッチングまで行う。
- **異常検知**：基本は既存スクリプト／クエリ実行だが、診療報酬改定やシステム変更によりコード変更が発生する場合がある。前月との大きな乖離があれば調査に入る。
- **リポジトリ・フォルダ・手順リンク**：
  - スクリプト置き場：`https://github.com/MedUpInc/outpatient-ops`（クローン・開発環境構築して利用）
  - 結果ファイル格納先：`https://drive.google.com/drive/u/0/folders/19ppu985HAQ0-nUPOewUsq_9penq0DfeI`
  - 受領データ確認フォルダ：`https://drive.google.com/drive/u/0/folders/1pyo8eIEPpCZl3bQIntUx7PmWeAM5-g-B`
  - 手順ドキュメント：`https://docs.google.com/document/d/1ej83SY35NtvjxKhsU4Lysb97reSYcbRl6yo8f0rYnHs/edit?tab=t.0#heading=h.m7jx1gyrn55e`（関連PDF：`Mission_Control_Blueprint.pdf`）
  - 議事録：`https://circleback.ai/meetings/umMgzIz4c8o5YkUrDiKOy` / `https://circleback.ai/meetings/55uKmpx98HYr9xcDCJTxk` / `https://circleback.ai/ja/meetings/BzEYf89SvOMd8HDoZrnhs?tab=notes`

| テナントID | 施設名 | 抽出方式 | 備考 |
| --- | --- | --- | --- |
| `11` | 済生会熊本病院 | `outpatient-ops` スクリプト | 泌尿器科・呼吸器外科 |
| `135` | 新東京病院 | `outpatient-ops` スクリプト | 心臓内科・糖尿病内科 |
| `274` | 鳥取県立中央病院 | 個別SQL（4-2参照） | 整形外科・脳神経外科・泌尿器科 |
| ~~`204`~~ | ~~がん研有明病院~~ | ~~`outpatient-ops` スクリプト~~ | **2026/08で終了** |

#### 病院別必須データ確認（前月末までの臨床データ＋翌月予約データ。欠けていれば抽出しない）
- **済生会熊本（`tenant 11`）**
  - Drive：`https://drive.google.com/drive/u/0/folders/1srhiPrDiCBbXWVYa4YtuBKq5Bw8hdiRW`
  - 前月までの外来EF：`https://admin-web.medup-core-service.com/foro-core/tenants/11?tab=outpatientEF`
  - 来月の外来予約CSV（泌尿器科・呼吸器外科）、来月の手術予約CSV（泌尿器科のみ）
- **新東京（`tenant 135`）**
  - Drive：`https://drive.google.com/drive/u/0/folders/1PuAtQQQqT1ivaPKaqvAD4Dg9EnBB1ZiP`
  - 前月までの外来EF：`https://admin-web.medup-core-service.com/foro-core/tenants/135?tab=outpatientEF`
  - 来月の心臓内科予約xlsx（外来予約・手術・内視鏡シート）、来月の糖尿病内科予約xlsx（同）
- **鳥取県立中央（`tenant 274`・脳神経外科）**
  - Drive：`https://drive.google.com/drive/u/0/folders/1uO2QVQyXjUbjWNS_q4vhXzzE7bypgI6o`
  - 前月までの外来EF（E行、明細行、造影加算）：`https://admin-web.medup-core-service.com/foro-core/tenants/274?tab=outpatientEF`
  - 前月までの入院データ、前月までの逆紹介データ（※逆紹介だけ遅れている場合は欠ける月をメモして候補抽出してよい）
  - 来月の通院予約は候補抽出後に患者IDと突合する（候補抽出の入力ではない）

#### 外来EF 3病院一括確認SQL（BigQuery）
※日付は対象月に読み替える。最大受診日が対象月末より前、または対象月のE行が0ならその病院は抽出しない。管理画面未登録の場合は受領データを確認し、なければCS経由で要求する。
```sql
SELECT
  tenant_id,
  MAX(IF(action_detail_number = 0, outpatient_date.value, NULL)) AS max_e_date,
  COUNTIF(action_detail_number = 0 AND outpatient_date.value BETWEEN DATE '2026-09-01' AND DATE '2026-09-30') AS month_e_rows
FROM `medup-foro.dpc_patient_records_prd.outpatient_ef`
WHERE tenant_id IN (11, 135, 274)
  AND data_type != 'SY'
GROUP BY tenant_id
```

#### 済生会熊本（`tenant 11`）実行手順・ログ例
1. Driveから外来予約データ・手術予約データをダウンロード。
2. 対象診療科のYAML（例：`src/pipeline/tenants/t011/respiratory_surgery.yaml`）に実行ファイル名と開始・終了日付を記載。
3. ドクターマスターの参照権限がない場合は、`src/bigquery/models/dpc.py` 143行目をスナップショット参照へ変更：
   `__tablename__ = "sk-kumamoto_doctor_master"` → `"sk-kumamoto_doctor_master_snapshot"`
4. 対象診療科のパイプラインを実行。

```text
$ uv run python -m src.pipeline.tenants.t011.urology
=== パイプライン開始: 11 泌尿器科 ===
  対象医師コード: 10件 / 15名
[1/5] 患者サマリー取得 + フィルタ -> 患者サマリー取得: 46706件 / フィルタ後候補: 111件
[2/5] 抗がん剤除外 -> 0件除外 → 残り 111件
[3/5] 直近紹介データ付与 -> 紹介データ: 109件
[4/5] CSV マージ + 手術除外 -> 外来予約: 33476件 / 手術予定除外: 0件 / 予約×候補マージ: 81件
[5/5] Excel出力（医師名ごと）
  三上　洋: 14件 / 右田　敏起: 1件 / 岡　保伸: 21件 / 濵﨑　和代: 11件 / 牧野　耕: 2件 / ﾛﾎﾞｯﾄ専門外来(泌): 17件
  出力先: outputs/11/2026-09-25/<医師名>Dr_泌尿器科.xlsx

$ uv run python -m src.pipeline.tenants.t011.respiratory_surgery
=== パイプライン開始: 11 呼吸器科 ===
  対象医師コード: 2件 / 3名
[1/4] 患者サマリー取得 + フィルタ -> 患者サマリー取得: 46706件 / 基本フィルタ後候補: 171件
[2/4] 退院 2 年以上経過フィルタ -> 171件 → 112件
[3/4] 直近紹介データ付与 -> 紹介データ: 104件
[4/4] CSV マージ + 出力 -> 外来予約: 33476件 / 予約×候補マージ: 67件
  出力: outputs/11/済生会熊本_呼吸器外科_逆紹介対象患者リスト_10_2026-09-25.xlsx (67件)
```

---

### 4-2. 鳥取県立中央病院（tenant_id: 274）：3診療科抽出条件・本番SQL
- **実装前提（2026-08-26更新）**：
  - 対象期間：直近1年（今回 `2025-08-01` 〜 `2026-07-31`、年度区切りではない・可変）。
  - 親集合：`patient_summary_by_department` は使わない。`outpatient_ef` を受診日で絞り、`dbt_int_doctor_department` の担当診療科で看る。最大行為・受診回数・併科は同期間のEFから再集計。
- **社内参照Notion**：
  - [20260805_逆紹介データ抽出クエリ作成](https://app.notion.com/p/20260805_-3b11da5a7f3b80fe9444f13313f96adf?pvs=21) / [20260807_抽出条件確認用_3診療科](https://app.notion.com/p/20260807_-_3-3b51da5a7f3b8191b014e46afe9ba2a4?pvs=21) / [20260807_外来適正化Weekly](https://app.notion.com/p/3b41da5a7f3b8026952dd3df4b233000?pvs=21) / [20260803_外来社内Mtg](https://app.notion.com/p/20260803_-_-Mtg-3b01da5a7f3b8035b72ddc9572603845?pvs=21) / [患者サマリー データ定義書](https://app.notion.com/p/3a61da5a7f3b80a5a56cceff49dd17ac?pvs=21)
  - 7/30 診療科面談：[整形](https://app.notion.com/p/3ad1da5a7f3b80bba612d1ebade32dee?pvs=21) / [脳外](https://app.notion.com/p/3ad1da5a7f3b809c951fc36b43a43277?pvs=21) / [泌尿](https://app.notion.com/p/3ad1da5a7f3b80adb16cdb5cd2530f69?pvs=21)

#### 病院確認結果まとめ
| 診療科 | 確認事項 | 確定結果 |
| --- | --- | --- |
| 整形外科 | 対象外OX2：「ギプス」「非観血的整復（骨折／脱臼）」のどちらか一方でも除外してよいか（参考：2025年度ギプス310件、非観血140件） | 「非観血的整復（骨折／脱臼）」のみを対象とする（OX1 CT/MRI・ギプスはv20で不採用）。 |
| 脳神経外科 | 悪性腫瘍除外に脳腫瘍DPC（`010010`）だけでは良悪判別不可。絞る条件を知りたい | 病院確認時点は神部医師の外来予約のみで除外（算定・病名判定なし）。**2026-09-30 CS合意によりクエリではNX3を適用しないのが正**（一覧の担当医フィルタで除外。予約JOINクエリはコメント残置）。 |
| 泌尿器科 | 積極的監視の除外は「直近3年以内の前立腺生検またはMRI」でよいか | 仮回答「3年確定。ただし除外ではなくフラグ設定し個別確認」→ 最終合意（8/25）：個別判断しかないため**除外条件・フラグともに削除**。 |

#### 4-2-1. 整形外科（面談：村田先生・山下先生・皆川様）
- **対象（`O1 ∧ O2 ∧ O3`、約2,187人）**：直近1年外来（O1）∧ 整形外科受診・**併科含む**（O2）∧ 最大行為が「処置／手術／注射」以下＝`1.入院`・`2.初診` 以外（O3）。点数・受診回数で除外しない（骨粗鬆症注射：プラリア年2回＋内服＋骨密度年1回も対象）。
- **除外フラグ**：`OX2_非観血`（期間内非観血的整復術）、`OX3_逆紹介済`（期間内整形外科から逆紹介済）。※`OX1`（CT/MRI）・ギプスは不採用。
- **個別論点メモ**：難病・特定疾患の書類対応患者（6月集中）は受け皿の書類作成可否をアンケート確認。抗生剤長期投薬はほぼいないため考慮せず。人工関節フォローは施設差があり一律ルールなし。ゾーン別の逆紹介実績を確認したい（山下先生）。

```sql
-- A. 候補リスト本体（整形外科 / tenant 274）
WITH
params AS (
  SELECT 274 AS tenant_id, DATE '2025-08-01' AS period_start, DATE '2026-07-31' AS period_end, '整形外科' AS target_department_name
),
ortho_doctors AS (
  SELECT SAFE_CAST(d.tenant_id AS INT64) AS tenant_id, d.doctor_code, d.department_name
  FROM `medup-foro.dpc_patient_records_prd.dbt_int_doctor_department` AS d
  CROSS JOIN params AS p
  WHERE SAFE_CAST(d.tenant_id AS INT64) = p.tenant_id AND d.department_name = p.target_department_name
),
ortho_ef AS (
  SELECT
    ef.tenant_id, CAST(SAFE_CAST(ef.patient_id AS INT64) AS STRING) AS patient_id,
    ef.doctor_code, od.department_name, ef.outpatient_date.value AS outpatient_date, ef.receipt_code, ef.action_name
  FROM `medup-foro.dpc_patient_records_prd.outpatient_ef` AS ef
  INNER JOIN ortho_doctors AS od ON ef.doctor_code = od.doctor_code AND SAFE_CAST(ef.tenant_id AS INT64) = od.tenant_id
  CROSS JOIN params AS p
  WHERE ef.tenant_id = p.tenant_id AND ef.data_type != 'SY' AND ef.action_detail_number = 0
    AND ef.outpatient_date.value BETWEEN p.period_start AND p.period_end
),
patient_flags AS (
  SELECT
    patient_id,
    COUNT(DISTINCT outpatient_date) AS visit_count_in_specialty,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '111') THEN 1 ELSE 0 END) AS first_visit_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '112') THEN 1 ELSE 0 END) AS follow_up_visit_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '140') THEN 1 ELSE 0 END) AS procedure_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '150') THEN 1 ELSE 0 END) AS surgery_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '13') THEN 1 ELSE 0 END) AS injection_flag,
    MAX(CASE WHEN receipt_code IN ('170033410', '170012070', '170011810', '170035010', '170020110', '170020470') THEN 1 ELSE 0 END) AS ct_imaging_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '170') THEN 1 ELSE 0 END) AS imaging_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '160') THEN 1 ELSE 0 END) AS exam_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '12') THEN 1 ELSE 0 END) AS medication_flag,
    MAX(CASE WHEN receipt_code IN ('180756010', '180024710') THEN 1 ELSE 0 END) AS rehabilitation_flag
  FROM ortho_ef
  GROUP BY patient_id
),
admission_ortho AS (
  SELECT DISTINCT CAST(SAFE_CAST(h.patient_number AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.foro_prd.public_hospital_admissions` AS h
  LEFT JOIN `medup-foro.foro_prd.public_dpc_specialty_codes` AS c ON c.code = h.dpc_specialty_code
  CROSS JOIN params AS p
  WHERE h.tenant_id = p.tenant_id AND (c.name = p.target_department_name OR c.name LIKE '%整形%')
    AND (h.admission_started_on BETWEEN p.period_start AND p.period_end OR h.admission_ended_on BETWEEN p.period_start AND p.period_end)
),
categorized AS (
  SELECT
    f.patient_id, f.visit_count_in_specialty,
    CASE
      WHEN a.patient_id IS NOT NULL THEN '1.入院'
      WHEN f.first_visit_flag = 1 THEN '2.初診'
      WHEN f.follow_up_visit_flag = 1 AND (f.procedure_flag = 1 OR f.surgery_flag = 1 OR f.injection_flag = 1) THEN '3.処置/手術/注射'
      WHEN f.follow_up_visit_flag = 1 AND f.ct_imaging_flag = 1 THEN '4.CT/MRI'
      WHEN f.follow_up_visit_flag = 1 AND (f.imaging_flag = 1 OR f.exam_flag = 1) THEN '5.検査（US・XP含む）'
      WHEN f.follow_up_visit_flag = 1 AND f.medication_flag = 1 THEN '6.投薬'
      WHEN f.follow_up_visit_flag = 1 AND f.rehabilitation_flag = 1 THEN '7.リハ'
      WHEN f.follow_up_visit_flag = 1 THEN '8.再診'
      ELSE '9.その他'
    END AS medical_service_category
  FROM patient_flags AS f
  LEFT JOIN admission_ortho AS a USING (patient_id)
),
target_patients AS (
  SELECT * FROM categorized WHERE medical_service_category NOT IN ('1.入院', '2.初診')
),
ox2_noninvasive_reduction AS (
  SELECT DISTINCT patient_id FROM ortho_ef
  WHERE receipt_code IN (
    '150016510', '150016610', '150016710', '150016810', '150016910', '150017010', '150017110', '150017210', '150017310', '150060410', '150114610', '150115010',
    '150033810', '150033910', '150034010', '150034110', '150034210', '150034310', '150034410', '150034510', '150034610', '150034710', '150035050', '150035110', '150114810'
  )
),
ox3_reverse_referred AS (
  SELECT DISTINCT CAST(SAFE_CAST(r.patient_id AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.foro_prd.public_reverse_referrals` AS r
  INNER JOIN `medup-foro.foro_prd.public_own_departments` AS dept ON dept.id = r.specialty_id
  INNER JOIN `medup-foro.foro_prd.public_own_department_tenants` AS odt ON odt.own_department_id = dept.id AND odt.tenant_id = r.tenant_id
  CROSS JOIN params AS p
  WHERE r.tenant_id = p.tenant_id AND dept.name = p.target_department_name
    AND r.reverse_referral_date BETWEEN p.period_start AND p.period_end
)
SELECT
  t.patient_id AS `患者ID`, t.medical_service_category AS `最大行為`, t.visit_count_in_specialty AS `受診回数`,
  IF(ox2.patient_id IS NOT NULL, 1, 0) AS `OX2_非観血`,
  IF(ox3.patient_id IS NOT NULL, 1, 0) AS `OX3_逆紹介済`,
  IF(ox2.patient_id IS NULL AND ox3.patient_id IS NULL, 1, 0) AS `最終候補`
FROM target_patients AS t
LEFT JOIN ox2_noninvasive_reduction AS ox2 USING (patient_id)
LEFT JOIN ox3_reverse_referred AS ox3 USING (patient_id)
ORDER BY t.patient_id;

-- B. ファネル件数（上記CTEの最終SELECTを以下に差し替えて実行）
/*
SELECT
  (SELECT COUNT(DISTINCT patient_id) FROM ortho_ef) AS n_o1_o2_ortho_visits,
  (SELECT COUNT(*) FROM target_patients) AS n_o3_target,
  COUNTIF(ox2.patient_id IS NOT NULL) AS n_ox2,
  COUNTIF(ox3.patient_id IS NOT NULL) AS n_ox3,
  COUNTIF(ox2.patient_id IS NULL AND ox3.patient_id IS NULL) AS n_final_candidates
FROM target_patients AS t
LEFT JOIN ox2_noninvasive_reduction AS ox2 USING (patient_id)
LEFT JOIN ox3_reverse_referred AS ox3 USING (patient_id);
*/
```

#### 4-2-2. 脳神経外科
- **対象**：本番SQL実装は `N1 ∧ N2 ∧ N3`（約660人：直近1年外来 ∧ 脳神経外科 ∧ 単純MRIフォロー・1.5T/3.0Tとも対象）。※確認用資料上の追加案：`OR (N4_1 ∧ N4_2)`（最大行為 `5.検査（US・XP含む）` ∧ 頸動脈狭窄エコー `160165010`、[参照](https://app.notion.com/p/3981da5a7f3b827182ef011707157ed1?pvs=21)）。
- **除外フラグ**：
  - `NX1_併科`：期間内に脳神経外科以外の受診あり
  - `NX2_CT`：期間内にCT算定あり（MRI混在コード除外済み16コード）
  - `NX3_神部`：2026-09-30合意によりクエリ上は空集合（一覧の担当医フィルタで除外）。※旧案の脳腫瘍DPC（`010010`）は良悪混在、かつ悪性脳腫瘍は血清腫瘍マーカーがなく管理料が算定されにくいため不採用。
  - `NX4_血管内`：血管内手術後（ステント・コイル）。MRI枠逼迫解消が主目的のため安全側除外。入院データ遡及限界（2019年以前）を補完するため、**(a) 外来EF血管内手術レセプト OR (b) 入院手術Kコード（`K178`系・`K609-2`、`K178-3`除外） OR (c) 血小板凝集能（代理指標）** の全期間OR条件で除外。
  - `NX5_造影MRI`：期間内の最新MRI算定日と同日に造影剤使用加算（`170020470`）あり（明細行 `action_detail_number > 0` を含む）。※当初検討の生化学的検査・BUN併算定は不採用。
  - `NX6_てんかん`：期間内にてんかん指導料あり
  - `NX7_SAH`：くも膜下出血DPC（`010020`）で退院日が直近5年以内（術後6年未満の簡便判定）
  - `NX8_逆紹介済`：期間内に脳神経外科から逆紹介済み

```sql
-- プローブ（任意）: 神部の表記確認
/*
SELECT d.name AS doctor_name, COUNT(*) AS n
FROM `medup-foro.foro_prd.public_op_optimizer_outpatient_reservations` AS r
INNER JOIN `medup-foro.foro_prd.public_op_optimizer_doctors` AS d ON d.id = r.doctor_id AND d.tenant_id = r.tenant_id
WHERE r.tenant_id = 274 AND d.name LIKE '%神部%'
GROUP BY d.name ORDER BY n DESC;
*/

-- 参考（旧NX3・不採用）: 脳腫瘍DPC調査クエリ
/*
SELECT patient_number, admission_ended_on
FROM `medup-foro.foro_prd.public_hospital_admissions`
WHERE tenant_id = 274 AND STARTS_WITH(LOWER(dpc_code), '010010')
  AND admission_ended_on BETWEEN DATE '2019-04-01' AND DATE '2026-03-31'
ORDER BY patient_number, admission_ended_on;
*/

-- A. 候補リスト本体（脳神経外科 / tenant 274）
WITH
params AS (
  SELECT 274 AS tenant_id, DATE '2025-08-01' AS period_start, DATE '2026-07-31' AS period_end,
         '脳神経外科' AS target_department_name, DATE_SUB(DATE '2026-07-31', INTERVAL 5 YEAR) AS sah_lookback_start
),
mri_codes AS (SELECT code FROM UNNEST(['170020110', '170033510']) AS code),
ct_codes AS (
  SELECT code FROM UNNEST([
    '170011710', '170011810', '170012070', '170027310', '170028610', '170033410', '170034910', '170038810',
    '170038910', '170040210', '170040250', '170040410', '170040610', '170704810', '170704910', '170901710'
  ]) AS code
),
vascular_receipt_codes AS (
  SELECT code FROM UNNEST(['150254910', '150344410', '150355410', '150273510', '150380850', '150337110', '150453450', '150372510']) AS code
),
platelet_codes AS (SELECT code FROM UNNEST(['160238710', '160238610']) AS code),
epilepsy_codes AS (SELECT code FROM UNNEST(['113002850', '113029610']) AS code),
neuro_doctors AS (
  SELECT SAFE_CAST(d.tenant_id AS INT64) AS tenant_id, d.doctor_code, d.department_name
  FROM `medup-foro.dpc_patient_records_prd.dbt_int_doctor_department` AS d
  CROSS JOIN params AS p
  WHERE SAFE_CAST(d.tenant_id AS INT64) = p.tenant_id AND d.department_name = p.target_department_name
),
ef_period AS (
  SELECT
    CAST(SAFE_CAST(ef.patient_id AS INT64) AS STRING) AS patient_id, ef.doctor_code,
    IFNULL(d.department_name, '未割当') AS department_name, ef.outpatient_date.value AS outpatient_date, ef.receipt_code
  FROM `medup-foro.dpc_patient_records_prd.outpatient_ef` AS ef
  LEFT JOIN `medup-foro.dpc_patient_records_prd.dbt_int_doctor_department` AS d
    ON ef.doctor_code = d.doctor_code AND SAFE_CAST(ef.tenant_id AS INT64) = SAFE_CAST(d.tenant_id AS INT64)
  CROSS JOIN params AS p
  WHERE ef.tenant_id = p.tenant_id AND ef.data_type != 'SY' AND ef.action_detail_number = 0
    AND ef.outpatient_date.value BETWEEN p.period_start AND p.period_end
),
neuro_patients AS (
  SELECT DISTINCT CAST(SAFE_CAST(ef.patient_id AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.dpc_patient_records_prd.outpatient_ef` AS ef
  INNER JOIN neuro_doctors AS nd ON ef.doctor_code = nd.doctor_code AND SAFE_CAST(ef.tenant_id AS INT64) = nd.tenant_id
  CROSS JOIN params AS p
  WHERE ef.tenant_id = p.tenant_id AND ef.data_type != 'SY' AND ef.action_detail_number = 0
    AND ef.outpatient_date.value BETWEEN p.period_start AND p.period_end
),
mri_patients AS (SELECT DISTINCT e.patient_id FROM ef_period AS e INNER JOIN mri_codes AS m ON e.receipt_code = m.code),
target_patients AS (SELECT n.patient_id FROM neuro_patients AS n INNER JOIN mri_patients AS m USING (patient_id)),
nx1_multi_dept AS (
  SELECT DISTINCT t.patient_id FROM target_patients AS t INNER JOIN ef_period AS e USING (patient_id)
  CROSS JOIN params AS p WHERE e.department_name != p.target_department_name
),
nx2_ct AS (
  SELECT DISTINCT t.patient_id FROM target_patients AS t INNER JOIN ef_period AS e USING (patient_id) INNER JOIN ct_codes AS c ON e.receipt_code = c.code
),
nx3_jinbu AS (
  SELECT CAST(NULL AS STRING) AS patient_id LIMIT 0
  -- 将来使う場合のクエリ（現行では実行しない）:
  -- SELECT DISTINCT CAST(SAFE_CAST(r.patient_number AS INT64) AS STRING) AS patient_id
  -- FROM `medup-foro.foro_prd.public_op_optimizer_outpatient_reservations` AS r
  -- INNER JOIN `medup-foro.foro_prd.public_op_optimizer_doctors` AS d ON d.id = r.doctor_id AND d.tenant_id = r.tenant_id
  -- CROSS JOIN params AS p
  -- WHERE r.tenant_id = p.tenant_id AND d.name LIKE '%神部%' AND DATE(r.reserved_at) BETWEEN p.period_start AND p.period_end
),
nx4_vascular_ef AS (
  SELECT DISTINCT CAST(SAFE_CAST(ef.patient_id AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.dpc_patient_records_prd.outpatient_ef` AS ef INNER JOIN vascular_receipt_codes AS v ON ef.receipt_code = v.code
  CROSS JOIN params AS p WHERE ef.tenant_id = p.tenant_id AND ef.data_type != 'SY' AND ef.action_detail_number = 0
),
nx4_vascular_adm AS (
  SELECT DISTINCT CAST(SAFE_CAST(h.patient_number AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.foro_prd.public_hospital_admissions` AS h CROSS JOIN params AS p
  WHERE h.tenant_id = p.tenant_id AND h.kcode_operation_1 IS NOT NULL
    AND ((STARTS_WITH(h.kcode_operation_1, 'K178') AND NOT STARTS_WITH(h.kcode_operation_1, 'K178-3')) OR STARTS_WITH(h.kcode_operation_1, 'K609-2'))
),
nx4_platelet AS (
  SELECT DISTINCT CAST(SAFE_CAST(ef.patient_id AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.dpc_patient_records_prd.outpatient_ef` AS ef INNER JOIN platelet_codes AS pl ON ef.receipt_code = pl.code
  CROSS JOIN params AS p WHERE ef.tenant_id = p.tenant_id AND ef.data_type != 'SY' AND ef.action_detail_number = 0
),
nx4_vascular_or_platelet AS (
  SELECT patient_id FROM nx4_vascular_ef UNION DISTINCT SELECT patient_id FROM nx4_vascular_adm UNION DISTINCT SELECT patient_id FROM nx4_platelet
),
latest_mri AS (
  SELECT e.patient_id, MAX(e.outpatient_date) AS latest_mri_date
  FROM ef_period AS e INNER JOIN mri_codes AS m ON e.receipt_code = m.code INNER JOIN target_patients AS t USING (patient_id)
  GROUP BY e.patient_id
),
nx5_contrast_mri AS (
  -- 造影剤使用加算は明細行（action_detail_number > 0）に載るため E行(=0) に絞らない
  SELECT DISTINCT lm.patient_id
  FROM latest_mri AS lm
  INNER JOIN `medup-foro.dpc_patient_records_prd.outpatient_ef` AS ef
    ON CAST(SAFE_CAST(ef.patient_id AS INT64) AS STRING) = lm.patient_id AND ef.outpatient_date.value = lm.latest_mri_date
  CROSS JOIN params AS p WHERE ef.tenant_id = p.tenant_id AND ef.data_type != 'SY' AND ef.receipt_code = '170020470'
),
nx6_epilepsy AS (
  SELECT DISTINCT t.patient_id FROM target_patients AS t INNER JOIN ef_period AS e USING (patient_id) INNER JOIN epilepsy_codes AS ep ON e.receipt_code = ep.code
),
nx7_sah AS (
  SELECT DISTINCT CAST(SAFE_CAST(h.patient_number AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.foro_prd.public_hospital_admissions` AS h CROSS JOIN params AS p
  WHERE h.tenant_id = p.tenant_id AND h.dpc_code IS NOT NULL AND STARTS_WITH(h.dpc_code, '010020')
    AND h.admission_ended_on > p.sah_lookback_start AND h.admission_ended_on <= p.period_end
),
nx8_reverse_referred AS (
  SELECT DISTINCT CAST(SAFE_CAST(r.patient_id AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.foro_prd.public_reverse_referrals` AS r
  INNER JOIN `medup-foro.foro_prd.public_own_departments` AS dept ON dept.id = r.specialty_id
  INNER JOIN `medup-foro.foro_prd.public_own_department_tenants` AS odt ON odt.own_department_id = dept.id AND odt.tenant_id = r.tenant_id
  CROSS JOIN params AS p
  WHERE r.tenant_id = p.tenant_id AND dept.name = p.target_department_name AND r.reverse_referral_date BETWEEN p.period_start AND p.period_end
)
SELECT
  t.patient_id AS `患者ID`,
  IF(nx1.patient_id IS NOT NULL, 1, 0) AS `NX1_併科`, IF(nx2.patient_id IS NOT NULL, 1, 0) AS `NX2_CT`,
  IF(nx3.patient_id IS NOT NULL, 1, 0) AS `NX3_神部`, IF(nx4.patient_id IS NOT NULL, 1, 0) AS `NX4_血管内`,
  IF(nx5.patient_id IS NOT NULL, 1, 0) AS `NX5_造影MRI`, IF(nx6.patient_id IS NOT NULL, 1, 0) AS `NX6_てんかん`,
  IF(nx7.patient_id IS NOT NULL, 1, 0) AS `NX7_SAH`, IF(nx8.patient_id IS NOT NULL, 1, 0) AS `NX8_逆紹介済`,
  IF(nx1.patient_id IS NULL AND nx2.patient_id IS NULL AND nx3.patient_id IS NULL AND nx4.patient_id IS NULL
     AND nx5.patient_id IS NULL AND nx6.patient_id IS NULL AND nx7.patient_id IS NULL AND nx8.patient_id IS NULL, 1, 0) AS `最終候補`
FROM target_patients AS t
LEFT JOIN nx1_multi_dept AS nx1 USING (patient_id) LEFT JOIN nx2_ct AS nx2 USING (patient_id)
LEFT JOIN nx3_jinbu AS nx3 USING (patient_id) LEFT JOIN nx4_vascular_or_platelet AS nx4 USING (patient_id)
LEFT JOIN nx5_contrast_mri AS nx5 USING (patient_id) LEFT JOIN nx6_epilepsy AS nx6 USING (patient_id)
LEFT JOIN nx7_sah AS nx7 USING (patient_id) LEFT JOIN nx8_reverse_referred AS nx8 USING (patient_id)
ORDER BY t.patient_id;

-- B. ファネル件数（上記CTEの最終SELECTを以下に差し替えて実行）
/*
SELECT
  (SELECT COUNT(*) FROM neuro_patients) AS n_n1_n2_neuro, (SELECT COUNT(*) FROM target_patients) AS n_n3_target,
  COUNTIF(nx1.patient_id IS NOT NULL) AS n_nx1, COUNTIF(nx2.patient_id IS NOT NULL) AS n_nx2,
  COUNTIF(nx3.patient_id IS NOT NULL) AS n_nx3, COUNTIF(nx4.patient_id IS NOT NULL) AS n_nx4,
  COUNTIF(nx5.patient_id IS NOT NULL) AS n_nx5, COUNTIF(nx6.patient_id IS NOT NULL) AS n_nx6,
  COUNTIF(nx7.patient_id IS NOT NULL) AS n_nx7, COUNTIF(nx8.patient_id IS NOT NULL) AS n_nx8,
  COUNTIF(nx1.patient_id IS NULL AND nx2.patient_id IS NULL AND nx3.patient_id IS NULL AND nx4.patient_id IS NULL
          AND nx5.patient_id IS NULL AND nx6.patient_id IS NULL AND nx7.patient_id IS NULL AND nx8.patient_id IS NULL) AS n_final_candidates
FROM target_patients AS t
LEFT JOIN nx1_multi_dept AS nx1 USING (patient_id) LEFT JOIN nx2_ct AS nx2 USING (patient_id)
LEFT JOIN nx3_jinbu AS nx3 USING (patient_id) LEFT JOIN nx4_vascular_or_platelet AS nx4 USING (patient_id)
LEFT JOIN nx5_contrast_mri AS nx5 USING (patient_id) LEFT JOIN nx6_epilepsy AS nx6 USING (patient_id)
LEFT JOIN nx7_sah AS nx7 USING (patient_id) LEFT JOIN nx8_reverse_referred AS nx8 USING (patient_id);
*/
```

#### 4-2-3. 泌尿器科（面談：村岡先生）
- **対象（`U1 ∧ U2 ∧ (U3 ∨ U4 ∨ U5)`、約986人）**：直近1年外来（U1）∧ 泌尿器科受診（U2）∧（最大行為 `4.CT/MRI` かつ受診年2回以下［U3］ ∨ 最大行為 `5.検査（US・XP含む）` かつ受診年5回以下・メインは年1〜2回のPSA［U4］ ∨ 最大行為 `6.投薬` / `8.再診`［U5］）。
- **除外フラグ**：
  - `UX1_併科`：期間内に泌尿器科以外の受診あり
  - `UX2_MRI`：期間内にMRI算定あり（診療科問わず。ほぼ前立腺がん院内フォロー）
  - `UX3_鳥取大`：鳥取大学医学部附属病院（ID：`75462`）から**泌尿器科へ**の紹介あり（紹介日は全期間・他科紹介は除外しない。ロボット手術後フォロー想定）
  - `UX4_術後`：前立腺がんDPC（`110080`）かつ `kcode_operation_1` が `K843` 系かつ退院が直近5年以内（6年目以降は対象可）
  - `UX6_逆紹介済`：期間内に泌尿器科から逆紹介済み
- **不採用条件メモ（UX5：積極的監視療法疑い）**：8/25合意により除外・フラグともに削除。当初検討ロジック（吉田案）：Step1（直近1年の悪性腫瘍特異物質治療管理料＋全期間の骨シンチ・前立腺針生検補助）→ Step2（K843手術歴なし）→ Step3((a)直近3年以内に生検orMRI→除外、(b)管理料初回算定1〜2年以内→フラグ残し、(c)その他→候補残し)。既知の限界：他院診断患者、腎腫瘍（C64・マーカーなし）、放射線治療後患者の扱い。

```sql
-- プローブ（任意）: 医師マスタ表記・鳥取大学ID確認（ID確定済: 75462）
/*
SELECT department_name, COUNT(*) AS n
FROM `medup-foro.dpc_patient_records_prd.dbt_int_doctor_department`
WHERE SAFE_CAST(tenant_id AS INT64) = 274 AND department_name LIKE '%泌尿%' GROUP BY department_name;

SELECT mi.medical_institution_id, mi.name, COUNT(*) AS n
FROM `medup-foro.foro_prd.public_referrals` AS r
INNER JOIN `medup-foro.foro_prd.tenant_29_view_medical_institutions` AS mi ON mi.medical_institution_id = r.medical_institution_id_from
WHERE r.tenant_id = 274 AND mi.name LIKE '%鳥取大学%' GROUP BY mi.medical_institution_id, mi.name ORDER BY n DESC;
*/

-- A. 候補リスト本体（泌尿器科 / tenant 274）
WITH
params AS (
  SELECT 274 AS tenant_id, DATE '2025-08-01' AS period_start, DATE '2026-07-31' AS period_end,
         '泌尿器科' AS target_department_name, 75462 AS tottori_univ_medical_institution_id,
         DATE_SUB(DATE '2026-07-31', INTERVAL 5 YEAR) AS prostate_surgery_lookback_start
),
mri_codes AS (SELECT code FROM UNNEST(['170020110', '170033510']) AS code),
uro_doctors AS (
  SELECT SAFE_CAST(d.tenant_id AS INT64) AS tenant_id, d.doctor_code, d.department_name
  FROM `medup-foro.dpc_patient_records_prd.dbt_int_doctor_department` AS d
  CROSS JOIN params AS p WHERE SAFE_CAST(d.tenant_id AS INT64) = p.tenant_id AND d.department_name = p.target_department_name
),
uro_ef AS (
  SELECT
    ef.tenant_id, CAST(SAFE_CAST(ef.patient_id AS INT64) AS STRING) AS patient_id,
    ef.doctor_code, ud.department_name, ef.outpatient_date.value AS outpatient_date, ef.receipt_code, ef.action_name
  FROM `medup-foro.dpc_patient_records_prd.outpatient_ef` AS ef
  INNER JOIN uro_doctors AS ud ON ef.doctor_code = ud.doctor_code AND SAFE_CAST(ef.tenant_id AS INT64) = ud.tenant_id
  CROSS JOIN params AS p
  WHERE ef.tenant_id = p.tenant_id AND ef.data_type != 'SY' AND ef.action_detail_number = 0
    AND ef.outpatient_date.value BETWEEN p.period_start AND p.period_end
),
ef_period AS (
  SELECT
    CAST(SAFE_CAST(ef.patient_id AS INT64) AS STRING) AS patient_id, ef.doctor_code,
    IFNULL(d.department_name, '未割当') AS department_name, ef.outpatient_date.value AS outpatient_date, ef.receipt_code
  FROM `medup-foro.dpc_patient_records_prd.outpatient_ef` AS ef
  LEFT JOIN `medup-foro.dpc_patient_records_prd.dbt_int_doctor_department` AS d
    ON ef.doctor_code = d.doctor_code AND SAFE_CAST(ef.tenant_id AS INT64) = SAFE_CAST(d.tenant_id AS INT64)
  CROSS JOIN params AS p
  WHERE ef.tenant_id = p.tenant_id AND ef.data_type != 'SY' AND ef.action_detail_number = 0
    AND ef.outpatient_date.value BETWEEN p.period_start AND p.period_end
),
patient_flags AS (
  SELECT
    patient_id, COUNT(DISTINCT outpatient_date) AS visit_count_in_specialty,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '111') THEN 1 ELSE 0 END) AS first_visit_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '112') THEN 1 ELSE 0 END) AS follow_up_visit_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '140') THEN 1 ELSE 0 END) AS procedure_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '150') THEN 1 ELSE 0 END) AS surgery_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '13') THEN 1 ELSE 0 END) AS injection_flag,
    MAX(CASE WHEN receipt_code IN ('170033410', '170012070', '170011810', '170035010', '170020110', '170020470') THEN 1 ELSE 0 END) AS ct_imaging_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '170') THEN 1 ELSE 0 END) AS imaging_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '160') THEN 1 ELSE 0 END) AS exam_flag,
    MAX(CASE WHEN STARTS_WITH(receipt_code, '12') THEN 1 ELSE 0 END) AS medication_flag,
    MAX(CASE WHEN receipt_code IN ('180756010', '180024710') THEN 1 ELSE 0 END) AS rehabilitation_flag
  FROM uro_ef GROUP BY patient_id
),
admission_uro AS (
  SELECT DISTINCT CAST(SAFE_CAST(h.patient_number AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.foro_prd.public_hospital_admissions` AS h
  LEFT JOIN `medup-foro.foro_prd.public_dpc_specialty_codes` AS c ON c.code = h.dpc_specialty_code
  CROSS JOIN params AS p
  WHERE h.tenant_id = p.tenant_id AND (c.name = p.target_department_name OR c.name LIKE '%泌尿%')
    AND (h.admission_started_on BETWEEN p.period_start AND p.period_end OR h.admission_ended_on BETWEEN p.period_start AND p.period_end)
),
categorized AS (
  SELECT
    f.patient_id, f.visit_count_in_specialty,
    CASE
      WHEN a.patient_id IS NOT NULL THEN '1.入院'
      WHEN f.first_visit_flag = 1 THEN '2.初診'
      WHEN f.follow_up_visit_flag = 1 AND (f.procedure_flag = 1 OR f.surgery_flag = 1 OR f.injection_flag = 1) THEN '3.処置/手術/注射'
      WHEN f.follow_up_visit_flag = 1 AND f.ct_imaging_flag = 1 THEN '4.CT/MRI'
      WHEN f.follow_up_visit_flag = 1 AND (f.imaging_flag = 1 OR f.exam_flag = 1) THEN '5.検査（US・XP含む）'
      WHEN f.follow_up_visit_flag = 1 AND f.medication_flag = 1 THEN '6.投薬'
      WHEN f.follow_up_visit_flag = 1 AND f.rehabilitation_flag = 1 THEN '7.リハ'
      WHEN f.follow_up_visit_flag = 1 THEN '8.再診'
      ELSE '9.その他'
    END AS medical_service_category
  FROM patient_flags AS f LEFT JOIN admission_uro AS a USING (patient_id)
),
target_patients AS (
  SELECT * FROM categorized
  WHERE (medical_service_category = '4.CT/MRI' AND visit_count_in_specialty <= 2)
     OR (medical_service_category = '5.検査（US・XP含む）' AND visit_count_in_specialty <= 5)
     OR medical_service_category IN ('6.投薬', '8.再診')
),
ux1_multi_dept AS (
  SELECT DISTINCT t.patient_id FROM target_patients AS t INNER JOIN ef_period AS e USING (patient_id)
  CROSS JOIN params AS p WHERE e.department_name != p.target_department_name
),
ux2_mri AS (
  SELECT DISTINCT t.patient_id FROM target_patients AS t INNER JOIN ef_period AS e USING (patient_id) INNER JOIN mri_codes AS m ON e.receipt_code = m.code
),
ux3_tottori_univ AS (
  SELECT DISTINCT CAST(SAFE_CAST(r.patient_id AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.foro_prd.public_referrals` AS r
  INNER JOIN `medup-foro.foro_prd.public_own_departments` AS dept ON dept.id = r.specialty_id
  INNER JOIN `medup-foro.foro_prd.public_own_department_tenants` AS odt ON odt.own_department_id = dept.id AND odt.tenant_id = r.tenant_id
  CROSS JOIN params AS p
  WHERE r.tenant_id = p.tenant_id AND dept.name = p.target_department_name AND r.medical_institution_id_from = p.tottori_univ_medical_institution_id
),
ux4_prostate_surgery AS (
  SELECT DISTINCT CAST(SAFE_CAST(h.patient_number AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.foro_prd.public_hospital_admissions` AS h CROSS JOIN params AS p
  WHERE h.tenant_id = p.tenant_id AND h.dpc_code IS NOT NULL AND STARTS_WITH(h.dpc_code, '110080')
    AND h.kcode_operation_1 IS NOT NULL AND STARTS_WITH(h.kcode_operation_1, 'K843')
    AND h.admission_ended_on > p.prostate_surgery_lookback_start AND h.admission_ended_on <= p.period_end
),
ux6_reverse_referred AS (
  SELECT DISTINCT CAST(SAFE_CAST(r.patient_id AS INT64) AS STRING) AS patient_id
  FROM `medup-foro.foro_prd.public_reverse_referrals` AS r
  INNER JOIN `medup-foro.foro_prd.public_own_departments` AS dept ON dept.id = r.specialty_id
  INNER JOIN `medup-foro.foro_prd.public_own_department_tenants` AS odt ON odt.own_department_id = dept.id AND odt.tenant_id = r.tenant_id
  CROSS JOIN params AS p
  WHERE r.tenant_id = p.tenant_id AND dept.name = p.target_department_name AND r.reverse_referral_date BETWEEN p.period_start AND p.period_end
)
SELECT
  t.patient_id AS `患者ID`, t.medical_service_category AS `最大行為`, t.visit_count_in_specialty AS `受診回数`,
  IF(ux1.patient_id IS NOT NULL, 1, 0) AS `UX1_併科`, IF(ux2.patient_id IS NOT NULL, 1, 0) AS `UX2_MRI`,
  IF(ux3.patient_id IS NOT NULL, 1, 0) AS `UX3_鳥取大`, IF(ux4.patient_id IS NOT NULL, 1, 0) AS `UX4_術後`,
  IF(ux6.patient_id IS NOT NULL, 1, 0) AS `UX6_逆紹介済`,
  IF(ux1.patient_id IS NULL AND ux2.patient_id IS NULL AND ux3.patient_id IS NULL AND ux4.patient_id IS NULL AND ux6.patient_id IS NULL, 1, 0) AS `最終候補`
FROM target_patients AS t
LEFT JOIN ux1_multi_dept AS ux1 USING (patient_id) LEFT JOIN ux2_mri AS ux2 USING (patient_id)
LEFT JOIN ux3_tottori_univ AS ux3 USING (patient_id) LEFT JOIN ux4_prostate_surgery AS ux4 USING (patient_id)
LEFT JOIN ux6_reverse_referred AS ux6 USING (patient_id)
ORDER BY t.patient_id;

-- B. ファネル件数（上記CTEの最終SELECTを以下に差し替えて実行）
/*
SELECT
  (SELECT COUNT(DISTINCT patient_id) FROM uro_ef) AS n_u1_u2_uro, (SELECT COUNT(*) FROM target_patients) AS n_u3_u5_target,
  COUNTIF(ux1.patient_id IS NOT NULL) AS n_ux1, COUNTIF(ux2.patient_id IS NOT NULL) AS n_ux2,
  COUNTIF(ux3.patient_id IS NOT NULL) AS n_ux3, COUNTIF(ux4.patient_id IS NOT NULL) AS n_ux4, COUNTIF(ux6.patient_id IS NOT NULL) AS n_ux6,
  COUNTIF(ux1.patient_id IS NULL AND ux2.patient_id IS NULL AND ux3.patient_id IS NULL AND ux4.patient_id IS NULL AND ux6.patient_id IS NULL) AS n_final_candidates
FROM target_patients AS t
LEFT JOIN ux1_multi_dept AS ux1 USING (patient_id) LEFT JOIN ux2_mri AS ux2 USING (patient_id)
LEFT JOIN ux3_tottori_univ AS ux3 USING (patient_id) LEFT JOIN ux4_prostate_surgery AS ux4 USING (patient_id)
LEFT JOIN ux6_reverse_referred AS ux6 USING (patient_id);
*/
```

---

### 4-3. 新東京病院グループ（tenant_id: 135）：自己注射除外条件追加・未受領EF調査
- **3施設統合テナント**：`tenant_id = 135` に以下3施設のEFファイルを統合投入（患者IDは3施設共通想定）。
  1. 新東京病院（主に入院中心。外来は基本なし）
  2. 新東京クリニック（外来）
  3. 新東京ハートクリニック（略称：`HCL`、外来）
- **クエリ管理**：RedashではなくGitHub上に保管されたクエリを修正・実行する。
- **変更依頼（2026/07/28、依頼：kanbara、対応：shota.kurokawa → Jun.Yamada 引き継ぎ）**：
  1. 今後は候補患者リストのみでOK（逆紹介先候補の施設は不要）。
  2. 糖尿病内科の抽出条件から「在宅自己注射指導管理料」を直近で算定している患者を除外（既存条件に合わせ「2025年度内算定」でまず出力。他期間案として直近3ヶ月＝最新データ月 `n-1` に対し `n-4〜n-1` 月等も比較フラグ付与）。
  3. 2025年度候補者リストのR列に「自己注射指導管理料」対象（シート1）を結合。
- **参照リンク**：
  - Slack：[依頼元スレッド](https://medup.slack.com/archives/C09E6V3SB98/p1784864664249839)
  - 出力Drive：[フォルダ](https://drive.google.com/drive/folders/1-YjXLluz-EQSzTusb9xREO9o06z4Em3c) / [R列フラグ結合シート](https://docs.google.com/spreadsheets/d/1W2D5au1yhUOxtLtHMLp9mmneqPFDqHLL0tfrLYerK2s/edit?gid=175538192#gid=175538192) / [先方算定項目シート](https://docs.google.com/spreadsheets/d/132wZd-mzWrdvhYhWDRNibsk-nrxypd4n/edit?usp=drive_link&ouid=116589248244187246615&rtpof=true&sd=true)
  - 調査マニュアル：[Circleback録画](https://circleback.ai/meetings/EW4eNoWa1uTO9aoC6Oowr) / [Driveフォルダ](https://drive.google.com/drive/folders/18cdNsg9P6JmV2LQKhXwh42EWo4XqgMtZ?usp=drive_link)

#### 抽出・調査用SQL
```sql
-- 1. シート1（期間別・自己注射指導管理料フラグ付与）
WITH base AS (
  SELECT
    tenant_id, patient_id, specialty_code, STRING_AGG(receipt_code) AS receipt_codes,
    LOGICAL_OR(outpatient_date.value BETWEEN DATE '2025-04-01' AND DATE '2026-03-31' AND receipt_code IN ('114021010', '114028410', '114040610', '114062910', '114063010', '114063110')) AS has_target_2025_fy,
    LOGICAL_OR(outpatient_date.value BETWEEN DATE '2026-01-01' AND DATE '2026-06-30' AND receipt_code IN ('114021010', '114028410', '114040610', '114062910', '114063010', '114063110')) AS has_target_2026_01_06,
    LOGICAL_OR(outpatient_date.value BETWEEN DATE '2026-04-01' AND DATE '2026-06-30' AND receipt_code IN ('114021010', '114028410', '114040610', '114062910', '114063010', '114063110')) AS has_target_2026_04_06,
    LOGICAL_OR(outpatient_date.value BETWEEN DATE '2026-06-01' AND DATE '2026-06-30' AND receipt_code IN ('114021010', '114028410', '114040610', '114062910', '114063010', '114063110')) AS has_target_2026_06
  FROM `medup-foro.dpc_patient_records_prd.outpatient_ef`
  WHERE tenant_id = 135 AND outpatient_date.value BETWEEN DATE '2025-04-01' AND DATE '2026-06-30'
  GROUP BY tenant_id, patient_id, specialty_code
)
SELECT a.tenant_id, a.patient_id, a.specialty_code, b.name, a.receipt_codes,
       a.has_target_2025_fy, a.has_target_2026_01_06, a.has_target_2026_04_06, a.has_target_2026_06
FROM base AS a
LEFT JOIN `medup-foro.foro_prd.public_dpc_specialty_codes` AS b ON a.specialty_code = b.code;

-- 2. 期間欠損調査（レセプトコードベース）
SELECT receipt_code, COUNT(*) AS count
FROM `medup-foro.dpc_patient_records_prd.outpatient_ef`
WHERE tenant_id = 135 AND CAST(receipt_code AS STRING) IN ('114021010', '114028410', '114040610', '114062910', '114063010', '114063110')
  AND outpatient_date.value BETWEEN DATE '2026-01-01' AND DATE '2026-06-30'
GROUP BY receipt_code ORDER BY receipt_code;

-- 3. 期間欠損調査（行為名称ベース）
SELECT receipt_code, COUNT(*) AS count
FROM `medup-foro.dpc_patient_records_prd.outpatient_ef`
WHERE tenant_id = 135 AND action_name LIKE '%自己注射%'
  AND outpatient_date.value BETWEEN DATE '2026-01-01' AND DATE '2026-06-30'
GROUP BY receipt_code ORDER BY receipt_code;

-- 4. 全テナント実データからの対象コード洗い出し
SELECT DISTINCT action_name, receipt_code, action_fee
FROM `medup-foro.dpc_patient_records_prd.outpatient_ef`
WHERE action_name LIKE '%在宅自己注射指導管理%' AND action_name NOT LIKE '%加算%' AND action_fee > 0
ORDER BY action_name;
```

#### 2026/08/03 発覚したデータ欠損・注意点
- **事象**：2025/12までは自己注射コードがEF上に存在するが、2026/01以降は0件となった。また同一データの2回投入形跡も確認された。
- **原因**：3施設のうち2026/01以降に受領できていたのは「新東京病院」のみで、外来を担うクリニック2施設が未受領だった。
- **再受領依頼対象**：
  - 新東京クリニック：`2026/01` 〜 `2026/06` の外来E・外来Fデータ
  - 新東京ハートクリニック（HCL）：`2026/01` 〜 `2026/06` の外来E・外来Fデータ
  - 新東京病院：`2025/08`, `2026/01` の外来E・外来Fデータ（過去再受領依頼放置分）
- **運用メモ**：先方提示リストの代表4桁（例：`8700`）ではなく9桁のレセプトフルコードで確認すること。入院EFは不要と伝えているが送付されてくる傾向あり。

---

## 5. MISマスタ運用・重複データ統合・不整合修正

### 5-1. MIS月次作業（厚生局マスタ更新）
1. **厚生局データの取得**：DataOpsメンバーがGoogle Driveへ格納した対象データを取得（`https://drive.google.com/drive/folders/1-BeEQGktTthtjLwd6_FZNMFw-gkr1bJe`）。
2. **厚生局データのコンバート**：`https://github.com/MedUpInc/data-customer-ops`（`update-mis/Pipfile`）を実行し、`output` 配下に生成。
   - `extract-overview = "python3 src/extract_from_overview.py"`
   - `extract-new = "python3 src/extract_from_new.py"`
   - `extract-closed = "python3 src/extract_from_closed.py"`
3. **MIS管理ツールでのファイル取り込み**（`https://admin-web.medup-core-service.com/mhlw-sync-works`）：
   - 新規・廃止一覧を登録（先行登録済リストと目検突合し、合致する場合は医療機関IDをコピー）。
   - 一覧ファイルをアップロードし医療機関コードで突合（差分＝UPDATE／一覧のみ＝厚生局のみ／MISのみ＝自社マスタのみ）。
4. **重複データの統合作業**：5-2の手順で統合を実施。
   - 重複解消対象一覧：`https://docs.google.com/spreadsheets/d/1WHt4zmYkwGbST_n9onbbdlYgxiDkg-OpszMmMf8dOPw/edit?gid=0#gid=0`
   - 議事録・手順資料：`https://circleback.ai/meetings/qtBXk0cnT5CwC4kl4HpCG` / `https://circleback.ai/meetings/EJMhpYibexu3x3IJ1SYqU?tab=transcript` / `https://docs.google.com/document/d/1vSSa62L-N71H7_hnNQKJ2_zaOg8Pzf_oqAOEXgK0qjI/edit?tab=t.0#heading=h.jg8znh54ny7r` / `https://drive.google.com/file/d/1VFYR-Az6TAf-Y4cZ2zT6Ld_up3msVkwp/view`

#### 発生したエラーと対処法（2026年8月〜9月実績）
- **エラー1：新規登録時の `medicalInstitutionCode` 重複エラー**
  - 該当ID：`40178` (`1415411087`), `404403` (`2342205453`), `402482` (`2343001521`)
  - 確認先：`https://admin-web.medup-core-service.com/medical-institutions/{40178|404403|402482}`
  - 原因・対処：3件とも既に正しく登録済でINSERT不要のため、ワークテーブル（`mis_prd.mhlw_sync_work_insert_records`）から対象行を `DELETE` し再実行。
- **エラー2：差分取り込み時の `medicalInstitutionCode` 重複エラー**
  - 該当ID：`53160` (`2313000487`, recordId `f48a3f80-10af-461d-a837-89235e7f046a`), `156062` (`3910119738`, recordId `b318bb5d-3b2d-4fa6-8471-5638f834f4b0`)
  - 確認先：`https://admin-web.medup-core-service.com/medical-institutions/{53160|156062}`
  - 原因・対処：重複データ側がアーカイブ済なのに医療機関コードが残存していた。本来は**アーカイブ時に医療機関コードを `0` に変更する**のが正しい運用。
- **エラー3：新規一覧に存在し、一覧ファイルに存在しないケース**
  - そのまま処理すると `insert` → `休止` となるため救済SQLが必要（最悪、1ヶ月後のマスタ更新で差分としてヒットするため救済可能）。

---

### 5-2. MIS重複医療機関データの統合（仕様・テーブル一覧・手順）

#### 発生背景・基本方針・全体手順
- **暫定登録ケース**：ユーザー依頼の手動登録／紹介データ受領時にマスタ未存在。
- **重複発生原因**：月1回の厚生局データ統合で、名称変更時は検知できない（新規医療機関はロジック統合可能）。
- **基本ルール**：
  - 厚生局データとMIS全件を突合してイレギュラー抽出し、MIS検索で複数ヒットしたものを重複とする。
  - **基本は厚生局データ側（医療機関コードと繋がっている方）を残す**。
  - **グループ病院機能注意**：`tenant_id = 13`（海老名座間）、`165`（音羽／音羽リハ／音羽記念）はレポート系の `tenant_medical_institution_ids` 等にIDが入っているため要確認。
  - **未調査領域**：`acute_cf` 関連は機能側の調査が必要。機能追加に伴い対象テーブルが増えるため都度確認。
- **全体手順**：(1) 統合依頼受領 → (2) 残す側を決定 → (3) Redashで使用テーブルと重複状況を調査し、消えるデータ詳細（メモ・ラベル等）をCSへ連絡 → (4) UPDATE/DELETE SQL実行 → (5) 不要な重複元をアーカイブ（医療機関コードを `0` に変更）。
- **確認用Redash・リンク**：
  - `foro_prd`：`https://data-mgmt.medup.jp/queries/2191?p_medical_institution_ids=51194%2C%20407252`（他例：`405574, 34576`）
  - `data_ops_prd`：`https://data-mgmt.medup.jp/queries/2287/source?p_medical_institution_ids=51194%2C%20407252`（他例：`405574, 34576`）
  - ラベル詳細調査：`https://data-mgmt.medup.jp/queries/2291/source?p_medical_institution_id=392993&p_medical_institution_ids=392993%2C404168&p_tenant_id=157`
  - スプレッドシート：[統合調査メモ](https://docs.google.com/spreadsheets/d/1tHStLUKcoJTqQVV6ZuGRmJ93dnkHq2KbPfBLw-wCGhw/edit?gid=16580683#gid=16580683) / [重複統合作業シート](https://docs.google.com/spreadsheets/d/1FX7KDjR11VVgxFxtvJ1VckRzLZfzC5vJMc1JV8i6oOU/edit?gid=1848597190#gid=1848597190) / 親Notion：`https://app.notion.com/p/20260708_MIS-3961da5a7f3b80ae92b5e56c35090533?pvs=21`
  - 過去事例：大寿ファミリークリニック（`51194`, `407252`）、土井消化器科内科医院（`76328`, `405419`）

#### 移行分類の定義と実務ルール
| 分類 | 意味 | SQL生成方針 |
| --- | --- | --- |
| **A: 単純UPDATE可能** | ユニーク制約なし、重複を作りにくい | `UPDATE ... SET medical_institution_id = dst WHERE medical_institution_id = src` |
| **B: 衝突処理が必要** | `medical_institution_id` を含むユニーク制約あり | 移行先既存行を優先し、移行元行は削除または手動マージ後に削除 |
| **C: 人間確認が必要** | メモ・特記事項・ラベル値など中身に意味がある | 詳細抽出SQLで中身を見てからマージSQLを決定 |
| **D: 特別扱い** | ID置換だけでは足りない、またはテナント設定に関わる | 個別方針を決めてからSQL化 |

- **実務運用上の重複NG項目（2026/08/04ルール）**：特記事項、医療機関ラベル（`medical_institution_label_values`）、医療機関メモ（`medical_institution_memos`）、フェーズ。
  - `action_list_medical_institutions`：別リストならUPDATE、同一リストならリンク情報なので1つあればよく統合不要。
  - `medical_institution_label_values` / `medical_institution_memos`：DataOpsでどちらを残すか判断せず、アーカイブ対象を出力してCSへ連携。

#### `foro` DB 対象テーブル一覧
| テーブル名 | 対象カラム | 分類 | ユニーク制約・注意点 | 移行方針 |
| --- | --- | --- | --- | --- |
| `action_list_medical_institutions` | `medical_institution_id` | B | `UNIQUE(action_list_id, medical_institution_id)` | 衝突行は移行先を残し移行元削除。非衝突行のみUPDATE。 |
| `communications` | `medical_institution_id` | A | ユニーク制約なし（`content` JSON内に名称残存の可能性あり） | 単純UPDATE。 |
| `crm_medical_institution_phases` | `medical_institution_id` | B/C | `UNIQUE(tenant_id, medical_institution_id)` | 片方のみならUPDATE。両方ある場合は人間確認（新しい `changed_at` 採用等）後片方削除。 |
| `crm_medical_institution_phase_change_histories` | `medical_institution_id` | A | 履歴テーブル・ユニーク制約なし | 単純UPDATE。 |
| `crm_medical_institution_special_notes` | `medical_institution_id` | C | `UNIQUE(tenant_id, medical_institution_id)`、`note_content` フリーテキスト | 片方のみならUPDATE。両方ある場合は内容確認し方針決定。 |
| `crm_vocs` | `medical_institution_id` | A | ユニーク制約なし | 単純UPDATE。 |
| `doctors` | `medical_institution_id` | A/C | ユニーク制約なし（同一医師重複登録の可能性あり） | 単純UPDATE。重複医師整理は別途人間確認。 |
| `email_stats` | `medical_institution_id` | A | nullable・ユニーク制約なし | 単純UPDATE。 |
| `group_medical_institutions` | `medical_institution_id` | B | `UNIQUE(tenant_id, medical_institution_id)` | 衝突行は移行先を残し移行元削除。非衝突行のみUPDATE。 |
| `medical_institution_label_values` | `medical_institution_id` | C | `UNIQUE(medical_institution_id, label_id, selection_id)` / `(..., area_id)` | 詳細抽出必須。非衝突行UPDATE。衝突行は `component_type` 別に判断（実務上はCS連携）。 |
| `medical_institution_memos` | `medical_institution_id` | C | `UNIQUE(tenant_id, medical_institution_id)`、`memo_content` フリーテキスト | 片方のみならUPDATE。両方ある場合は内容確認（実務上はアーカイブ＋CS連携）。 |
| `medical_institution_report_item_aggregation_conditions` | `medical_institution_id` | A | nullable・ユニーク制約なし（配列/JSON `tenant_medical_institution_ids` は原則対象外） | `medical_institution_id` のみ単純UPDATE。 |
| `referrals` | `medical_institution_id_from` | A | ユニーク制約なし（`to` は対象外） | `medical_institution_id_from` のみ単純UPDATE。 |
| `reverse_referrals` | `medical_institution_id_to` | A | ユニーク制約なし（`from` は対象外） | `medical_institution_id_to` のみ単純UPDATE。 |
| `tag_medical_institutions` | `medical_institution_id` | B | `UNIQUE(tag_id, medical_institution_id)` | 衝突行は移行先を残し移行元削除。非衝突行のみUPDATE。 |
| `tasks` | `medical_institution_id` | A | nullable・ユニーク制約なし | 単純UPDATE。 |
| `tenants` | `medical_institution_id` | D | テナント自身の自院設定 | 人間確認後にUPDATE。 |
| `medical_institution_attachment_files` | `medical_institution_id` | A | ユニーク制約なし（確認SQLは `GROUP BY tenant_id, medical_institution_id` に修正済） | 単純UPDATE。 |
| `acute_cf_referral_medical_institutions` | `medical_institution_id` | D | `UNIQUE(tenant_id, medical_institution_id)`。主キーUUIDを `acute_cf_patients.referral_medical_institution_id` が参照 | 片方のみならUPDATE。両方ある場合は患者側UUIDを移行先行へ付け替えてから移行元削除。`display_name`, `order` 要確認。 |

#### `medical_institution_label_values` の `component_type` 別扱い
| check SQL `table_name` | `component_type` | 表示名 | 主な保存先 | 関連テーブル | 衝突・統合判断 |
| --- | --- | --- | --- | --- | --- |
| `medical_institution_label_values.text` | `0`, `5` | `text`, `textarea` | `text_value` | なし | 同一 `label_id` で文字列同一なら移行元削除。異なれば人間確認。 |
| `medical_institution_label_values.integer` | `1` | `integer` | `integer_value` | なし | 数値同一なら移行元削除。異なれば人間確認。 |
| `medical_institution_label_values.single_selection` | `2`, `3` | `select`, `radio` | `selection_id` | `medical_institution_label_selections` | `selection_id` 同一なら移行元削除。異なれば人間確認。 |
| `medical_institution_label_values.checkbox` | `4` | `checkbox` | `id`（親）＋中間テーブル | `medical_institution_label_value_selections` | `selection_id` 集合で比較。同一なら親行ごと削除。統合時は中間テーブルを移行先親行へ追加後、移行元親行削除。 |
| `medical_institution_label_values.area` | `6` | `area` | `area_id` | `medical_institution_label_areas`, `..._area_addresses` | 自動生成されるため移行対象外。 |

#### `data_ops` DB 対象テーブル一覧
| テーブル名 | 対象カラム | 分類 | ユニーク制約・注意点 | 移行方針 |
| --- | --- | --- | --- | --- |
| `tenant_medical_institution_code_ids` | `medical_institution_id` | A/B | schema上ユニーク制約なし（`tenant_id`, `dpc_medical_institution_code` の実質重複に注意） | 原則UPDATE。同一tenant・コード重複時は確認。 |
| `own_department_mappings` | `target_medical_institution_id` | B/C | migrationで `COALESCE(target_medical_institution_id, -1)` のカスタムユニーク制約あり | 非衝突行UPDATE。衝突時は内容確認し片方削除。 |
| `medical_institution_mappings` | `medical_institution_id` | B/C | migrationで `(tenant_id, received_medical_institution_name, COALESCE(received_medical_institution_address, ''))` のカスタムユニーク制約あり | 原則UPDATE。同一受領名・住所で移行先行があれば移行元削除。異なれば両方残してID寄せ可。 |
| `medical_institution_mapping_work_sheet_section_records` | `medical_institution_id` | C | 受領名・住所系のカスタムユニーク制約あり（`memo`, `is_edited`, `is_checked`, `pattern`） | 基本UPDATE可能だが作業履歴変更の必要性を判断。 |
| `medical_institution_mapping_work_sheet_review_records` | `medical_institution_id` | C | `UNIQUE(tenant_id, review_id, received_medical_institution_name, received_medical_institution_address)`＋section recordユニーク参照 | 基本UPDATE可能。重複削除は慎重に実施。 |
| `referral_masters` | `medical_institution_id_from`, `..._to` | A | `UNIQUE(tenant_id, referral_control_id)` | 原則 `from` を更新、`to` は必要時のみ。 |
| `referral_master_work_sheet_records` | `medical_institution_id_from`, `..._to` | C | 取り込み作業シート行 | 履歴保持優先なら未変更可。再利用優先ならUPDATE。 |
| `reverse_referral_masters` | `medical_institution_id_from`, `..._to` | A | `UNIQUE(tenant_id, reverse_referral_control_id)` | 原則 `to` を更新、`from` は必要時のみ。 |
| `reverse_referral_master_work_sheet_records` | `medical_institution_id_from`, `..._to` | C | 取り込み作業シート行 | 履歴保持優先なら未変更可。再利用優先ならUPDATE。 |
| `tmp_ref_naviis` | `medical_institution_id` | B/C | `UNIQUE(medical_institution_id)` | 両IDに行がある場合は中身確認必須。consultation hours親子関係も考慮。 |
| `navii_medical_institution_mappings` | `medical_institution_id` | A/C | nullable、`UNIQUE(navii_name, navii_address)` | 原則UPDATE。名前/住所がどちらに近いか確認。 |

#### `mis` DB 対象テーブル一覧（`medical_institutions.id` 自体は変更しない）
| テーブル名 | 対象カラム | 分類 | ユニーク制約・注意点 | 移行方針 |
| --- | --- | --- | --- | --- |
| `medical_institutions` | `id` | D | マスタ本体（主キー） | ID更新しない。参照移行後、重複元を `archived` 相当にするか人間判断。 |
| `medical_institution_facility_types` | `medical_institution_id` | B | `UNIQUE(medical_institution_id, facility_type_id)` | 衝突行は移行先を残し移行元削除。非衝突行UPDATE。 |
| `medical_institution_specialties` | `medical_institution_id` | B | `UNIQUE(medical_institution_id, specialty_id)` | 衝突行は移行先を残し移行元削除。非衝突行UPDATE。 |
| `ref_naviis` | `medical_institution_id` | C/D | `UNIQUE(medical_institution_id)`。子 `ref_naviis_consultation_hours` が `ref_navii_id` 参照 | 両IDに行がある場合は中身確認必須。採用行を決め子テーブル付け替え/統合後に片方削除。 |
| `mhlw_sync_work_insert_records` / `_update_records` / `_close_records` / `_overview_diff_records` / `_overview_only_list_records` / `_overview_only_mis_records` | `medical_institution_id` | D | 厚労省同期作業の履歴・結果テーブル | 原則履歴として残す（未処理・再処理対象や現在参照検索運用時のみ検討）。 |
| `medical_institution_logs` | `medical_institution_id` | D | 変更ログ（`medical_institution` 削除時はCascade） | 原則履歴として残す（統合は監査観点で人間判断）。 |

#### 対象外だが存在する関連テーブル
- `alert_medical_institutions.medical_institution_id`：**無視**（他データとの関連でアラートの意味自体が変わるため）。
- `medup_foro_medical_institution_segments.medical_institution_id`：**無視**（1日経てば自動生成されるため）。
- レポート系 `tenant_medical_institution_ids`：**`tenant_id = 13`, `165` の場合のみ確認**。

---

### 5-3. MIS重複医療機関 確認作業ログ（2026/08/04 カン）
| 医療機関ID | コード | 医療機関名 | 住所 | ステータス / 区分 | 確認結果・対応 |
| --- | --- | --- | --- | --- | --- |
| `409670` | `1` | セタファミリークリニック | 愛知県名古屋市北区黒川本通４丁目３７番地カーサビアンカ黒川2階 | `open` / `2` | 2件ヒット。`104502` が正。opsのみ `medical_institution_mappings` 重複あり（tenant `102`: `104502` 11件 / `409670` 2件）。抽出シート：[prd](https://docs.google.com/spreadsheets/d/1FX7KDjR11VVgxFxtvJ1VckRzLZfzC5vJMc1JV8i6oOU/edit?gid=1291569985#gid=1291569985) / [ops](https://docs.google.com/spreadsheets/d/1FX7KDjR11VVgxFxtvJ1VckRzLZfzC5vJMc1JV8i6oOU/edit?gid=469875116#gid=469875116) |
| `409702` | `1` | 六町ハートクリニック | 東京都足立区六町3-5-23 5階503号 | `open` / `2` | 1件ヒット（手動）。新規届出待ち（2026/08/03開院） |
| `409705` | `1` | 築地駅前内科・婦人科クリニック | 東京都中央区築地2-12-8 大広ビル5階 | `open` / `2` | 1件ヒット（手動）。新規届出待ち（2026/08/03開院） |
| `409723` | `1` | おもと内科・リウマチ膠原病クリニック | 滋賀県大津市粟津町３−２ JR 石山駅 NKビル | `open` / `2` | 1件ヒット（手動）。新規届出待ち（2026/07/07開院） |
| `409724` | `1` | しもおか脳神経外科 | 大阪府松原市新堂4丁目1154イオンタウン松原1階 | `open` / `2` | 1件ヒット（手動）。新規届出待ち（2026/09/01開院） |
| `409725` | `1` | なるおリハビリペインクリニック | 大阪府羽曳野市白鳥1丁目4-15 白鳥メディカルビル 4階 | `open` / `2` | 1件ヒット。新規届出待ち（2026/09/01開院） |
| `409851` | `1` | 中野みらい矯正歯科 | 東京都中野区中野2-13-23 岡デンタルビル1F | `open` / `3` | 住所で2件ヒット（`409851`:コード1、`147055`:コード0）。2021/07「岡歯科医院」継承開院から5年届出なし・HP保険記載なし → **自費診療のみと判断** |
| `409858` | `1` | 登戸産婦人科 | 神奈川県川崎市多摩区登戸１６８０−１ | `open` / `2` | 1件ヒット。新規届出待ち（2026/08/01開院） |
| `409859` | `1` | センター北循環器・内科NCクリニック | 神奈川県横浜市都筑区中川中央1丁目1番3号 1階 | `open` / `2` | （個別確認結果の記載なし） |
| `409860` | `1` | ホームクリニック西橋本 | 神奈川県相模原市緑区西橋本5-4-3 | `open` / `2` | 1件ヒット。新規届出待ち（2026/08/01開院） |
| `409912` | `1` | うえだ在宅ケアクリニック | 神奈川県川崎市麻生区上麻生6-39-31 コートレット1F | `open` / `2` | 2件ヒット。`410042` が正。foro_prdのみ・重複なし（[シート](https://docs.google.com/spreadsheets/d/1FX7KDjR11VVgxFxtvJ1VckRzLZfzC5vJMc1JV8i6oOU/edit?gid=1699243347#gid=1699243347)）。**`409912` アーカイブのみで完了** |

---

### 5-4. アーカイブ医療機関マッピング修正（2026/10/14）
- **背景**：医療機関アーカイブ時の `medical_institution_mappings` 修正漏れにより、アーカイブ済IDへの紐付け時にエラーが発生する。定期的に洗い出して Data Mgmt Admin から正規IDへ更新する。
- **手順と事例（テナント `135`：船橋中央病院）**：
  1. Redash（`https://data-mgmt.medup.jp/queries/2278`）でアーカイブ医療機関に紐づいているマッピング一覧を表示。
  2. Data Mgmt Admin で対象テナント（`135`）を開き、`医療機関` タブで該当ID（`120718`）を絞り込み確認。
  3. MIS Admin で「船橋中央病院」を検索し、`120718` がアーカイブ済・正規IDが `20769` であることを特定。
  4. Data Mgmt Admin で `120718` → `20769` に更新し、正しい名称・住所が表示されることを確認。

---

### 5-5. MIS診療科更新（マッピング修正・全件更新機能追加）
- **リンク**：Slack（`https://medup.slack.com/archives/C036KMJT06L/p1784876169135069?thread_ts=1784865476.724569&cid=C036KMJT06L`） / 管理画面（`https://admin-web.medup-core-service.com/department-update`） / リリースノート（2026/08/17：`https://app.notion.com/p/3bf1da5a7f3b80c4a402c1ab3a8cf1c0?pvs=21`）
- **事象・原因**：元データに「糖尿病内科」があるのにforo CRM／連携マップ上で非表示・検索不可だった。原因は `mis_prd.department_mappings`（`id = 252b65de-44e1-40d9-bfd3-6dcc234f9d22`）で「糖尿病内科」が「消化器内科」に誤マッピングされていたこと（修正済）。
- **機能改修（2026/08/17リリース）**：従来の「最新状態に更新」は前回実行日時以降の差分のみ対象だったため、既存ボタンとは別に**「全件最新状態に更新」ボタン**を新設（対象ステータス：`open`, `suspend`。「最新状態を確認」実行後に押下可能）。

---

### 5-6. MISデータ調査・referral紐付け修正実績（2026/08/19）
- **概要**：DataOpsクレンジングミスによる不整合をDB直接更新で修正。
- **リンク**：Slack（`https://medup.slack.com/archives/C06HQ2TLJKF/p1787115867985779`） / TBL関連図Figma（`https://www.figma.com/board/JsNLqHrflfzMpqle1P53hd/%E7%B4%B9%E4%BB%8B%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AE%E6%B5%81%E3%82%8C?node-id=3-19&t=b0XKRQBSXjOnQBzH-0`）
- **対象**：`data_ops.referral_masters` および `foro_prd.referrals`（`tenant_id = 23`、`'四谷内科・内視鏡クリニック'`、誤 `23197` → 正 `229401`、計73件）

```sql
-- 1. 確認 & 対象 referral_control_id 抽出（data_ops）
SELECT rm.* FROM public.referral_masters AS rm
WHERE rm.tenant_id = 23 AND rm.medical_institution_id_from = 23197 AND rm.received_medical_institution_name = '四谷内科・内視鏡クリニック';

SELECT string_agg(quote_literal(rm.referral_control_id::text), ', ' ORDER BY rm.referral_control_id)
FROM public.referral_masters AS rm
WHERE rm.tenant_id = 23 AND rm.medical_institution_id_from = 23197 AND rm.received_medical_institution_name = '四谷内科・内視鏡クリニック';

-- 2. data_ops.referral_masters 更新
BEGIN;
UPDATE public.referral_masters AS rm
SET medical_institution_id_from = 229401, updated_at = NOW()
WHERE rm.tenant_id = 23 AND rm.medical_institution_id_from = 23197 AND rm.received_medical_institution_name = '四谷内科・内視鏡クリニック';

SELECT COUNT(*) AS remaining_wrong FROM public.referral_masters AS rm
WHERE rm.tenant_id = 23 AND rm.medical_institution_id_from = 23197 AND rm.received_medical_institution_name = '四谷内科・内視鏡クリニック'; -- 期待: 0
SELECT COUNT(*) AS fixed_count FROM public.referral_masters AS rm
WHERE rm.tenant_id = 23 AND rm.medical_institution_id_from = 229401 AND rm.received_medical_institution_name = '四谷内科・内視鏡クリニック'; -- 期待: 73
COMMIT; -- おかしければ ROLLBACK;

-- 3. foro_prd.referrals 確認 & 更新
SELECT r.id, r.referral_control_id, r.patient_id, r.referral_date, r.medical_institution_id_from, r.medical_institution_id_to, r.updated_at
FROM public.referrals AS r
WHERE r.tenant_id = 23 AND r.medical_institution_id_from = 23197
  AND r.referral_control_id IN (
    '99742', '100906', '101381', '101646', '102501', '103079', '103080', '103725', '104063', '106907',
    '107022', '107361', '108367', '108524', '108809', '108928', '109540', '109842', '114929', '115358',
    '116586', '116609', '118616', '120343', '120492', '120535', '121008', '122801', '124044', '125730',
    '127898', '129461', '131102', '132079', '132972', '133775', '137259', '140293', '140844', '146790',
    '147104', '147340', '149574', '150665', '151133', '151694', '152393', '152401', '154716', '155317',
    '158799', '158831', '159841', '160082', '160315', '160806', '160852', '161591', '164128', '166212',
    '166776', '167759', '168523', '173887', '174394', '175286', '175986', '176969', '177048', '177052',
    '177364', '182860', '184935'
  )
ORDER BY r.referral_date DESC, r.id;

BEGIN;
UPDATE public.referrals AS r
SET medical_institution_id_from = 229401, updated_at = NOW()
WHERE r.tenant_id = 23 AND r.medical_institution_id_from = 23197
  AND r.referral_control_id IN (
    '99742', '100906', '101381', '101646', '102501', '103079', '103080', '103725', '104063', '106907',
    '107022', '107361', '108367', '108524', '108809', '108928', '109540', '109842', '114929', '115358',
    '116586', '116609', '118616', '120343', '120492', '120535', '121008', '122801', '124044', '125730',
    '127898', '129461', '131102', '132079', '132972', '133775', '137259', '140293', '140844', '146790',
    '147104', '147340', '149574', '150665', '151133', '151694', '152393', '152401', '154716', '155317',
    '158799', '158831', '159841', '160082', '160315', '160806', '160852', '161591', '164128', '166212',
    '166776', '167759', '168523', '173887', '174394', '175286', '175986', '176969', '177048', '177052',
    '177364', '182860', '184935'
  );

SELECT COUNT(*) AS remaining_wrong FROM public.referrals AS r
WHERE r.tenant_id = 23 AND r.medical_institution_id_from = 23197
  AND r.referral_control_id IN (
    '99742', '100906', '101381', '101646', '102501', '103079', '103080', '103725', '104063', '106907',
    '107022', '107361', '108367', '108524', '108809', '108928', '109540', '109842', '114929', '115358',
    '116586', '116609', '118616', '120343', '120492', '120535', '121008', '122801', '124044', '125730',
    '127898', '129461', '131102', '132079', '132972', '133775', '137259', '140293', '140844', '146790',
    '147104', '147340', '149574', '150665', '151133', '151694', '152393', '152401', '154716', '155317',
    '158799', '158831', '159841', '160082', '160315', '160806', '160852', '161591', '164128', '166212',
    '166776', '167759', '168523', '173887', '174394', '175286', '175986', '176969', '177048', '177052',
    '177364', '182860', '184935'
  ); -- 期待: 0
COMMIT; -- おかしければ ROLLBACK;
```

---

## 6. 通知設定・その他定例マスタ運用

### 6-1. データ反映完了メール宛先変更（2026/10/01 実績・SOP）
- **目的**：データ投入時の作業完了通知メール送信先を更新する月次定例業務（特に3月・4月の年度替わりは依頼急増）。
- **対象DB / テーブル**：`data_ops_prd` の `public.completion_notification_users`（Prisma管理下）。個人情報（PII）を含むためBigQuery非同期。SSHトンネル経由でDBeaver等から本番PostgreSQLへ直接接続。
- **リンク**：依頼Slack（`https://medup.slack.com/archives/C02NHN8KM6F/p1790299374131099`） / CS送信先シート（`https://docs.google.com/spreadsheets/d/171mV_B6Pyqyv1t3Y1UEex-9pf-t8VCkNJIq_egBnjg0/edit?gid=0#gid=0`） / Circleback（`https://circleback.ai/ja/meetings/lw1pqBcqIvOTnu5PwkEog?tab=notes`）

#### テーブル仕様・制約・バグ回避策
| カラム名 | 型・内容 | 備考・制約 |
| --- | --- | --- |
| `id` | UUID | UNIQUE / NOT NULL。INSERT時は `gen_random_uuid()` |
| `tenant_id` | テナントID | NOT NULL |
| `name` | 氏名 | NOT NULL |
| `honorific` | 敬称（例：`様`） | NOT NULL |
| `email` | メールアドレス | NOT NULL。末尾の空白は除去する |
| `enabled_notification` | boolean | NOT NULL。`false` が論理削除、新規は `true`（※SOP内では `enable_notification` 表記揺れあり） |
| `last_sent_at` | timestamp | **NOT NULL**。INSERT時は `CURRENT_TIMESTAMP` を明示指定（**NULLバグ回避**：NULLだと通知配信バッチや描画コンポーネントで例外が発生し配信停止する既存バグあり） |
| `created_at` / `updated_at` | timestamp | NOT NULL。INSERT時は `CURRENT_TIMESTAMP` |
| `"order"` | 宛先順 | NOT NULL。PostgreSQL予約語のため必ずダブルクォート `"order"` で記述 |

- **制約・運用ルール**：
  - 全項目 `NOT NULL`、`(tenant_id, email)` ユニーク制約（`completion_notification_users_tenant_id_email_key`）。異テナント間の同一アドレスは許容。
  - **論理削除厳守**：物理削除（`DELETE`）は厳禁（CS問合せの追跡可能性と障害時リカバリのため）。
  - **一次情報保全**：Redash出力の論理削除対象IDリストは監査用マスターデータのため編集・削除・上書き保存厳禁。

#### 事前クレンジング・実行SQL・トラブルシューティング
- **2026/10/01 実績**：
  - 論理削除：33件。
  - 新規追加：元データ54件 → シート内重複除外（tenant `167` `y-hase@teikyo-u.ac.jp` の「長谷山 / order=2」※SOP内「初山様」表記）で53件 → 既存DB重複（tenant `143` 藤本美生 `Mio_Fujimoto@pref.hyogo.lg.jp`、ユニーク違反 `23505`・確定リスト取り下げ）を除外／`DO NOTHING` でスキップし **52件** 投入。
- **同一タブ実行の必須性**：PostgreSQL（Read Committed）の仕様上、未コミットの変更は同一セッションからしか見えない。検証用 `SELECT` は必ず `UPDATE` と同一DBeaverタブ内で実行する。
- **トラブルシューティング**：
  - エラーで `current transaction is aborted` になったら直ちに `ROLLBACK;` してリセットし、`BEGIN;` からやり直す。
  - DBeaverでテキストを部分選択したまま実行すると選択範囲のみ送信され構文エラーになるため、未選択または全選択で実行する。`VALUES` 末尾等の閉じ括弧 `)` 欠落に注意。
- **改善展望**：3月・4月繁忙期の工数・ミス削減のため、管理コンソールへのCSVアップロード＋自動バリデーションによるセルフサービスUI化が望まれる。

```sql
-- 事前確認（既存DBとの重複チェック）
SELECT tenant_id, email, "name", "order", enabled_notification
FROM public.completion_notification_users
WHERE (tenant_id, email) IN ((143, 'example@example.com'));

-- 作業1：論理削除（同一タブで件数確認後にCOMMIT）
BEGIN;
UPDATE public.completion_notification_users
SET enabled_notification = false
WHERE id IN ('00000000-0000-0000-0000-000000000000');

SELECT id, enabled_notification
FROM public.completion_notification_users
WHERE id IN ('00000000-0000-0000-0000-000000000000');
-- COMMIT; / ROLLBACK;

-- 作業2：新規追加（既存行スキップ時は ON CONFLICT ... DO NOTHING。新規分は created_at で確認）
BEGIN;
INSERT INTO public.completion_notification_users (
  id, tenant_id, "name", honorific, email, enabled_notification,
  last_sent_at, created_at, updated_at, "order"
)
VALUES
  (gen_random_uuid(), 0, '氏名', '様', 'example@example.com', true, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP, 1)
ON CONFLICT (tenant_id, email) DO NOTHING;

SELECT tenant_id, "name", honorific, email, "order", enabled_notification, created_at
FROM public.completion_notification_users
WHERE (tenant_id, email) IN ((0, 'example@example.com'))
ORDER BY tenant_id, "order";
-- COMMIT; / ROLLBACK;
```