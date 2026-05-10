# Unit: ambient-sensing

## 目的
位置・天気・音声などの環境信号を起点に、美咲がトリガを引かなくても他ユニットの機能を自動発火させるコンテキスト。

## 対象ユーザーストーリー
- US-16: 位置・天気コンテキストトリガ自動実行
- US-17: 音声ワンショット代行

---

## ストーリー詳細

### US-16: 位置・天気コンテキストトリガ自動実行
- **As a** 美咲
- **I want** 位置イベント（保育園着・帰宅・夫の退社）と天気イベント（雨・暑熱・寒波予報）に連動して、必要な準備や調整が自動発火してほしい
- **So that** 「あ、〇〇しなきゃ」「今日雨だから長靴…」と自分で気づく作業すらやめたい
- **AC**:
  - 位置トリガ: 保育園到着→「登園済」自動更新／お迎え予測時刻を夫に共有／帰宅検知→ 夕飯導線（解凍・着替え・お風呂順序）を提示／夫の退社検知→ お迎えバトン引継ぎ通知（オプトイン時は美咲を経由せず直接）
  - 天気トリガ: 雨予報→ 長靴・カッパを翌日持ち物リスト（→ [household-supplies.md](household-supplies.md) の US-03）に自動追加し玄関配置リマインド／猛暑予報→ 帽子・大型水筒・冷感タオル追加／寒波予報→ 防寒着追加。前夜と当朝の2回通知
  - 「お迎えバトン引継ぎ」は [partner-collaboration.md](partner-collaboration.md) の US-08（夫へのタスクアサイン）でお迎えタスクが夫に割り当て済みであることを前提とする
  - 各トリガは領域別オプトイン制御（→ [command-center.md](command-center.md) の US-12）に従う
- **優先度**: Should

### US-17: 音声ワンショット代行
- **As a** 美咲
- **I want** 手が塞がっている朝に「HomePilotに依頼: 〇〇発熱」と1フレーズ言うだけで、欠席連絡・夫通知・予定再調整が並列実行されてほしい
- **So that** 子を抱えながらスマホを操作する地獄から解放されたい
- **AC**:
  - スマホ音声アシスタント連携で起動可能
  - 1音声トリガで複数アクション（園連絡＋夫通知＋カレンダー再調整＋必要物の自動補充）が並列発火
  - 完了は通知またはホーム画面（→ [command-center.md](command-center.md) の US-11）で進捗表示
  - 実行内容は事後報告ログ（→ [command-center.md](command-center.md) の US-14）に残る
- **優先度**: Should

---

## 本ユニットから他ユニットへの参照（呼び出し方向）
- US-16 → [household-supplies.md](household-supplies.md) (US-03 翌日持ち物リスト)、[partner-collaboration.md](partner-collaboration.md) (US-08 お迎えタスクアサイン前提)、[command-center.md](command-center.md) (US-12 オプトイン制御)
- US-17 → [nursery-context.md](nursery-context.md) (US-07 欠席連絡)、[partner-collaboration.md](partner-collaboration.md) (US-08 夫通知)、[household-supplies.md](household-supplies.md) (US-04 必要物自動補充)、[command-center.md](command-center.md) (US-11 進捗表示・US-14 ログ記録)

## 他ユニットからの参照
- 本ユニットは下流の各ユニットを呼び出す側であり、他ユニットから本ユニットへの直接参照は無い。
