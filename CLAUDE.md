<!-- このファイルはプロジェクト固有ルールのみを書く。個人/グローバル AI ルール
（言語・確認スタイル・出力フォーマット等）は各 AI ツールのグローバル設定へ。
fresh public clone でも有効な内容に保つこと。 -->

# fressay-netsuper-cart 開発ガイド

> **このファイルは索引であって本文ではない。** 全 AI セッションで全文がロードされるので、
> ルールの本文はここへ書かず、破る人が必ず開く場所（コード・検査スクリプト・skill・guide・台帳）へ置き、
> ここには索引の 1 行だけを残す。新しいルールを足す前に既存の CLAUDE.md・skill・guide・台帳を検索し、
> 正本が既にあれば参照だけにする。詳細は下記「設計原則の索引」。

## プロジェクト概要

フレッセイネットスーパーで、文章の買い物リストを AI（Claude / ChatGPT のブラウザ操作機能）がカートに入れる、非公式の Agent Skill（`SKILL.md`）。利用者本人がログインしたブラウザで、サイトの画面にあるボタンを押すだけで動き、カートに入れて一覧を見せたところで止まる。注文の確定は利用者が行う。

株式会社フレッセイとは関係のない個人の作品。成果物は `skills/fressay-netsuper-cart/SKILL.md` 1 本で、Claude（Claude Code・Cowork・Chrome 拡張）と ChatGPT の skill の両方で読める形にする。

## やらないこと（スコープ外）

- 「お会計へ」から先（支払い・注文確定）の自動化
- ID・パスワードの入力、ログインの自動化
- 配達の便（当日／翌日・時間帯）を AI が選ぶこと。利用者がサイトの画面で選ぶ
- 背景での一括取得（`fetch` での巡回）や、非公開の API の直接呼び出し
- 自前の Chrome 拡張・API キーを使うアプリ（必要と分かるまで作らない）
- 商品データ・画像・サイトの文章をリポジトリへ入れること

## 技術スタック

| 種類 | 内容 |
|---|---|
| 成果物 | Markdown（Agent Skills 形式の `SKILL.md`） |
| 検査 | `scripts/check-claude-md.mjs`（CLAUDE.md の構造検査）。秘密・個人情報の検査は doxguard と gitleaks（CI） |

## ディレクトリ構成

- `skills/fressay-netsuper-cart/SKILL.md` — skill 本体（このフォルダを ZIP にして配る）
- `README.md` — 利用者向けの説明と入れ方
- `scripts/` — 検査スクリプト（CI の doxguard 実行ラッパーを含む）
- `.githooks/` — pre-commit（CLAUDE.md の検査。秘密の検査は封印済みの hook が担う）

## 主要コマンド

- 秘密情報の検査: `npx doxguard scan --staged --block --strict` （CI は `scripts/secrets-scan.dual.mjs --native-only`）
- CLAUDE.md の構造検査: `node scripts/check-claude-md.mjs`

## 設計原則の索引（本文は正本にある）

| ルール | 正本（本文はここ） | 機械検査 |
|---|---|---|
| カートに入れて止まる。注文確定・ログイン・便の選択を AI にさせない | `skills/fressay-netsuper-cart/SKILL.md` の「やらないこと」 | なし（実機テストで確かめる） |
| 迷った品は当て推量で入れず、最後に候補を出す | `skills/fressay-netsuper-cart/SKILL.md` の「品名を商品に決める」 | なし |
| 公開ファイルに個人情報（住所・氏名・電話・注文番号・担当店）を入れない | 本ファイル「AI 作業共通ルール」 | doxguard strict（CI は構造パターンのみ）+ gitleaks |

**新しいルールを足す前に、まずこの表に 1 行足せる形にできないかを考える。** できないもの
（機械検査も、決まったファイルも無いもの）だけが本文を持ってよい。

## AI 作業共通ルール

ビルド・コミット禁止、secrets-scan 責務、plan/bugfix/pending md の作成ルール等の AI 作業共通ルールは、各利用者のグローバル AI 設定に従う（作者環境の例: `~/.claude/CLAUDE.md` および `~/.claude/guides/`）。

- 実機で試すときは、画面に出る住所・氏名・電話番号・注文番号をファイルやログに書かない。例は架空のデータで書く
- 注文の確定ボタンは、試験でも押さない

## Obsidian artifacts

If `docs/obsidian/README.md` exists, use it as an index for related knowledge artifacts.
Use the repository-relative `docs/obsidian` entry. Do not write to a central absolute
path and do not silently fall back to `docs/local` when the entry is missing.

## secrets-scan（このリポジトリの配線）

書く瞬間の責務（固有名詞の一般化・fixture は合成データ等）は上記「AI 作業共通ルール」の参照先に従う。このリポジトリ固有の配線は以下:

- scanner: doxguard 0.2.0（手動実行: `npx doxguard scan --staged --block --strict`）と gitleaks。旧 `scripts/secrets-scan.mjs` は撤去済み
- layer 2: `.githooks/pre-commit`（`git config core.hooksPath .githooks`）/ layer 3: `.github/workflows/secrets-scan.yml`（doxguard strict、`doxguard.ci.json`）・`.github/workflows/gitleaks.yml`
- CI は監視語リストを持たないため構造パターン（絶対パス・private IP・メール）のみ。監視語は作者のローカル環境で layer 2 が検査する

## 関連ドキュメント

| 項目 | パス |
|---|---|
| ユーザー向け README | `README.md` |
| Codex/他 AI 用入口 | `AGENTS.md` |
| ローカル作業ノート（非公開） | `docs/local/`（存在する場合） |
