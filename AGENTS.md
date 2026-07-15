# AGENTS.md — プロジェクト規約(このリポジトリで作業するすべてのエージェントと人間の共通ルール)

本書が規約の正本。Codex / Cline / Cursor / Windsurf 等はルートの本ファイルを直読みし、
Claude Code は `CLAUDE.md` 冒頭の `@AGENTS.md` インポート経由で読む(.guardrails/GUARDRAILS.md §6。
本文をどちらかへ複製しない——分割であって複製ではないのがドリフトしない理由・G5)。
コミット・push・CI の門(.guardrails/GUARDRAILS.md §3〜§5)は git フックと CI なので全エージェント共通。
Claude Code と Codex が持つ追加の門(編集直後・操作直前・ターン終了のフック層)は、前者は
`CLAUDE.md`、後者は `.codex/hooks.json` を参照。

**ブートストラップの現状(2026-07-16)**: Step -1a(キット配置)完了・Step 1(本書と CLAUDE.md の整備)を
このコミットで実施。**Step 0(採用列 `ts-react-web@12` の入力確定・`fill_bindings.py` によるバインディング
充填)はまだ未実行**——`.guardrails/BOOTSTRAP.md` の台帳は Step 0/1 とも `🚧` のまま(番号順の
✅ 化を守るため、Step 0 が先に済むまで Step 1 を ✅ にはしない)。そのため下記の §0・§5・§7・§8 の
一部はバインディング充填後に確定する。詳細な経緯は [PLAN.md](PLAN.md) を参照。

## §0 よく使うコマンド(ランタイム共通動詞 — .guardrails/GUARDRAILS.md §12.1)
すべて `uv run scripts/dev.py <動詞>`。動詞名は全プロジェクト共通・未配線は明示エラー:

| 動詞 | 何をするか |
|---|---|
| `check` | 構造検査(§3.3)。既定配線済み |
| `probe "<cmd>"` | 迂回防止(§2)への事前照会 |
| `doctor` | 環境・シム・hooksPath・フック配線の事実表示 |
| `gates` | 全門の現在状態(常時有効/列充填待ち)を一覧 |
| `selftest` | 門コーパス一括再生 |
| `up` / `reset` / `seed` / `test` / `e2e` / `db` | 採用列(`ts-react-web@12`)固有のコマンド。
  Step 0 の `fill_bindings.py` 実行までは未配線(このリポジトリ自体は稼働アプリを持たない
  kit 配布リポジトリのため、`up`/`e2e` 等は Step 0 で「対象外」判定になる可能性が高い) |

- 索引再生成: `uv run scripts/generate_structure.py`(STRUCTURE.md を書いてよい唯一の主体)
- 静的解析: 採用列確定後に記載(現状 Python は `uv run` 経由のみ・専用 lint 未導入)

## §1 ファイル規模
1ファイル500行以内を目安とする(超過は check-structure の soft 警告)。超えそうなら分割する。

## §2 フォルダ規模
1フォルダに CLAUDE.md 以外で7ファイルまでを目安とする(`scripts/` は例外)。
超えそうならサブフォルダへ整理する。`.claude/skills/` 配下(ベンダーコピー)は対象外——
生成物扱いで index/検査から除外されている(§6 参照)。

## §3 ファイル先頭ヘッダー
すべてのコードファイルの先頭に役割一行コメントを書く。書式: `<ファイル名> — 役割`
(例: `# check_structure.py — 構造検査`)。`.claude/skills/` 配下のベンダーコピーは
生成物扱いのため対象外。

## §4 ドキュメントの置き場の分担
- 索引 = `STRUCTURE.md`(自動生成・手編集禁止)
- 全体計画・アーキテクチャ・技術選定理由 = `PLAN.md`(様式は [doc/PLAN_FORMAT.md](doc/PLAN_FORMAT.md) — §6 参照)
- 導入手順 = `README.md`
- 上流選定・特別対応の記録 = `.upstream/sources.yaml`
- フォルダ固有知見 = 各フォルダの `CLAUDE.md`
- 出戻り防止の地図 = `.guardrails/GUARDRAILS.md`
- 目標の正本 = `.guardrails/GOALS.md`
- バインディングの正本 = `bindings/catalog.md`(採用列: `ts-react-web@12`)

## §5 フォルダ独立性・依存方向
このリポジトリはアプリのレイヤー構成を持たない(kit 配布物 + 上流参照のみ)。
`upstream/` → 読み取り専用の submodule。`.claude/skills/` → ベンダーコピー(手編集禁止)。
`scripts/` → kit のインストーラ CLI(`install_kit.py`)と workbench 固有インストーラ
(`install_workbench.py`・`revendor_uipro.py`)。依存の向きは常に
「利用先リポジトリ ← このリポジトリの配布物」の一方向で、このリポジトリの中に
双方向依存は存在しない。`layer-violation` 系の hard 規則は Step 0 で
`LAYER_FORBIDDEN_IMPORTS`(空リスト=不発)のまま据え置く判断になる見込み(アプリ層が無いため)。

## §6 プロジェクト固有の規約

`upstream/` 配下は読み取り専用の submodule(参照元)。編集しない。

### UI スキル(`.claude/skills/` — ベンダー領域)

[.claude/skills/](.claude/skills/) は
[nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)
からのベンダーコピー(オーケストレータ + サブスキル6つ)。**手で編集しない**。
出所 SHA と更新手順は [.upstream/sources.yaml](.upstream/sources.yaml)(id: ui-ux-pro-max-skill)が正本。

採用時の特別対応3点(2026-07-15 決定・経緯は sources.yaml の rationale)。
いずれも管理区画(`>>> GUARDRAILS BINDING >>>`)への機械充填で、
**適用は `scripts/install_workbench.py` が行う**(手貼り不要。下のコードは充填内容の正本)。
kit の `install_kit.py` は版上げ時に区画の中身を引き継ぐため、充填は更新で消えない:

1. **Python 実行**: `scripts/dev.py` の COMMANDS(加算形)へ次を充填し、
   スキル検索は `uv run scripts/dev.py design "<query>"` の動詞で呼ぶ
   (「Python は必ず uv 経由」kit GUARDRAILS §7.1。読み替え規約でなく動詞レールにする。
   uv 直呼びでの動作は確認済み):
   ```python
   COMMANDS.update({
       "design": [["uv", "run", "python",
                   ".claude/skills/ui-ux-pro-max/scripts/search.py", "{args}"]],
   })
   ```
2. **kit 検査の除外**: `scripts/repo_scan.py` の BINDING 区画へ次を充填する。
   `GENERATED_PATTERNS` は「手編集禁止・索引/検査から除外」の意味論で、内容系検査
   (ヘッダー必須・print 直呼び・ログ被覆・テスト非決定等)と STRUCTURE.md 索引の
   両方から外れる(check_structure.py は生成物を読み込まない)。
   gitleaks(秘密検出)は除外されない——それが正しい挙動:
   ```python
   GENERATED_PATTERNS += [re.compile(r"^\.claude/skills/")]
   ```
3. **生成物の扱い**: 同じ BINDING 区画へ次を充填する。`--persist` が書く
   `design-system/` はデザイン決定の記録としてコミット対象:
   ```python
   GENERATED_PATTERNS += [re.compile(r"^design-system/")]
   ```

emilkowalski/skills 由来の5スキル(アニメーション/デザインエンジニアリング系)は
`upstream/ui-skills/` の submodule 参照(ベンダーコピーではない)。repo_scan の列挙は
`git ls-files`(親リポジトリの追跡ファイルのみ)なので submodule の中身は最初から
検査対象外——特別対応は不要。

### デザイン参照資料

UI の見た目・方向性を検討・実装するときは
[upstream/design-md/awesome-design-md/design-md/](upstream/design-md/awesome-design-md/design-md/)
を参照する——74ブランド分の実在デザインシステムの DESIGN.md 集
(submodule + sparse-checkout。例: `design-md/apple/DESIGN.md`・`design-md/linear.app/DESIGN.md`)。
ui-ux-pro-max の検索(スタイル・パレットの一般則)と役割が違い、こちらは
「実在ブランドの具体的なトーン・トークン・原則」を引く時に使う。

### UI開発時のテスト方針

ボタン遷移などの操作でエラーが起きないかの確認は、guardrails-kit 側に既に機構がある
(`upstream/guardrails-kit/.guardrails/GUARDRAILS.md` §12.4「操作レール」):
UI操作要素へのテストID属性必須化(`ui-missing-testid`・hard)+ Playwright MCP による
実UI操作 + 再現できたバグの E2E spec 化(fix と同一コミット)+ `e2e` CI ジョブ
(採用列 `ts-react-web@12` で Playwright が配線済み)。

ただしこの workbench 自体は UI 機能を実装する対象ではない(汎用キットであり、実装先は
配布先の個別リポジトリ——PLAN.md「目的」参照)。Step 0 のバインディング充填が済んでも、
このリポジトリ自身に E2E で検証すべき UI 機能が生まれるとは限らない。

### PLAN.md の編集方針

PLAN.md を編集するときは、様式(節構成・書式)を [doc/PLAN_FORMAT.md](doc/PLAN_FORMAT.md) に
合わせる。PLAN_FORMAT.md はあくまで様式の雛形であり、中身(目的・アーキテクチャ・
技術選定理由等)はそのまま転記せず、このリポジトリ固有の内容に置き換える。

## §7 ログ規則
- 秘匿: トークン・パスワード・APIキーをログに渡さない(コミット面は gitleaks が機械検査。
  ログ面はこの規約が最後の責務 — .guardrails/GUARDRAILS.md §8.3)。識別子は載せてよいが中身は載せない。
- 例外を握りつぶさない(空 catch 禁止 — lint で error 化)。
- 出力基準・形式・ログ単一出口・ログ境界パターンは、Step 0 で `ts-react-web@12` 列から
  `bindings/catalog.md` 経由で確定する(現状このリポジトリは Python インストーラ CLI のみで、
  アプリのログ出口を持たない)。

## §8 テスト戦略
- テストが通る状態でのみコミットする(pre-push と CI が機械検査)。
- 一度直したバグは回帰テストに固定し、fix と同一コミットに同梱する。
- 新機能(feat)もテストを同梱する。
- flaky の温床を持ち込まない(sleep・現在時刻・seed なし乱数・外部I/O直呼びは hard 違反)。
- 本命の E2E: 採用列 `ts-react-web@12` の Playwright。ただし現状このリポジトリにテストは無い
  (Step -1b 棚卸しで「既存テストの状態: なし」と記録済み)。E2E が必要になるのは
  実際に UI 機能を実装する下流リポジトリ側であり、このリポジトリ自身では
  今のところ対象が無い(§6「UI開発時のテスト方針」参照)。
- UI の操作要素にはテストID属性を必ず付ける(hard `ui-missing-testid` — §12.4。同上の理由で
  現状このリポジトリでは該当箇所なし)。

## §9 上流の更新運用

上流ごとの取り込み・更新方式は [.upstream/sources.yaml](.upstream/sources.yaml) に記録する。
全上流を週1・同じタイミング(月曜 06:00 UTC)でチェックする(`update-upstreams.yml`)。
新しい上流 / Skill を採用するときは README「新しい上流 / Skill を採用するとき」の手順に従う。

## §10 Git 規則
- GitHub Flow: main へ直接 push しない。1トピック=1ブランチ=1PR。
- コミットは小さく(純変更 400 行超で soft `commit-too-large` が警告——生成物・lockfile は除外)。
- コミットメッセージ規約: `^(feat|fix|test|docs|refactor|chore): .+`
  (commit-msg フックが機械検査。Merge / Revert / fixup! / squash! は素通し)。
- `.guardrails/GOALS.md`・`.guardrails/GUARDRAILS.md`・`bindings/catalog.md` を変更するコミットは、
  本文に効くGを1行書く。
- 依存マニフェストへの追加は本文に `依存追加: <名前> — 理由1行` を書く。

### §10-4 フック(commit / push の門)との付き合い方 — 全エージェント共通
pre-commit / commit-msg / pre-push の門は git フックなので、どのエージェントで作業しても発火する。
- 迂回禁止: `--no-verify`・`SKIP=` は使わない。フックが落ちるなら迂回せず違反そのものを直す。
- 未コミットの作業を消すコマンド(`git reset --hard`・`git clean -f`・広域 checkout/restore)を
  使わない: 消してよい変更なら先に `git stash` で退避する。
- 自動修正系フックで落ちたら: 書き換えられたファイルを `git add` して同じコミットを再実行するだけ。
- `generate-structure` で落ちたら: `git add STRUCTURE.md` して再実行するだけ。
- 同じフックが2回連続で落ちたら機械的リトライをやめて原因調査に切り替える。

## §11 導入方法(このリポジトリを配布物として使う側の手順)

このリポジトリ自体は汎用キットであり、README.md「セットアップ(導入方法)」に
ケース0〜4(新規/既存リポジトリへの導入・更新の取り込み)を記載している。
このリポジトリ自身への敷設(ケース2の自己適用)の進捗は `.guardrails/BOOTSTRAP.md` を参照。

## §12 作業開始の定型手順
1. `STRUCTURE.md` を読む(いまの全体像)
2. `PLAN.md` を読む(なぜこの構成か・現状・ロードマップ)
3. 触るフォルダの `CLAUDE.md` を読む(フォルダ固有の知見・ハマりどころ)
4. 環境が要る作業なら `uv run scripts/dev.py verbs` で配線状態を確認する

## §13 発見の記録先(中央メモは作らない)
- 再現できるバグ → 回帰テスト(fix と同一コミット)
- 直感に反する箇所 → その場の近接コメント
- フォルダ固有の知見 → そのフォルダの `CLAUDE.md`
- 昇格ルール: 近接コメントに書いた制約がそのファイルの外で噛んだら、そのフォルダの
  `CLAUDE.md` へ昇格して記録する。
