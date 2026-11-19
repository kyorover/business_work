# Figma制作プロンプト｜確定DPC 9-Step：概念的処理とデータの由来

## 作成する図

編集可能なFigma Designの依存図を作成してください。`dpc-dependencies-explicit.pdf`を構成の基準とし、処理関数名を概念的な処理に置き換え、④にマスタの元データまでの依存関係を追加します。この文書だけで作成できるよう、内容を以下に収録しています。

## レイアウト

Step1〜9を左から右へ横一列に配置します。各列は以下の5段です。

1. ① Step名・対象桁・概要
2. ② Step全体のIN / OUT（モデル名・候補型・フィールド）
3. ③ 概念的な処理の流れ（各処理のIN / OUT、機械的処理かLLM処理かを明示）
4. ④ 参照データと由来（ユーザーデータ／マスタを区分し、マスタ ← 元データ ← さらに元データまで描く）
5. ⑤ 手書き補足欄（未確認の出典・運用上の生成元を記入する空白）

③は各Step内で上から下へ処理カードを配置します。④は左から「利用マスタ」「直接の元データ」「元ファイル」の列を取り、右から左へ依存矢印を引きます。Repositoryは③と④をつなぐ小さな参照ノードとして残します。処理関数・メソッドの実装名を主役にしないでください。生成関数、生成スクリプト、変換ジョブはノードにしません。

列幅1,400px程度を開始点にし、必要なら拡張してください。全体を画面幅に押し込まず、拡大して各Stepを読める大きさを保ちます。本文20〜24px、識別子18〜20px、余白24〜40pxを目安にし、文字に合わせてノードを拡張してください。見出し・各段上端・手書き欄の位置を揃え、短い列には余白を残してください。

## 表現ルール

- 実線矢印：処理と候補の受け渡し。Step間は左→右、Step内は上→下。
- 点線矢印：参照・データ由来。元データ→マスタ→Repository→概念処理。接続ラベルで「元データ」「実行時参照」を区別する。
- ④の例：`DPC6分類マスタ ← 分類名称シート ← DPC電子点数表Excel`。シート→ファイルの関係は処理工程ではなくデータの包含・由来である。
- 複数の元データは並列に分岐させる。「A＋B」はA→Bという直列依存にしない。
- 同じデータはStepごとに再掲してよい。同じStep内では同一ノードを共用できるが、長い共通バスや交差線は避ける。
- データノードは日本語名と正確なテーブル名／シート名／ファイル名を併記する。長いパスはノード下の補足に移してよい。
- ③のカードには「機械的」または「LLM」の文字バッジを付ける。LLMは文脈に基づく抽出・推定・批評で、結果に不確実性がある。ルールによる検証や照合とは分ける。
- 処理カードに実装関数名は掲載しない。データ型・モデル名はI/O補足として残す。
- 条件分岐・再試行は対応するカードへ接続する。単に全処理が必ず順番に実行されるように描かない。
- マスタ紫、ユーザーデータ緑、元ファイル灰青、機械的処理淡青、LLM処理淡黄。Step1〜3の見出し青、4〜6緑青、7〜9紫。色に加えてラベルでも識別する。
- テキスト・ノード・線を個別編集可能にする。画像1枚の貼り付けで完了しない。

## 確認範囲と情報の扱い

StepのI/Oと概念処理は既存の実装確認資料を基準とします（ai-dpc-multi-agent、基準PDF記載HEAD e6f16da）。マスタの由来は2026-11-11に確認したai-dpc-branch-masterのローカル生成実装（HEAD ad93025）に基づきます。両者は同一リリースとしての整合を検証したものではありません。

生成実装は原資料からメモリ上で分岐IDやコード一覧を作り直す場合があります。既存Parquetを読むという意味のマスタ→マスタ依存を推測で追加しないでください。元データのさらに上流が実装から分からない場合は「上流出典未確認」で止めます。
※ 2026/11/11の引き継ぎ会議にて共有された仕様・課題（候補数の上限、レセプト電算コードへの移行検討、ダミーコードの手動調査など）を補足として追記しています。

## 各Stepの内容

### Step1｜主病名候補｜1〜6桁

概要：ICD-10抽出 → DPC6対応付け。LLM＋マスタ検索。
※ 約10万件のICD-10マスタから検索し、N対1の対応表で機械的にDPC6桁へ変換。候補は最大5件に絞られ、それぞれが最後まで独立して後続Stepへ渡される。

**② IN**

- 入力モデル：`PatientContext`。
- `patient_id`、`admission_date`。`discharge_date` / `age` は省略可。
- 依存引数：`Step1Deps`。

**OUT**

- 出力モデル：`Dpc6CandidatesOutput`。
- 候補型：`DiagnosisCandidate`。
- 格納形式：`candidates: list[DiagnosisCandidate]`。

掲載フィールド・補足（字下げ行は直前の項目の内訳）：

```text
病名・ICD-10：diagnosis_name
icd10 / icd10_name
DPC6候補：dpc6_candidates[]
  Dpc6Candidate {dpc6, dpc6_name}
信頼度・主病名らしさ：
  confidence / primary_likelihood
根拠：evidence
他候補との比較：primary_vs_others
鑑別対象：differential_against
→ DPC6単位に展開・重複整理
→ list[Dpc6FlatCandidate] をStep2へ（最大5件）
```

**③ 概念的な処理の流れ**

| ID | 判定方式 | 処理 | IN | OUT | 条件・補足 |
|---|---|---|---|---|---|
| S1-1 | 機械的 | カルテ・EFの取得／入力確認 | PatientContext、患者ID・入退院日 | list\[EhrNoteDto\]、list\[EfProcedureCodeDto\] | カルテ0件はInsufficientInputError。マスタ検索ツールも構築。 |
| S1-2 | LLM | 主病名・ICD-10候補を推定 | カルテ・EF・患者情報、検索ツール／再試行時の指摘 | LlmDiagnosisOutput、病名・ICD-10・DPC6候補、信頼度・根拠 | LLMが必要に応じて検索ツールを呼ぶ。構造化出力失敗も再試行対象。 |
| S1-3 | 機械的 | LLM出力の形式・内容を検証 | LlmDiagnosisOutput | 検証済みの候補、またはStep1ValidationError | 不正な候補等は再試行。LLM呼出は例外を含め最大3回。 |
| S1-4 | 機械的 | ICD-10を正式なDPC6に対応付け | 検証済みの病名候補、DPC6対応マスタ・患者年齢 | Dpc6CandidatesOutput、list\[DiagnosisCandidate\] | マスタで対応付けを再確認。対応なし候補の扱い・小児優先順を適用。 |
| S1-5 | 機械的 | 最終出力の根拠等を検証 | Dpc6CandidatesOutput | 検証済みStep1出力、またはStep1ValidationError | 不備があれば②へ戻る。上限到達時はエラー。 |
| S1-6 | 機械的 | Step2向けに展開・重複整理 | Dpc6CandidatesOutput | list\[Dpc6FlatCandidate\] | 同一DPC6はprimary\_likelihood、同値ならconfidenceで選択。Step間の処理として実行。 |

通常は番号順に実線接続。表内の丸数字はこのStep内の処理番号を指す。

- 再試行：S1-3 → S1-2。上限・採用条件は表の補足に従う。
- 再試行：S1-5 → S1-2。上限・採用条件は表の補足に従う。

**③と④をつなぐ実行時参照**

Repositoryのクラス・メソッドは参照経路の補足。左端は上記の概念処理IDとする。各データからRepositoryへ個別の点線を引く。

| 概念処理 | Repository / メソッド | 参照するデータ | 条件 |
|---|---|---|---|
| S1-1 | `DuckDbEhrNoteRepository.find_by_patient` | 電子カルテ `ehr_notes` | — |
| S1-1 | `DuckDbEfFileRepository.find_procedure_codes` | 診療行為明細 `ef_records` | — |
| S1-2 | `DuckDbIcd10Repository.find_candidates_by_code` | ICD-10病名マスタ `icd10_name_master`、ICD-10コードマスタ `icd10_code_master` | LLM検索ツール：コード検索時 |
| S1-2 | `DuckDbIcd10Repository.search_by_keyword` | ICD-10病名マスタ `icd10_name_master`、ICD-10コードマスタ `icd10_code_master` | LLM検索ツール：キーワード検索時 |
| S1-2 | `DuckDbDpc6Repository.find_by_icd10` | DPC6・ICD-10対応マスタ `dpc6_icd10_map`、DPC6分類マスタ `dpc6_master` | LLM検索ツール：ICD-10 → DPC6 |
| S1-2 | `DuckDbDpcBranchingRepository.find_surgery_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、手術分岐マスタ `dpc_surgery_branch` | LLM検索ツール：関連手術検索 |
| S1-2 | `DuckDbDpc6Repository.find_by_k_code` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、手術分岐マスタ `dpc_surgery_branch`、DPC6分類マスタ `dpc6_master` | LLM検索ツール：Kコード → DPC6 |
| S1-2 | `DuckDbIcd10CodingCautionRepository.find_by_icd10` | ICD-10コーディング留意事例 `icd10_coding_caution` | LLM検索ツール：留意事例（元データはPDFからの抽出） |
| S1-2 | `DuckDbDpc6CodingCautionRepository.find_by_dpc6` | DPC6コーディング留意事例 `dpc6_coding_caution` | LLM検索ツール：留意事例（元データはPDFからの抽出） |
| S1-4 | `DuckDbDpc6Repository.find_by_icd10` | DPC6・ICD-10対応マスタ `dpc6_icd10_map`、DPC6分類マスタ `dpc6_master` | LLM出力をプログラム側で対応付け |

④には上記参照データを置き、マスタには次の由来一覧を展開する。

### Step2｜病態等分類｜7桁目

概要：病態の分岐を判定。ルール＋必要時LLM。
※ 構造化された数値データがカルテに無いため、年齢以外はLLMがカルテから読み取ってルールベースに当てはめる。

**② IN**

- 入力モデル：`list[Dpc6FlatCandidate]`。
- Step1の出力をDPC6単位に展開・重複整理した候補群。
- `PatientContext`：患者ID・入退院日・年齢。
- 依存引数：`Step2Deps`。

**OUT**

- 出力モデル：`Step2Digit7Output`。
- 候補型：`Step2Digit7Candidate`。
- 格納形式：`candidates: list[Step2Digit7Candidate]`。

掲載フィールド・補足（字下げ行は直前の項目の内訳）：

```text
Dpc6FlatCandidateの情報を継承
  DPC6・病名・ICD-10・名称
  confidence / primary_likelihood
  evidence / evidence_excerpts
追加：7桁目 digit7
判定条件：digit7_condition
判断理由：digit7_reasoning
根拠引用：digit7_evidence[]
要確認：digit7_needs_review
```

**③ 概念的な処理の流れ**

| ID | 判定方式 | 処理 | IN | OUT | 条件・補足 |
|---|---|---|---|---|---|
| S2-1 | 機械的 | DPC6ごとの7桁目分岐を取得 | list\[Dpc6FlatCandidate\]、PatientContext | list\[Digit7BranchDto\] | 候補ごとに以下を実行。分岐なしはdigit7=Noneを保持。 |
| S2-2 | 機械的 | 単一分岐・年齢ルールを適用 | 分岐・判定ルール・年齢 | 暫定digit7・理由、肺炎追加判定の必要性 | 肺炎判定が不要なら⑦へ迂回。必要時だけカルテを取得。 |
| S2-3 | LLM | 肺炎種別を判定［必要時］ | カルテ・患者情報、候補ラベル・追加指摘 | LlmPneumoniaCategoryResult、市中／院内／判定不能、根拠・判断理由 | — |
| S2-4 | 機械的 | 根拠期間・ラベル等を検証 | 肺炎判定結果・入退院日、許容ラベル | list\[VerifyIssue\] | 根拠の有無・入院期間・結論とラベルの整合を規則で確認。 |
| S2-5 | LLM | 判定内容を別ロールで批評 | 肺炎判定結果・カルテ、患者情報・判定指示 | 追加のlist\[VerifyIssue\] | 意味・臨床文脈の確認にもLLMを用いる。 |
| S2-6 | 機械的 | 再試行制御・判定を分岐へ適用 | 判定結果・検証／批評の指摘、肺炎分岐 | digit7・reasoning、evidence・needs\_review | 指摘ありなら③へ1回再試行。指摘残留・判定不能も結果に反映。 |
| S2-7 | 機械的 | 候補へ7桁目を追加して返す | 元のDPC6候補、digit7と判定情報 | Step2Digit7Output、candidates\[\] | — |

通常は番号順に実線接続。表内の丸数字はこのStep内の処理番号を指す。

- 条件付き迂回：S2-2 → S2-7（肺炎不要）。
- 再試行：S2-6 → S2-3。上限・採用条件は表の補足に従う。

**③と④をつなぐ実行時参照**

Repositoryのクラス・メソッドは参照経路の補足。左端は上記の概念処理IDとする。各データからRepositoryへ個別の点線を引く。

| 概念処理 | Repository / メソッド | 参照するデータ | 条件 |
|---|---|---|---|
| S2-1 | `DuckDbDpcBranchingRepository.find_digit7_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、病態等分類判定ルール `dpc_digit7_rule` | — |
| S2-3 | `DuckDbEhrNoteRepository.find_by_patient` | 電子カルテ `ehr_notes` | 肺炎の分類が必要なときに取得 |

④には上記参照データを置き、マスタには次の由来一覧を展開する。

### Step3｜年齢・出生時体重等｜8桁目

概要：抽出した数値を条件照合。ルール＋必要時LLM。

**② IN**

- 入力モデル：`Step2Digit7Output`。
- `candidates[]`：前段までの候補情報。
- `PatientContext`：患者ID・入退院日・年齢。
- 依存引数：`Step3Deps`。

**OUT**

- 出力モデル：`Step3Digit8Output`。
- 候補型：`Step3Digit8Candidate`。
- 格納形式：`candidates: list[Step3Digit8Candidate]`。

掲載フィールド・補足（字下げ行は直前の項目の内訳）：

```text
Step2Digit7Candidateを継承
追加：8桁目 digit8
判定条件：digit8_condition
判断理由：digit8_reasoning
根拠引用：digit8_evidence[]
要確認：digit8_needs_review
情報不足時は仮値・要確認を保持
```

**③ 概念的な処理の流れ**

| ID | 判定方式 | 処理 | IN | OUT | 条件・補足 |
|---|---|---|---|---|---|
| S3-1 | 機械的 | カルテ・8桁目分岐を取得 | Step2Digit7Output、PatientContext | list\[Digit8BranchDto\]、カルテ・候補情報 | digit7未確定なら評価を省略し、digit8=xで出力へ。 |
| S3-2 | 機械的 | 条件種別を判別／年齢を直接使用 | 分岐・トリガー、patient.age | condition\_type、年齢なら数値を確定 | 年齢は⑦へ。分岐・条件なし等は仮値／xのルールを適用。 |
| S3-3 | LLM | カルテから条件値を抽出 | JCS・出生時体重等の抽出指示、カルテ・患者情報 | ClinicalValueResult、raw\_value・unit・evidence | 年齢以外で必要な条件値を抽出。 |
| S3-4 | 機械的 | 抽出値の根拠・期間等を検証 | ClinicalValueResult、患者の入退院日 | list\[VerifyIssue\] | — |
| S3-5 | LLM | 抽出内容を別ロールで批評 | 抽出値・カルテ・患者情報 | 追加のlist\[VerifyIssue\] | — |
| S3-6 | 機械的 | 再試行制御・数値の正規化 | raw\_value・unit、検証／批評の指摘 | 正規化した数値またはNone、根拠・残留指摘 | 指摘ありなら③へ1回再試行。kg→g、週数等を機械的に正規化。 |
| S3-7 | 機械的 | 数値と分類範囲を照合 | 正規化値または年齢、range\_lower / range\_upper | digit8・reasoning、evidence・needs\_review | 下限≦値＜上限。値不足や指摘残留は仮値・要確認に反映。 |
| S3-8 | 機械的 | 候補へ8桁目を追加して返す | Step2候補・8桁目判定情報 | Step3Digit8Output、candidates\[\] | — |

通常は番号順に実線接続。表内の丸数字はこのStep内の処理番号を指す。

- 条件付き迂回：S3-2 → S3-7（年齢）。
- 再試行：S3-6 → S3-3。上限・採用条件は表の補足に従う。

**③と④をつなぐ実行時参照**

Repositoryのクラス・メソッドは参照経路の補足。左端は上記の概念処理IDとする。各データからRepositoryへ個別の点線を引く。

| 概念処理 | Repository / メソッド | 参照するデータ | 条件 |
|---|---|---|---|
| S3-1 | `DuckDbDpcBranchingRepository.find_digit8_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、年齢等判定条件マスタ `dpc_digit8_trigger` | — |
| S3-1 | `DuckDbEhrNoteRepository.find_by_patient` | 電子カルテ `ehr_notes` | — |

④には上記参照データを置き、マスタには次の由来一覧を展開する。

### Step4｜手術｜9〜10桁目

概要：必須・除外Kコードを照合。ルール判定（LLMなし）。
※ 運用課題：現在はEFファイルのKコードで完全一致判定しているが、病院側が使用する「レセプト電算コード」へマスタを寄せて分岐する改善案が合意されている。

**② IN**

- 入力モデル：`Step3Digit8Output`。
- `candidates[]`：前段までの候補情報。
- `PatientContext`：患者ID・入退院日・年齢。
- 依存引数：`Step4Deps`。

**OUT**

- 出力モデル：`Step4SurgeryOutput`。
- 候補型：`Step4SurgeryCandidate`。
- 格納形式：`candidates: list[Step4SurgeryCandidate]`。

掲載フィールド・補足（字下げ行は直前の項目の内訳）：

```text
Step3Digit8Candidateを継承
追加：9〜10桁目 surgery
判断理由：surgery_reasoning
一致Kコード：matched_k_codes[]
  list[MatchedKCode]
  code：Kコード
  name：手術名
```

**③ 概念的な処理の流れ**

| ID | 判定方式 | 処理 | IN | OUT | 条件・補足 |
|---|---|---|---|---|---|
| S4-1 | 機械的 | EFの手術Kコードを取得 | PatientContext・EF明細 | list\[MatchedKCode\]、code・name | Repositoryで対象の診療区分や除外コードを絞り込む。 |
| S4-2 | 機械的 | 対象候補の手術分岐を取得 | Step3候補、DPC6・digit7・digit8 | list\[SurgeryBranchDto\] | digit7/8が未確定なら検索キーにxを用いる。 |
| S4-3 | 機械的 | 必須・除外Kコードを照合 | 手術分岐・実施Kコード | SurgeryEvalResult またはNone、手術値・理由・一致Kコード | LLMなし。条件と優先順位で機械的に判定。 |
| S4-4 | 機械的 | 手術情報を候補に追加して返す | Step3候補・照合結果 | Step4SurgeryOutput、surgery / matched\_k\_codes等 | 分岐なし・該当なしはsurgery=None。 |

通常は番号順に実線接続。表内の丸数字はこのStep内の処理番号を指す。

**③と④をつなぐ実行時参照**

Repositoryのクラス・メソッドは参照経路の補足。左端は上記の概念処理IDとする。各データからRepositoryへ個別の点線を引く。

| 概念処理 | Repository / メソッド | 参照するデータ | 条件 |
|---|---|---|---|
| S4-2 | `DuckDbDpcBranchingRepository.find_surgery_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、手術分岐マスタ `dpc_surgery_branch` | — |
| S4-1 | `DuckDbEfFileRepository.find_k_codes` | 診療行為明細 `ef_records` | — |

④には上記参照データを置き、マスタには次の由来一覧を展開する。

### Step5｜手術・処置等1｜11桁目

概要：処置の実施条件を照合。ルール判定（LLMなし）。
※ 化学療法などのダミーコードは、成分（YJコード等）や個別定義を手作業で調べて分解・マスター化している。

**② IN**

- 入力モデル：`Step4SurgeryOutput`。
- `candidates[]`：前段までの候補情報。
- `PatientContext`：患者ID・入退院日・年齢。
- 依存引数：`ProcedureDeps`。

**OUT**

- 出力モデル：`Step5Procedure1Output`。
- 候補型：`Step5Procedure1Candidate`。
- 格納形式：`candidates: list[Step5Procedure1Candidate]`。

掲載フィールド・補足（字下げ行は直前の項目の内訳）：

```text
Step4SurgeryCandidateを継承
追加：11桁目 procedure1
判断理由：procedure1_reasoning
照合結果：procedure1_candidates[]
  list[ProcedureMatch]
  value / trigger_codes
  matched_ef_codes / is_selected
確認理由：procedure1_review_reasons[]
  list[ProcedureReviewReason]
要確認：procedure1_needs_review
  確認理由の有無から算出
```

**③ 概念的な処理の流れ**

| ID | 判定方式 | 処理 | IN | OUT | 条件・補足 |
|---|---|---|---|---|---|
| S5-1 | 機械的 | EF取得・対象明細の集約 | PatientContext・EF明細 | ProcedureCodePools、本体／全有効明細のコード集合 | 手術区分50の薬剤を除外し、receipt\_code単位で実施日を集約。 |
| S5-2 | 機械的 | 前段確認・分岐と対応表を準備 | Step4候補・ダミーコード対応表、事前読込のDefinitionCodeResolver | 処置分岐・trigger\_codes、判定に使う対応表・判定器 | 前段の手術・必要な処置桁が未確定なら値をNoneのまま出力。 |
| S5-3 | 機械的 | トリガーのコード解決・条件照合 | トリガー・対応表・定義コード、ProcedureCodePools | 分岐別の一致EF・実施日、list\[ProcedureMatch\] | 通常／ダミー／定義コードと必須組合せを規則で照合。LLMなし。 |
| S5-4 | 機械的 | 一致分岐から値・根拠を選択 | 全分岐の照合結果、規定の分岐順 | selected\_value・reasoning、all\_matches（is\_selected付） | 分岐順の下位を優先。全xならx、該当なしなら0。 |
| S5-5 | 機械的 | 処置値・確認理由を候補に追加 | 前段候補・選択値・照合結果 | Step5Procedure1Output、procedure1\_review\_reasons等 | 選択された分岐の確認理由を保存し、確認要否を算出。 |

通常は番号順に実線接続。表内の丸数字はこのStep内の処理番号を指す。

**③と④をつなぐ実行時参照**

Repositoryのクラス・メソッドは参照経路の補足。左端は上記の概念処理IDとする。各データからRepositoryへ個別の点線を引く。

| 概念処理 | Repository / メソッド | 参照するデータ | 条件 |
|---|---|---|---|
| S5-2 | `DuckDbDpcBranchingRepository.find_procedure1_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、手術分岐マスタ `dpc_surgery_branch`、手術・処置等1分岐マスタ `dpc_procedure1_branch`、手術・処置等1判定条件マスタ `dpc_procedure1_trigger` | — |
| S5-1 | `DuckDbEfFileRepository.find_procedure_codes` | 診療行為明細 `ef_records` | 処置コードを取得 |
| S5-1 | `DuckDbEfFileRepository.find_all_records` | 診療行為明細 `ef_records` | 全有効明細を取得 |
| S5-2 | `DuckDbDummyCodeRepository.get_all_mappings` | ダミーコード対応マスタ `dummy_code_mapping` | — |
| S5-2 | `DuckDbDefinitionCodeRepository.get_all` | 定義コード対応・評価ルールマスタ `dpc_definition_code_mapping` | 共通の事前読込。Step5・6では受け取った判定器を使用（毎Stepで再読込しない） |

④には上記参照データを置き、マスタには次の由来一覧を展開する。

### Step6｜手術・処置等2｜12桁目

概要：処置の実施条件を照合。ルール判定（LLMなし）。

**② IN**

- 入力モデル：`Step5Procedure1Output`。
- `candidates[]`：前段までの候補情報。
- `PatientContext`：患者ID・入退院日・年齢。
- 依存引数：`ProcedureDeps`。

**OUT**

- 出力モデル：`Step6Procedure2Output`。
- 候補型：`Step6Procedure2Candidate`。
- 格納形式：`candidates: list[Step6Procedure2Candidate]`。

掲載フィールド・補足（字下げ行は直前の項目の内訳）：

```text
Step5Procedure1Candidateを継承
追加：12桁目 procedure2
判断理由：procedure2_reasoning
照合結果：procedure2_candidates[]
  list[ProcedureMatch]
  value / trigger_codes
  matched_ef_codes / is_selected
確認理由：procedure2_review_reasons[]
  list[ProcedureReviewReason]
要確認：procedure2_needs_review
  確認理由の有無から算出
```

**③ 概念的な処理の流れ**

| ID | 判定方式 | 処理 | IN | OUT | 条件・補足 |
|---|---|---|---|---|---|
| S6-1 | 機械的 | EF取得・対象明細の集約 | PatientContext・EF明細 | ProcedureCodePools、本体／全有効明細のコード集合 | 手術区分50の薬剤を除外し、receipt\_code単位で実施日を集約。 |
| S6-2 | 機械的 | 前段確認・分岐と対応表を準備 | Step5候補・ダミーコード対応表、事前読込のDefinitionCodeResolver | 処置分岐・trigger\_codes、判定に使う対応表・判定器 | 前段の手術・必要な処置桁が未確定なら値をNoneのまま出力。 |
| S6-3 | 機械的 | トリガーのコード解決・条件照合 | トリガー・対応表・定義コード、ProcedureCodePools | 分岐別の一致EF・実施日、list\[ProcedureMatch\] | 通常／ダミー／定義コードと必須組合せを規則で照合。LLMなし。 |
| S6-4 | 機械的 | 一致分岐から値・根拠を選択 | 全分岐の照合結果、規定の分岐順 | selected\_value・reasoning、all\_matches（is\_selected付） | 分岐順の下位を優先。全xならx、該当なしなら0。 |
| S6-5 | 機械的 | 処置値・確認理由を候補に追加 | 前段候補・選択値・照合結果 | Step6Procedure2Output、procedure2\_review\_reasons等 | 選択された分岐の確認理由を保存し、確認要否を算出。 |

通常は番号順に実線接続。表内の丸数字はこのStep内の処理番号を指す。

**③と④をつなぐ実行時参照**

Repositoryのクラス・メソッドは参照経路の補足。左端は上記の概念処理IDとする。各データからRepositoryへ個別の点線を引く。

| 概念処理 | Repository / メソッド | 参照するデータ | 条件 |
|---|---|---|---|
| S6-2 | `DuckDbDpcBranchingRepository.find_procedure2_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、手術分岐マスタ `dpc_surgery_branch`、手術・処置等1分岐マスタ `dpc_procedure1_branch`、手術・処置等2分岐マスタ `dpc_procedure2_branch`、手術・処置等2判定条件マスタ `dpc_procedure2_trigger` | — |
| S6-1 | `DuckDbEfFileRepository.find_procedure_codes` | 診療行為明細 `ef_records` | 処置コードを取得 |
| S6-1 | `DuckDbEfFileRepository.find_all_records` | 診療行為明細 `ef_records` | 全有効明細を取得 |
| S6-2 | `DuckDbDummyCodeRepository.get_all_mappings` | ダミーコード対応マスタ `dummy_code_mapping` | — |
| S6-2 | `DuckDbDefinitionCodeRepository.get_all` | 定義コード対応・評価ルールマスタ `dpc_definition_code_mapping` | 共通の事前読込。Step5・6では受け取った判定器を使用（毎Stepで再読込しない） |

④には上記参照データを置き、マスタには次の由来一覧を展開する。

### Step7｜副傷病｜13桁目

概要：LLM抽出＋EFから推定。ICD-10と条件照合。
※ 手術記録や放射線レポートなどの部門データは運用上未連携のケースが多く、現状はカルテ記事優先で判定する議論がある。

**② IN**

- 入力モデル：`Step6Procedure2Output`。
- `candidates[]`：前段までの候補情報。
- `PatientContext`：患者ID・入退院日・年齢。
- 依存引数：`Step7Deps`。

**OUT**

- 出力モデル：`Step7ComorbidityOutput`。
- 候補型：`Step7ComorbidityCandidate`。
- 格納形式：`candidates: list[Step7ComorbidityCandidate]`。

掲載フィールド・補足（字下げ行は直前の項目の内訳）：

```text
Step6Procedure2Candidateを継承
追加：13桁目 comorbidity
判断理由：comorbidity_reasoning
照合結果：comorbidity_matches[]
  list[ComorbidityMatch]
  trigger_code / matched_icd10
  source：カルテ・レポート・EF
  evidence / source_excerpts
  EF由来はef_code等も保持
```

**③ 概念的な処理の流れ**

| ID | 判定方式 | 処理 | IN | OUT | 条件・補足 |
|---|---|---|---|---|---|
| S7-1 | 機械的 | 資料・候補分岐・トリガーを準備 | Step6候補・患者情報、カルテ・レポート・EF・マスタ | 候補別分岐・共通トリガー、病名キーワード・患者資料 | 前段未確定／分岐なしの候補は副傷病Noneのまま出力。 |
| S7-2 | 機械的 | EF対応表から副傷病候補を推定 | EF処置コード、comorbidity\_ef\_mapping | ef\_hits: list\[FoundComorbidityHit\] | 名称は「推定」でも、対応表による機械的処理。LLMなし。 |
| S7-3 | LLM | カルテ・レポートから副傷病を探索 | 共通トリガー・病名キーワード、カルテ・放射線レポート | LlmComorbidityResult、該当病名・ICD-10・根拠 | トリガーをバッチ分割。資料またはトリガーなしなら⑦へ迂回。 |
| S7-4 | 機械的 | 副傷病の根拠を検証 | LLMの抽出結果・入退院日 | list\[VerifyIssue\] | 根拠の有無と入院期間を確認。 |
| S7-5 | LLM | 抽出内容を別ロールで批評 | 副傷病抽出結果・カルテ、患者情報・抽出指示 | 追加のlist\[VerifyIssue\] | — |
| S7-6 | 機械的 | 再試行制御・EF結果と統合 | LLM結果・検証指摘・ef\_hits | all\_hits = ef\_hits + llm\_hits、list\[FoundComorbidityHit\] | 指摘ありなら③へ1回再試行。残留指摘はログ、最後の結果をbest-effort採用。 |
| S7-7 | 機械的 | ICD-10と分岐条件を照合 | all\_hits・候補別の分岐 | comorbidity・reasoning、list\[ComorbidityMatch\] | 全xならx。トリガーなし／該当なしは0。出典・根拠も保持。 |
| S7-8 | 機械的 | 副傷病判定を候補へ追加 | Step6候補・副傷病判定 | Step7ComorbidityOutput、candidates\[\] | — |

通常は番号順に実線接続。表内の丸数字はこのStep内の処理番号を指す。

- 条件付き迂回：S7-2 → S7-7（LLM不要）。
- 再試行：S7-6 → S7-3。上限・採用条件は表の補足に従う。

**③と④をつなぐ実行時参照**

Repositoryのクラス・メソッドは参照経路の補足。左端は上記の概念処理IDとする。各データからRepositoryへ個別の点線を引く。

| 概念処理 | Repository / メソッド | 参照するデータ | 条件 |
|---|---|---|---|
| S7-1 | `DuckDbDpcBranchingRepository.find_comorbidity_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、手術分岐マスタ `dpc_surgery_branch`、手術・処置等1分岐マスタ `dpc_procedure1_branch`、手術・処置等2分岐マスタ `dpc_procedure2_branch`、副傷病分岐マスタ `dpc_comorbidity_branch`、副傷病判定条件マスタ `dpc_comorbidity_trigger` | — |
| S7-1 | `DuckDbEhrNoteRepository.find_by_patient` | 電子カルテ `ehr_notes` | — |
| S7-1 | `DuckDbDeptSystemReportRepository.find_by_patient` | 放射線レポート `dept_system_reports` | 実運用上は未連携のケースあり |
| S7-2 | `DuckDbEfFileRepository.find_procedure_codes` | 診療行為明細 `ef_records` | — |
| S7-2 | `DuckDbComorbidityEfMappingRepository.get_all_mappings` | EFコード・副傷病対応マスタ `comorbidity_ef_mapping` | — |
| S7-1 | `DuckDbIcd10Repository.find_by_prefix` | ICD-10病名マスタ `icd10_name_master`、ICD-10コードマスタ `icd10_code_master` | 副傷病トリガーの病名を補完 |

④には上記参照データを置き、マスタには次の由来一覧を展開する。

### Step8｜重症度等｜14桁目

概要：重症度を判定・DPC14組立。ルール＋必要時LLM。

**② IN**

- 入力モデル：`Step7ComorbidityOutput`。
- `candidates[]`：前段までの候補情報。
- `PatientContext`：患者ID・入退院日・年齢。
- 依存引数：`Step8Deps`。

**OUT**

- 出力モデル：`Step8SeverityOutput`。
- 候補型：`Step8SeverityCandidate`。
- 格納形式：`candidates: list[Step8SeverityCandidate]`。

掲載フィールド・補足（字下げ行は直前の項目の内訳）：

```text
Step7ComorbidityCandidateを継承
追加：14桁目 severity
完成コード：dpc14（未確定はNone）
判断理由：severity_reasoning
根拠：severity_evidence[]
  list[SevEvidence]
  date / excerpt / source
要確認：severity_needs_review
複数候補を維持してStep9へ渡す
```

**③ 概念的な処理の流れ**

| ID | 判定方式 | 処理 | IN | OUT | 条件・補足 |
|---|---|---|---|---|---|
| S8-1 | 機械的 | 前段確認・カルテ・分岐を取得 | Step7候補・PatientContext | list\[SeverityBranchDto\]、カルテ・判定コンテキスト | 前段未確定／分岐なしはseverity=None・dpc14=Noneで出力。 |
| S8-2 | 機械的 | 単一分岐・条件種別・年齢を処理 | 分岐条件・PatientContext | 選択済みseverity または、数値／LLMに渡す判定条件 | 年齢は⑦へ。単一分岐・全x等で値が決まれば⑧へ。 |
| S8-3 | LLM | 重症度の数値・ラベルを抽出 | カルテ・候補ラベル・患者情報、必要時の手術照合ツール | LlmSevExtractResult、numeric / label / evidence | — |
| S8-4 | 機械的 | ラベル・根拠・眼の手術条件を検証 | LLM結果・許容ラベル・入退院日、眼では手術分岐・EF | list\[VerifyIssue\] | ラベル正規化後に検証。眼の分類ではKコード算定実績も規則で確認。 |
| S8-5 | LLM | 重症度判定を別ロールで批評 | 重症度抽出結果・カルテ、判定指示・患者情報 | 追加のlist\[VerifyIssue\] | — |
| S8-6 | 機械的 | 再試行制御・値と要確認を確定 | 抽出結果・検証／批評の指摘 | SevColumnValue、numeric / label / needs\_review | 指摘ありなら③へ1回再試行。残留時は最終値を暫定採用＋要確認。 |
| S8-7 | 機械的 | 数値・ラベルと分類条件を照合 | SevColumnValue・分岐条件 | severity・reasoning、evidence・needs\_review | 複数一致は先頭採用。不一致等はデフォルト経路へ。 |
| S8-8 | 機械的 | 各桁を連結してDPC14を組立 | 1〜13桁＋severity | dpc14 またはNone | ここでは候補を1件に絞らない。 |
| S8-9 | 機械的 | 重症度・DPC14を候補へ追加 | Step7候補・重症度判定・DPC14 | Step8SeverityOutput、candidates\[\] | — |

通常は番号順に実線接続。表内の丸数字はこのStep内の処理番号を指す。

- 条件付き迂回：S8-2 → S8-7（年齢）。
- 条件付き迂回：S8-2 → S8-8（値確定）。
- 再試行：S8-6 → S8-3。上限・採用条件は表の補足に従う。

**③と④をつなぐ実行時参照**

Repositoryのクラス・メソッドは参照経路の補足。左端は上記の概念処理IDとする。各データからRepositoryへ個別の点線を引く。

| 概念処理 | Repository / メソッド | 参照するデータ | 条件 |
|---|---|---|---|
| S8-1 | `DuckDbDpcBranchingRepository.find_severity_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、手術分岐マスタ `dpc_surgery_branch`、手術・処置等1分岐マスタ `dpc_procedure1_branch`、手術・処置等2分岐マスタ `dpc_procedure2_branch`、副傷病分岐マスタ `dpc_comorbidity_branch`、重症度等分岐マスタ `dpc_severity_branch`、重症度等判定条件マスタ `dpc_severity_condition` | — |
| S8-1 | `DuckDbEhrNoteRepository.find_by_patient` | 電子カルテ `ehr_notes` | — |
| S8-4 | `DuckDbDpcBranchingRepository.find_surgery_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、手術分岐マスタ `dpc_surgery_branch` | 眼の分類：同一手術分岐を確認 |
| S8-4 | `DuckDbEfFileRepository.find_k_codes` | 診療行為明細 `ef_records` | 眼の分類：手術の算定実績を確認 |
| S8-3 | `DuckDbDpcBranchingRepository.find_surgery_branches` | 病態等分類分岐マスタ `dpc_digit7_branch`、年齢等分岐マスタ `dpc_digit8_branch`、手術分岐マスタ `dpc_surgery_branch` | Step8のLLM補助ツール：呼出時 |
| S8-3 | `DuckDbEfFileRepository.find_by_patient` | 診療行為明細 `ef_records` | Step8のLLM補助ツール：呼出時 |

④には上記参照データを置き、マスタには次の由来一覧を展開する。

### Step9｜最終候補の選定｜DPC14・点数

概要：実在確認・信頼度で絞り込み。点数比較＋僅差時LLM。
※ 請求に使えない出来高請求コードや非実在コードを除外し、信頼度と点数を重み付けして最終的な推奨コードを1件決める。

**② IN**

- 入力モデル：`Step8SeverityOutput`。
- `candidates[]`：前段までの候補情報。
- `PatientContext`：患者ID・入退院日・年齢。
- `Dpc6CandidatesOutput`：Step1結果、省略可。
- 依存引数：`Step9Deps`。

**OUT**

- 出力モデル：`Step9FinalOutput`。
- 候補型：`Step9FinalCandidate`。
- 格納形式：`selected: Step9FinalCandidate | None` / `rejected: list[Step9FinalCandidate]`。

掲載フィールド・補足（字下げ行は直前の項目の内訳）：

```text
selected：選定1件、またはNone
rejected：非選定候補のリスト
候補の型：Step9FinalCandidate
  dpc14 / dpc6 / 病名・ICD-10
  total_score / score_per_day
  length_of_stay / period1_days等
  confidence / evidence
  is_valid / is_bundled
  rejection_reason：棄却理由
  procedure1/2_review_reasons
  tiebreaker_decision：僅差選定の記録
```

**③ 概念的な処理の流れ**

| ID | 判定方式 | 処理 | IN | OUT | 条件・補足 |
|---|---|---|---|---|---|
| S9-1 | 機械的 | 完成候補を抽出し、実在を照合 | Step8SeverityOutput、dpc14\_pattern | 実在する候補と期間別点数、非実在候補＋棄却理由 | DPC14未完成や出来高請求コードは除外。完成候補なしなら空の最終結果。 |
| S9-2 | 機械的 | 入院日数・点数を計算 | 期間別点数・PatientContext | list\[Step9FinalCandidate\]、total\_score / score\_per\_day等 | 退院日ありは実日数（期間III上限）。未定は期間IIまで。 |
| S9-3 | 機械的 | 信頼度・点数で基準候補を選ぶ | 有効候補・confidence・点数 | base\_winner・base\_rejected | 信頼度で絞り、最大点数→信頼度順。空なら閾値を段階的に下げる。 |
| S9-4 | 機械的 | LLM選定の実行条件を確認 | 基準候補・他候補・SelectorContext、LLM選定の有効設定 | 判定対象top\_k または、LLM不要の基準結果 | 既定は基準との差1,000点以内、最大3候補。不要なら⑦へ。 |
| S9-5 | LLM | 僅差候補から主病名として選定 | top\_k・カルテ・手術情報、Step1結果・留意事例・患者情報 | TiebreakerLlmResult、winner\_index・reasoning・confidence | LLM選定が有効で、条件を満たす場合のみ実行。 |
| S9-6 | 機械的 | LLM提案の採否を確認 | LLM結果・top\_k・基準結果 | 最終選定候補・非選定候補、TiebreakerDecision | 失敗／範囲外／上書き信頼度0.75未満は基準維持（既定設定）。 |
| S9-7 | 機械的 | 最終結果を構成 | 最終選定候補・非選定候補、実在確認で棄却した候補 | selected: 候補 | None、rejected: list\[候補\] |

通常は番号順に実線接続。表内の丸数字はこのStep内の処理番号を指す。

- 条件付き迂回：S9-4 → S9-7（LLM不要）。

**③と④をつなぐ実行時参照**

Repositoryのクラス・メソッドは参照経路の補足。左端は上記の概念処理IDとする。各データからRepositoryへ個別の点線を引く。

| 概念処理 | Repository / メソッド | 参照するデータ | 条件 |
|---|---|---|---|
| S9-1 | `DuckDbDpc14PatternRepository.find_scores_by_dpc14_list` | DPC14定義・期間別点数マスタ `dpc14_pattern` | — |
| S9-5 | `DuckDbEhrNoteRepository.find_by_patient` | 電子カルテ `ehr_notes` | 選定補助情報を構成する場合 |
| S9-5 | `DuckDbEfFileRepository.find_k_codes` | 診療行為明細 `ef_records` | 選定補助情報を構成する場合 |
| S9-5 | `DuckDbIcd10CodingCautionRepository.find_by_icd10` | ICD-10コーディング留意事例 `icd10_coding_caution` | 選定補助情報を構成する場合 |
| S9-5 | `DuckDbDpc6CodingCautionRepository.find_by_dpc6` | DPC6コーディング留意事例 `dpc6_coding_caution` | 選定補助情報を構成する場合 |

④には上記参照データを置き、マスタには次の由来一覧を展開する。

## ④ 元データの定義

以下のIDはこのプロンプト内の参照用です。図では日本語名と実ファイル名を表示してください。点数表・傷病名マスターは実行時に明示指定するファイルで、下記名はリポジトリREADMEの配置例です。採用済みバンドルの版を断定しないでください。

| ID | 元ファイル（リポジトリ相対パス） | 説明・上流 |
|---|---|---|
| E | `data/dpc_fee_schedule/診断分類群(DPC)電子点数表(正式版)_20260812.xlsx` | DPC電子点数表。シート別に下表の依存がある。入手URLは未確認。 |
| D | `data/disease_master/b_20260601.txt` | 傷病名マスター。入手URLは未確認。 |
| P | `data/procedure_triggers/master_curated_procedure_triggers.csv` | 処置等1・2の補正データ。さらに上流の出典は未確認。 |
| C | `data/definition_codes/master_curated_definition_codes.json` | 定義コードの補正・評価ルールデータ。さらに上流の出典は未確認。（化学療法等のダミーコード定義は手作業で調査・作成） |
| U | `data/code_mappings/dummy_code_mapping.json` | ダミーコード対応データ。さらに上流の出典は未確認。（手作業で調査・作成） |
| F | `data/code_mappings/comorbidity_ef_mapping.json` | EFコード・副傷病対応データ。さらに上流の出典は未確認。 |
| I | `data/coding_cautions/留意すべきICDコードへの対応例.csv` | ICD留意事例データ。厚労省等のコーディングテキスト(PDF)から抽出。 |
| J | `data/coding_cautions/DPC上6桁別_注意すべきDPCコーディング事例集.csv` | DPC6留意事例データ。厚労省等のコーディングテキスト(PDF)から抽出。 |

## ④ マスタ別の由来

表中の「シート ← E」はシートを独立ノードにして元Excelノードに接続します。「＋」で並べた依存はそれぞれマスタへ接続します。「コード一覧 ← D」はメモリ上のデータ集合であり、既存Parquetへの依存ではありません。

| マスタ（日本語名／テーブル名） | 元データへの依存（右から左） |
|---|---|
| EFコード・副傷病対応マスタ / `comorbidity_ef_mapping` | F |
| DPC14定義・期間別点数マスタ / `dpc14_pattern` | `11）診断群分類点数表` シート ← E（上位分岐IDも同シート由来） |
| DPC6コーディング留意事例 / `dpc6_coding_caution` | J |
| DPC6・ICD-10対応マスタ / `dpc6_icd10_map` | `４）ＩＣＤ` シート ← E ＋ 既知ICD-10コード一覧 ← D |
| DPC6分類マスタ / `dpc6_master` | `２）分類名称` シート ← E |
| 副傷病分岐マスタ / `dpc_comorbidity_branch` | `11）診断群分類点数表` シート ← E（上位分岐IDも同シート由来） |
| 副傷病判定条件マスタ / `dpc_comorbidity_trigger` | `９）定義副傷病名` シート ← E ＋ 分岐対応データ ← `11）診断群分類点数表` シート ← E |
| 定義コード対応・評価ルールマスタ / `dpc_definition_code_mapping` | C |
| 病態等分類分岐マスタ / `dpc_digit7_branch` | `11）診断群分類点数表` シート ← E（上位分岐IDも同シート由来） |
| 病態等分類判定ルール / `dpc_digit7_rule` | `３）病態等分類` シート ← E |
| 年齢等分岐マスタ / `dpc_digit8_branch` | `11）診断群分類点数表` シート ← E（上位分岐IDも同シート由来） |
| 年齢等判定条件マスタ / `dpc_digit8_trigger` | `５）年齢、出生時体重等` シート ← E ＋ 分岐対応データ ← `11）診断群分類点数表` シート ← E |
| 手術・処置等1分岐マスタ / `dpc_procedure1_branch` | `11）診断群分類点数表` シート ← E（上位分岐IDも同シート由来） |
| 手術・処置等1判定条件マスタ / `dpc_procedure1_trigger` | `７）手術・処置等１` シート ← E ＋ 分岐対応データ ← `11）診断群分類点数表` シート ← E ＋ P（trigger\_type・receipt\_codes補完） ＋ C（定義コード確認） |
| 手術・処置等2分岐マスタ / `dpc_procedure2_branch` | `11）診断群分類点数表` シート ← E（上位分岐IDも同シート由来） |
| 手術・処置等2判定条件マスタ / `dpc_procedure2_trigger` | `８）手術・処置等２` シート ← E ＋ 分岐対応データ ← `11）診断群分類点数表` シート ← E ＋ P（条件補完） ＋ C（定義コード・名称） |
| 重症度等分岐マスタ / `dpc_severity_branch` | `11）診断群分類点数表` シート ← E（上位分岐IDも同シート由来） |
| 重症度等判定条件マスタ / `dpc_severity_condition` | `10－1）重症度等（ＪＣＳ等）`、`10－2）重症度等（手術等）`、`10－3）重症度等（重症・軽症）`、`10－4）重症度等（脳卒中の発症時期等）` の各シート ← E ＋ 分岐対応データ ← `11）診断群分類点数表` シート ← E |
| 手術分岐マスタ / `dpc_surgery_branch` | `11）診断群分類点数表` シート ← E ＋ \`６）手術 \` シート ← E（シート名末尾に空白あり） |
| ダミーコード対応マスタ / `dummy_code_mapping` | U |
| ICD-10コードマスタ / `icd10_code_master` | ICD-10コード列 ← D |
| ICD-10コーディング留意事例 / `icd10_coding_caution` | I |
| ICD-10病名マスタ / `icd10_name_master` | 病名・読み・ICD-10等のレコード ← D |

### ユーザーデータ

`ehr_notes`（電子カルテ）、`ef_records`（診療行為明細）、`dept_system_reports`（放射線レポート）はマスタと別枠で配置します。ai-dpc-branch-masterは患者データの取込を扱わないため、これらの上流システム・取込ファイルは「本確認範囲外」とし、⑤に記入欄を残してください。
※運用上、現状は手術記録や放射線レポートなどの部門システムデータは連携されていないケースが多く、主に電子カルテ記事（ehr_notes）を優先して読み取っています。

### 混同しない事項

- `dpc_definition_code_dependency` は生成側のv1.2のみのテーブルです。基準9-Stepの参照先に含まれないため、今回の図に追加しません。
- `master_curated_definition_dependencies.csv` を定義コード対応マスタの元データとして接続しないでください。現行の生成分岐では別のdependency table用です。
- 旧ルート直下の `dpc_points.xlsx` / `001675226.xlsx`、旧V3 JSON・Level 2 JSONは、現行生成の自動参照元として接続しません。
- `k_code_master` は今回の確定9-Stepの直接参照先に追加しません。
- 補正CSV・JSONが点数表や別マスタから作られたという上流関係は未確認です。名称だけで接続しないでください。

## ⑤ 手書き補足欄

全Stepの最下部に「ユーザーデータの出力元」「マスタ原資料の出典・補足」の2つの空欄を配置します。④で確認済みの由来は記載済みとし、⑤には未確認の情報を推測して埋めません。

## 検収

- 9列、5段が揃い、③には概念処理と個々のI/O・LLM／機械的の区別がある。
- 処理関数名がカードの見出しになっていない。
- ④にマスタ・元データ・元ファイルの連鎖があり、生成処理ノードがない。
- 全参照マスタに日本語名・実テーブル名・由来がある。補正元も省略していない。
- 実行時のSQL上位分岐参照と、生成時の元データ依存を混同していない。
- 原資料や補正データを読む関係と、既存マスタParquetを読む関係を混同していない。
- 文字の欠け、縮小しすぎ、線の交差がなく、ノード・文字・線を個別編集できる。

## 作図根拠（フレーム外の資料メモに記載）

- `ai-dpc-branch-master/README.md`：入力配置、対応版、旧資料の扱い。
- `ai-dpc-branch-master/master_data/generation/simple.py`：テーブル別の実際の読込ファイル、シート、補正データへの依存。
- `ai-dpc-branch-master/docs/design/master-responsibilities.md`：生成側と利用側の責務、患者データが範囲外であること。
- 既存 `dpc-dependencies-explicit.pdf` と概念処理の整理資料：StepのI/O・参照経路・条件付き処理。
- 2026/11/11の業務引き継ぎミーティング（Circleback等）に基づく運用仕様・課題事項の補足。

生成を実行した検証ではなく、読込実装と資料の静的確認に基づく図です。

---

# 詳細データ（元テキスト）

**確定DPC 9-Step｜概念的処理とデータの由来**

実線：候補の受け渡し　　点線：元データ・処理への参照　　[M01]〜[M23]：同一マスタは全Step共通番号　　機械的＝ルール照合・検証　　LLM＝文脈抽出・推定・批評（不確実性あり）

**① Step1｜主病名候補｜1〜6桁**
ICD-10抽出 → DPC6対応付け。LLM＋マスタ検索。

**① Step2｜病態等分類｜7桁目**
病態の分岐を判定。ルール＋必要時LLM。

**① Step3｜年齢・出生時体重等｜8桁目**
抽出した数値を条件照合。ルール＋必要時LLM。

**① Step4｜手術｜9〜10桁目**
必須・除外Kコードを照合。ルール判定（LLMなし）。

**① Step5｜手術・処置等1｜11桁目**
処置の実施条件を照合。ルール判定（LLMなし）。

**① Step6｜手術・処置等2｜12桁目**
処置の実施条件を照合。ルール判定（LLMなし）。

**① Step7｜副傷病｜13桁目**
LLM抽出＋EFから推定。ICD-10と条件照合。

**① Step8｜重症度等｜14桁目**
重症度を判定・DPC14組立。ルール＋必要時LLM。

**① Step9｜最終候補の選定｜DPC14・点数**
実在確認・信頼度で絞り込み。点数比較＋僅差時LLM。

**② Step全体の IN / OUT**
各Stepごとの詳細なIN/OUT・モデル型・格納形式などは、前半のプロンプトセクションに整理されています。

**③ 概念的な処理の流れ**
各カード：方式／入力 IN／出力 OUT／条件・補足。候補ごとに評価。
詳細なテーブルは前半のプロンプトセクションに整理されています。

**④ 参照データと由来**
利用マスタ　←　直接の元データ　←　元ファイル
点線「元データ」＝生成時の由来。シート／ファイルは処理工程ではない。
詳細な対応・Repositoryとの依存関係は前半のプロンプトセクションに整理されています。

**⑤ 手書き補足欄**
ユーザーデータの出力元
マスタ原資料の出典・補足

**資料メモ｜確認範囲・作図根拠**
利用側：ai-dpc-multi-agent／基準PDF記載 HEAD e6f16da
生成側：ai-dpc-branch-master／HEAD ad93025
2026-11-11の読込実装・資料の静的確認。同一リリースとしての整合は未検証。生成を実行した検証ではない。

根拠資料
・ai-dpc-branch-master/README.md：入力配置、対応版、旧資料の扱い
・ai-dpc-branch-master/master_data/generation/simple.py：テーブル別の読込ファイル、シート、補正データへの依存
・ai-dpc-branch-master/docs/design/master-responsibilities.md：生成側／利用側の責務、患者データが範囲外であること
・既存 dpc-dependencies-explicit.pdf と概念処理の整理資料：StepのI/O、参照経路、条件付き処理
・ai-dpc引き継ぎ1回目（2026/11/11）の会議メモ（Circleback）

読み方・確認限界
・電子点数表Excel／傷病名マスターの表示名はREADMEの配置例。採用済みバンドルの版や入手URLは未確認。
・補正CSV／JSONのさらに上流は未確認。名称から推測して接続していない。
・分岐対応データ／ICD-10コード一覧は原資料からメモリ上で構成。既存マスタParquet間の生成依存ではない。
・患者データの上流システム・取込ファイルは本確認範囲外。⑤は未記入。
・Step5／6の定義コード判定器は同一の共通事前読込結果の再掲。各Stepでの再読込を意味しない。
・元ファイルは灰青、マスタは紫、患者データは緑。実線は候補受渡し、点線は元データ／実行時参照。