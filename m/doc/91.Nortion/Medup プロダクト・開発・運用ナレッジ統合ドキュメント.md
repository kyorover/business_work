# Medup プロダクト・開発・運用ナレッジ統合ドキュメント

## (9+) Data OpsナレッジDB
**ドキュメント名:** [(9+) Data OpsナレッジDB](https://app.notion.com/p/medup/22c1da5a7f3b80d48386ec353993e206?v=22c1da5a7f3b805bb315000cb5d79ba0)
**プラットフォーム:** Notion

### 概要
Data Ops（データ運用）に関するナレッジを集約したデータベース（リストビュー）ページ。紹介データや入院データの紐づけ、DPCデータの概要、施設名寄せ（データクレンジング）の仕組み、オンボーディング用資料など、データ仕様や運用ルールに関するドキュメントがタグ（初級、中級、対象データ種別など）とともに一覧化されている。

### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（データベースの管理項目）**
  * 各ナレッジドキュメントに対して、対象者のレベルを示す「初級」「中級」のタグが付与されていること。
  * 関連するシステムやデータの種類として「入院データ」「foro CRM」「紹介データ」「DPC」「データクレンジング」「MIS」といったタグが付与されていること。
  * 各ドキュメントの日付（2025/07/10、2025/09/02、2026/04/14など）が記録されていること。

* **サブページやリスト項目から得られる情報（各ナレッジドキュメント）**
  * [[紹介データと入院データの紐づけの概要]](https://app.notion.com/p/medup/22c1da5a7f3b80fca4ccff27821dcfe5?pvs=25) : 紹介データと入院データを紐づける仕組みや仕様の概要（タグ：初級、入院データ、foro CRM、紹介データ）についてわかること
  * [[入院データ ≒ DPCデータの概要]](https://app.notion.com/p/medup/DPC-22c1da5a7f3b80e3a34ded3995e0fd70?pvs=25) : 入院データおよびDPCデータに関する基本的な概要や関係性（タグ：初級、入院データ、DPC、foro CRM）についてわかること
  * [[foro CRMのデータの概要]](https://app.notion.com/p/medup/foro-CRM-22c1da5a7f3b8056baf9e112a6727bb4?pvs=25) : foro CRM内で扱われるデータの全体的な概要（タグ：初級、foro CRM、紹介データ、入院データ）についてわかること
  * [[データクレンジング（施設の名寄せ）の仕組みの概要]](https://app.notion.com/p/medup/22c1da5a7f3b80bda8b7ee81fd35a170?pvs=25) : 医療機関等の施設データを名寄せ（クレンジング）する仕組みの概要（タグ：初級、データクレンジング）についてわかること
  * [[アクセス不可]](https://app.notion.com/p/medup/22d1da5a7f3b8012b2dfe90249ef8030?pvs=25) : 権限等の理由により現在アクセス不可となっているページ（タグ：初級、紹介データ、入院データ、DPC）
  * [[【オンボーディング用】メダップのデータについて + Data Opsについて]](https://app.notion.com/p/medup/Data-Ops-2611da5a7f3b80fab9efcec947e3a394?pvs=25) : 新規参画者（オンボーディング）向けに、メダップのデータ全体像やData Ops業務について解説した資料（タグ：初級）
  * [[foro に入ってるデータがすべてわかるページ！]](https://app.notion.com/p/medup/foro-644259e4620349159ce380cd8586e7bf?pvs=25) : foroシステムに格納されている全データの仕様や詳細がわかるページ（タグ：中級、MIS、foro CRM）
  *(※補足：テキストデータ上のリスト項目のURL `url?id=...` は内部ID形式のまま記載)*

---

## プロダクト foro 連携マップ
**ドキュメント名:** [プロダクト foro 連携マップ](https://app.notion.com/p/medup/foro-3923828e2ff34d91a5babc954d668fa5)
**プラットフォーム:** Notion

### 概要
医療機関向けサービス「foro 連携マップ」に関する総合的なプロダクトポータルページ。プロダクトのビジョンやロードマップ、開発目標、PRDやドメインモデリングなどの「開発コンテンツ」、マニュアルなどの「ユーザーコンテンツ」、藤沢湘南台病院などの「議事録」、および具体的な機能開発や改善タスクを管理する「プロダクトバックログ」が集約されている。

### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（プロジェクトの運用方針やカンバンの状態）**
  * ページ上部の関係者として「miho.mihoya」「ShimboKatsuyuki」「Jun.Yamada」「Yasutaka Umehara」などの名前が記録されていること。
  * 議事録リストの記載内容から、「藤沢湘南台病院」との定例会議（外部）が定期的に行われていること、および「クロスセル提案の振り返り」や開発のWeekly MTGが実施されていること。
  * 「foro 連携マップ プロダクトバックログ」としてカンバンが設置されており、タスクのステータスとして「アイスボックス 4」「ToDo 3」「仮説検証 0」「要件定義 4」「デザイン待ち 0」「デザイン中 1」「確認中 0」「開発待ち 0」「見積・設計中 0」「開発中 0」という区分で管理されていること。
  * バックログタスクの担当者として「Takayuki Guzman」「Junsei Tokazi」「yuya tabata」などの名前が記載されていること。

* **サブページやリスト項目から得られる情報（各カテゴリのドキュメント、運用ナレッジなど）**
  * **Product Vision&ロードマップ**
    * [[外来適正化 Vision]](https://app.notion.com/p/medup/Vision-98fab65e85be4bca8b2617146427aa71?pvs=25) : プロダクトにおける外来適正化のビジョンや方針についてわかること
    * [[2024 foro 連携マップ プロダクトロードマップ]](https://app.notion.com/p/medup/2024-foro-711d288180d64f3a84804c709a00d1a4?pvs=25) : 2024年のforo連携マップに関するプロダクト開発ロードマップ
    * [[2023 foro 外来 プロダクトロードマップ]](https://app.notion.com/p/medup/2023-foro-8e04ce95b61e408190ce10f76ea1fc3d?pvs=25) : 2023年のforo外来に関する過去のプロダクトロードマップ
  * **目標**
    * [[2024/02 - 04 連携マップユニット目標]](https://app.notion.com/p/medup/2024-02-04-318a6b466a124f19ba2ac9803784837c?pvs=25) : 2024年2月〜4月期の連携マップユニットにおける開発・事業目標（OKR等）についてわかること
  * **開発コンテンツ**
    * [[連携マップ開発のホームページ]](https://app.notion.com/p/medup/d74d5a51b1e2422c991d523b8eb24765?pvs=25) : 連携マップの開発チーム向けポータル・ホームページ
    * [[ドメインモデリング]](https://app.notion.com/p/medup/3da77d132b374a59b3a31f35d2619468?pvs=25) : システム開発におけるドメインモデル・データ構造の定義
    * [[ユビキタス言語]](https://app.notion.com/p/medup/52d14d72e42d490b8cc631eb186f9a25?pvs=25) : プロダクトや開発チーム内で用いる共通の専門用語・ユビキタス言語の定義
    * [[PRD]](https://app.notion.com/p/medup/PRD-30625fd05cba429ea53a85705b83a535?pvs=25) : プロダクト要求仕様書（Product Requirements Document）
    * [[foro 連携マップ/Zのデータ分析]](https://app.notion.com/p/medup/foro-Z-bf967a40a15545a3bf0df52b2be84954?pvs=25) : プロダクトの利用状況やZに関するデータ分析情報
    * [[市区町村データのメンテ方法]](https://app.notion.com/p/medup/1181da5a7f3b80b29d02eefad96119fe?pvs=25) : システム内で利用する市区町村マスターデータのメンテナンスや更新手順
  * **ユーザーコンテンツ**
    * [[ユーザー向けマニュアル]](https://app.notion.com/p/medup/12a467d56e3146f4bc4c02e47bac5ead?pvs=25) : 顧客・ユーザー向けのシステム操作マニュアル
  * **その他**
    * [[アクセス許可テナントリスト]](https://app.notion.com/p/medup/237ef18d171e47a2a2c23c74edd2f3a5?pvs=25) : 連携マップへのアクセスが許可されているテナント（医療機関等）の一覧リスト
    * [[VOC（foro 連携マップ）]](https://app.notion.com/p/medup/8a2ade784d8b4ff2b8070b76f70e2cc9?v=dc8b0da470e74eb6b95fb921571ddf91&pvs=25) : 顧客やユーザーからの要望・フィードバック（VoC）まとめ
    * [[[連携マップ] ローカルデータの取り扱いルール策定]](https://app.notion.com/p/medup/f0a26c78eaf54dd0ad3f58407ea05ad2?pvs=25) : ローカル環境等でのデータ取り扱いに関するセキュリティや運用ルール
    * [[foro 連携マップにアクセスするためのドメインリスト]](https://app.notion.com/p/medup/foro-755ce23028634cf8b5888b0ff095970d?pvs=25) : プロダクトにアクセスするための関連ドメイン一覧
    * [[foro外来_プロダクト名称アップデート検討]](https://app.notion.com/p/medup/foro-_-5af6b5fbda46493c9d6ae0150ede9c4f?pvs=25) : プロダクト名称（foro外来等）の変更・アップデートに関する検討記録
    * [[仮想プレスリリース]](https://app.notion.com/p/medup/90a53aaff26e4b6992dcf3a416dd9b5e?pvs=25) : サービス開発・企画に関する仮想プレスリリース（PR/FAQ等）のドキュメント
    * [[foro 連携マップ_通常CSのオンボーディング作業概要]](https://app.notion.com/p/medup/foro-_-CS-1191da5a7f3b80488d1df11eb60ec4f4?pvs=25) : カスタマーサクセス（CS）による顧客への通常オンボーディング作業の概要フロー
    * [[連携マップリリースノート]](https://app.notion.com/p/medup/1f21da5a7f3b80b28573e9d98bf6744a?v=1f21da5a7f3b80b0968a000c60c9625c&pvs=25) : 機能追加や修正などのリリース履歴
  * **プロダクトバックログ（具体的な開発・改善タスク）**
    * [[逆紹介すべき外来患者を発見できる]](https://app.notion.com/p/medup/a5be257bd7a94ae8907a44b8fd0ccc43?pvs=25) : 逆紹介の対象候補となる外来患者を抽出・発見する機能の要件
    * [[外来患者の個別状況を可視化・管理できる]](https://app.notion.com/p/medup/7a4b22a15e7c4e888a48ab87992553f2?pvs=25) : 患者の個別状況をシステム上で可視化・管理する機能の要件
    * [[文字見えない問題への対応]](https://app.notion.com/p/medup/10b1da5a7f3b8049a1cfd5bf833e8c24?pvs=25) : 画面上の文字の視認性改善・UI修正に関するタスク
    * [[医療機関ラベル増えて見づらい問題]](https://app.notion.com/p/medup/1ccf71cc5066440d900dfe3d05621a57?pvs=25) : 医療機関ラベルが密集した際の表示UI改善タスク
    * [[ナビゲーションの改善]](https://app.notion.com/p/medup/10b1da5a7f3b80c8a68fc04b842a0765?pvs=25) : プロダクト内のナビゲーションUI・導線改善タスク
    * [[連携マップの医療機関ピンで医科（病院・診療所）と歯科を分ける]](https://app.notion.com/p/medup/1541da5a7f3b80d7a59ceba91d809087?pvs=25) : マップ上のピン表示において、医科と歯科を識別可能にする機能追加タスク
    * [[リストを印刷したい]](https://app.notion.com/p/medup/1121da5a7f3b8067a43bdb8313f9597b?pvs=25) : 検索結果等のリスト印刷機能の追加タスク
    * [[患者向けUI切り替え機能]](https://app.notion.com/p/medup/UI-10b1da5a7f3b80bb9316e5f70cba04f8?pvs=25) : 医療従事者向けUIから患者向けUIへ画面を切り替える機能のタスク
    * [[ユーザーのプロダクト活用状況を把握したい]](https://app.notion.com/p/medup/14c1da5a7f3b80f288c2cf9639cc86da?pvs=25) : プロダクトの利用トラッキングや利用状況の可視化に関するタスク
    * [[患者向け連携登録医検索システム]](https://app.notion.com/p/medup/10b1da5a7f3b807fb11ce5d934e6282e?pvs=25) : 患者向けに連携登録医を検索できるシステムの構築タスク

---

## 運用ナレッジ・対応履歴ドキュメント集

### 20260827_外来適正化月次データ抽出
**ドキュメント名:** [20260827_外来適正化月次データ抽出](https://app.notion.com/p/medup/20260827_-3c91da5a7f3b8027973ffdc63e2c2254)
**プラットフォーム:** Notion

#### 概要
2026年8月27日に実施された「外来適正化」の月次データ抽出（逆紹介対象リストの抽出）作業に関する記録ドキュメント。月末に実施する理由（翌月予約患者への逆紹介）や、クエリ実行とデータ突合のフロー、報酬改定等の影響によるクエリ変更の可能性、前月データとの乖離発生時の調査方針など、月次運用の具体的なナレッジが記載されている。

#### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（運用ルールや作業ナレッジ）**
  * ミーティング・作業の参加者として「ShimboKatsuyuki」「Takeshi Yanagiuchi」「Jun.Yamada」が記録されていること。
  * 「逆紹介対象リストの抽出」を行う月次作業であり、翌月予約患者の逆紹介を出すために「概ね月末に実施している」という運用スケジュール。
  * 抽出したデータと予約データを突合することで「逆紹介する人（候補）リスト」が完成し、逆紹介先の医療機関とのマッチングに利用されるという業務フロー。
  * 抽出作業は「基本は用意されているクエリ実行でそのままいける想定」であるものの、報酬改定などのコード変更があるとクエリの変更が発生するというシステム運用の注意点。
  * 抽出結果データを確認し、「前月との大きな乖離等があるとその辺り（報酬改定等）を疑って調査に入る」というデータ品質確認のノウハウ。
* **サブページやリスト項目から得られる情報（関連ドキュメントやツールへのリンク）**
  * [[circleback.ai /mee…rDiKOy]](https://circleback.ai/meetings/umMgzIz4c8o5YkUrDiKOy) : 本ミーティング・作業に関するCirclebackの議事録や録画URL。
  * [[Circleback Circleback]](https://circleback.ai/meetings/umMgzIz4c8o5YkUrDiKOy) : 上記と同様のCirclebackリンク。
  * [[Circleback Circleback]](https://circleback.ai/meetings/55uKmpx98HYr9xcDCJTxk) : 本作業に関連するCirclebackの記録リンク。
  * [[Circleback Meeting notes - Circleback]](https://circleback.ai/ja/meetings/BzEYf89SvOMd8HDoZrnhs?tab=notes) : Circleback上に生成された、本作業のミーティングノート・議事録詳細。
  * [[外来予約データ月次処理 業務運用手順書]](https://docs.google.com/document/d/1ej83SY35NtvjxKhsU4Lysb97reSYcbRl6yo8f0rYnHs/edit?tab=t.0#heading=h.m7jx1gyrn55e) : 抽出や突合など、実際の作業手順が詳細に記載された公式の業務運用マニュアルへのリンク。

### 20260909_紹介データ重複削除
**ドキュメント名:** [20260909_紹介データ重複削除](https://app.notion.com/p/medup/20260909_-3d61da5a7f3b80deaaffe772b1df7bbb)
**プラットフォーム:** Notion

#### 概要
2026年9月9日に記録された、紹介データのファイル単位での重複削除対応に関するメモ。2024年秋以前の古いデータであるために管理ツール（MIS管理画面）上での削除処理ができず手動対応となった背景や、削除対象となる各テーブル（referrals、referral_masters）ごとの具体的な削除仕様について記載されている。

#### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（古いデータの削除仕様や対応履歴）**
  * 2024年秋より前の紹介データにおいてファイル単位でのデータ重複が発生していること。
  * 通常のデータ削除はMIS管理画面から実行できるが、今回のケースは削除機能が搭載される以前に登録された古いデータであるため、MIS管理画面から削除できず手動対応となるという仕様。
  * 対応の経緯となる[[Slackのメッセージ]](https://medup.slack.com/archives/C036KMJT06L/p1788752098064459?thread_ts=1788499315.547009&cid=C036KMJT06L)へのリンク情報。
  * 削除対象テーブル「referrals」は、管理画面からの削除で消える仕様であること。
  * 削除対象テーブル「referral_masters」は、昔のデータは管理画面で消しても消えないが、2024年10月以降分については「data mgmt admin」で消せる仕様であること。
  * ミーティングや対応のメタデータ（開催日：2026/09/09、[[Circleback URL]](https://circleback.ai/ja/meetings/soZvQHfmBsyXupJeBS6xj?tab=transcript)の議事録・スクリプトリンクなど）。
* **サブページやリスト項目から得られる情報**
  * （※本文内に記載されているサブページやリスト項目はありません）

### 20260909_入院データ重複削除
**ドキュメント名:** [20260909_入院データ重複削除](https://app.notion.com/p/medup/20260909_-3d61da5a7f3b803c9dffcfb31f355ed6)
**プラットフォーム:** Notion

#### 概要
2026年9月9日に記録された、がん研有明における入院データの重複削除および洗替プロセス（患者番号変換 → 削除 → 再投入）に関する手順と仕様のドキュメント。DPCデータにおいて匿名化された患者番号を診療系IDと紐づけるための復号ロジック（変換ルール）や、ファイル名の指定、データ構造の前提条件について詳細に定義されている。

#### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（データの仕様や変換ロジック等）**
  * 開催メタデータ：開催日は2026/09/09。参加者としてShota Kurokawa、byeongseon kang、Takeshi Yanagiuchi、Jun.Yamadaの記載がある。
  * 【背景】がん研有明のDPCは厚生局提出用に「患者番号を匿名化したデータ」しか作成できない。そのため、紹介データ等の「診療系ID」と紐づけるには、システムへの投入前に復号処理が必要となること。
  * 【IT推進課の匿名化ロジック（変換ルール）】診療系IDの頭の「0」を削除し、末尾に「0」を追加している仕様（例：診療系ID `0007925233` → DPCデータ識別番号 `0079252330`）。
  * 【投入時の復号ロジック】末尾の「0」を除去し、元の桁数になるように先頭を0埋めして戻す仕様（例：`0079252330` → `0007925233`）。
  * 復号後の出力ファイル名の指定ルールは `*_patient_number_fixed` であること。
  * 【ファイル要件】様式1ファイルおよびDファイルともに、施設コードの次列（2列目）が「データ識別番号」になっているという前提条件。
  * 【手順の開始】「1. 元ファイル（様式1とDファイル、月ごとセット）を受領する」というオペレーションの第一ステップ。
* **サブページやリスト項目から得られる情報（関連ドキュメントやツールへのリンク）**
  * [[circleback.ai /ja/…script]](https://circleback.ai/ja/meetings/soZvQHfmBsyXupJeBS6xj?tab=transcript) : ミーティング（Circleback）の文字起こしスクリプトや議事録情報。
  * [[Slackのメッセージ]](https://medup.slack.com/archives/C036KMJT06L/p1788918416671009?thread_ts=1788860270.129589&cid=C036KMJT06L) : 入院データの洗替・重複削除対応に関するSlack上でのやり取りや依頼スレッド。
  * [[アクセス不可]](https://app.notion.com/p/2c01da5a7f3b80e0bcbdd497bda02a96?pvs=24#2c01da5a7f3b80e0bcbdd497bda02a96) : テナント個別の情報（※現在アクセス不可のステータス）。
  * [[入院データ投入手順]](https://app.notion.com/p/2651da5a7f3b80c99cf8e94756160a4a?pvs=24) : データをシステムへ投入する際の公式マニュアル・手順書。
  * [[foro Core Admin 入院タブ]](https://admin-web.medup-core-service.com/foro-core/tenants/204?tab=hospitalAdmission) : データの削除や再投入などのオペレーションで使用する管理画面（foro Core Admin）の該当タブへのリンク。

---

## 02. Engineer
**ドキュメント名:** [02. Engineer](https://app.notion.com/p/medup/02-Engineer-b32e0a1f71114560a9a6beaedda734e6)
**プラットフォーム:** Notion

### 概要
開発組織「02. Engineer」のポータルページ。開発組織の全体ルール（組織戦略、Techスタンダード、セキュリティ等）をはじめ、「foro CRM」「foro 外来適正化」「refery」などの各プロダクト開発ページへのリンク、および「Data & CSE」カテゴリにおける管理ツールの仕様やデータ更新ワークフロー（ナビイ、緯度経度更新、MIS更新など）が集約されている。また、「for 副業の皆さんへ」の定例MTGメモや、旧Devページ配下の「輪読会・勉強会・LT」「開発WIKI」などのナレッジも包括的にリスト化されている。

### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（プロジェクトの運用方針やミーティング記録など）**
  * 「Data & CSE」セクションにおけるops関連は現在「アクセス不可」であること
  * 「Data & CSE」セクションに記載されているドキュメント群は「管理ツールの仕様等devよりのもの(以降予定無し)」という前提であること
  * 「for 副業の皆さんへ」として、「Tech - 週次 for 副業」ミーティングが2025年12月15日から2026年3月23日まで定期開催されており、参加者として「chihiro.naganuma」「隼 有冨」「sawadaichi」が記録されていること

* **サブページやリスト項目から得られる情報（各部門のドキュメント、運用ナレッジなど）**
  * [[プロダクト開発組織戦略]](https://app.notion.com/p/medup/f4a7125445894a53a68569214cb44974?pvs=25) : 開発組織全体の戦略や方針について
  * [[Delivery - Tech スタンダード]](https://app.notion.com/p/medup/Delivery-Tech-b18b24753f3f4f559d280c0e11fcf67f?pvs=25) : 開発（Delivery）における技術的な標準ルールやコーディングスタンダードについて
  * [[セキュリティ]](https://app.notion.com/p/medup/7b10c016568d4e85971c18ec5325487f?pvs=25) : システム開発・運用において遵守すべきセキュリティ規定やルールについて
  * [[PdM向けドメイン知識キャッチアップ]](https://app.notion.com/p/medup/PdM-94967e6c4bdf4e4096b20119a7add136?pvs=25) : PdM（プロダクトマネージャー）向けの事業や医療ドメイン知識の習得用資料
  * [[Delivery - Tech Bi-Weekly]](https://app.notion.com/p/medup/Delivery-Tech-Bi-Weekly-7cdcd9d342b248098abe8022a6756479?pvs=25) : 開発チームの隔週（Bi-Weekly）定例ミーティングに関する情報
  * [[緊急時に指差し確認ができない場合のDBの操作ルール]](https://app.notion.com/p/medup/DB-1521da5a7f3b804f8ee9d4467d5a9b97?pvs=25) : 複数人での確認が困難な緊急時におけるデータベース（DB）の操作ルールについて
  * [[本番環境とSTG環境]](https://app.notion.com/p/medup/STG-3e61da5a7f3b80a48030de0837858460?pvs=25) : プロダクション（本番）環境およびSTG（ステージング）環境に関する定義・運用ルールについて
  * [[リリース手順]](https://app.notion.com/p/medup/3e91da5a7f3b806797e2f1ce57018d97?pvs=25) : プロダクトを本番環境へリリースする際の手順
  * [[04.foro CRM]](https://app.notion.com/p/04-foro-CRM-0e464442ff284798913a991395c5b487?pvs=24) : プロダクト「foro CRM」に関する開発・運用情報
  * [[DataOps引き継ぎタスク]](https://app.notion.com/p/medup/DataOps-3aa1da5a7f3b80cf8a48d3f951bfbac1?pvs=25) : 「Product - MIS」に関連するDataOpsの引き継ぎタスク
  * [[04.foro 外来適正化]](https://app.notion.com/p/04-foro-2681da5a7f3b809d8ff5c961ee79fde2?pvs=24) : プロダクト「foro 外来適正化」に関する開発・運用情報
  * [[Old_FDEチーム]](https://app.notion.com/p/medup/Old_FDE-3df1da5a7f3b8082891bdddc8bf59bbb?pvs=25) : FDEカテゴリにおける、旧FDEチームの過去情報やアーカイブ
  * [[04.refery]](https://app.notion.com/p/04-refery-3affae5e901e46d0b07fabb55c64ed97?pvs=24) : プロダクト「refery」に関する開発・運用情報
  * [[アクセス不可]](https://app.notion.com/p/22c1da5a7f3b80a2a68ce12eb5b116ff?pvs=24#22c1da5a7f3b80a2a68ce12eb5b116ff) : ops関連のアクセス権限（アクセス不可）に関するリンク先
  * [[【closed】パフォーマンス]](https://app.notion.com/p/medup/closed-1501da5a7f3b80cfb13bcae133fed5f0?pvs=25) : クローズ済みのパフォーマンス改善・検証に関するタスク・仕様について
  * [[medup - foro 管理ツール]](https://app.notion.com/p/medup/medup-foro-2070773f9faa4c4a9878b437f74febf8?pvs=25) : medup・foroにおける管理ツールの仕様等（dev寄り）について
  * [[DataOps - Wiki]](https://app.notion.com/p/medup/DataOps-Wiki-f9323921fe7b40c494cd9f64e9db3729?pvs=25) : DataOpsに関する開発者向けのWiki・ナレッジベース
  * [[AlloyDBと仲良くなるために]](https://app.notion.com/p/medup/AlloyDB-47a11c673ba14ea7b960d6684cce7c3b?pvs=25) : Google Cloud AlloyDBの技術的キャッチアップや仕様に関する情報
  * [[ナビイデータスクレイピングについて]](https://app.notion.com/p/medup/2ce1da5a7f3b80b2807cdfaf7576d3a2?pvs=25) : ナビイ（navii）データのスクレイピング処理に関するシステム仕様や手順
  * [[ナビイ(navii)手動マッチングのワークフロー]](https://app.notion.com/p/medup/navii-1461da5a7f3b802488fbe124c0753f9b?pvs=25) : ナビイデータと自社データ等を手動マッチングする際のワークフロー
  * [[医療機関の位置情報(緯度経度)更新ワークフロー]](https://app.notion.com/p/medup/1731da5a7f3b80608e13c5ef0e7f7fcb?pvs=25) : 医療機関の住所等に基づく緯度・経度情報を更新するための運用フロー
  * [[MISの更新フロー〜厚生局の医科歯科編〜]](https://app.notion.com/p/medup/MIS-1821da5a7f3b805fba57efa6396f362a?pvs=25) : 厚生局が公開する医科歯科データに基づく、MIS（医療情報システム）の更新フロー
  * [[MISの(更新)履歴管理 - 履歴データ生成]](https://app.notion.com/p/medup/MIS-18a1da5a7f3b80628a40f23003b9500c?pvs=25) : MIS（医療情報システム）の更新履歴管理および履歴データの生成ロジック仕様
  * [[アーカイブ]](https://app.notion.com/p/medup/1851da5a7f3b80a1a775f446f03d49c2?pvs=25) : Data & CSE配下の過去の不要になったドキュメントアーカイブ
  * [[MTG Memo]](https://app.notion.com/p/medup/85a90ad73b2a42d28332134a3ae4a424?v=d4865d65d8ae47779c1a05000c74ba07&pvs=18) : 「for 副業の皆さんへ」カテゴリにおける定例ミーティングの汎用メモ
  * [[プロダクト開発戦略]](https://app.notion.com/p/medup/bd92d8844fa14cd8bf4eddaec491fee7?pvs=25) : 旧Devページ配下の、プロダクト開発組織の全体戦略に関する情報
  * [[四半期Vision]](https://app.notion.com/p/medup/Vision-8f661e1915e54c499df437eb01e1e79f?pvs=25) : 開発組織における四半期ごとのビジョンや目標設定に関するドキュメント
  * [[メンバー毎・キックオフ(オンボーディング)]](https://app.notion.com/p/medup/962ecf8042f54a188f50e7c0b37372b9?pvs=25) : 開発組織へ新規参画するメンバー向けのキックオフおよびオンボーディング資料
  * [[MTG]](https://app.notion.com/p/medup/MTG-c796eeadf40e4d2f9883b084c13a66b8?pvs=25) : 開発組織内の各種ミーティングに関する運用ルールや情報
  * [[Grade]](https://app.notion.com/p/medup/Grade-f380a89d233c4e0ebfafef9a5c60005f?pvs=25) : エンジニアの等級（グレード）制度や評価・スキル基準に関するドキュメント
  * [[週チャレ]](https://app.notion.com/p/medup/f4d6f168bb904decbd98979d63f59d87?pvs=25) : 開発チーム内の定期的なチャレンジや取り組みに関する記録
  * [[エンジニア面談・面接]](https://app.notion.com/p/medup/9f7aa7b1a4db40389b57d4a7bbfff025?pvs=25) : エンジニア採用の面接フローや、メンバー面談に関する運用・ナレッジ
  * [[厚生局データ整備]](https://app.notion.com/p/medup/c70385a97fcd4a9198f457a5aebd0686?pvs=25) : 厚生局のオープンデータ整備プロジェクトに関する運用ドキュメント
  * [[ベストプラクティス集]](https://app.notion.com/p/medup/d967f950237f4b239a9a40fff87ca7c8?pvs=25) : 開発・運用における技術的・組織的なベストプラクティスをまとめたナレッジベース
  * [[ゆとり]](https://app.notion.com/p/medup/14fbd1ea3fea4af4b3de61ac2aa1aa33?pvs=25) : 開発組織における「ゆとり」というテーマや取り組みに関連する情報
  * [[権限移譲プログラム]](https://app.notion.com/p/medup/4b099aa85f46478d95e124834be96d25?pvs=25) : 組織内における権限移譲（デリゲーション）のプログラムに関するドキュメント
  * [[カレンダー]](https://app.notion.com/p/medup/721986a63c8a4e68bdbe8995fa2d9ae4?pvs=25) : 開発組織共通のカレンダー運用やスケジュール情報
  * [[メダップ流?! オニオン Functional DDD]](https://app.notion.com/p/medup/Functional-DDD-5ef678ec18cb4f95b52dfebd54f57321?pvs=25) : メダップにおけるオニオンアーキテクチャや関数型DDD（ドメイン駆動設計）の実践に関する技術ドキュメント
  * [[AI Cording Agent 導入pjt]](https://app.notion.com/p/medup/AI-Cording-Agent-pjt-1f91da5a7f3b805faac2f64cb037e2de?pvs=25) : AIコーディングエージェントの組織的な導入プロジェクトに関する情報
  * [[それ、n8nでなんとかならんか コーディングエージェント活用知見交換会 2025年10月15日]](https://app.notion.com/p/medup/n8n-28d1da5a7f3b8012ad9bf2edaf72220a?pvs=25) : n8nとコーディングエージェントの活用に関する知見交換会（chihiro.naganuma）の記録
  * [[comet ブラウザ試した コーディングエージェント活用知見交換会 2025年10月1日]](https://app.notion.com/p/medup/comet-27f1da5a7f3b807ca1fdf675e3cf020d?pvs=25) : cometブラウザの試用を通じたエージェント活用に関する知見交換会（byeongseon kang）の記録
  * [[Codex CLIお試し コーディングエージェント活用知見交換会 2025年9月3日]](https://app.notion.com/p/medup/Codex-CLI-2631da5a7f3b805d9a23df18c21f4be9?pvs=25) : Codex CLIの試用に関する知見交換会（chihiro.naganuma）の記録
  * [[claude code をより強力に使う コーディングエージェント活用知見交換会 2025年8月27日]](https://app.notion.com/p/medup/claude-code-25c1da5a7f3b8093a1b8f7d10d549455?pvs=25) : Claude Codeの高度な活用方法に関する知見交換会（byeongseon kang）の記録
  * [[Serenaよさそうじゃない？ コーディングエージェント活用知見交換会 2025年8月6日]](https://app.notion.com/p/medup/Serena-2471da5a7f3b80d1a456dfd567159af5?pvs=25) : Serenaツールの活用に関する知見交換会（chihiro.naganuma）の記録
  * [[Claude Code用のルールを分割整理しよう コーディングエージェント活用知見交換会 2025年7月16日]](https://app.notion.com/p/medup/Claude-Code-2321da5a7f3b80db99b7cd3d201513d1?pvs=25) : Claude Codeのルール分割・整理に関する知見交換会（chihiro.naganuma）の記録
  * [[医療情報システムで求められるセキュリティ関係のガイドライン等まとめ 2025年7月14日]](https://app.notion.com/p/medup/2171da5a7f3b806db832decab753414d?pvs=25) : 医療情報システム向けのセキュリティガイドラインに関する勉強会（chihiro.naganuma）の記録
  * [[メダップで扱っている医療オープンデータ初歩編勉強会 勉強会 2024年12月6日]](https://app.notion.com/p/medup/14b1da5a7f3b806ca592d12515e8392e?pvs=25) : メダップで扱う医療オープンデータの基礎に関する勉強会（chihiro.naganuma）の記録
  * [[言語化の習慣が認知と判断を高める仮説 LT 2024年11月19日]](https://app.notion.com/p/medup/13c1da5a7f3b80e8ac35c838d1fc6e76?pvs=25) : 言語化の習慣と認知・判断に関するLT（chihiro.naganuma）の記録
  * [[CRM戦略 is 何 LT 2024年9月24日]](https://app.notion.com/p/medup/CRM-is-10b1da5a7f3b80a28f4cccebf378fd2b?pvs=25) : CRM戦略の概要・定義に関するLT（YugiShunsuke）の記録
  * [[TypeScript, React, Node.js]](https://app.notion.com/p/medup/TypeScript-React-Node-js-43c3efb039724c4484af73ab4b892df0?pvs=25) : TypeScript、React、Node.jsといった主要技術に関する開発WIKI
  * [[Architecture Design]](https://app.notion.com/p/medup/Architecture-Design-c88603c3fa914bbfbdd878e958d6c95f?pvs=25) : アーキテクチャ設計・システム設計に関するナレッジ・標準化ドキュメント
  * [[Team Development (Scrum, Agile, ...etc)]](https://app.notion.com/p/medup/Team-Development-Scrum-Agile-etc-75607dd5a70844fd8221e0af52079b96?pvs=25) : スクラムやアジャイル等のチーム開発手法に関するナレッジ
  * [[プロダクト(マネジメント・戦略)]](https://app.notion.com/p/medup/3669f99bcf594e1b8bf2eaa3881841e9?pvs=25) : プロダクトマネジメントやプロダクト戦略に関するナレッジ
  * [[util]](https://app.notion.com/p/medup/util-39a22d77803d419a88fe927542ec8080?pvs=25) : ユーティリティ機能や共通処理等に関する技術ナレッジ
  * [[管理ツール移行]](https://app.notion.com/p/medup/1dfb543560aa455a9c444c39627d3784?pvs=25) : Core Serviceプロジェクトにおける管理ツール移行に関する情報
  * [[アーカイブ]](https://app.notion.com/p/medup/2563bc87cf4543fa8988b17e8ce76a0a?pvs=25) : 旧Dev関連の不要になったドキュメントが格納されているアーカイブページ
  * [[DataOps アーカイブ]](https://app.notion.com/p/medup/DataOps-268e7b783e5549d8ab01dc31acbbd92f?pvs=25) : DataOps関連の不要になったドキュメントが格納されているアーカイブページ

---

## 02.Design
**ドキュメント名:** [02.Design](https://app.notion.com/p/medup/02-Design-3aee66c58191483cb6b114e70debc484)
**プラットフォーム:** Notion

### 概要
デザイン組織・チーム「02.Design」のポータルページ。「チーム」「プロダクト」「ナレッジ」の3つのカテゴリでドキュメントやリソースが集約されているほか、ページ内にデザインタスクを管理するカンバンボードや、直近1ヶ月のMTG議事録一覧（ビュー）が配置されている。

### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（カンバンの状態やアクセス権限など）**
  * ページ内にデザインタスク管理用のカンバンボードが設置されており、タスクのステータスが「ICEBOX」「要件定義」「未デザイン」「デザイン中」「確認待ち」「デザインQA」「Master反映」というフローで管理されていること。
  * 議事録の閲覧用として「MTG Minutes(1ヶ月最新)」および「チーム外MTG(1ヶ月最新)」のデータビューが用意されていること。
  * 現在「チーム外MTG(1ヶ月最新)」のビューについて、「このビューで参照されているデータベースへのアクセス権限がありません（アクセス不可）」という状態になっていること。

* **サブページやリスト項目から得られる情報（各カテゴリのドキュメント、運用ナレッジなど）**
  * **チーム**
    * [[MTG Minutes]](https://app.notion.com/p/medup/6e5ea36452694d9d9498124c91c89aca?v=168717532b404c38ac436c1373337344&pvs=25) : デザインチーム内の定例ミーティングや各種議事録・記録
    * [[チーム外MTG Minutes]](https://app.notion.com/p/medup/ef411093fe614b3a855b14cfafa8e3e3?v=7dde400fc8274e82831bf244abfdca70&pvs=25) : チーム外・他部署とのミーティングに関する議事録や記録
  * **プロダクト**
    * [[Spooon (medup DesignSystem)]](https://app.notion.com/p/medup/Spooon-medup-DesignSystem-23a6fc4d1eab447cba3c725aca68a5b5?pvs=25) : メダップのデザインシステム「Spooon」に関する仕様やガイドライン
    * [[Figma除くデザインファイル(Gdrive)]](https://drive.google.com/drive/folders/1_ygpL3iqAakFSBH3NAfgYhsFaxFUxT6i) : Figma以外のデザインリソース・素材が格納されているGoogle Driveへのリンク
  * **ナレッジ**
    * [[おすすめ記事・本・音声・動画]](https://app.notion.com/p/medup/3600184bc38d4ffc8fdd0432b6d071f1?v=035e2f76e3144ed3a5a545fa220d4c17&pvs=25) : デザインや関連技術・業務に関する学習・インプット用のおすすめコンテンツまとめ
    * [[フロントエンド開発のお約束]](https://app.notion.com/p/de5fbf8d96f6489e9fdbc5ab931ab168?pvs=24) : フロントエンド開発における実装ルールや、デザインチームとの連携・運用上のナレッジ
  * **アーカイブ**
    * [[Archives]](https://app.notion.com/p/medup/Archives-8b73e84482c74c4780d9ba7ccac779a9?pvs=25) : 過去の不要になったデザイン関連情報のアーカイブページ

---

## 04.foro CRM
**ドキュメント名:** [04.foro CRM](https://app.notion.com/p/medup/04-foro-CRM-0e464442ff284798913a991395c5b487)
**プラットフォーム:** Notion

### 概要
医療機関向けCRMサービス「foro CRM」に関するプロジェクトポータルページ。全体像、Vision、ロードマップ、サービス紹介資料、仕様、開発バックログ、プロダクトOps（Problem / Solution Discovery）、開発・オンボーディング情報などを集約している。「foroシリーズ（foro連携マップ、外来適正化など）」としてのページ統合計画等についても記載されている。

### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（プロジェクトの運用方針や前提事項）**
  * このページ内には最新情報と更新されていない情報が混じっている可能性があること
  * foro CRM単体ではなく、foro連携マップや外来適正化など「foroシリーズ」全体でのまとめ・ページ統合の必要性が高まっていること
  * 「foro 外来適正化」が「foro CRMのオプション」という位置づけであること
  * ユーザー事例シェアのミーティングが「毎月第3木曜日」に実施されていること
  * VoC開発予定が「2026年3月」に計画されていること

* **全体資料（全体像・Vision・営業資料等）のサブページ・リストから得られる情報**
  * [[foro CRMの提供価値まとめ_20250807]](https://app.notion.com/p/medup/foro-CRM-_20250807-2481da5a7f3b80f3a8e9cb8cc6cf5d19?pvs=25) : foro CRMが提供する価値に関する2025年8月7日時点のまとめ情報
  * [[foro CRM (w/PS) のサマリ]](https://app.notion.com/p/medup/foro-CRM-w-PS-f4b956b2a3f74f3ba3624eb203ea4823?pvs=25) : PS（プロフェッショナルサービス）込みでのforo CRM全体のサマリ資料
  * [[プロダクト部門ロードマップ]](https://app.notion.com/p/1841da5a7f3b803f8eaac543d50be5d6?pvs=24) : foro CRMのVision達成に向けたプロダクト部門の中長期ロードマップ
  * [[サービス紹介資料 (drive.google.com)]](https://drive.google.com/drive/folders/1eN8Aocn3zHQ8-TkYecsuK4JmaiILBWJR) : foro CRMのサービス紹介資料（Google Driveリンク）
  * [[20241010_デモ説明]](https://app.notion.com/p/medup/20241010_-1181da5a7f3b80d2bcead0bf7eef0a79?pvs=25) : 2024年10月10日時点のプロダクトデモに関する説明資料
  * [[暗号鍵運用管理規程]](https://app.notion.com/p/medup/2711da5a7f3b8090a0b3cf71c956fdf1?pvs=25) : foro CRMにおけるセキュリティ運用に関する文書（暗号鍵の管理ルール等）
  * [[サービス紹介資料（foro 外来適正化用）]](https://drive.google.com/drive/folders/1pObW3mkXn3XexbJJT3RPNkCiXSXvozl3) : foro CRMオプションである外来適正化向けのサービス紹介資料
  * [[ユーザー事例シェア（毎月第３木曜日）]](https://app.notion.com/p/medup/2551da5a7f3b8070b56ce5814ef451ce?pvs=25) : 毎月開催されるユーザー事例共有に関する詳細や履歴

* **プロダクト(仕様等)のサブページ・リストから得られる情報**
  * [[VoC（CS用）]](https://app.notion.com/p/d857f41c1d1a485782a37f7f3cf24dcf?pvs=24) / [[VoC]](https://app.notion.com/p/medup/VoC-235f9d1561b64efea1cd365a0b7d3b8f?pvs=25) : 顧客からの要望やフィードバック（VoC: Voice of Customer）の集約情報
  * [[対応環境、ブラウザ]](https://app.notion.com/p/medup/1aa24b23027846ec9d9b4b08e9e54a93?pvs=25) : foro CRMがサポート・対応している利用環境や推奨ブラウザの仕様
  * [[セールス用セキュリティ要点まとめ]](https://app.notion.com/p/medup/b4a8e743bf8e4f5da30cc0e4f1491cce?pvs=25) : 営業（セールス）時に回答・活用するためのセキュリティ要件のまとめ
  * [[障害報告]](https://app.notion.com/p/medup/05531d92bc5345648a3ee150a7cddfa1?pvs=25) : システム障害に関する過去の報告履歴や対応記録
  * [[リリースノート]](https://app.notion.com/p/medup/e6a66d9e2de946feb8e1f87276ba101b?pvs=25) : 新機能リリースやアップデートに関する詳細な履歴情報
  * [[redash]](https://app.notion.com/p/medup/redash-92622b44e1cd471f8241cb8c33c906ba?pvs=25) : Redashを利用したデータ抽出・クエリに関する情報
  * [[開発バックログ一覧]](https://app.notion.com/p/medup/94aa7802a27748439110b86b10649d59?pvs=25) : 開発待ちのタスク、機能要望、改善リスト（バックログ）
  * [[Google Analytics Event]](https://app.notion.com/p/medup/Google-Analytics-Event-756ed2749b324228ba871a68865dc998?pvs=25) : GA（Google Analytics）で設定・取得しているイベントトラッキングの定義と仕様
  * [[UXライティングガイドライン]](https://app.notion.com/p/medup/UX-020a04348430432296748a4754c90ee6?pvs=25) : プロダクト内のUI/UX上のテキスト・文言に関するライティングルール
  * [[Looker Studio（カスタムダッシュボード）でforoのデータを利用するための仕組みと利用方法]](https://app.notion.com/p/medup/Looker-Studio-foro-1fe7ebaf837e41f58c9b42a46c61437d?pvs=25) : Looker Studioを用いたカスタムダッシュボード構築とデータ連携の手順

* **プロダクトOps・開発(Delivery)のサブページ・リストから得られる情報**
  * [[foro CRM ProblemDiscovery]](https://app.notion.com/p/medup/foro-CRM-ProblemDiscovery-63a4544de6c7485cbe867a73ed07e156?pvs=25) : 顧客課題の発見・特定（Problem Discovery）に関するプロセスやリサーチ結果
  * [[foro CRM SolutionDiscovery]](https://app.notion.com/p/medup/foro-CRM-SolutionDiscovery-a7504f72a71f41e5bd6bda5319462d0e?pvs=25) : 課題解決策の検証（Solution Discovery）に関するプロセスや結果
  * [[サイドプロジェクト]](https://app.notion.com/p/medup/e9392d589e724d1db0e09a258c243605?pvs=25) : メインのプロダクト開発外で進行している関連サイドプロジェクトの情報
  * [[開発チームオンボーディング]](https://app.notion.com/p/medup/74c648cc3155444ea2b4858216ca0c85?pvs=25) : 開発チームへ新規参画するエンジニア向けのオンボーディング資料・手順
  * [[foro CRMチーム参画初回キックオフテンプレート]](https://app.notion.com/p/medup/foro-CRM-57a07ab12217430faa8bdb51cde95ec7?pvs=25) : 新規メンバー参画時に実施するキックオフ用資料のテンプレート
  * [[スクラム]](https://app.notion.com/p/medup/e11fc846410541b786d34079f30e6f3b?pvs=25) : 開発チームにおけるスクラムイベント（スプリント、デイリー等）の運用ルール
  * [[wiki]](https://app.notion.com/p/medup/wiki-fbf9466ee4f041f4b1664135200e5cd0?pvs=25) : 開発者向けの技術的なナレッジやTipsをまとめたWiki
  * [[暫定ゴールデンパス]](https://app.notion.com/p/medup/40605ae2ef3d41b0b9c79b927b8516ce?pvs=25) : 開発環境の構築や主要な開発フローなどの標準的な手順・ガイド
  * [[コーディングエージェント活用促進プロジェクト]](https://app.notion.com/p/medup/21f1da5a7f3b80f58508d8f757c57d5a?pvs=25) : AIコーディング支援ツールの社内活用・導入促進に関する取り組み
  * [[IPsec-VPN 検証まとめ]](https://app.notion.com/p/medup/IPsec-VPN-2491da5a7f3b800389e8c452bc5a41d4?pvs=25) : IPsec-VPN接続に関するネットワーク技術検証のまとめ
  * [[IPアドレス制限をかける必要がある場合の対応について]](https://app.notion.com/p/medup/IP-25b1da5a7f3b8049ba05fda9e7b876eb?pvs=25) : 顧客からIPアドレス制限（アクセス制限）を求められた際の技術対応・運用ルール

---

## 04.foro 外来適正化
**ドキュメント名:** [04.foro 外来適正化](https://app.notion.com/p/medup/04-foro-2681da5a7f3b809d8ff5c961ee79fde2)
**プラットフォーム:** Notion

### 概要
医療機関向けサービス「04.foro 外来適正化」に関するプロジェクトの「WIP_Vision（柳内作成中 as of 20260116）」に基づく事業戦略（初期顧客・通常顧客ごとのセグメント、Value Proposition、重視すること／しないこと、ユーザー例）および、Data/CSE・CS部門が運用する各種ナレッジベース、データ定義、Ops関連ドキュメント等を集約したポータルページ。

### このページを参照すべきケース（得られる情報）
* **WIP_Vision（柳内作成中 as of 20260116）から得られる事業戦略および顧客ターゲット定義**
  * 初期顧客のセグメント：「企画能力が高めで、経営状態が悪くないトップ病院」および「外来適正化ニーズが高い」こと
  * 初期顧客へのValue Proposition：外来適正化の、体制・プロセスを、ユーザー病院内に構築すること（PS込みオンリー）
  * 初期顧客に対して重視すること：価値提供のために、必須となるデータや業務の見定め。そのために、価値を実現すること。振り返りを行い、今後のサービスとして必須・不要を峻別すること
  * 初期顧客に対して重視しないこと：先方情シスがデータ授受に自発的に動くことを期待すること（こちらが動くしかなさそう）
  * 初期顧客のユーザー例：熊本、新東京、音羽、有明
  * 通常顧客（PS込み想定）のセグメント：「外来適正化ニーズが高い病院」および「初期顧客でわかった、セグメントに合致する病院」
  * 通常顧客（PS込み想定）へのValue Proposition：外来適正化の、体制・プロセスを、（※原文途切れ）
  * 通常顧客に対して重視すること：契約前の提供範囲の合意。データ・フォーマットやExport・授受についての合意
  * 通常顧客に対して重視しないこと：特になし（空欄）
  * 通常顧客のユーザー例：まだいない

* **部門別まとめ（Data/CSE）のサブページから得られる情報**
  * [[foro CRM・外来適正化のデータ投入 ユーザー相談用資料マスタ]](https://app.notion.com/p/medup/foro-CRM-2831da5a7f3b803daf01f110c9c41306?pvs=25) : システムへのデータ投入にあたり、ユーザー病院と相談する際に使用する資料のマスタ情報
  * [[外来適正化のデータの流れとOps]](https://app.notion.com/p/medup/Ops-2d41da5a7f3b8066b2b8eca510d55bf1?pvs=25) : 外来適正化プロジェクトにおけるデータフローや運用オペレーション（Ops）の具体的な手順
  * [[診療行為とEFファイル(レセプト)の読み替えナレッジ]](https://app.notion.com/p/medup/EF-2e91da5a7f3b8029a379cf571fa9e7e4?pvs=25) : 診療行為とEFファイル（レセプトデータ）を紐付ける・読み替えるためのナレッジ・ノウハウ
  * [[DirectCloud トライアル情報]](https://app.notion.com/p/medup/DirectCloud-2fc1da5a7f3b8083bf55f0a0d135b7d8?pvs=25) : クラウドストレージ「DirectCloud」のトライアル環境に関する情報
  * [[Tableauまとめ]](https://app.notion.com/p/medup/Tableau-3881da5a7f3b80b591a7e4159b858e7d?pvs=25) : Tableauを利用したデータ可視化やダッシュボードに関するまとめ情報
  * [[データ定義]](https://app.notion.com/p/medup/3a61da5a7f3b807ab2f5c96e09dc4283?pvs=25) : 外来適正化プロジェクトで扱うデータの定義や仕様

* **部門別まとめ（CS）のサブページから得られる情報**
  * [[foro 外来適正化 CS履歴]](https://app.notion.com/p/medup/foro-CS-2261da5a7f3b80478c98f3d2f66bb3b0?pvs=25) : 外来適正化サービスのカスタマーサクセス（CS）における顧客対応履歴や議事録
  * [[済生会熊本病院_外来適正化PJT]](https://app.notion.com/p/medup/_-PJT-61d32d8602544c4b956b3a73ae4d5883?pvs=25) : 初期顧客ユーザー例の「熊本」に該当する、済生会熊本病院との外来適正化プロジェクト個別情報

---

## 04.refery
**ドキュメント名:** [04.refery](https://app.notion.com/p/medup/04-refery-3affae5e901e46d0b07fabb55c64ed97#main)
**プラットフォーム:** Notion

### 概要
医療機関向けプロダクト「04.refery」に関する総合ポータルページ。AIが全文検索する際のインデックスとして、「過去の事業戦略（2023年OKR、MRR60万円目標など）」、「プロダクトの仕様・開発情報（ユビキタス言語集、ワーディングルール、リリースノート）」、「ビジネス面での重要決定（refery開発停止に伴う現ユーザーへの案内内容）」、および「医療・予約ドメインの知識（紹介業務フロー、WEB予約過去調査）」といった広範なナレッジが集約されている。
また、ページ下部には「refery Dev バックログ FY23 4Q」として開発カンバンが直接表示されており、UI崩れ（必須アイコンの縦書き化）や新機能（予約履歴、通知機能など）の具体的なタスクが管理されている。

### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（プロジェクトの運用方針やカンバンの状態）**
  * ページ上部に「以下は更新されていない可能性あり。Notionの作成者・最終更新者に確認が必要」という運用上の警告事項が明記されていること。
  * ページに関わっているメンバーとして、「KoyamaYoshiki」「ShigemitsuYosuke」「ObataShota」「Shota Kurokawa」「Takayuki Guzman」「moeko natsui」「satoshi.nakamatsu」などの名前が記録されていること。
  * 「プロダクトカンバン（refery Dev バックログ FY23 4Q）」が設置されており、タスクの進捗ステータスとして「新規」「アイスボックス 14」「バックログ 4」「要件定義 0」「デザイン待ち 0」「デザイン中 0」「デザイン確認待ち 0」「開発待ち 0」「見積・設計中 0」「開発中 0」「QA中 0」という区分で管理されていること。
  * カンバン上の具体的なタスク名として、「[崩れ]質問文が長い時、必須アイコンが縦書きになる」「ログ運用方針を決める システム」「予約の履歴機能を追加 5 プロダクト」「通知機能を追加 5 プロダクト」「【シフト】勤務条件が紐づいていない医師のシフトも調整できるように見える 2 プロダクト」「ユーザーへのフィードバック用レポートのデータ取得方法を構築する システム」「ログイン画面のデザイン差異の解消」「テナントユーザーは診療科に予約条件を設定しないで予約を受けることができる 2 プロダクト」「ライブラリのバージョンアップ関連 2 システム」「e2eの導入 2 システム」が直接テキストで記載されていること。

* **サブページやリスト項目から得られる情報（各カテゴリのドキュメント、運用ナレッジなど）**
  * **ピン留め**
    * [[referyの全体像_20230413]](https://app.notion.com/p/medup/refery-_20230413-3aeef58640fb413cbccf30ccce4a376d?pvs=25) : 2023年4月13日時点でのreferyの全体像やコンセプトについてわかること
    * [[refery_202305-07_OKR]](https://app.notion.com/p/medup/refery_202305-07_OKR-1063d1fddd6645bf88ba129c186a4719?pvs=25) : 2023年5月〜7月期におけるreferyチームのOKR（目標と成果指標）についてわかること
    * [[refery 事業戦略 ver.1]](https://app.notion.com/p/medup/refery-ver-1-f46d17303c2a4ac5b16fa1bfe1e75d2b?pvs=25) : referyの事業戦略（第1版）に関するドキュメント
    * [[refery 事業戦略 ver.2]](https://app.notion.com/p/medup/refery-ver-2-a937ab46e97e4dfbbda8539b3a5e7af1?pvs=25) : referyの事業戦略（第2版）に関するドキュメント
    * [[refery 事業戦略 ver.3]](https://app.notion.com/p/medup/refery-ver-3-edb211568a424ca6a4314403bd033cc6?pvs=25) : referyの事業戦略（第3版）に関するドキュメント
    * [[2023年4月末までの歩み]](https://app.notion.com/p/medup/2023-4-d659e381546c451db21b79d1a9efa355?pvs=25) : 2023年4月末までのプロジェクトの軌跡や経緯のまとめ
    * [[PMM/PdM共有ミーティング]](https://app.notion.com/p/medup/PMM-PdM-d76d8c5c14d44b03bf9c597fdd75cbbb?pvs=25) : PMM（プロダクトマーケティング）とPdM（プロダクトマネジメント）間の共有・連携ミーティングの記録
    * [[2025_福寿会病院PoC]](https://app.notion.com/p/medup/2025_-PoC-20f1da5a7f3b809fb84ce1f6576fdecc?pvs=25) : 2025年に実施または計画された「福寿会病院」でのPoC（概念実証）に関する情報
  * **プロダクト**
    * [[プロダクトマネジメント]](https://app.notion.com/p/medup/eec75a9de0b7499dab5ded3a1d7cb884?pvs=25) : プロダクトマネジメント業務全般に関するドキュメント
    * [[refery: 開発 - トップページ]](https://app.notion.com/p/medup/refery-d407dc9a1859455fad66bb6b6a71d3a1?pvs=25) : refery開発チームのトップページポータル
    * [[デザイン]](https://app.notion.com/p/medup/88992679755d47999290b98046908ec6?pvs=25) : デザイン関連の仕様やリソースに関するドキュメント
    * [[referyワーディングルール]](https://app.notion.com/p/medup/3a84ded059f14dbebe2f749293ab9b23?v=e4a74e00ad224abdb8598359319d4253&pvs=25) : プロダクト内で使用するテキスト・文言（ワーディング）のルール
    * [[ヘルプセンター]](https://app.notion.com/p/medup/f88a70ee93f244458e0cc0f6b0453a82?pvs=25) : ヘルプセンターの構築・運用に関するドキュメント
    * [[refery ユビキタス言語集]](https://app.notion.com/p/medup/refery-d14f512dcff7433db89eb054136519ed?pvs=25) : チーム共通で用いる専門用語やユビキタス言語の定義まとめ
    * [[ライティング (Spooon)]](https://app.notion.com/p/79659ee5780a4935b5e26eff65c30123?pvs=24) : プロダクト内のライティング（Spooon関連）に関するルール・ドキュメント
    * [[refery リリースノート]](https://app.notion.com/p/medup/eb08e5f4780e4929b1171b7a74e73782?v=2391da5a7f3b8013b379000c512868db&pvs=25) : これまでの機能追加や修正などのリリース履歴
    * [[OLD]](https://app.notion.com/p/medup/OLD-b6e91cf8924749a3bffe5f5df6c77793?pvs=25) : 古いプロダクト情報のアーカイブ
  * **ビジネス**
    * [[事業仮説]](https://app.notion.com/p/medup/7a6393ea58224470baa7b6f3f3bbf61c?pvs=25) : refery事業における仮説構築のまとめ
    * [[refery登り方]](https://app.notion.com/p/medup/refery-88d1ccb5a6694abd9136716fddfb8519?pvs=25) : 事業・プロジェクトのマイルストンや成長プロセス（登り方）に関する戦略
    * [[50施設アタック！！]](https://app.notion.com/p/medup/50-fd4ed363261144abac847bc9a831b82e?pvs=25) : 50施設への営業・導入アプローチに関する戦略や行動記録
    * [[202305-07_refery_CPF/PSF検証]](https://app.notion.com/p/medup/202305-07_refery_CPF-PSF-2e5c47be834049d383304aeaf0d48836?pvs=25) : 2023年5月〜7月にかけて実施されたCPF/PSFの検証に関する記録
    * [[Road to MRR600,000円]](https://app.notion.com/p/medup/Road-to-MRR600-000-5f6441be414641f29fd6274aef527275?pvs=25) : MRR（月次経常収益）60万円達成に向けたロードマップや施策
    * [[refery開発停止に伴う現ユーザーへの案内内容(全体方針含む)]](https://app.notion.com/p/medup/refery-1a4deda231df4c7ca3dfb846ee895eb6?pvs=25) : プロダクトの開発停止が決定した際の、既存ユーザーへの案内内容や全体方針のドキュメント
    * [[old]](https://app.notion.com/p/medup/old-a05f6b89dd654959becc001d03eb998c?pvs=25) : 古いビジネス関連情報のアーカイブ
  * **ドメイン知識キャッチアップ**
    * [[既存プロダクトリサーチ]](https://app.notion.com/p/medup/acd9cbf48f6c4023a1cb66d07322043f?pvs=25) : 関連する既存プロダクトや他社サービスの調査情報についてわかること
    * [[WEB予約過去調査・検討まとめ]](https://app.notion.com/p/medup/WEB-b011f70ee07e4028ae098e6b5577247e?pvs=25) : WEB予約機能に関する過去の調査内容や検討事項のまとめ
    * [[紹介業務フロー]](https://app.notion.com/p/medup/231f983b1f07430dbf0dd2af604a829d?pvs=25) : 医療機関における「紹介業務」の具体的なフローや手順といった業務知識
    * [[インタビューまとめ]](https://app.notion.com/p/medup/e99c4d55a4634f53a640a6c38340d469?pvs=25) : 顧客や関係者に対して実施したインタビュー結果のまとめ
  * **プロダクトカンバン（開発タスク）**
    * [[refery Dev バックログ FY23 4Q]](https://app.notion.com/p/medup/dd799b46f34a4292a8545eb1ab5670e1?v=ac14b113dbc843469a382c0f77e1bf26&pvs=18) : 2023年度第4四半期の開発バックログボード全体
    * [[[崩れ]質問文が長い時、必須アイコンが縦書きになる]](https://app.notion.com/p/medup/480a6b48084f4c1e969e8105b5114a0b?pvs=25) : UI崩れ（必須アイコンの縦書き）に関する修正タスク
    * [[ログ運用方針を決める]](https://app.notion.com/p/medup/baed3ec8942a4accaadc0ce0179ed94b?pvs=25) : システムログの運用方針策定に関するタスク
    * [[予約の履歴機能を追加]](https://app.notion.com/p/medup/1fff1a3cc0c54d22aed18ebcda0ef11d?pvs=25) : プロダクトへの予約履歴機能の追加タスク
    * [[通知機能を追加]](https://app.notion.com/p/medup/944bdc13e9b74e44b54996d7d93ef6d2?pvs=25) : プロダクトへの通知機能追加タスク
    * [[【シフト】勤務条件が紐づいていない医師のシフトも調整できるように見える]](https://app.notion.com/p/medup/ea061cf408274228b447229046aa4be0?pvs=25) : 医師のシフト調整機能における仕様・表示の修正タスク
    * [[ユーザーへのフィードバック用レポートのデータ取得方法を構築する]](https://app.notion.com/p/medup/15e3bf47356f46208ec0af2a468388ef?pvs=25) : レポート機能に必要なデータ取得ロジック構築のシステムタスク
    * [[ログイン画面のデザイン差異の解消]](https://app.notion.com/p/medup/4a6ac0e981f5430c9b0dba44f4b8f307?pvs=25) : ログイン画面のデザイン修正タスク
    * [[テナントユーザーは診療科に予約条件を設定しないで予約を受けることができる]](https://app.notion.com/p/medup/a92a59665f714eb498321f767f527473?pvs=25) : 診療科の予約条件設定に関するプロダクト仕様変更タスク
    * [[ライブラリのバージョンアップ関連]](https://app.notion.com/p/medup/29e962b70ff44b3d9ef87475db2e82f0?pvs=25) : 利用ライブラリのバージョンアップ対応タスク
    * [[e2eの導入]](https://app.notion.com/p/medup/e2e-56db47309c6741b2861d7a6a80407bdf?pvs=25) : E2Eテスト環境の導入に関するシステムタスク
    * [[新規ページ]](https://app.notion.com/p/medup/159ca73742d34c00bacbc5101849d2bc?pvs=25) : 新規で作成されたタスクページ

---

## AI DPCコーディング_LP制作
**ドキュメント名:** [AI DPCコーディング_LP制作](https://app.notion.com/p/medup/AI-DPC-_LP-3c51da5a7f3b80f3be7dd87fbd9135e4)
**プラットフォーム:** Notion

### 概要
メダップの新規サービス「メダップ AI DPCコーディング」のランディングページ（LP）制作に関する要件定義および構成案ドキュメント。済生会熊本病院とのプレスリリース実施までを「初期リリース」と定義し、不信の払拭や競合（富士通、Ubie）を意識した情報設計・SEO方針、サービス名称の由来、LPの具体的なセクション構成（ヒーロー、課題、提供価値など）が詳細に記載されている。

### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（LP制作の事業戦略、方針、構成案など）**
  * 初期リリース方針（目的）：サービスサイトの最低要件である「不信の払拭」をメインとし、競合である富士通やUbieのサービスサイトと比較して情報（提供価値や導入の流れなど）に不足がないようにする戦略。
  * 初期リリース方針（SEO・マーケティング）：現状「DPCコーティング AI」の検索上位がUbieの独壇場であるため、LLMO/AIO対策やインフルエンサーへの早期アプローチを重視すること。
  * 初期リリース方針（劣後事項）：初期段階での「CVR向上」や実績・事例のアピールは優先度を下げ（捨てる）、代わりに「済生会熊本病院」の業務内容や他病院との違い、想定効果の調査を進めること。
  * サービス名称の決定理由：「メダップ AI DPCコーディング」という名称は、後発サービスとして何ができるか直感的に伝わることと、将来的な「AIシリーズ」への拡張期待感を作る目的があること。
  * LP構成案（1. ヒーロー）：「DPCコーディングをAIで最適化 〜限られた人員で、収益とコーディング品質を最大化〜」というキャッチコピー方針。
  * LP構成案（3. 課題提起）：病院側の課題として「欠員が出ても簡単には補充できない(Structure)」「品質のばらつき(Process)」「熟練者のレビューや医師確認の負荷が高い(Process)」「適正請求できず収益機会を逸している/残業の常態化(Outcome)」を取り上げること。
  * LP構成案（4. 提供価値）：ユーザーの業務が「AI結果のレビューを中心とした業務へと変革」すること。「日次データ提出」という内容はUbieと内容が被るためLPには入れないこと。

* **サブページやリスト項目から得られる情報（参考資料や競合リンクなど）**
  * [[メダップAI DPCコーディングサービス概要_20260816]](https://docs.google.com/presentation/d/1FGtLIOLpPGGX0iM71w_UxaNMeoq3LclFfPcqt8aN_OY/edit?usp=sharing) : 2026年8月16日時点のAI DPCコーディングサービスの概要資料
  * [[WIP_メダップAI DPCコーディング_ご提案_20260901_v2.pptx]](https://docs.google.com/presentation/d/1uwunzSufzGYDRORILgc3epIN9bKevqYR/edit?usp=sharing&ouid=116589248244187246615&rtpof=true&sd=true) : 2026年9月1日版のサービス提案資料（概要資料よりも分かりやすいと指定されている）
  * [[【完成版】referyガイドライン＆テンプレート ver230529 のコピー]](https://docs.google.com/presentation/d/1UK8MT9yK64IqjaCNwl96K5t8yEv3n12MqU70vb1YHhk/edit?usp=sharing) : デザインや構成の参考となるreferyのガイドライン・テンプレート
  * [[アクセス不可]](https://app.notion.com/p/3c91da5a7f3b80948615f55b1aaccba9?pvs=24#3c91da5a7f3b80948615f55b1aaccba9) : 権限等の理由によりアクセスできない参考資料リンク
  * [[grits DPCコーディングについて病院のサイトで初心者にもわかる簡単な説明を作りたい時のポイント - 医科・歯科専門HP制作会社…]](https://medical-grits.jp/news/column/6295/) : 外部のHP制作会社による、初心者向けにDPCコーディングを説明する際の参考記事
  * [[DPCサポーター]](https://intro.dr-ubie.com/products/dpc-supporter) : LP制作において比較対象・競合としてマークしているUbieの提供サービスサイトへのリンク
  * [[https://rdlp.jp/image/lp-archive/60309/full.png]](https://rdlp.jp/image/lp-archive/60309/full.png) : LPの「共同開発病院ロゴ枠」をデザインする際のイメージ参考画像URL

---

## data ops マニュアル/ルール
**ドキュメント名:** [data ops マニュアル/ルール](https://app.notion.com/p/medup/22c1da5a7f3b80b7bf2bfff24975a28e?v=22c1da5a7f3b806aa84d000c3fcc0fa7)
**プラットフォーム:** Notion

### 概要
Data Ops（データ運用）領域の業務フローやルールに関するドキュメントを集約したデータベース（リストビュー）ページ。AIが全文検索する際のインデックスとして、「医療機関や介護系の新規登録・クレンジング（名寄せ）判断基準」「入院データ・紹介データの投入手順」「CSからの非定常作業・修正依頼フロー」「解約後データの削除対応」「Data Opsチームの体制やリーダー業務」など、データ管理の実務に関する具体的なマニュアル群が関連ロールとともに網羅されている。

### このページを参照すべきケース（得られる情報）
* **ページ内の直接記載情報から得られること（データベースの管理項目と属性）**
  * 各マニュアルやルールに対して、「Data Opsオーナー」「Data Opsリーダー」「Data Opsメンバー」「CS」「MIS」といった業務ロール・権限のタグが付与されていること。
  * ドキュメントの対象業務として「入院データ投入」「紹介データ/逆紹介データ投入」「その他のデータ関連業務」「カスタムダッシュボード」といったカテゴリタグが付与されていること。
  * 各ドキュメントの最終更新・作成日付として、直近の2026/10/02から2025/07/29までの履歴が記録・管理されていること。

* **サブページやリスト項目から得られる情報（各マニュアル・ルールの詳細）**
  * [[介護系 登録の判断基準]](https://app.notion.com/p/medup/3971da5a7f3b8098abf9e58a09ab768a?pvs=25) : 介護系施設のデータをシステム登録する際の判断基準やルールについてわかること（対象：Data Opsオーナー、Data Opsリーダー、Data Opsメンバー）
  * [[データ修正依頼方法 for CS]](https://app.notion.com/p/medup/for-CS-3641da5a7f3b8083b04cf92dfadebb61?pvs=25) : CS（カスタマーサクセス）担当者がデータの修正を依頼する際の具体的な手順やフローについてわかること（対象：その他のデータ関連業務、CS）
  * [[CS→DataOpsへの非定常作業依頼（ID不要・洗替等）手順]](https://app.notion.com/p/medup/CS-DataOps-ID-3421da5a7f3b809a977dda73a82105eb?pvs=25) : CSからData Opsへ、ID不要や洗替などの非定常作業を依頼する際の手順についてわかること（対象：CS、Data Opsリーダー）
  * [[医療機関の新規登録依頼対応手順]](https://app.notion.com/p/medup/2711da5a7f3b802e9406c62a108954cf?pvs=25) : 外部から医療機関のデータを新規登録する依頼があった際の対応フローについてわかること
  * [[医療機関の新規登録手順]](https://app.notion.com/p/medup/2711da5a7f3b8054af42fd69aedd36ba?pvs=25) : 医療機関のデータを新規登録する際の具体的な実務・操作手順についてわかること
  * [[入院データ投入手順]](https://app.notion.com/p/medup/2651da5a7f3b80c99cf8e94756160a4a?pvs=25) : 入院データをシステムへ投入する際の具体的な手順・マニュアル（対象：入院データ投入、Data Opsリーダー、Data Opsオーナー）
  * [[クレンジング時の医療機関重複登録時の業務フロー]](https://app.notion.com/p/medup/22c1da5a7f3b80b99717e15810373ac3?pvs=25) : データクレンジング（名寄せ）において、医療機関の重複登録が発覚した際のMIS・Data Ops間の業務フローについてわかること（対象：MIS、Data Opsオーナー、Data Opsリーダー、Data Opsメンバー）
  * [[Data Ops 全体業務フロー俯瞰図]](https://app.notion.com/p/medup/Data-Ops-2321da5a7f3b80269b61d426b86b558d?pvs=25) : 紹介データ/逆紹介データ投入、その他のデータ投入を含めたData Ops業務全体のフローを俯瞰したドキュメント（対象：Data Opsオーナー、Data Opsリーダー、Data Opsメンバー）
  * [[Data Opsチームの体制/役割]](https://app.notion.com/p/medup/Data-Ops-23a1da5a7f3b8058bd70dbce7f9dc95d?pvs=25) : Data Opsチーム内の体制や、オーナー・リーダー・メンバーそれぞれの役割定義についてわかること
  * [[Data Opsリーダー業務マニュアル]](https://app.notion.com/p/medup/Data-Ops-23f1da5a7f3b8003b09afb7ab95fd76e?pvs=25) : Data Opsリーダーが担うデータ投入や管理業務に関する詳細なマニュアル（対象：紹介データ/逆紹介データ投入、入院データ投入、その他のデータ投入）
  * [[ユーザーデータのセキュリティ規定草案]](https://app.notion.com/p/medup/2491da5a7f3b80c5acb2ed9904ce4c0b?pvs=25) : ユーザーデータを扱う上でのセキュリティ規定の草案についてわかること
  * [[解約済み/デモ終了後のユーザーのデータ及び環境削除の暫定対応]](https://app.notion.com/p/medup/2621da5a7f3b80f29d9fed4af6c652e9?pvs=25) : サービス解約済み、またはデモ終了後のユーザーデータ・システム環境の削除に関する暫定的な対応ルールについてわかること（対象：その他のデータ関連業務、Data Opsオーナー）
  * [[クレンジングルール ver1.0]](https://app.notion.com/p/medup/ver1-0-2641da5a7f3b80869a01ec2c6fb68649?pvs=25) : データクレンジング（データ整形・名寄せ）を実施する際の基準となるルールブック（ver1.0）
  * [[データソース×ユーザーデータ×ロール整理]](https://app.notion.com/p/medup/2851da5a7f3b804c9f12c0030b44a04f?pvs=25) : データソースとユーザーデータ、およびそれらにアクセスするロール（権限）の関連性を整理したマトリックス・ドキュメント
  * [[クレンジング全体の流れと手順]](https://app.notion.com/p/medup/28c1da5a7f3b8065a677e14adb6c36dc?pvs=25) : データクレンジング業務の全体フローと具体的な作業手順についてわかること
  * [[クレンジングフロー【介護系の登録判断】（確認中）]](https://app.notion.com/p/medup/36f1da5a7f3b804ab27bfc87bb249c2a?pvs=25) : クレンジング業務における、介護系施設の登録判断に特化したフロー（確認中ステータス）についてわかること
  * [[クレンジングフロー【受領データが閉院IDのケース】（作成中）]](https://app.notion.com/p/medup/ID-36f1da5a7f3b80d3bfb8c1f4c7037704?pvs=25) : クレンジングにおいて、受領データが既に閉院しているIDと合致した場合の処理フロー（作成中ステータス）についてわかること
  * [[Tableauの管理方法]](https://app.notion.com/p/medup/Tableau-3741da5a7f3b80509e42edd129e80956?pvs=25) : カスタムダッシュボード等で利用するTableauのシステム管理方法や運用ルールについてわかること（対象：カスタムダッシュボード、CS、Data Opsオーナー）
  * [[data integration asanaチケット発行手順]](https://app.notion.com/p/medup/data-integration-asana-3ea1da5a7f3b808e9feed5685f42c95b?pvs=25) : data integration関連のタスク管理において、Asanaのチケットを発行・起票する際の手順についてわかること

---

## VoC（CS用）
**ドキュメント名:** [VoC（CS用）](https://app.notion.com/p/medup/d857f41c1d1a485782a37f7f3cf24dcf?v=32a1da5a7f3b80649cd3000c8e0b594e)
**プラットフォーム:** Notion

**概要**
医療機関向けサービス（foro CRM、foro 連携マップ、外来適正化など）に対する顧客やカスタマーサクセス（CS）・プロフェッショナルサービス（PS）担当者からの要望・フィードバック（VoC: Voice of Customer）を一覧管理するデータベースページ。レポートの集計条件や連携マップの検索UI改善、AIを用いた逆紹介リストのレコメンド機能など、多岐にわたる大量の機能改善要望が蓄積されている。

**このページを参照すべきケース（得られる情報）**
* **ページ内のテーブル（データベース）から得られる大まかな情報**
  * **各VoCの管理項目:** 要望内容、入力日、入力者、対象機能（連携先カルテ、レポート、連携マップ、ナビイ連携等）、ユーザーの所属・役職（医事課、連携室、経営企画室等）、ユーザー背景、PdMコメント、対応ステータスなど。
  * **蓄積されている要望の主なカテゴリ（概略）:**
    * **レポート・データ関連:** CSV出力時のファイル名指定、年度指定・月平均・移動平均などの集計・期間指定の高度化、エリア構成比率の可視化。
    * **連携マップ・UI改善:** 住所検索時の自院非表示、エコー検査などの特定条件検索、印刷機能・QRコード・患者用画面の追加、検索条件項目のUI最適化。
    * **AI・逆紹介機能:** AIによる逆紹介先候補の事前リスト化やレコメンド表示、複雑な条件でのスピーディーな検索機能の追加。
    * **その他システム改善:** ナビイ連携の外国語対応、データの実態に合わせたユーザー側での手動編集機能、ラベルのデフォルト設定変更など。
* **サブページやリスト項目から得られる情報**
  * 本ページ内に固有のサブページリンクは存在しない（※「VoCユニット整理用」「機能絞ったもの」などの別ビュータブを用いて、用途に応じたデータベースの表示切り替えが可能）。
  