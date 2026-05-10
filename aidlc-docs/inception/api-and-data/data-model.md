# HomePilot DynamoDB データモデル（物理）

> 入力整合: [architecture.md §9 データモデル概要](../architecture/architecture.md) / [openapi.yaml](openapi.yaml)
> スコープ: **DDB 単一テーブル物理表現**（PK/SK・GSI 設計の物理構造）

---

## 1. 設計方針

- **単一テーブル設計**（Single-Table Design）
- TableName: `HomePilot`
- 課金モード: On-Demand（PAY_PER_REQUEST）
- 暗号化: KMS-CMK
- DynamoDB Streams 有効化（NEW_AND_OLD_IMAGES）
- 一部エンティティに TTL 属性（短期キャッシュ系）

世帯（HOUSEHOLD）単位で全データをパーティション化することで、世帯内クエリを効率化し、世帯間 JOIN を不要化する。

---

## 2. 主キー設計

| 属性 | 型 | 説明 |
|---|---|---|
| `PK`（パーティションキー） | String | `HOUSEHOLD#<householdId>` |
| `SK`（ソートキー） | String | エンティティ種別＋ID（プレフィックス階層） |

---

## 3. グローバルセカンダリインデックス（GSI）

| GSI | PK | SK | 用途 |
|---|---|---|---|
| `gsi1` | `GSI1PK` | `GSI1SK` | 担当者 × 締切日（US-09 夫の自走化UI） |
| `gsi2` | `GSI2PK` | `GSI2SK` | 種別 × 時刻（US-14 事後報告ログ、US-01 お便り時系列、US-12 承認待ち時系列 等） |

---

## 4. アクセスパターン × インデックス

| # | アクセスパターン | 関連 US | 利用キー |
|---|---|---|---|
| 1 | 世帯メタ取得（含 commander） | 共通 / US-15 | PK=HOUSEHOLD#<hid>, SK=META# |
| 2 | 世帯ユーザー一覧 | 共通 | PK=HOUSEHOLD#<hid>, SK begins_with USER# |
| 3 | お便り一覧（時系列） | US-01 | gsi2: GSI2PK=TYPE#LETTER#<hid>, GSI2SK begins_with TS# |
| 4 | 単一お便り取得 | US-01 | PK=HOUSEHOLD#<hid>, SK=LETTER#<lid> |
| 5 | 当日タスク（夫向け） | US-09 | gsi1: GSI1PK=ASSIGN#<sub>, GSI1SK begins_with DUE#<today> |
| 6 | 持ち物リスト（指定日） | US-03 | PK=HOUSEHOLD#<hid>, SK=SUPLIST#<date> |
| 7 | 連絡帳下書き | US-06 | PK=HOUSEHOLD#<hid>, SK=DIARY#<date> |
| 8 | 承認待ち一覧（時系列） | US-12 | gsi2: GSI2PK=TYPE#APPROVAL#<hid>, GSI2SK begins_with TS# |
| 9 | 領域別オプトイン設定 | US-12 | PK=HOUSEHOLD#<hid>, SK=SET# |
| 10 | 事後報告ログ（時系列） | US-14 | gsi2: GSI2PK=TYPE#LOG#<hid>, GSI2SK begins_with TS# |
| 11 | 在庫予測 | US-04 | PK=HOUSEHOLD#<hid>, SK begins_with INV# |
| 12 | 抽出予定（指定日） | US-02 | gsi2: GSI2PK=TYPE#EVENT#<hid>, GSI2SK=DATE#<date> |
| 13 | 園連絡履歴 | US-07 / US-09 | gsi2: GSI2PK=TYPE#NURMSG#<hid>, GSI2SK begins_with TS# |
| 14 | 環境イベント時系列 | US-16 | gsi2: GSI2PK=TYPE#AMB#<hid>, GSI2SK begins_with TS# |

---

## 5. エンティティ × キーマップ

| エンティティ | SK プレフィックス | GSI1PK | GSI1SK | GSI2PK | GSI2SK | TTL |
|---|---|---|---|---|---|---|
| Household | `META#` | - | - | - | - | - |
| User | `USER#<sub>` | - | - | - | - | - |
| Letter | `LETTER#<lid>` | - | - | `TYPE#LETTER#<hid>` | `TS#<uploadedAt>` | - |
| ParsedEvent | `EVENT#<eid>` | - | - | `TYPE#EVENT#<hid>` | `DATE#<date>` | - |
| CalendarSync | `CALSYNC#<id>` | - | - | - | - | 90日 |
| Diary | `DIARY#<date>` | - | - | - | - | - |
| NurseryMessage | `NURMSG#<id>` | - | - | `TYPE#NURMSG#<hid>` | `TS#<sentAt>` | - |
| Task | `TASK#<tid>` | `ASSIGN#<sub>` | `DUE#<dueDate>` | `TYPE#TASK#<hid>` | `TS#<createdAt>` | - |
| TaskAssign | `ASSIGN#<aid>` | `ASSIGN#<sub>` | `DUE#<dueDate>` | - | - | - |
| SuppliesList | `SUPLIST#<date>` | - | - | - | - | 60日 |
| InventoryItem | `INV#<sku>` | - | - | - | - | - |
| Approval | `APP#<id>` | - | - | `TYPE#APPROVAL#<hid>` | `TS#<proposedAt>` | - |
| AutomationLog | `LOG#<ts>#<id>` | - | - | `TYPE#LOG#<hid>` | `TS#<executedAt>` | 365日 |
| UserSettings | `SET#` | - | - | - | - | - |
| AmbientEvent | `AMB#<ts>#<id>` | - | - | `TYPE#AMB#<hid>` | `TS#<occurredAt>` | 90日 |

---

## 6. ER 図（物理：テーブル間の論理関係 + キー設計）

```mermaid
erDiagram
    HOUSEHOLD ||--o{ USER : "メンバ"
    HOUSEHOLD ||--o{ LETTER : "受領"
    HOUSEHOLD ||--o{ DIARY : "下書き"
    HOUSEHOLD ||--o{ NURSERY_MESSAGE : "送信"
    HOUSEHOLD ||--o{ TASK : "保有"
    HOUSEHOLD ||--o{ TASK_ASSIGN : "アサイン"
    HOUSEHOLD ||--o{ SUPPLIES_LIST : "日次"
    HOUSEHOLD ||--o{ INVENTORY_ITEM : "在庫"
    HOUSEHOLD ||--o{ APPROVAL : "提案"
    HOUSEHOLD ||--|| USER_SETTINGS : "設定"
    HOUSEHOLD ||--o{ AUTOMATION_LOG : "自動実行"
    HOUSEHOLD ||--o{ AMBIENT_EVENT : "観測"
    HOUSEHOLD ||--o{ CALENDAR_SYNC : "同期"
    LETTER ||--o{ PARSED_EVENT : "抽出"
    USER ||--o{ TASK : "担当"
    USER ||--o{ TASK_ASSIGN : "受領"
    APPROVAL ||--o| AUTOMATION_LOG : "承認結果"
    TASK ||--o{ TASK_ASSIGN : "履歴"

    HOUSEHOLD {
        string pk PK "HOUSEHOLD#hid"
        string sk PK "META#"
        string commander "mom or dad, US-15"
        string handoverUntil "ISO datetime"
        string createdAt
    }
    USER {
        string pk PK "HOUSEHOLD#hid"
        string sk PK "USER#sub"
        string sub "Cognito sub"
        string role "mom or dad"
        string displayName
        string platformEndpointArn "SNS Mobile Push"
    }
    LETTER {
        string pk PK
        string sk PK "LETTER#lid"
        string gsi2pk "TYPE#LETTER#hid"
        string gsi2sk "TS#uploadedAt"
        string s3Key
        string status "ocr_pending or parsing or parsed or failed"
        number confidence
    }
    PARSED_EVENT {
        string pk PK
        string sk PK "EVENT#eid"
        string gsi2pk "TYPE#EVENT#hid"
        string gsi2sk "DATE#yyyy-mm-dd"
        string letterId FK "LETTER#lid"
        string title
        date eventDate
        string targetClass
    }
    DIARY {
        string pk PK
        string sk PK "DIARY#date"
        string body
        string status "draft or sent"
        boolean smoothedFromOriginal
    }
    NURSERY_MESSAGE {
        string pk PK
        string sk PK "NURMSG#id"
        string gsi2pk "TYPE#NURMSG#hid"
        string gsi2sk "TS#sentAt"
        string msgType "absent or late or pickup_change or custom or diary"
        string sender "mom or dad or system"
    }
    TASK {
        string pk PK
        string sk PK "TASK#tid"
        string gsi1pk "ASSIGN#sub"
        string gsi1sk "DUE#yyyy-mm-dd"
        string gsi2pk "TYPE#TASK#hid"
        string gsi2sk "TS#createdAt"
        string title
        string status "open or done or declined"
        string assignee "mom or dad or unassigned"
        boolean autoAssigned
    }
    TASK_ASSIGN {
        string pk PK
        string sk PK "ASSIGN#aid"
        string gsi1pk "ASSIGN#sub"
        string gsi1sk "DUE#yyyy-mm-dd"
        string taskId FK "TASK#tid"
        string status "proposed or approved or completed or declined"
    }
    SUPPLIES_LIST {
        string pk PK
        string sk PK "SUPLIST#date"
        json items
        string generatedAt
    }
    INVENTORY_ITEM {
        string pk PK
        string sk PK "INV#sku"
        string sku
        number unitsRemaining
        number consumptionPerDay
        date predictedEmptyAt
        boolean autoOrderEnabled
        number maxOrderJpy
    }
    APPROVAL {
        string pk PK
        string sk PK "APP#id"
        string gsi2pk "TYPE#APPROVAL#hid"
        string gsi2sk "TS#proposedAt"
        string domain
        string status "pending or approved or rejected or expired"
        json payload
    }
    AUTOMATION_LOG {
        string pk PK
        string sk PK "LOG#ts#id"
        string gsi2pk "TYPE#LOG#hid"
        string gsi2sk "TS#executedAt"
        string domain
        string action
        string status "executed or cancelled or failed"
        json details
    }
    USER_SETTINGS {
        string pk PK
        string sk PK "SET#"
        json automationByDomain "domain to autoApprove autoExecute exclusions"
        json notificationPreferences
    }
    AMBIENT_EVENT {
        string pk PK
        string sk PK "AMB#ts#id"
        string gsi2pk "TYPE#AMB#hid"
        string gsi2sk "TS#occurredAt"
        string source "geofence or weather or voice"
        json payload
    }
    CALENDAR_SYNC {
        string pk PK
        string sk PK "CALSYNC#id"
        string source "google"
        string status "ok or conflict"
        string externalEventId
    }
```

---

## 7. データ整合性・トランザクション

- 関連エンティティを同時更新する場合は `TransactWriteItems`
  - 例: Task 完了 → AutomationLog 追記（US-09 → US-14）
  - 例: Approval 承認 → AutomationLog 追記（US-12 → US-14）
- DynamoDB Streams から CDC Lambda が SNS topic に publish（ユニット間連携）

---

## 8. ライフサイクル

| エンティティ | TTL/保管 |
|---|---|
| Letter（画像） | S3 にライフサイクル: 180日後 IA、365日後 Glacier |
| AutomationLog | DDB TTL 365日 |
| AmbientEvent | DDB TTL 90日 |
| SuppliesList | DDB TTL 60日 |
| CalendarSync | DDB TTL 90日 |
| DLQ メッセージ | SQS DLQ で14日保持 → S3 アーカイブ |

---

## 9. キャパシティ・スケーリング

- On-Demand により世帯数増に対して透過的にスケール
- パーティション偏り対策: `HOUSEHOLD#<hid>` でユーザー分散済（世帯数 ≒ パーティション数）
- ホットパーティション懸念がある時系列ログ系（gsi2）は GSI 側を `TYPE#<...>#<hid>` とすることで世帯ごとに分散

---

## 10. OpenAPI スキーマとの対応

| OpenAPI スキーマ | DDB エンティティ | 主要マッピング |
|---|---|---|
| `Letter` / `ParsedLetter` | LETTER, PARSED_EVENT | `LETTER#<lid>` ＋ 関連 `EVENT#<eid>` |
| `DiaryDraft` | DIARY | `DIARY#<date>` |
| `NurseryMessage` | NURSERY_MESSAGE | `NURMSG#<id>` |
| `Task` | TASK | `TASK#<tid>`、gsi1 で当日担当検索 |
| `SuppliesList` / `SupplyItem` | SUPPLIES_LIST, INVENTORY_ITEM | `SUPLIST#<date>` ／ `INV#<sku>` |
| `SupplyOrder` | （イベントとして AUTOMATION_LOG に記録） | `LOG#<ts>#<id>` |
| `AutomationDomainSetting` / `AutomationSettings` | USER_SETTINGS | `SET#` 内 `automationByDomain` |
| `Approval` | APPROVAL | `APP#<id>` |
| `AutomationLog` / `AutomationLogPage` | AUTOMATION_LOG | `LOG#<ts>#<id>` ＋ gsi2 で時系列 |
| `HomeView` | （複数エンティティの集約ビュー） | TASK + AUTOMATION_LOG + APPROVAL を Lambda が集約 |
| `AmbientLocationEvent` | AMBIENT_EVENT | `AMB#<ts>#<id>` |
| `VoiceIntentResult` | AMBIENT_EVENT + AUTOMATION_LOG | 受領＝AMB、実行結果＝LOG |
