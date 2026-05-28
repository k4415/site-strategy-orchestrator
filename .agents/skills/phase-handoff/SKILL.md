---
name: phase-handoff
description: 前段エージェントの成果物が done になった時、次エージェントを aachat session run で起動する。「フェーズ完了」「次に進める」などのトリガーで起動。
---

# phase-handoff

前段成果物 doc を検証し、次フェーズのエージェントを session run で起動する。

## いつ起動するか

- 前段エージェントから「<doc> を保存しました、status: done です」と通知された時
- `progress-check` skill が「次に進むべき」と判定した時

## 実行手順

1. **前段成果物の検証**
   - 該当 doc を読む
   - frontmatter の `status: done` を確認
   - `unresolved_questions` や `followup_questions` が許容範囲か（フェーズごとの基準は `knowledge/recovery-playbook.md`）

2. **case doc 更新**
   - `cases/<case-id>.md` の `status` を次フェーズ enum に更新
   - `assignee` を次エージェント名に更新
   - `children` 配列に該当成果物 doc を追加

3. **重複起動チェック**
   ```bash
   aachat session list --project <project> --agent <next-agent>
   ```
   running session があれば判断：継続するか stop して fresh で起動するか

4. **依頼本文を組み立てる**
   - `knowledge/handoff-templates.md` の該当フェーズテンプレを使う
   - case_id / wiki link を埋める

5. **session run で起動**
   ```bash
   aachat session run <next-agent> --project <project> "<組み立てた依頼本文>"
   ```

6. **完了ログ**
   - case doc の本文に「YYYY-MM-DD HH:MM: <prev-agent> → <next-agent> ハンドオフ完了」と追記

## フェーズ遷移表

| 完了 doc | 次の status | 次の assignee |
|---|---|---|
| hearing/<case-id>.md | requirements | site-requirements-agent |
| requirements/<case-id>.md | research | site-research-agent |
| research/<case-id>.md + research-insight/<case-id>.md | drafting_brief | site-brief-agent |
| briefs/<case-id>.md | delivered | (案件オーナーへ通知) |

## NG

- 前段 doc が `status: done` でないのに次に進める
- 同一エージェントの session を確認せず重複起動する
- 依頼本文に wiki link を含めず曖昧な指示で投げる
- `--via claude-code` を付け忘れる
