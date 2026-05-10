# API仕様（OpenAPI）／ ER図 設計 計画書

## 背景・目的

- 入力: [user_stories.md](../user_stories.md) / [units/](../units/) / [architecture.md](../architecture/architecture.md)
- 目的:
  - HomePilot が公開する **REST API の仕様**を OpenAPI で定義する
  - DynamoDB の **データモデル**を ER 図で可視化する
- 制約:
  - ハッカソン Inception フェーズ成果物として、書類審査での技術的具体性を補強する
  - architecture.md（§9 データモデル概要、§7 ストーリー対応マトリクス、§8 シーケンス）と整合させる

---

## 進め方の原則

- 各ステップは順次実行し、完了後にチェック（`[x]`）
- 独自判断は行わない。`[Question]` に `[Answer]` が入るまで進めない
- 計画はまずレビュー・承認を受ける

---

## ステップ一覧

### ステップ 0: 計画レビューと承認
- [x] ユーザーが本計画書をレビューし、承認する
- [x] 下記の `[Question]` にすべて回答が入っていることを確認する

---

### ステップ 1: スコープと出力形式の確定（質問）

#### 1-1. OpenAPI

[Question 1-1-1] OpenAPI のバージョンは？
  - (A) OpenAPI 3.1（最新、JSON Schema 2020-12 互換）
  - (B) OpenAPI 3.0.3（API Gateway インポート互換性が高い）
  - (C) お任せ

[Answer 1-1-1] A

[Question 1-1-2] ファイルフォーマットは？
  - (A) YAML
  - (B) JSON
  - (C) 両方

[Answer 1-1-2] A

[Question 1-1-3] スコープに含めるストーリーは？
  - (A) Must優先度 のみ（API化が必要な8本）
  - (B) Must + Should（より広範）
  - (C) 全API化対象ストーリー（背景cron系・SQS連携系を除く全部）
  - (D) お任せ

[Answer 1-1-3] B

[Question 1-1-4] エラーレスポンスの記述粒度は？
  - (A) Happy path のみ
  - (B) 主要エラー（400/401/403/404/409/500）込み、共通エラースキーマ定義
  - (C) お任せ

[Answer 1-1-4] B

[Question 1-1-5] 認証方式の表現は？
  - (A) Cognito JWT Bearer のみ（`securitySchemes: bearerAuth`）
  - (B) Cognito + APIキー（Siri Shortcuts 等のサービス間用に検討）
  - (C) お任せ

[Answer 1-1-5] A

[Question 1-1-6] ファイル分割方針は？
  - (A) 単一ファイル `aidlc-docs/inception/api-and-data/openapi.yaml`
  - (B) ユニット別に分割（`openapi-nursery-context.yaml` ほか）
  - (C) Components 共通化のみ別ファイル、本体は単一
  - (D) お任せ

[Answer 1-1-6] A

#### 1-2. ER 図

[Question 1-2-1] ER図のフォーマットは？
  - (A) Mermaid `erDiagram`（md内に埋込）
  - (B) draw.io / dbdiagram.io 等の画像
  - (C) Mermaid + 補助の表

[Answer 1-2-1] A

[Question 1-2-2] ER図の表現対象は？（DDBは単一テーブル設計なので2解釈ある）
  - (A) **論理ER**（家族・ユーザー・タスク・お便り… の関係を「もし RDB だったら」風に表現。可読性重視）
  - (B) **DDB 単一テーブル物理表現**（PK/SK・GSI 設計の物理構造）
  - (C) 両方（論理ER → 物理マッピング表）

[Answer 1-2-2] B

[Question 1-2-3] ER図ファイル配置は？
  - (A) `aidlc-docs/inception/api-and-data/data-model.md`
  - (B) architecture.md §9 を拡張
  - (C) お任せ

[Answer 1-2-3] A

---

### ステップ 2: API インベントリ（エンドポイント抽出）
- [x] architecture.md §6（ユニット別マッピング）と §8（シーケンス）から、外部公開すべきエンドポイントを抽出
- [x] HTTPメソッド／パス／用途 を一覧化
- [x] ユーザーロール（mom / dad / commander）でのアクセス権を明示

---

### ステップ 3: OpenAPI ドキュメント作成
- [x] ステップ1の回答に従い、`info` / `servers` / `securitySchemes` / `paths` / `components.schemas` を作成
- [x] 各エンドポイントに対応する US-ID をコメント／`tags` で紐付け
- [x] エラースキーマを共通化（指定があれば）

---

### ステップ 4: ER 図作成
- [x] ステップ1-2の回答に従い、論理／物理／両方 のいずれかを作成
- [x] architecture.md §9 のエンティティ一覧と整合
- [x] DDB単一テーブル物理表現を含む場合: PK/SK/GSI の設計を併記

---

### ステップ 5: 整合性チェック
- [x] OpenAPI で定義した path と architecture.md §8 のシーケンス内 path が一致
- [x] ER 図のエンティティと OpenAPI の `schemas` が対応する（フィールド名・型）
- [x] §7 ストーリー対応マトリクスのエンドポイントが OpenAPI に存在する

---

### ステップ 6: ユーザーレビュー
- [x] 完成物をユーザーに提示し、レビューを依頼
- [x] フィードバック反映
- [x] 計画チェックボックスを最終更新

---

## 未解決の `[Question]` 一覧

### OpenAPI
- [x] 1-1-1 OpenAPI バージョン
- [x] 1-1-2 ファイルフォーマット
- [x] 1-1-3 スコープ
- [x] 1-1-4 エラーレスポンスの粒度
- [x] 1-1-5 認証方式の表現
- [x] 1-1-6 ファイル分割

### ER 図
- [x] 1-2-1 フォーマット
- [x] 1-2-2 表現対象（論理／物理／両方）
- [x] 1-2-3 ファイル配置
