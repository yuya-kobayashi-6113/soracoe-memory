---
name: github-repos
description: allsmile-inc/sola-coe と、このリポジトリ固有の事情 — org・コネクタ・ID基盤との権限差・積み上げPR事故の履歴
type: reference
---
# GitHubリポジトリ

**`allsmile-inc/sola-coe`**。ID基盤の `allsmile-corp` とは**org が別**。アクセスは `Claude Github MCP Connector` 経由(切れたらclaude.aiのコネクタ設定から再認証)。

**GitHub MCP の環境共通の罠(日本語破損・全置換・403・search_codeの0件・サイズ耐性・積み上げPRの回復手順)は `env-pitfalls` スキルが正本。ここには書かない。**

## 積み上げPRの事故履歴【2026-08-05】

**PR#7/#8がmainに届かない事故が起きた。**ベースブランチを残したまま順にマージしたため、後続PRが1つ前のブランチにマージされていた。PR#9は merge commit、PR#10は `update_pull_request` で base を main へ付け替えて回復した。

**手順は `env-pitfalls`「積み上げPRを作ってしまった場合の回復手順」を見ること。**

**このリポジトリの現状**: `Automatically delete head branches` の設定状況は未確認。**マージ済みの `docs/s0-08-*` が10本残っている**ので、オンになっていない可能性が高い。次にPRを作る前に確認する。

## ID基盤から持ち込まなかったもの【org・Appの権限差】

**別プロジェクトで成り立っていた制約が、こちらでは成り立たなかった実例。**

| ID基盤での制約 | SolaCoeでの実測 |
|---|---|
| 「`push_files` は15KBまで」 | 単一ファイルなら149KBのYAMLが1回で通った(`create_or_update_file` 側) |
| 「`.github/workflows/` 配下が403」 | `deploy.yml` を削除できた |

**org と GitHub App の権限が違うため。**他プロジェクトの制約を無検証で持ち込まないこと(no-sycophancy)。
