# Recovery Playbook

各エージェントが詰まった時の判断フロー。

## 完了通知が来ない

1. `aachat session list --project <project> --agent <target>` で session 状態確認
2. 30 分以上 turn が更新されていない → `aachat session stop <session-id>` で停止
3. case doc に状況を追記し、新規 session を `aachat session run` で起動
4. 同じエージェントで 3 回失敗 → `asks` で人間判断を仰ぐ

## カバレッジが上がらない（hearing）

site-hearing-agent から「クライアントがこれ以上答えられない」と通知が来た場合：

1. coverage の必須セル（business_overview / jtbd_decision_moment / customer_pains / competitive_alternatives / offer）が埋まっているか確認
2. 埋まっていれば次フェーズへ進む（completeness を犠牲にする判断）
3. 必須セルが欠けていれば `asks` で「この情報なしで進めるか」を人間に確認

## リサーチで競合が見つからない

site-research-agent から「research_needs に挙がった競合がスクレイピング不能 / 該当競合不在」と通知された場合：

1. requirements-agent に research_needs の再生成を依頼（session run）
2. それでも見つからない → `asks` で「市場カテゴリの再定義が必要かもしれない」と人間に確認
3. 部分的なリサーチだけでブリーフに進む判断もあり（research-insight に明示）

## brief のスキーマ準拠失敗

site-brief-agent から「必須フィールドが上流 doc から取得不能」と通知された場合：

1. 不足フィールドを確認
2. hearing / research に該当情報があるはず → brief-agent に再 merge を依頼
3. それでも欠ける → requirements-agent or hearing-agent に再依頼（再ヒアリング）
4. 「ない情報は埋められない」と判明 → `asks` で「該当フィールドを除いて納品するか」を確認

## エスカレーションの最終手段

すべての自動リカバリーで詰まったら：

```yaml
# case doc の frontmatter に追加
asks:
  - id: escalation_<phase>_<n>
    question: "<状況の簡潔な説明> ── どう進めますか？"
    options:
      - { value: retry_same, label: "もう一度同じエージェントを起動" }
      - { value: skip_phase, label: "このフェーズをスキップして次に進む" }
      - { value: shrink_scope, label: "スコープを縮小して再開" }
      - { value: pause, label: "一旦中断（オーナーが個別対応）" }
    blocking: true
```

人間が答えるまで `phase-handoff` は動かさない。
