> **このファイルは「Soracoe システム開発」案件の記憶の索引です。**
> **正本はこのリポジトリ（`yuya-kobayashi-6113/soracoe-memory`）です。**Cowork のプロジェクトドキュメント側にある同じ内容は移設時点の写しなので、更新しないでください。読むときも書くときも、こちらを見てください。
> トピック別の中身は `memory/` 配下にあります。索引を読んで、必要になったものだけを開いてください。

進め方・報告の型・メモリ運用・迎合禁止・指示書の型は **allsmile-ops プラグインのスキルが正本**(v0.6.0)。ここには案件固有の情報だけを置く。

- [SolaCoeの今](memory/solacoe.md) — **S0とS1-01が完了し実装が動き出した**。ADR-0018の反映4本とADR-0019(pnpm)はマージ済み。**次はS1-02**。確立した進め方(実機で通してから書く/独立監査に一次情報で裏取り/blob SHAで挟む)と、環境の限界(ロックファイルは投入不可・Dockerは検証不可)
- [決定と却下案の在り処](memory/decisions.md) — **蒸し返されやすい9件**と却下理由の在り処。**ADRの本数も最新番号も書き写さない**。**ADRは実機と食い違うことがある**(決定10の撤回の実例と教訓)。**「影響」節は母集団ではない**
- [技術の外から見えない制約](memory/tech_stack.md) — **ID基盤(PostgreSQL/Keycloak)とは別物**。MySQL / EF Core(Oracle公式) / Tailwind 4 / ESLint 9 / pnpm の**実機検証済みの罠**。**この環境でできる検証・できない検証**
- [顧客に聞かれたら即答する用](memory/customer_answers.md) — なぜMySQLか/なぜモバイル専用UIを作らないか/なぜWAFを入れないか/なぜ外部サービスを使わないか
- [設計書の落とし穴](memory/spec_hazards.md) — 「前版参照」の前版が存在しない/他DB・他クラウド前提の残骸/05章に定義がない/数値は数え直す
- [GitHubリポジトリ](memory/repos.md) — allsmile-**inc**/sola-coe。**MCPの環境共通の罠は env-pitfalls スキルが正本**。ここは org・コネクタ・積み上げPR事故の履歴・ID基盤との権限差だけ
- [用語の罠](memory/terminology.md) — F1〜F3'の遷移。Ownerは集約ルートでない。`branches`が自社拠点と取引先支店の二義

**設計の正本はリポジトリ `allsmile-inc/sola-coe`。ここに複製しない。**
