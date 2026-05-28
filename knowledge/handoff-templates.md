# Handoff Templates

`aachat session run` で各エージェントに依頼する時の本文テンプレ。manual.md §3.7 の 3 要素（動詞+目的、context wiki link、期待成果物）に準拠。

## ヒアリング起動

```bash
aachat session run site-hearing-agent.<owner> --project <project> "
クライアント <name> のヒアリングを開始してください。

context:
- 案件ハブ: [[aachat/docs/<team>/<project>/cases/<case-id>.md]]
- 想定 brief_type: <lp|saas|未確定>

期待する成果物:
- [[aachat/docs/<team>/<project>/hearing/<case-id>.md]] に発話ログ + 構造化要約を保存
- coverage が 80% 以上、followup_questions が 3 件以下になったら status: done
- 完了したら cases/<case-id>.md の status を requirements、assignee を site-requirements-agent に更新
"
```

## 要件整理起動

```bash
aachat session run site-requirements-agent.<owner> --project <project> "
ヒアリング結果を要件定義に整理してください。

context:
- 案件ハブ: [[aachat/docs/<team>/<project>/cases/<case-id>.md]]
- 入力: [[aachat/docs/<team>/<project>/hearing/<case-id>.md]]

期待する成果物:
- [[aachat/docs/<team>/<project>/requirements/<case-id>.md]] を保存
- research_needs（競合候補・市場リサーチ論点・VOC キーワード）を必ず含める
- brief_type の暫定確定（決められない場合は asks で人間判断）
- 完了したら cases/<case-id>.md の status を research、assignee を site-research-agent に更新
"
```

## リサーチ起動

```bash
aachat session run site-research-agent.<owner> --project <project> "
要件定義の research_needs に従ってリサーチを実行してください。

context:
- 案件ハブ: [[aachat/docs/<team>/<project>/cases/<case-id>.md]]
- 入力: [[aachat/docs/<team>/<project>/requirements/<case-id>.md]]

期待する成果物:
- [[aachat/docs/<team>/<project>/research/<case-id>.md]] に事実レポートを保存
- [[aachat/docs/<team>/<project>/research-insight/<case-id>.md]] に考察レポートを保存
- 競合LP 3〜5本、市場リサーチ、VOC（X 経由）、提供資料解析を並列で実施
- 完了したら cases/<case-id>.md の status を drafting_brief、assignee を site-brief-agent に更新
"
```

## ブリーフ起動

```bash
aachat session run site-brief-agent.<owner> --project <project> "
要件定義とリサーチ結果を統合し、戦略ブリーフを生成してください。

context:
- 案件ハブ: [[aachat/docs/<team>/<project>/cases/<case-id>.md]]
- 要件: [[aachat/docs/<team>/<project>/requirements/<case-id>.md]]
- リサーチ事実: [[aachat/docs/<team>/<project>/research/<case-id>.md]]
- リサーチ考察: [[aachat/docs/<team>/<project>/research-insight/<case-id>.md]]
- スキーマ: [[aachat/docs/agent-development/site-creation-suite/schemas/brief-schema.md]]

期待する成果物:
- [[aachat/docs/<team>/<project>/briefs/<case-id>.md]] に YAML frontmatter + Markdown 本文で保存
- brief-schema.json 準拠（validate_brief.py でチェック）
- provenance フィールドで上流 4 doc に wiki link を張る
- 完了したら cases/<case-id>.md の status を delivered に、orchestrator に完了通知
"
```

## 重複起動チェック（必須）

`aachat session run` の前に必ず実行：

```bash
aachat session list --project <project> | jq '.data.sessions[] | select(.agent == "<対象エージェント>")'
```

該当エージェントの running session があれば、`session send <session-id>` で追加依頼するか、stop してから fresh で起動する。
