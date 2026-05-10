# HomePilot

> AWS Summit Japan 2026 AI-DLC ハッカソン提出プロジェクト
> テーマ「人をダメにするサービス」 — 家庭を自動操縦するスマホアプリ

---

## コンセプト

共働き世帯の意思決定者（ペルソナ: 中村美咲・36歳・PM）が抱える本質的悩みは家事ではなく「**家庭全体の意思決定負荷**」。HomePilot は、家庭運営の判断・調整・実行を AI が代行し、ユーザーを徹底的に「考えなくていい人」にダメにする。

「**判断を奪う／調整を奪う／記憶を奪う**」をオプトインで完全自動運転まで踏み込めるよう設計。

---

## Inception フェーズ成果物

```
aidlc-docs/inception/
├── user_stories.md            ユーザーストーリー15本／6エピック
├── user_stories_plan.md
├── units/                     境界づけられたコンテキスト（5ユニット）
│   ├── nursery-context.md     保育園との双方向I/O
│   ├── household-supplies.md  物資の期日逆算
│   ├── partner-collaboration.md 夫婦協働・自走化
│   ├── command-center.md      司令塔体験
│   ├── ambient-sensing.md     環境信号トリガ
│   └── units_plan.md
├── architecture/
│   ├── architecture.md        AWS アーキテクチャ設計
│   └── design_plan.md
└── api-and-data/
    ├── openapi.yaml           OpenAPI 3.1（25エンドポイント）
    ├── data-model.md          DynamoDB 単一テーブル物理設計
    └── plan.md
```

ペルソナ定義: [persona.md](persona.md)

---

## ペルソナ → ストーリー → ユニット → アーキの流れ

| フェーズ | アウトプット |
|---|---|
| **ペルソナ定義** | 中村美咲（36歳・共働きPM・6歳/2歳の子持ち）／本質悩み「意思決定負荷」 |
| **ユーザーストーリー** | Connextra＋AC＋優先度＋ダメにする軸（判断/調整/記憶/実行）の15本 |
| **ユニット分割** | サブドメイン軸で5ユニット、ユニット間は SNS/SQS の参照リンクで疎結合 |
| **AWS アーキテクチャ** | フルサーバーレス（API GW + Lambda + Step Functions）／Bedrock + Textract／DDB Single-Table |
| **API・データ** | OpenAPI 3.1（Cognito JWT 認証・25エンドポイント）／DDB 物理設計（PK/SK/GSI/TTL） |

---

## 主要技術スタック

| レイヤー | 採用 |
|---|---|
| モバイル | Flutter（iOS/Android 単一コード） |
| Edge | CloudFront + WAF + API Gateway HTTP API |
| 認証 | Amazon Cognito User Pool + Apple/Google Federation |
| コンピュート | AWS Lambda + Step Functions + EventBridge Scheduler |
| ユニット間連携 | Amazon SNS（topic/unit）+ SQS（subscriber/DLQ） |
| AI/OCR | Amazon Bedrock（Claude Haiku/Sonnet）+ Amazon Textract |
| データ | Amazon DynamoDB（Single-Table）+ S3 |
| プッシュ | SNS Mobile Push（APNs/FCM） |
| 外部連携 | Google Calendar / 気象庁 API / 楽天 API / Siri Shortcuts / Google Assistant |
| IaC | AWS CDK（TypeScript） |
| リージョン | ap-northeast-1（東京） |

---

## 5 つの境界づけられたコンテキスト

| ユニット | 担当ストーリー |
|---|---|
| `nursery-context` | US-01 / US-02 / US-06 / US-07 |
| `household-supplies` | US-03 / US-04 |
| `partner-collaboration` | US-08 / US-09 / US-10 / US-15 |
| `command-center` | US-11 / US-12 / US-14 |
| `ambient-sensing` | US-16 / US-17 |

---

## ハッカソン評価軸とのトレース

| 評価観点 | 対応 |
|---|---|
| ビジネス意図(Intent)の明確さ | [persona.md](persona.md) / [user_stories.md §1](aidlc-docs/inception/user_stories.md) |
| Unit分解の適切さ | [units/](aidlc-docs/inception/units/) で5ユニット境界＋参照リンク |
| 創造性・テーマ適合性 | 各ストーリーに「ダメにする軸」、オプトイン完全自動の踏み込み |
| AI-DLCプロセス実践 | 各フェーズに `plan.md` を残し、ステップごとの Q&A で意思決定を記録 |
