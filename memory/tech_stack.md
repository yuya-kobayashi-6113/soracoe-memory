---
name: tech-stack-hazards
description: SolaCoeの技術で外から見えない制約 — ID基盤(PostgreSQL/Keycloak)とは別物。MySQL固有の罠。pnpm固有の罠。スタック自体はCLAUDE.md §2
type: reference
---
# 技術の外から見えない制約

⚠️ **共通ID基盤とは別システム。混同しない。**ID基盤=C#/.NET 10・**PostgreSQL**・Keycloak・Razor Pages。SolaCoe=Next.js + ASP.NET Core + **MySQL** + Redis。

**スタックの一覧と版数はリポジトリの `CLAUDE.md` §2 が正本。ここには写さない**(ADR-0018 と ADR-0019 で更新済み。版数を書き写すと必ず腐る)。書くのは、読んでも分からない罠だけ。

## MySQL固有の罠(踏んだもの)
- **`[Timestamp] byte[]` は使えない。**SQL Server専用。楽観ロックは `row_version BIGINT` のアプリ管理カウンタ + `IsConcurrencyToken()`、加算は `SaveChanges` インターセプタで一括(ADR-0014)
- **`EF.Functions.ILike` は EF Core 本体に存在しない**(Npgsql の拡張)。MySQLプロバイダでは**実行時エラーではなくコンパイルエラー**(`CS1061`)。`EF.Functions.Like` を使う(`utf8mb4_0900_ai_ci` が大小を同一視することは実測済み)
- **CHECK制約は1行の中しか検証できない。**状態遷移の禁止はDomainで弾く
- CHECK制約のためMySQL **8.0.16以上が必須**(これは「機能が入った版の閾値」。**採用版の指定と混同して書き換えない**)
- 予約語回避: `floor → floor_label`、`rank → rank_label`。**MySQL 8.4 の新規予約語は実機で `QUALIFY` と `TABLESAMPLE`**(03章の列名との衝突は無し)
- 列挙は `VARCHAR(N)` + CHECK。**`ENUM`型は使わない**
- 主キーは `id CHAR(36)`(UUID・アプリ層生成)。ログ系のみ `BIGINT AUTO_INCREMENT`
- **FULLTEXT を EF の fluent API で定義しない。**BTREE が作られ `MATCH ... AGAINST` が `ERROR 1191` で落ちる。生SQL固定 + `INDEX_TYPE` 検査テスト
- **MySQL 8.4 は `mysql_native_password` が既定で無効**(`caching_sha2_password` 前提に揃える。アプリの接続文字列は不変)

## EF Core の Oracle 公式プロバイダ(`MySql.EntityFrameworkCore`)の罠【実機検証済み】
ADR-0018 で Pomelo から乗り換えた。**Pomelo 向けのコード例をそのまま持ち込むと動かない。**

- **接続文字列に `Old Guids` を設定してはいけない。**設定すると `CHAR(36)` の主キーへの INSERT が `Incorrect string value` で**必ず失敗する**。**ADR-0018 の本文と 2026-08-17 の後日追記は「必須」と書いているが誤りで、2026-08-18(1) の後日追記で撤回済み**
- **拡張メソッド名は `UseMySQL`(`SQL` が大文字)。`ServerVersion` 引数を取らない**
- **`ConnectionIdleTimeout` はサポートされない**(`ArgumentException: Option not supported`)。アイドル回収は既定に委ねる(`ConnectionLifeTime` は生成時刻起点で意味が違う)
- `TreatTinyAsBoolean` は受理される。**指定しない既定でも `TINYINT(1)` は `Boolean` で返る**
- `EnableRetryOnFailure` / `CommandTimeout` などは第3引数 `Action<MySQLDbContextOptionsBuilder>` から使える
- **公式ドキュメントが EF Core 10 に追随していない。挙動は実機で確かめる**

## Tailwind 4 の罠【実機検証済み】
- **`--text-*` は「フォントサイズ」の名前空間。**文字色を `--text-primary` にすると衝突する。**背景は `surface`、文字色は `fg`**(`bg-surface-secondary` / `text-fg-secondary`)
- **`@theme` は既定テーマとマージされる。**`--text-xs` だけ上書きすると既定の `--text-xs--line-height` が残る。**`--text-*: initial` で名前空間ごとリセットし、行の高さは `leading-*` で明示する**
- **`translate-*` は `transform` ではなく個別の `translate` プロパティを出す**(`@keyframes` の `transform` と衝突しない)
- 余白は **`--spacing` 1個の基準値**から計算(`p-4` = `calc(var(--spacing) * 4)`)。**px 固定にするとブラウザの文字サイズ設定に追従しない**
- **duration の名前空間は無い。**イージングだけ `--ease-*` に置き、時間は `duration-150` で指定
- `tailwindcss-animate` は peer 非互換 → **`tw-animate-css`**。`autoprefixer` は不要。PostCSS は `@tailwindcss/postcss`

## ESLint 9 + Flat Config の罠【実機検証済み】
- **ESLint 10 では動かない。**`eslint-config-next` が内包する `eslint-plugin-react` がクラッシュする。**9 系を厳密固定**
- **`next lint` は Next 16 の CLI から消えている。**`pnpm lint` は `eslint .` を直接呼ぶ
- **型情報を使うルールは `files` で TS に絞る。**絞らないと JS/MJS が `Parsing error: ... was not found by the project service` になる
- **`@typescript-eslint/*` のルールは `files` を付けた同じブロックに書く。**別ブロックだと `could not find plugin` で落ちる
- **`react-hooks` は `eslint-config-next/core-web-vitals` に内包**(別途足さない)。`exhaustive-deps` の既定は warning
- 既定の `ignores`(`.next/**` など)は展開すれば引き継がれる。**自分で `ignores` を書いて上書きしない**

## pnpm の罠【実機検証済み・ADR-0019】
- **`pnpm-workspace.yaml` の `allowBuilds` を書かないと `pnpm install` が異常終了する**(`ERR_PNPM_IGNORED_BUILDS`)。**`package.json` の `pnpm.onlyBuiltDependencies` では効かない**(pnpm 11 で設定の置き場が変わった)。現時点の対象は `unrs-resolver`。**ネイティブ依存を足すたびに承認の追記が要る**
- **`package.json` が `frontend/` 配下にあるため、CI では `pnpm/action-setup` に `package_json_file: frontend/package.json`、`setup-node` に `cache-dependency-path: frontend/pnpm-lock.yaml` が要る。**無いと必ず落ちる
- **`npx` は使わない。**ツールは devDependencies に入れて `pnpm exec`。**`pnpm dlx` も使わない**(版が固定されない)。**例外は `preinstall` の `npx only-allow pnpm` だけ**
- **`prepare` は pnpm でも走る。**ただし **`husky install` は husky 9 で非推奨**(実行すると DEPRECATED と出る)。`"prepare": "husky"` を使う
- `node:24-alpine` に **corepack 0.35.0 が同梱**(`corepack enable pnpm` が使える)
- `prettier` の対象に `pnpm-lock.yaml` が入るので `.prettierignore` に足す

## インフラの前提(ADR-0011 / 0012)
EC2 t3.medium ×1にfrontend/backend/Caddy/Redisを同居、RDS Single-AZ、**NAT Gatewayなし**、**ALB・WAF・ElastiCache・Secrets Manager・外部SaaSはすべて不採用**。TLSはCaddy。デプロイはSSH(公開鍵認証のみ)。

**アウトバウンド443を絞るとECR pullも証明書更新も止まる。**Security Groupの「443/80のみ」はインバウンドの話

## この環境でできる検証 / できない検証【実証済み】
**できる**: `.NET 10 SDK`(apt) / NuGet / npm / pnpm / Docker(`sudo dockerd` を自分で起動する) / `mysql:8.4` の実コンテナ / `dotnet ef` のマイグレーション / Next のビルドと Lint。**「クラウド環境だから検証できない」は成り立たない。**

**できない**:
- **Docker コンテナの中からパッケージレジストリに到達できない**(TLS がサンドボックスのプロキシに阻まれる)。**Dockerfile のビルド検証は不可**
- **ロックファイルを GitHub MCP で push できない**(142KB でも 232KB でも出力上限超え)。**ローカルの git 経路で入れてもらう**
