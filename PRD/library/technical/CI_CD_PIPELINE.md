# CI/CD パイプライン仕様 - Library (CI/CD Pipeline - Library)

本ドキュメントでは、`isidorica-library` リポジトリの CI/CD パイプライン仕様を定義する。

**ステータス**: フェーズ4実装予定
**最終更新**: 2025-12-22

---

## 1. 概要

`isidorica-library` は Isidorica の記事、出典、設定ファイル（コンテンツ）を管理するリポジトリである。
本パイプラインは、**コンテンツ品質の保証**、**パーマリンクの保全**、および**運用の自動化**を実現する。

> **注意**: UI・ビルドロジックに関するチェックやデプロイは `isidorica-engine` リポジトリで管理される。[Engine CI/CD](../../engine/technical/CI_CD_PIPELINE.md) を参照。

### 1.1 ワークフロー一覧

| ワークフロー名 | ファイル名 | トリガー | 目的 |
| :--- | :--- | :--- | :--- |
| **Pull Request Checks** | `ci.yml` | PR作成・更新 | コンテンツバリデーション |
| **Topics Check** | `topics-check.yml` | PR作成・更新 | Topicsバリデーション、alias自動修正 |
| **Post-Merge Processing** | `post-merge.yml` | mainへのプッシュ | アーカイブ保存、画像処理、Engineデプロイトリガー |
| **Image Upload** | `image-upload.yml` | `articles/**/images/**` 変更時 | 画像の最適化とR2アップロード |
| **Review Check** | `review-check.yml` | 週次（Schedule） | レビュー停滞PRの通知 |
| **Translation Sync** | `translation-sync.yml` | オリジナル記事更新時 | 翻訳要Issueの自動作成 |
| **Monthly Report** | `monthly-report.yml` | 月次（Schedule） | 健全性レポート、リマインダー |

### 1.2 アーキテクチャ

```
┌─────────────────── Pull Request ───────────────────┐
│                                                    │
│  ┌──────────────┐  ┌──────────────┐               │
│  │ Frontmatter  │  │ Topics       │               │
│  │ Validation   │  │ Validation   │               │
│  └──────────────┘  └──────────────┘               │
│         ↓                ↓                        │
│  ┌──────────────┐  ┌──────────────┐               │
│  │ Slug         │  │ textlint     │               │
│  │ Consistency  │  │ (日本語)     │               │
│  └──────────────┘  └──────────────┘               │
│                      ↓                            │
│              All Pass? → Merge可能                 │
└────────────────────────────────────────────────────┘
                       ↓
┌─────────────────── main マージ ────────────────────┐
│                                                    │
│  ┌──────────────┐  ┌──────────────┐               │
│  │ Archive      │  │ Image        │               │
│  │ (R2保存)     │  │ Optimization │               │
│  └──────────────┘  └──────────────┘               │
│         ↓                                         │
│  ┌──────────────┐  ┌──────────────┐               │
│  │ Catalog      │  │ Usage Index  │               │
│  │ Generation   │  │ Generation   │               │
│  └──────────────┘  └──────────────┘               │
│         ↓                                         │
│  ┌──────────────────────────────────────────────┐ │
│  │ Trigger Engine Deploy (Webhook)              │ │
│  └──────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘
```

---

## 2. Pull Request Checks (Quality Gate)

マージを許可するための必須チェック。

### 2.1 Content Validation (コンテンツ整合性検証)

| チェック | 説明 | 失敗時の動作 | 参照ドキュメント |
| :--- | :--- | :--- | :--- |
| **Frontmatter検証** | Zodスキーマによる型チェック（必須フィールド、enum値） | PRブロック | [MARKDOWN_SPEC.md](./MARKDOWN_SPEC.md) |
| **予約語チェック** | Slugが予約語でないか | PRブロック | [URL_AND_TAXONOMY_DESIGN.md](./URL_AND_TAXONOMY_DESIGN.md) |
| **Slug一致チェック** | 翻訳記事のSlugがオリジナルと一致 | PRブロック | [URL_AND_TAXONOMY_DESIGN.md](./URL_AND_TAXONOMY_DESIGN.md) |
| **ディレクトリ追従チェック** | 翻訳版の主題ディレクトリがオリジナルと一致 | 警告 | [URL_AND_TAXONOMY_DESIGN.md](./URL_AND_TAXONOMY_DESIGN.md) |
| **学習パス整合性** | `learning-paths.yml` の記事slugが存在 | ビルドエラー | [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) |
| **概念関係整合性** | `concept-relations.yml` の概念が `topics.yml` に存在 | 警告 | [GOVERNANCE.md](../GOVERNANCE.md) |

### 2.2 Topics Validation (Topicsバリデーション)

> **参照**: [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) セクション2.3

| ケース | 処理 | 人間の介入 |
| :--- | :--- | :--- |
| **正規ID マッチ** | ✅ 通過 | なし |
| **alias マッチ** | 🔧 自動修正（正規IDに置換してコミット） | なし |
| **未登録** | 🏷️ `new-topic` ラベル付与、レビュワーに通知 | 必要 |

### 2.3 Writing Quality (文章品質) - textlint

> **参照**: [ARCHITECTURE_DECISION_RECORD.md](../../engine/technical/ARCHITECTURE_DECISION_RECORD.md) セクション2.7

#### ツール構成

| ツール | 用途 | 備考 |
| :--- | :--- | :--- |
| **textlint** | 日本語文章リンター | CLI + VS Code拡張 |
| **textlint-rule-preset-ja-technical-writing** | 技術文書向けルールセット | ベースプリセット |
| **カスタムルール** | Isidorica固有ルール | ひらく語、外来語長音、禁止表現 |

#### CI/CD実行

| 項目 | 内容 |
| :--- | :--- |
| **実行タイミング** | PRごと（`ci.yml`） |
| **対象ファイル** | `articles/**/*.md` |
| **コマンド** | `pnpm textlint articles/` |

#### チェック項目と失敗時の動作

| チェック | ルール | 失敗時の動作 |
| :--- | :--- | :--- |
| **文長** | `sentence-length` (60文字) | 警告 |
| **読点過多** | `max-ten` (3個まで) | 警告 |
| **敬体/常体混在** | `no-mix-dearu-desumasu` | 警告 |
| **二重否定** | `no-double-negative-ja` | 警告 |
| **冗長表現** | `ja-no-redundant-expression` | 警告 |
| **表記ゆれ** | `prh`（辞書ファイル） | 🔧 自動修正（`--fix`） |
| **ひらく語** | カスタムルール | 警告 |
| **外来語長音** | カスタムルール | 警告 |
| **禁止表現** | カスタムルール（「あなた」等） | 警告 |

#### 運用方針

| フェーズ | 動作 | 備考 |
| :--- | :--- | :--- |
| **Phase 4（初期）** | 警告のみ（PRブロックしない） | レビュワーが確認 |
| **Phase 5以降** | 重要ルールはPRブロック | 段階的に厳格化 |
| **表記ゆれ** | 常に自動修正 | `--fix` でコミット |

---

## 3. Post-Merge Processing (マージ後処理)

`main` ブランチへのマージ後に実行される処理。

### 3.1 Content Archiving (パーマリンク保存)

> **参照**: [URL_AND_TAXONOMY_DESIGN.md](./URL_AND_TAXONOMY_DESIGN.md) セクション2

#### 処理フロー

```
main ブランチへの PR マージ
  ↓
GitHub Actions: アーカイブ生成ジョブが起動
  ↓
変更された記事を検出（git diff）
  ↓
各記事に対して:
  1. 正規化処理（updated_at除外、改行統一、末尾空白除去）
  2. ハッシュ計算（SHA-256 先頭12文字）
  3. permalinks.json で重複チェック
     ├─ 重複なし → R2 にアップロード、permalinks.json を更新
     └─ 重複あり → スキップ（同一コンテンツ）
  ↓
permalinks.json をコミット（自動）
```

#### ステップ詳細

| ステップ | 説明 |
| :--- | :--- |
| **1. 変更検出** | `git diff` で更新された記事ファイルを特定 |
| **2. 正規化** | `updated_at` 除外、改行コード統一（LF）、末尾空白除去、EOF改行追加 |
| **3. ハッシュ計算** | 正規化後のMarkdown全体の `SHA-256` ハッシュ（先頭**12文字**）を計算 |
| **4. 重複チェック** | `permalinks.json` で同一ハッシュを検索。同一コンテンツならスキップ |
| **5. R2アップロード** | 正規化後のMarkdownを `isidorica-archives/{lang}/{slug}/{hash}.md` にアップロード |
| **6. 履歴更新** | `permalinks.json` に新ハッシュとメタデータを追記 |
| **7. Git Commit** | `permalinks.json` の変更をBotが自動コミット |

#### permalinks.json スキーマ

```json
{
  "ja": {
    "game-theory": {
      "currentHash": "a1b2c3d4e5f6",
      "history": [
        { "hash": "a1b2c3d4e5f6", "date": "2025-12-22", "message": "バージョン2.0" },
        { "hash": "f7e8d9c0b1a2", "date": "2025-11-01", "message": "初版公開" }
      ]
    }
  },
  "en": {
    "game-theory": {
      "currentHash": "c3d4e5f6a1b2",
      "history": [
        { "hash": "c3d4e5f6a1b2", "date": "2025-12-22", "message": "Initial translation" }
      ]
    }
  }
}
```

| フィールド | 説明 |
| **lang** | 言語コード（`ja`, `en` 等） |
| **slug** | 記事のSlug |
| **currentHash** | 最新版のハッシュ（12文字） |
| **history** | 過去バージョンの履歴（ハッシュ、日付、メッセージ） |

> **アーカイブ閲覧時の処理**: ユーザーがアーカイブURLにアクセスした際の処理（最新版リダイレクト、警告バナー表示等）はランタイム処理（Cloudflare Workers）であり、CI/CDとは別です。詳細は [URL_AND_TAXONOMY_DESIGN.md](./URL_AND_TAXONOMY_DESIGN.md) セクション2.7を参照。

### 3.2 Image Processing (画像処理)

> **参照**: [IMAGE_MANAGEMENT.md](../../engine/IMAGE_MANAGEMENT.md)

| ステップ | 説明 |
| :--- | :--- |
| **1. 画像検出** | PRで追加された `articles/**/images/**` を検出 |
| **2. 元画像バックアップ** | `isidorica-originals` バケットにアップロード |
| **3. 最適化** | Sharp で WebP/AVIF 変換、リサイズ（最大1200px） |
| **4. 配信用アップロード** | `isidorica-assets` バケットにアップロード |
| **5. パス変換** | Markdown内の相対パスをR2 URLに変換 |

### 3.3 Artifact Generation (成果物生成)

| 成果物 | ファイル | 内容 | 参照 |
| :--- | :--- | :--- | :--- |
| **カタログ** | `CATALOG.md` | 全記事のタイトル、Slug、物理パス、タグ | [URL_AND_TAXONOMY_DESIGN.md](./URL_AND_TAXONOMY_DESIGN.md) |
| **出典使用状況** | `sources/usage-index.json` | 各出典IDと使用記事のリスト | [GOVERNANCE.md](../GOVERNANCE.md) |

### 3.4 Engine Deploy Trigger

コンテンツ更新を検知し、`isidorica-engine` リポジトリのデプロイワークフローをトリガーする。

| 方法 | 説明 |
| :--- | :--- |
| **Repository Dispatch** | GitHub APIでEngineリポジトリのワークフローを起動 |

---

## 4. Scheduled Workflows (定期実行)

### 4.1 Weekly: Review Check

> **参照**: [GOVERNANCE.md](../GOVERNANCE.md) セクション4.4

| 項目 | 内容 |
| :--- | :--- |
| **実行タイミング** | 毎週月曜日 |
| **処理内容** | レビュー停滞中のPRを検出し、通知Issueを作成 |
| **通知対象** | 7日以上レビューがないPR |

### 4.2 Weekly: Link Check

| 項目 | 内容 |
| :--- | :--- |
| **実行タイミング** | 毎週 |
| **処理内容** | 記事内の外部リンク切れをチェック |
| **出力** | リンク切れがあればIssue作成 |

### 4.3 Monthly: Report Generation

> **参照**: [GOVERNANCE.md](../GOVERNANCE.md) セクション5

| 項目 | 内容 |
| :--- | :--- |
| **実行タイミング** | 毎月1日 |
| **処理内容** | 健全性レポート生成、リマインダーIssue作成 |

#### 生成されるレポート

| レポート | 内容 |
| :--- | :--- |
| **健全性レポート** | 記事数、平均出典数、レビュー待ち件数 |
| **孤立した関係** | `concept-relations.yml` の概念が `topics.yml` にない |
| **未使用トピック** | どの記事も使用していないトピック |
| **定期見直しリマインダー** | 更新頻度に基づく見直し対象記事 |

### 4.4 Monthly: Unused Source Cleanup

> **参照**: [GOVERNANCE.md](../GOVERNANCE.md) セクション6.4

| 項目 | 内容 |
| :--- | :--- |
| **実行タイミング** | 毎月1日 |
| **処理内容** | `usage-index.json` と `bibliography.json` を照合し、deprecatedかつ未使用の出典を削除するPRを自動作成 |
| **出力** | 削除候補PR（レビュワー確認待ち） |

### 4.5 On-Demand: Translation Sync

> **参照**: [GITHUB_TEMPLATES.md](../../shared/GITHUB_TEMPLATES.md)

| 項目 | 内容 |
| :--- | :--- |
| **トリガー** | オリジナル記事の更新がマージされたとき |
| **処理内容** | 翻訳版の更新が必要なことを示すIssueを自動作成 |

---

## 5. Bot Operations (Botによる自動操作)

### 5.1 自動コミット

| 操作 | トリガー | 内容 |
| :--- | :--- | :--- |
| **Topics alias修正** | PR時のTopicsチェック | aliasを正規IDに置換してコミット |
| **permalinks.json更新** | mainマージ後 | ハッシュ履歴を更新してコミット |
| **画像パス変換** | mainマージ後 | 相対パスをR2 URLに変換してコミット |

### 5.2 自動ラベル付与

| ラベル | 条件 | 参照 |
| :--- | :--- | :--- |
| `new-topic` | 未登録のTopicが含まれる | [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) |
| `needs-translation-review` | 翻訳記事の更新PR | - |
| `priority:high/medium/low` | レビュー優先度スコアに基づく | [GOVERNANCE.md](../GOVERNANCE.md) |

### 5.3 自動Issue作成

| Issue | トリガー | 参照 |
| :--- | :--- | :--- |
| **翻訳更新要求** | オリジナル記事更新 | [GITHUB_TEMPLATES.md](../../shared/GITHUB_TEMPLATES.md) |
| **レビュー停滞通知** | 週次チェック | [GOVERNANCE.md](../GOVERNANCE.md) |
| **リンク切れ報告** | 週次チェック | - |
| **定期見直しリマインダー** | 月次チェック | [GOVERNANCE.md](../GOVERNANCE.md) |

### 5.4 自動PR作成

| PR | トリガー | 参照 |
| :--- | :--- | :--- |
| **未使用出典削除** | 月次チェック | [GOVERNANCE.md](../GOVERNANCE.md) |
| **翻訳版ディレクトリ移動** | オリジナルの主題ディレクトリ変更時 | [URL_AND_TAXONOMY_DESIGN.md](./URL_AND_TAXONOMY_DESIGN.md) |

---

## 6. ツールスタック

| カテゴリ | ツール | 用途 |
| :--- | :--- | :--- |
| **CI Platform** | GitHub Actions | ワークフロー実行基盤 |
| **Storage** | Cloudflare R2 | 画像、アーカイブデータの保存 |
| **Linter** | textlint | 日本語文章チェック |
| **Validation** | Zod | Frontmatterスキーマ検証 |
| **Validation** | Custom Scripts | Slug一致、Topics検証等 |
| **Image** | Sharp | 画像最適化 |
| **Upload** | wrangler | R2アップロード |

---

## 7. ワークフローファイル一覧

| ファイル | 用途 | 実装フェーズ |
| :--- | :--- | :--- |
| `.github/workflows/ci.yml` | PR時のコンテンツバリデーション | Phase 4 |
| `.github/workflows/topics-check.yml` | Topicsバリデーション | Phase 4 |
| `.github/workflows/post-merge.yml` | マージ後処理（アーカイブ、画像、トリガー） | Phase 4 |
| `.github/workflows/image-upload.yml` | 画像処理 | Phase 4 |
| `.github/workflows/review-check.yml` | 週次レビューチェック | Phase 4 |
| `.github/workflows/translation-sync.yml` | 翻訳同期Issue作成 | Phase 4 |
| `.github/workflows/monthly-report.yml` | 月次レポート | Phase 4 |
| `.github/workflows/source-cleanup.yml` | 未使用出典削除PR作成 | Phase 4 |
| `.github/workflows/link-check.yml` | 週次リンクチェック | Phase 4 |
| `.github/workflows/translation-dir-sync.yml` | 翻訳版ディレクトリ追従PR作成 | Phase 4 |

---

## 8. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [Engine CI/CD](../../engine/technical/CI_CD_PIPELINE.md) | UI・ビルドロジックのCI/CD |
| [URL_AND_TAXONOMY_DESIGN.md](./URL_AND_TAXONOMY_DESIGN.md) | パーマリンク、Slug一致ルール |
| [NETWORK_ARCHITECTURE.md](./NETWORK_ARCHITECTURE.md) | Topicsバリデーション |
| [IMAGE_MANAGEMENT.md](../../engine/IMAGE_MANAGEMENT.md) | 画像処理フロー |
| [GOVERNANCE.md](../GOVERNANCE.md) | レビューポリシー、定期見直し |
| [GITHUB_TEMPLATES.md](../../shared/GITHUB_TEMPLATES.md) | Issue/PRテンプレート |
| [MARKDOWN_SPEC.md](./MARKDOWN_SPEC.md) | Frontmatterスキーマ |
