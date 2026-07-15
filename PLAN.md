<!-- PLAN.md — guardrails-workbench の全体計画・現状・技術選定理由の正本 -->
# PLAN.md — guardrails-workbench 全体計画

## 目的

[zappyzed100/guardrails-kit](https://github.com/zappyzed100/guardrails-kit) に
UI Skill・デザイン参照資料・インストーラを揃えた"汎用キット"(ワークベンチ)。
このリポジトリ自体では特定の UI 機能は作らない——実際にどの UI 機能を作るかは、
このワークベンチをテンプレートとして clone した先の個別リポジトリ
(README「ケース1: 新規リポジトリに導入する場合」)で決める話であり、対象外。
このリポジトリの役割は、その土台(submodule・ベンダーコピー・インストーラ・
上流監視)を整備し続けることに限定される。

## 現状(2026-07-16 時点)

上流の取り込みは完了、workbench 自身への kit 敷設はまだ。

- 完了: `upstream/guardrails-kit` submodule(ts-react-web@12 相当まで更新済み)・
  emilkowalski/skills 全5本の submodule + sparse-checkout・
  awesome-design-md submodule + sparse-checkout(74ブランドの DESIGN.md、参照専用)・
  ui-ux-pro-max-skill のベンダーコピー(`.claude/skills/`、特別対応3点は
  [CLAUDE.md](CLAUDE.md) と [.upstream/sources.yaml](.upstream/sources.yaml) が正本)
- 週次監視(`update-upstreams.yml`・月曜 06:00 UTC)と Dependabot は運用中
- 未着手: この workbench 自身への `install_kit.py` 実行(ルートに `scripts/dev.py`・
  `scripts/repo_scan.py`・`.guardrails/` がまだ無く、ゲートは未有効化 —
  [CLAUDE.md](CLAUDE.md)「UI開発時のテスト方針」に詳細)。目的は自前の UI 機能を
  作ることではなく、配布先で追体験する敷設手順がこのリポジトリ自身でも
  そのまま通ることを検証する(dogfooding)ため。

## アーキテクチャ

```text
guardrails-workbench/
├─ upstream/
│  ├─ guardrails-kit/                 submodule(全体を利用。未敷設)
│  ├─ ui-skills/emilkowalski-skills/  submodule + sparse-checkout(採用Skill 5本)
│  └─ design-md/awesome-design-md/    submodule + sparse-checkout(参照専用・74ブランド)
├─ .claude/skills/                    ui-ux-pro-max ベンダーコピー(7スキル・手編集禁止)
├─ scripts/                           workbench 固有インストーラ
│  ├─ install_workbench.py            特別対応3点の機械充填(導入方法ケース1/2/4)
│  ├─ revendor_uipro.py               ui-ux-pro-max の再ベンダー(導入方法ケース3)
│  └─ setup-upstreams.ps1             clone後の sparse-checkout 再現
├─ .github/
│  ├─ dependabot.yml                  guardrails-kit の更新(skills系は ignore)
│  └─ workflows/update-upstreams.yml  週1の上流監視・更新PR作成
└─ .upstream/sources.yaml             上流選定結果・特別対応の正本
```

依存関係・充填内容・セットアップ手順の詳細は [README.md](README.md) と
[CLAUDE.md](CLAUDE.md) が正本。ここでは重複させず要約のみ扱う。

## 技術選定理由(要約)

- **submodule(全体利用)** — guardrails-kit: cherry-pick 不要、リポジトリ全体が製品土台。
- **submodule + sparse-checkout** — emilkowalski/skills(自己完結 Markdown、install 不要)・
  awesome-design-md(参照資料のみ抽出、~2.3MB に抑制)。
- **ベンダーコピー** — ui-ux-pro-max-skill: npm 配布(`ui-ux-pro-max-cli`)が GitHub main に
  遅れているため、pinned SHA からインストーラの render+copy を再現して `.claude/skills/` へ
  直接コピーする方式を採用。
- 各上流の比較・却下理由・改訂履歴は [.upstream/sources.yaml](.upstream/sources.yaml) の
  `rationale` が正本(例: awesome-design-md は当初リンクのみで検討し、
  「実際に参照されるのはローカル資料」という指摘で submodule 化に改訂)。

## ロードマップ

1. **kit の自己敷設**(未着手・dogfooding目的): README「ケース2: この workbench 自身に
   敷設する場合」の手順で `install_kit.py`(★列 `ts-react-web@12`)→ 敷設完了後に
   `install_workbench.py` を適用し、ゲート(`repo_scan.py`・`dev.py`・`.guardrails/`)を
   有効化する。ここで何かの UI 機能を作るためではなく、配布先で使われる手順そのものが
   このリポジトリでも通ることを検証するために行う。
2. **上流の継続追随**: guardrails-kit・emilkowalski/skills・awesome-design-md・
   ui-ux-pro-max-skill の週次監視は運用中。新しい上流/Skill 候補が出た場合のみ
   README「新しい上流 / Skill を採用するとき」の手順で採否を検討する。
3. **インストーラ自体の改善**: `install_workbench.py`・`revendor_uipro.py`・
   `setup-upstreams.ps1` を、配布先での実運用で気づいた不備に応じて直す。

実際の UI 機能の企画・実装は、このワークベンチを clone した個別の利用先リポジトリの
PLAN.md が扱う話であり、ここには書かない。

## 運用

- 全上流のチェックは週1・月曜 06:00 UTC に統一(README「上流の更新フロー」表が正本)。
- kit との充填ドリフト検出は `install_kit.py --check`(敷設後に利用可能)。
- `upstream/` 配下は読み取り専用・`.claude/skills/` はベンダーコピーにつき手編集しない
  ([CLAUDE.md](CLAUDE.md)冒頭)。

## タスク(機械可読)

書式:
- `- [ ] タイトル` … 未完了。行末に `` `状態タグ` `` が無ければ `backlog` 扱い
- `- [x] タイトル` … 完了。行末にタグが無ければ `done` 扱い
- 状態を明示したい時だけ行末にタグを付ける: `` `next` `` / `` `in_progress` `` /
  `` `blocked` `` / `` `cancelled` ``(`done`/`backlog` はチェック状態で表せるため省略可)
- `unknown` はこの記法では書かない — 収集側が行を解釈できなかった場合にのみ付与する

- [x] guardrails-kit を submodule として導入する(ts-react-web@12 相当)
- [x] emilkowalski/skills 5本を submodule + sparse-checkout で導入する
- [x] awesome-design-md を submodule + sparse-checkout で導入する(参照専用)
- [x] ui-ux-pro-max-skill をベンダーコピーし、特別対応3点を install_workbench.py に充填する
- [x] 上流の週次監視ワークフロー(update-upstreams.yml)を用意する
- [ ] この workbench 自身に guardrails-kit を敷設し、手順の追体験可能性を検証する(install_kit.py → install_workbench.py) `next`
