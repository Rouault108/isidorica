# CLIツール仕様 (CLI Tools Specification)

本ドキュメントでは、Isidorica の開発・執筆を支援するCLIツールの仕様を定義する。

---

## 1. 概要

Isidoricaでは、以下のCLIツールを提供する。

| ツール | コマンド | 用途 |
| :--- | :--- | :--- |
| **記事作成CLI** | `pnpm new-article` | 新規記事ファイルの作成を支援 |

---

## 2. 多層防御の設計

記事の品質と一貫性を保つため、**多層防御**アプローチを採用する。CLIは「作成時」の防御を担い、他のツールと連携する。

| 段階 | 対策 | 目的 |
| :--- | :--- | :--- |
| **作成時** | スキャフォールディング（CLI） | 正しい構造・テンプレートを事前に提供 |
| **編集後** | ビルド時バリデーション（Zod） | 必須項目の欠落や型エラーを検出 |
| **開発中** | VS Codeスニペット | 既存記事への構造追加を支援 |

---

## 3. 記事作成CLI (`pnpm new-article`)

適切なディレクトリ配置とFrontmatter入力を支援するCLIツール。

### 3.1 目的

| 課題 | 解決策 |
| :--- | :--- |
| **ディレクトリ選択の難しさ** | 分野を選択すれば、引用形式に基づく適切なディレクトリに自動配置 |
| **Slug重複リスク** | 既存Slugとの重複を事前チェック |
| **Frontmatter入力ミス** | 必須フィールドを含むテンプレートを自動生成 |

### 3.2 対話フロー（インタラクティブモード）

```
$ pnpm new-article

> タイトル: 囚人のジレンマ
> Slug: prisoner-dilemma

> 分野を選択してください:
   1. psychology (心理学) - APA
   2. economics (経済学) - Chicago
   3. philosophy (哲学) - Chicago
   4. mathematics (数学) - AMS
   5. computer-science (CS) - IEEE
   ...
   20. misc (その他) - Harvard
> 選択: 2

> 言語 [ja]: ja

✅ Created: content/articles/ja/economics/prisoner-dilemma.md
```

### 3.3 非対話モード（フラグ指定）

CLIフラグを覚えている場合や、スクリプトからの呼び出し時に使用。

```bash
pnpm new-article --slug prisoner-dilemma --discipline economics --lang ja
```

| フラグ | 説明 | 必須 |
| :--- | :--- | :--- |
| `--slug` | 記事のSlug（ケバブケース） | ✅ |
| `--discipline` | 分野ID（`config/disciplines.yml` のキー） | ✅ |
| `--lang` | 言語コード（デフォルト: `ja`） | - |
| `--title` | タイトル（省略時はSlugから推測） | - |

### 3.4 機能詳細

#### 分野選択

*   `config/disciplines.yml` から分野一覧を読み込み、対話形式で表示。
*   選択された分野に基づいてディレクトリパスを決定。
*   分野と引用形式のマッピングは [GOVERNANCE.md](../../library/GOVERNANCE.md) セクション1.1に準拠。

#### Slug重複チェック

*   `content/articles/` 配下の全Markdownファイルをスキャンし、既存Slugを収集。
*   入力されたSlugが既存Slugと重複している場合、警告を表示し、続行するか確認。

```
⚠️  Slug "prisoner-dilemma" は既に存在します (ja/economics/prisoner-dilemma.md)
続行しますか？ [y/N]: 
```

#### Frontmatterテンプレート

生成されるファイルには、以下の必須フィールドを含むFrontmatterテンプレートが挿入される。

```yaml
---
title: 囚人のジレンマ
slug: prisoner-dilemma
description: ""  # ← 執筆者が記入
topics: []       # ← 執筆者が記入
created_at: 2024-12-22
updated_at: 2024-12-22
status: draft
---

## 概要

<!-- ここに概要を記述 -->
```

#### 引用形式の自動決定

*   執筆者は引用形式を意識する必要がない。
*   ディレクトリ（分野）から引用形式が自動推論される（現行仕様どおり）。
*   `discipline` フィールドは追加しない（Frontmatterの最小化）。

### 3.5 設計原則

| 原則 | 説明 |
| :--- | :--- |
| **ディレクトリ構造の隠蔽** | 執筆者は「引用形式」を知らなくても、「分野」を選ぶだけで適切な場所にファイルが作成される。 |
| **Frontmatterの最小化** | `discipline` フィールドは追加しない。引用形式はディレクトリから自動推論する。 |
| **CLIがメインワークフロー** | GitHub Web UIからの直接編集は限定的なユースケースとして許容するが、CLIがメインのワークフロー。 |
| **対話・非対話の両対応** | 初心者は対話モード、上級者・スクリプトは非対話モードを使用。 |

### 3.6 実装技術（案）

| 項目 | 技術 |
| :--- | :--- |
| **言語** | TypeScript (Node.js) |
| **対話UI** | `enquirer` または `prompts` |
| **ファイル操作** | Node.js `fs` モジュール |
| **YAML解析** | `yaml` パッケージ |
| **引数パース** | `commander` または `yargs` |

### 3.7 将来拡張

| 機能 | 説明 | 優先度 | 備考 |
| :--- | :--- | :--- | :--- |
| **記事タイプ選択** | `--type person` 等で記事タイプを指定し、Infoboxテンプレートを自動挿入 | 中 | 辞書的機能の設計後に実装 |
| **翻訳版作成** | `pnpm new-translation --source ja/economics/prisoner-dilemma.md --lang en` | 中 | - |
| **トピック候補表示** | 既存トピック一覧から選択・補完 | 低 | - |
| **既存記事へのInfobox追加** | `pnpm add-infobox --type person --file path/to/article.md` | 低 | 辞書的機能の設計後に実装 |
| **テンプレート継承** | 記事タイプ間の共通構造を継承（タイプ数増加時に検討） | 低 | 辞書的機能の設計後に実装 |

> **注意**: 現在のフェーズでは「教科書」（学習パス内の記事）にフォーカスしている。「辞書的機能」（Wikipedia風のInfobox、記事タイプ別テンプレート）は将来フェーズで設計・実装予定。

---

## 4. VS Codeスニペット（将来）

既存記事へのInfobox追加を支援するスニペットを提供予定。辞書的機能の設計後に実装する。

*   `infobox-person` → 人物Infoboxを挿入
*   `infobox-concept` → 概念Infoboxを挿入

> **ステータス**: 未実装。辞書的機能の設計後に検討。

---

## 5. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [CONTRIBUTING.md](../CONTRIBUTING.md) | 開発・執筆ガイド |
| [URL_AND_TAXONOMY_DESIGN.md](../../library/technical/URL_AND_TAXONOMY_DESIGN.md) | ディレクトリ構造、分野と引用形式のマッピング |
| [GOVERNANCE.md](../../library/GOVERNANCE.md) | 分野と引用形式のマッピング定義 |
| [MARKDOWN_SPEC.md](../../library/technical/MARKDOWN_SPEC.md) | Infoboxのスキーマ定義 |

