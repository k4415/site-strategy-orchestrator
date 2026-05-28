---
name: progress-check
description: 案件の現在状態を読み、次にやるべきアクションを返す。「進捗どう？」「今どこ？」「ステータス確認」などのトリガーで起動。
---

# progress-check

案件ハブ doc を読み、現状サマリと次アクションを返す。

## いつ起動するか

- 「進捗どう？」「今どこ？」「ステータス確認」と聞かれた時
- 案件オーナーから状況問い合わせがあった時
- `/loop` で定期実行する設定の時

## 実行手順

1. **case doc 読み込み**
   `cases/<case-id>.md` を読み、frontmatter から以下を取得：
   - status
   - assignee
   - children（各成果物 doc）
   - asks（人間判断待ちのもの）

2. **現在のフェーズで動いているか確認**
   - assignee の session 状態を `aachat session list --agent <assignee> --project <project>` で確認
   - last_updated が 30 分以上前なら「stalled」と判定

3. **各成果物 doc の status 確認**
   - children のリンク先 doc を読む
   - 各 status を表にまとめる

4. **次アクション提示**
   - 全 done で次フェーズへ進めるなら → `phase-handoff` を呼ぶ
   - 現フェーズが進行中なら → 「<assignee> が working」と返す
   - stalled の場合 → `recovery` skill を呼ぶ
   - asks が blocking なら → 「人間判断待ち」と返す

## 出力フォーマット

```markdown
## <案件名> 進捗

- 現在: <status> （<assignee>）
- 開始からの経過: <X 時間>
- 成果物:
  - hearing/<case-id>.md: done
  - requirements/<case-id>.md: in_progress
  - research/<case-id>.md: 未着手
  - briefs/<case-id>.md: 未着手
- 次アクション: <具体的に何が次に動くか>
- 人間判断待ち: <あれば asks の概要>
```

## NG

- 全 doc を読まずに「進んでいます」とだけ返す
- stalled 判定で recovery を呼ばない
