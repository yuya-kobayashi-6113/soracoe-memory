---
name: terminology
description: SolaCoeの用語の罠 — F1〜F3'の遷移。Ownerは集約ルートでない。branchesが自社拠点と取引先支店の二義
type: reference
---
# 用語の罠

**用語の定義自体はリポジトリの `CLAUDE.md` §1 にある。**ここには間違えやすいものだけ。

## 業務の流れ
**見積(F1) → 内諾 → 案件・受注(F2) → 完了(F3) / 失注(F3')**

- **フェーズごとに必須項目が変わる**
- **禁止遷移**(完了→失注 / 完了→受注 / 失注→内諾 / 受注→内諾)は**Domain層で弾く**。CHECK制約では表現できない
- 内諾は見積から案件を生成する移行(`transition-to-project`)。**見積は一覧に残す**(ADR-0010 #4)

## 間違えやすいもの
- **`branches` が二義。**`/api/v1/branches` = **自社拠点**、`/api/v1/partners/{id}/branches` = **取引先の支店**。権限も `company` と `partner` で別。取り違えると認可を間違える
- **施主(Owner)は集約ルートではない。**Project配下の内部エンティティ(ADR-0002)。`row_version` を持たず、競合検知は親の `projects.row_version` で行う
- **取引先(Partner)は複数分類を同時に持てる**(得意先 `customer` / 施工業者 `contractor` / 仕入先 `supplier`)。検索は包含検索
- **営業担当とナカザワ支店は支店単位**で持つ(ADR-0016)。取引先本体ではない
- 権限モジュールは6つ(`estimate` / `project` / `claim` / `partner` / `company` / `settings`)× view/edit の2値のみ。**編集者と閲覧のみは `company` と `settings` を付与できない**
