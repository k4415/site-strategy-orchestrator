---
name: case-init
description: 案件キックオフ時に cases/<case-id>.md を作成する。「新規案件」「キックオフ」「<クライアント名>を開始」「case doc を作成」などのトリガーで起動。
---

# case-init

新規案件の hub doc を作成し、初期状態を設定する。

## いつ起動するか

- **hearing-agent から「新規案件の case doc を作成してください」と session run で依頼された時**（標準ルート）
- 人間から「新規案件 <name> を開始」「キックオフ」と直接依頼された時（API 起動ルート）
- 既存の `cases/<case-id>.md` が存在しない時のみ実行

## 入力

依頼本文から以下を抽出：
- クライアント名 / 案件名 / case_id 候補
- brief_type（`lp` / `saas`）
- 案件オーナー（人間）
- 現フェーズ（hearing-agent からの依頼なら `hearing` で進行中）

**納期（due_date）は必須ではない**。hearing-agent が詳細シート完了後に session send で追記してくる前提。

**人間から直接ではなく hearing-agent から呼ばれた場合、追加質問はせずに頂いた情報で case doc を作る**（クライアント対話の流れを止めない）。人間から直接の場合だけ、不足項目を確認する。

## 実行手順

### 1. case_id を決める

- クライアント名を kebab-case 化（例: `acme-saas-launch-2026q3`）
- 既存の cases/ 配下に同名がないか確認
- 衝突する場合は末尾に番号付与（`-2`, `-3`...）

### 2. frontmatter を組み立てる

`knowledge/case-hub-schema.md` の規約に従う：

```yaml
---
title: "<クライアント名> サイト制作案件"
summary: "ヒアリング進行中"
status: hearing
assignee: site-hearing-agent
owner: <human-username>
brief_type: <lp|saas>
# due_date は hearing 完了時に hearing-agent から追記される
pre_shared_assets: []
creative_assets: []
children: []
---
```

`due_date` は **入力にあれば** frontmatter に入れる、無ければ省略（空フィールドにしない）。

### 3. ファイル作成

- パス: `aachat/docs/<team>/<project>/cases/<case-id>.md`
- 本文: 案件概要、クライアント情報、ヒアリング開始メモ

```markdown
# <クライアント名> サイト制作案件

クライアント: <name>
brief_type: <lp|saas>
オーナー: <human>
キックオフ: <ISO8601>

ヒアリング進行中（site-hearing-agent が担当）。
```

### 4. hearing-agent からの依頼の場合 → 終了

- hearing-agent が既にクライアントと対話中なので、追加で session run しない
- case doc 作成完了を hearing-agent に通知して終了（自分は待機）

### 5. 人間からの直接キックオフの場合 → site-hearing-agent を起動

- `phase-handoff` skill を呼んで site-hearing-agent を session run で起動

### 判定の仕方

- 依頼本文に「私が継続中」「ヒアリング中」「hearing-agent から」などの記述がある → 4 へ
- そうでなければ → 5 へ

## 追記依頼（hearing-agent からの follow-up）

hearing-agent から以下のような session run が来たら case doc を更新する（case-init の責務範囲内）:

### pre_shared_assets 追記

```
cases/<case-id>.md の pre_shared_assets に以下を追記:
- { type: existing_website, url: '...', status: received }
- ...
```

→ frontmatter の `pre_shared_assets` 配列に append。重複（同じ type + url）は弾く。

### creative_assets 追記

```
cases/<case-id>.md の creative_assets に以下を追記:
- { type: logo, url: '...', status: received }
- ...
```

→ frontmatter の `creative_assets` 配列に append。同様に重複弾き。

### due_date 追記

```
cases/<case-id>.md の due_date を '2026-08-31' に更新してください。
```

→ frontmatter に `due_date: 2026-08-31` を追加（既存値があれば上書き）。

## NG

- 同名 case が既存なのに上書きしない（必ず stop して人間確認）
- 納期未指定を理由に作成を止めない（hearing 中に追記される前提）
- brief_type が「ec」「コーポレート」など対象外の場合は「対応範囲外」と asks でエスカレーション
- pre_shared_assets / creative_assets の重複追記
