# Case ハブ doc の frontmatter 規約

案件ハブは `aachat/docs/<team>/<project>/cases/<case-id>.md`。

## 必須 frontmatter

```yaml
---
title: "<クライアント名> サイト制作案件"
summary: "<次にやること1文>"
status: hearing              # enum: intake | hearing | requirements | research | drafting_brief | delivered
assignee: site-hearing-agent # 現在の責任者エージェント名
owner: <人間オーナーのユーザー名>
brief_type: lp | saas        # 案件キックオフ時に確定（不明なら hearing 終了時に確定）

# 任意フィールド（hearing 進行に伴って hearing-agent が orchestrator 経由で追記）
due_date: <YYYY-MM-DD>       # 任意。詳細シート Q22 で hearing-agent が取得して追記
pre_shared_assets: []        # クライアント提供資料（URL リスト中心）
creative_assets: []          # 制作素材（ロゴ・ブランドガイド等）

children:                    # 各フェーズ成果物 doc への wiki link
  - "../hearing/<case-id>.md"
  - "../requirements/<case-id>.md"
  - "../research/<case-id>.md"
  - "../research-insight/<case-id>.md"
  - "../briefs/<case-id>.md"
---
```

## pre_shared_assets の構造

クライアントが事前共有する資料の URL リスト。hearing-agent の事前資料リクエスト（一括シート）で受領 → orchestrator が逐次追記。

```yaml
pre_shared_assets:
  - { type: existing_website, url: "https://example.com/", status: received, received_at: "2026-05-28T10:00:00Z" }
  - { type: product_catalog, url: "https://example.com/products", status: received }
  - { type: sales_deck, url: "https://drive.google.com/...", status: received }
  - { type: voc_review, url: "https://example.com/customer-voices", status: received }
  - { type: competitor_ref, url: "https://competitor-a.com/", status: received }
  - { type: competitor_ref, url: "https://competitor-b.com/", status: received }
  - { type: inspiration_like, url: "https://...", status: received }
  - { type: inspiration_dislike, url: "https://...", status: received }
  - { type: industry_report, url: "https://...", status: received }
  - { type: other, url: "https://...", status: received, description: "業務フロー図" }
```

`type` の enum:
- `existing_website` / `product_catalog` / `sales_deck` / `voc_review`
- `competitor_ref` / `inspiration_like` / `inspiration_dislike`
- `industry_report` / `other`

`status`:
- `pending`: 共有予定だが URL 未着
- `received`: 受領済み
- `parsed`: research-agent の client-asset-parse skill で構造化済み

## creative_assets の構造

サイト制作チーム向けの素材。hearing-agent が制作素材リクエスト（一括シート）で受領 → orchestrator が追記。

```yaml
creative_assets:
  - { type: logo, url: "https://drive.google.com/...", status: received }
  - { type: brand_guide, url: "https://...", status: received }
  - { type: brand_colors, value: "#1A73E8 / #F4B400 / #34A853", status: received }
  - { type: fonts, value: "Inter (heading) / Noto Sans JP (body)", status: received }
  - { type: product_images, status: pending, note: "撮影予定（6月中旬）" }
  - { type: video, url: "https://youtu.be/...", status: received }
  - { type: mandatory_badge, url: "https://...", description: "ISO27001 認証バッジ", is_mandatory: true, status: received }
  - { type: compliance, value: "特商法表記必須・薬機法対応必要", status: received }
```

`type` の enum:
- `logo` / `brand_guide` / `brand_colors` / `fonts`
- `product_images` / `video` / `shoot_planned`
- `mandatory_badge` / `compliance`

`status`:
- `pending`: 「これから作る・撮る」
- `received`: 受領済み
- `not_available`: 無い・該当しない（明示的に確認済み）

`is_mandatory: true` を指定すると brief-agent が `constraints.mandatory_elements` に取り込む。

## status 遷移

```
hearing → requirements → research → drafting_brief → delivered
```

`intake` は廃止（hearing-agent がいきなり hearing から開始）。

| status | assignee | 完了条件 |
|---|---|---|
| hearing | site-hearing-agent | hearing/<case-id>.md が status: done。pre_shared_assets / creative_assets も完了 |
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

## hearing-agent からの追記依頼パターン

hearing 進行中に case doc に追記する典型パターン（hearing-agent から session run で orchestrator を呼ぶ）:

### 事前資料受領後

```
cases/<case-id>.md の pre_shared_assets に以下を追記してください:
- { type: existing_website, url: '<url>', status: received }
- { type: competitor_ref, url: '<url>', status: received }
... (全件)
```

### 制作素材受領後

```
cases/<case-id>.md の creative_assets に以下を追記してください:
- { type: logo, url: '<url>', status: received }
- { type: brand_colors, value: '<#hex hex>', status: received }
... (全件)
```

### 詳細シート完了後（納期確定）

```
cases/<case-id>.md の due_date を '<YYYY-MM-DD>' に更新してください。
```
