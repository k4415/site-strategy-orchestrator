# site-strategy-orchestrator

サイト制作上流工程（ヒアリング→要件整理→リサーチ→戦略ブリーフ）を直列パイプラインで進行管理する aachat agent。

## できること

- 案件キックオフ時に `cases/<case-id>.md` を作成し、フェーズ進行を管理
- `site-hearing-agent` / `site-requirements-agent` / `site-research-agent` / `site-brief-agent` を順に `aachat session run` で起動
- 各フェーズ完了時に case ハブ doc の `status` と `assignee` を更新
- 詰まった時は `asks` フィールドで人間判断を仰ぐ

## 使い方

事前にこの project に `site-hearing-agent` / `site-requirements-agent` / `site-research-agent` / `site-brief-agent` を assign してください。

```bash
aachat agent clone <owner>/site-strategy-orchestrator --name site-strategy-orchestrator
aachat project assign <project> --agent site-strategy-orchestrator
aachat session run site-strategy-orchestrator --project <project> \
  "新規案件 <クライアント名> を開始。サイト種別 <lp|saas>、納期 <YYYY-MM-DD>。ヒアリングセッションを立ち上げて。"
```

## 構成

- `identity.md` — エージェントの役割・行動方針
- `environment.yaml` — 実行環境（依存パッケージ・env 宣言）
- `memory/` — 案件進行中の状態・未完了の引き継ぎ
- `knowledge/` — handoff テンプレ・case ハブ schema・recovery playbook
- `.agents/skills/case-init` — 案件ハブ doc 作成
- `.agents/skills/phase-handoff` — 次エージェントへの session run
- `.agents/skills/progress-check` — 案件状態の確認
- `.agents/skills/recovery` — リトライ・人間エスカレーション

## 設計ドキュメント

詳細設計は aachat shared doc を参照: [[aachat/docs/agent-development/site-creation-suite/specs/site-strategy-orchestrator.md]]

全体パイプライン: [[aachat/docs/agent-development/site-creation-suite/specs/overview.md]]

## 必要な env

なし。`aachat` CLI が project 経由で他エージェントを起動するため、自前のAPIキーは不要。

## 注意

secret、token、JWT、PAT、秘密鍵は repo に含めないでください。
