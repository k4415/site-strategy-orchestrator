---
name: case-init
description: 案件キックオフ時に cases/<case-id>.md を作成する。「新規案件」「キックオフ」「<クライアント名>を開始」などのトリガーで起動。
---

# case-init

新規案件の hub doc を作成し、初期状態を設定する。

## いつ起動するか

- 「新規案件 <name> を開始」「キックオフ」「<クライアント名>のサイト制作を始める」などの依頼を受けた時
- 既存の `cases/<case-id>.md` が存在しない時のみ実行

## 入力

依頼本文から以下を抽出：
- クライアント名 / 案件名
- 想定 brief_type（`lp` / `saas` / 未確定）
- 納期
- 案件オーナー（人間）

不足があれば質問して埋める。

## 実行手順

1. **case_id を決める**
   - クライアント名を kebab-case 化（例: `acme-saas-launch-2026q3`）
   - 既存の cases/ 配下に同名がないか確認

2. **frontmatter を組み立てる**
   `knowledge/case-hub-schema.md` の規約に従う：
   ```yaml
   ---
   title: "<クライアント名> サイト制作案件"
   summary: "ヒアリングセッションを開始する"
   status: hearing
   assignee: site-hearing-agent
   owner: <human-username>
   brief_type: <lp|saas|未確定>
   due_date: <YYYY-MM-DD>
   children: []
   ---
   ```

3. **ファイル作成**
   - パス: `aachat/docs/<team>/<project>/cases/<case-id>.md`
   - 本文: 案件概要、クライアント情報、ヒアリング開始メモ

4. **次フェーズへ即ハンドオフ**
   - `phase-handoff` skill を呼んで site-hearing-agent を起動

## NG

- 同名 case が既存なのに上書きしない（必ず stop して人間確認）
- 納期が未指定のまま進めない
- brief_type が「ec」「コーポレート」など対象外の場合は「対応範囲外」とエスカレーション
