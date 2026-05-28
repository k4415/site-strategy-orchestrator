---
name: case-init
description: 案件キックオフ時に cases/<case-id>.md を作成する。「新規案件」「キックオフ」「<クライアント名>を開始」などのトリガーで起動。
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
- 納期
- 案件オーナー（人間）
- 現フェーズ（hearing-agent からの依頼なら `hearing` で進行中）

不足があれば呼び出し元に確認を返す。**人間から直接ではなく hearing-agent から呼ばれた場合、追加質問はせずに頂いた情報で case doc を作る**（クライアント対話の流れを止めない）。

## 実行手順

1. **case_id を決める**
   - クライアント名を kebab-case 化（例: `acme-saas-launch-2026q3`）
   - 既存の cases/ 配下に同名がないか確認

2. **frontmatter を組み立てる**
   `knowledge/case-hub-schema.md` の規約に従う：
   ```yaml
   ---
   title: "<クライアント名> サイト制作案件"
   summary: "ヒアリング進行中"
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

4. **hearing-agent からの依頼の場合 → 終了**
   - hearing-agent が既にクライアントと対話中なので、追加で session run しない
   - case doc 作成完了を hearing-agent に通知して終了（自分は待機）

5. **人間からの直接キックオフの場合 → site-hearing-agent を起動**
   - `phase-handoff` skill を呼んで site-hearing-agent を session run で起動

判定の仕方:
- 依頼本文に「私が継続中」「ヒアリング中」「hearing-agent から」などの記述がある → 4 へ
- そうでなければ → 5 へ

## NG

- 同名 case が既存なのに上書きしない（必ず stop して人間確認）
- 納期が未指定のまま進めない
- brief_type が「ec」「コーポレート」など対象外の場合は「対応範囲外」とエスカレーション
