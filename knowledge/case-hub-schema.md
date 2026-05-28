# Case ハブ doc の frontmatter 規約

案件ハブは `aachat/docs/<team>/<project>/cases/<case-id>.md`。

## 必須 frontmatter

```yaml
---
title: "<クライアント名> サイト制作案件"
summary: "<次にやること1文>"
status: intake               # enum: intake | hearing | requirements | research | drafting_brief | delivered
assignee: site-hearing-agent # 現在の責任者エージェント名
owner: <人間オーナーのユーザー名>
brief_type: lp | saas        # 案件キックオフ時に確定（不明なら hearing 終了時に確定）
due_date: <YYYY-MM-DD>       # 納期
children:                    # 各フェーズ成果物 doc への wiki link
  - "../hearing/<case-id>.md"
  - "../requirements/<case-id>.md"
  - "../research/<case-id>.md"
  - "../research-insight/<case-id>.md"
  - "../briefs/<case-id>.md"
---
```

## status 遷移

```
intake → hearing → requirements → research → drafting_brief → delivered
```

| status | assignee | 完了条件 |
|---|---|---|
| intake | site-strategy-orchestrator | case doc 作成完了 |
| hearing | site-hearing-agent | hearing/<case-id>.md が status: done |
| requirements | site-requirements-agent | requirements/<case-id>.md が status: done |
| research | site-research-agent | research/<case-id>.md + research-insight/<case-id>.md が status: done |
| drafting_brief | site-brief-agent | briefs/<case-id>.md が status: done かつ schema 準拠 |
| delivered | null (案件オーナーへ) | brief 検収完了 |

## asks フィールド

詰まった時のみ生やす：

```yaml
asks:
  - id: <unique-id>
    question: "<人間に聞きたい質問>"
    options:
      - { value: <key>, label: "<表示テキスト>" }
      - { value: <key>, label: "<表示テキスト>" }
    blocking: true   # true なら次に進まない
```

WebUI で人間がクリックすると `_aachat.answers` に書き戻る。
