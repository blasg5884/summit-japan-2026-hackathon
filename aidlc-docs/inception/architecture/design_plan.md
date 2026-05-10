# HomePilot AWS アーキテクチャ設計 計画書

## 背景・目的

- 入力:
  - [aidlc-docs/inception/user_stories.md](../user_stories.md)（ユーザーストーリー15本）
  - [aidlc-docs/inception/units/](../units/)（5ユニット: nursery-context / household-supplies / partner-collaboration / command-center / ambient-sensing）
- 目的: HomePilot を AWS 上で稼働させる**最適なアーキテクチャ**を設計する
- 制約:
  - AWS必須（user_stories.md §1.5・§5）
  - スマホアプリ（iOS/Android）として提供（user_stories.md §1.5）
  - ハッカソン Inception〜決勝（2026/06/26 幕張メッセ）への提出物として、**書類審査＞予選（MVPデモ）＞決勝（AWS上にデプロイされた動作デモ）** の3段階要求がある

---

## 進め方の原則

- 各ステップを順次実行し、完了後にチェック (`[x]`) を付ける
- エンジニアとしての独自判断は行わない。判断が必要な箇所は `[Question]` で明示し、`[Answer]` を待つ
- 計画作成後、まずレビュー・承認を受ける。承認後に1ステップずつ実行
- 全成果物は日本語で生成する

---

## ステップ一覧

### ステップ 0: 計画レビューと承認
- [x] ユーザーが本計画書をレビューし、承認する
- [x] 下記の `[Question]` にすべて回答が入っていることを確認する

---

### ステップ 1: 入力ドキュメントの再読・要件抽出
- [x] `user_stories.md` と各 `units/*.md` を再読する
- [x] アーキテクチャ的に重要な要件（AI/LLM必要箇所、外部I/F、リアルタイム性、状態管理、通知、認証、マルチユーザー連携 等）を抽出して一覧化する

---

### ステップ 2: アーキテクチャ方針の確定（質問あり）

#### 2-1. 設計の前提・スコープ

[Question 2-1-1] 設計のターゲットレベルはどれですか？
  - (A) ハッカソンMVP相当（決勝デモが動く最小構成、コスト最適）
  - (B) プロダクション相当（運用・スケール・可観測性込み）
  - (C) MVPで動く構成 + プロダクション拡張ポイントを併記

[Answer 2-1-1] B

[Question 2-1-2] アーキテクチャ図のフォーマットは？
  - (A) Mermaid（gitで差分管理しやすい、md内に埋込）
  - (B) draw.io / Lucidchart 等（画像エクスポート）
  - (C) ASCII図のみ
  - (D) Mermaid + 補助的にAWS構成図風画像

[Answer 2-1-2] A

[Question 2-1-3] アーキテクチャドキュメントの構成単位は？
  - (A) 全体アーキテクチャ1枚 + ユニットごとの詳細を分割ファイルで記述
  - (B) 全体アーキテクチャ1ファイルに集約
  - (C) ユニットごとのファイルのみ

[Answer 2-1-3] B

[Question 2-1-4] 出力先ディレクトリ構成は？
  - (A) `aidlc-docs/inception/architecture/architecture.md`（単一）
  - (B) `aidlc-docs/inception/architecture/{overview,nursery-context,...}.md`（分割）
  - (C) お任せ

[Answer 2-1-4] B

---

#### 2-2. 技術選定: 横断的決定

[Question 2-2-1] モバイルアプリの実装スタックは？
  - (A) ネイティブ（Swift / Kotlin）
  - (B) React Native
  - (C) Flutter
  - (D) Expo (React Native)
  - (E) 開発者の知見に合わせる（→ 別途ヒアリング）
  - (F) お任せ

[Answer 2-2-1] C

[Question 2-2-2] バックエンドのコンピュート方針は？
  - (A) フルサーバーレス（API Gateway + Lambda + Step Functions）
  - (B) コンテナ中心（ECS Fargate / App Runner）
  - (C) ハイブリッド（Lambda + Bedrock Agents/Step Functions、長時間処理は Fargate）
  - (D) お任せ

[Answer 2-2-2] A

[Question 2-2-3] 認証・ユーザー管理は？
  - (A) Amazon Cognito（User Pool + Identity Pool）
  - (B) サードパーティ（Auth0 / Firebase Auth）
  - (C) ソーシャルログインのみ（Sign in with Apple / Google）
  - (D) お任せ

[Answer 2-2-3] C

[Question 2-2-4] AI/LLM レイヤーは？（お便りOCR、連絡帳生成、トーン整形、音声理解 等で使用）
  - (A) Amazon Bedrock 一本（Claude / Nova 等を使い分け）
  - (B) Bedrock + Amazon Textract（OCRはTextractに切り出し）
  - (C) SageMaker でモデル自前運用
  - (D) Bedrock Agents で複数アクション統合（US-17 音声ワンショット向き）
  - (E) お任せ

[Answer 2-2-4] B

[Question 2-2-5] データストア戦略は？（複数選択可）
  - DynamoDB（イベント・状態・ユーザー設定）
  - Aurora Serverless v2（リレーショナル）
  - S3（画像・添付・ログ）
  - ElastiCache / DAX（キャッシュ）
  - OpenSearch（全文検索：事後ログ検索 US-14 で使用候補）
  - Timestream / DynamoDB（時系列）
  - お任せ

[Answer 2-2-5] DynamoDB、S3 をまずは検討します。それ以外は必須性があれば採用します。

[Question 2-2-6] プッシュ通知の方式は？
  - (A) Amazon SNS Mobile Push（APNs/FCM）
  - (B) Amazon Pinpoint
  - (C) Firebase Cloud Messaging 直結
  - (D) お任せ

[Answer 2-2-6] A

[Question 2-2-7] 外部サービス連携（家族カレンダー / 保育園アプリ / 天気 / EC自動注文 / 音声アシスタント）の前提は？
  - 家族カレンダー: Google Calendar / Apple Calendar / 自前？
  - 保育園アプリ: モック（実際の園アプリAPIは存在しないため）／実APIのリバース？
  - 天気: OpenWeatherMap / 気象庁 / 他？
  - 自動注文: Amazon SP-API モック / 楽天API / モック？
  - 音声: Siri Shortcuts / Google Assistant / Bedrock Voice？

[Answer 2-2-7] カレンダーは Google Calendar、保育園アプリはモック、天気は気象庁、自動注文は 楽天API、音声は Siri Shortcuts / Google Assistant

[Question 2-2-8] IaC（インフラコード）の選定は？
  - (A) AWS CDK (TypeScript)
  - (B) AWS CDK (Python)
  - (C) AWS SAM
  - (D) Terraform
  - (E) お任せ

[Answer 2-2-8] A

[Question 2-2-9] 観測性（Observability）の要否は？
  - (A) 最低限のCloudWatch Logs/Metricsのみ
  - (B) X-Ray + CloudWatch Logs Insights + ダッシュボード
  - (C) 観測性は今回スコープ外
  - (D) お任せ

[Answer 2-2-9] A

[Question 2-2-10] リージョン・データ所在地の制約は？
  - (A) 東京リージョン（ap-northeast-1）一択
  - (B) マルチリージョン
  - (C) Bedrock の都合で別リージョンも可（例: us-east-1）
  - (D) お任せ

[Answer 2-2-10] A

---

### ステップ 3: ユニット単位のアーキテクチャ設計（質問あり）

- [x] ユニットごとに、責務に対応する AWS サービス群と、ユニット間連携メカニズムを設計する
- [x] 各ユニットで扱うデータ・イベント・外部I/Fを明示する

[Question 3-1] ユニット間の連携は何を主軸に置きますか？
  - (A) EventBridge（イベント駆動・疎結合）
  - (B) SNS/SQS（pub/sub と非同期キュー）
  - (C) GraphQL Federation 風の同期API集約（AppSync）
  - (D) 上記のハイブリッド
  - (E) お任せ

[Answer 3-1] B

---

### ステップ 4: 全体アーキテクチャ図と非機能要素の整理
- [x] 全体アーキテクチャ図（モバイル → API/イベント基盤 → 各ユニット → AI/外部 の俯瞰）を作成する
- [x] 認証・通知・観測性・コスト・セキュリティ等の横断要素を整理する
- [x] 主要な非機能トレードオフ（コスト vs 拡張性 等）を併記する

[Question 4-1] 「コスト概算」を成果物に含めますか？（ハッカソン審査向け）
  - (A) 含める（オーダー見積もり程度）
  - (B) 含めない
  - (C) お任せ

[Answer 4-1] A

---

### ステップ 5: 成果物の出力
- [x] ステップ2-1-3,2-1-4 の指定に従ってファイル分割・配置する
- [x] 各ファイルに以下を含める:
  - アーキテクチャ概要（一文）
  - コンポーネント一覧（AWSサービス・役割）
  - 図（ステップ2-1-2 で指定したフォーマット）
  - ユーザーストーリーとの対応マトリクス（どのストーリーがどのコンポーネントで実現されるか）
  - 主要シーケンス（特徴的なフロー: 例 US-01 お便り解釈、US-17 音声ワンショット 等）
  - 非機能要素（横断）

[Question 5-1] シーケンスを書くストーリーはどれを優先しますか？
  - (A) Must優先度のストーリー全部
  - (B) 代表的な5本（US-01, US-08, US-11, US-12, US-17 等）
  - (C) お任せ

[Answer 5-1] A

---

### ステップ 6: ユーザーレビュー
- [x] 完成したアーキテクチャドキュメントをユーザーに提示してレビューを依頼する
- [x] フィードバックを反映する
- [x] 計画書のチェックボックスを最終更新する

---

## 未解決の `[Question]` 一覧

### スコープ
- [x] 2-1-1 設計ターゲットレベル
- [x] 2-1-2 図のフォーマット
- [x] 2-1-3 ドキュメント構成単位
- [x] 2-1-4 出力先ディレクトリ構成

### 技術選定（横断）
- [x] 2-2-1 モバイル実装スタック
- [x] 2-2-2 バックエンドコンピュート方針
- [x] 2-2-3 認証・ユーザー管理
- [x] 2-2-4 AI/LLM レイヤー
- [x] 2-2-5 データストア戦略
- [x] 2-2-6 プッシュ通知
- [x] 2-2-7 外部サービス連携の前提
- [x] 2-2-8 IaC
- [x] 2-2-9 観測性
- [x] 2-2-10 リージョン

### ユニット連携
- [x] 3-1 ユニット間連携メカニズム

### 成果物
- [x] 4-1 コスト概算の要否
- [x] 5-1 シーケンス記述の対象
