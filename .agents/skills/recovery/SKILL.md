---
name: recovery
description: エージェント停滞・エラー時のリカバリ判断と人間エスカレーション。「stalled」「エラー」「3回失敗」などのトリガーで起動。
---

# recovery

エージェントが詰まった時の判断フロー。

## いつ起動するか

- `progress-check` が stalled を検知した時
- 前段エージェントからエラー通知が来た時
- 同じ session run が 3 回連続で失敗した時

## 実行手順

1. **状況把握**
   - case doc を読む
   - 該当エージェントの直近 session を `aachat session read <session-id>` で確認
   - エラーメッセージ / 詰まりのパターンを抽出

2. **`knowledge/recovery-playbook.md` の判定フローに沿って分類**
   - 完了通知が来ない
   - カバレッジが上がらない（hearing）
   - リサーチで競合が見つからない
   - brief のスキーマ準拠失敗
   - 想定外のエラー

3. **自動リカバリーを試す**
   - 1 回目: session stop → fresh session run（同じ依頼）
   - 2 回目: 依頼本文を縮小（スコープを狭める）して session run
   - 3 回目: 別エージェントに代替依頼（例: brief-agent が詰まったら requirements-agent に戻して再ヒアリング依頼）

4. **3 回失敗したら `asks` を生やす**
   - case doc の frontmatter に `asks` 配列を追加
   - 質問 + 選択肢（retry_same / skip_phase / shrink_scope / pause）を用意
   - blocking: true で phase-handoff を止める

5. **オーナーに通知**
   - `aachat project send <project> "@<owner> case <case-id> で recovery が必要です。詳細: <doc-link>" --via claude-code`

## NG

- 詰まりの原因を読まず無限リトライ
- `asks` を生やさず勝手にスコープを変える
- オーナー通知を忘れて自動進行を続ける
