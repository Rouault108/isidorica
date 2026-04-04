# GitHub テンプレート仕様

本ドキュメントでは、Isidoricaプロジェクトで使用する GitHub PR/Issue テンプレートの仕様を定義する。

**ステータス**: ドラフト

---

## 1. ディレクトリ構成

```
.github/
├── PULL_REQUEST_TEMPLATE/
│   ├── article-update.md      # 記事更新用（Library リポジトリ）
│   └── engine-change.md       # Engine変更用（Engine リポジトリ）
│
├── ISSUE_TEMPLATE/
│   ├── article-review.md      # 記事レビュー依頼
│   ├── translation-needed.md  # 翻訳依頼（自動生成用テンプレート）
│   ├── new-article.md         # 新規記事提案
│   └── bug-report.md          # バグ報告
│
└── workflows/
    ├── translation.yml        # 翻訳Issue自動作成
    ├── review-check.yml       # 定期更新チェック
    └── topics-check.yml       # Topics バリデーション
```

---

## 2. PR テンプレート

### 2.1 記事更新用 (`article-update.md`)

Library リポジトリで記事を追加・更新する際に使用する。

#### テンプレート内容

```markdown
## 変更内容

<!-- 変更内容を簡潔に記述してください -->

## 変更の種類

- [ ] 📝 新規記事
- [ ] ✏️ 既存記事の更新
- [ ] 🐛 誤字・誤植の修正
- [ ] 📚 出典の追加・修正

## チェックリスト

- [ ] Frontmatter が正しく記述されている
- [ ] 見出し構造が適切（H2 → H3 → H4 の順序）
- [ ] 出典が正しく引用されている
- [ ] ローカルでプレビューを確認した（該当する場合）

---

## 翻訳優先度（オプション）

以下のいずれかをチェックすると、翻訳優先度の自動計算をオーバーライドします。
何もチェックしない場合は、学習パス含有数・被依存数等から自動計算されます。

- [ ] 🚨 `critical` - 緊急で翻訳が必要（30日後にAI翻訳フォールバック）
- [ ] ⬆️ `high` - 優先的に翻訳が必要（45日後にAI翻訳フォールバック）
- [ ] ⬇️ `low` - 翻訳の優先度を下げる（90日後にAI翻訳フォールバック）
- [ ] 🚫 `none` - 翻訳対象外（翻訳Issueを作成しない）

---

## 関連Issue

<!-- 関連するIssueがあればリンクしてください -->
<!-- Closes #123 -->
```

#### チェックボックスの意味

| チェックボックス | 効果 | AI翻訳フォールバック |
| :--- | :--- | :--- |
| 🚨 `critical` | 優先度スコアを100に設定 | 30日後 |
| ⬆️ `high` | 優先度スコアを80に設定 | 45日後 |
| ⬇️ `low` | 優先度スコアを20に設定 | 90日後 |
| 🚫 `none` | 翻訳Issueを作成しない | - |
| 未チェック | 自動計算（学習パス、被依存数、安定性から算出） | スコアに応じて決定 |

> **参照**: 優先度スコアの計算ロジックは [GOVERNANCE.md](../library/GOVERNANCE.md) のセクション8.3「翻訳優先度」を参照。

### 2.2 Engine変更用 (`engine-change.md`)

**TODO**: Engine リポジトリ用のPRテンプレートを定義する。

---

## 3. Issue テンプレート

### 3.1 翻訳依頼 (`translation-needed.md`)

オリジナル記事が更新された際に、GitHub Actions が自動で作成する Issue のテンプレート。

#### テンプレート内容

```markdown
---
name: 🌐 翻訳更新依頼
about: オリジナル記事が更新されたため、翻訳版の更新が必要です
labels: translation-needed, auto-generated
---

## 🌐 翻訳版の更新が必要です

| 項目 | 値 |
|:-----|:---|
| **記事** | [{{slug}}](/{{lang}}/{{slug}}) |
| **言語** | {{language_name}} |
| **オリジナル更新日** | {{original_updated_at}} |
| **オリジナル更新PR** | #{{original_pr_number}} |
| **翻訳版最終更新** | {{translation_updated_at}} |
| **優先度スコア** | {{priority_score}}/100 |
| **優先度ラベル** | `{{priority_label}}` |
| **AI翻訳フォールバック** | {{fallback_days}}日後 |

### スコア内訳

- 学習パス含有: {{learning_path_count}}パス → {{learning_path_score}}点
- 被依存数: {{required_by_count}}記事 → {{required_by_score}}点
- 安定性: {{days_since_update}}日 → {{stability_score}}点
- 手動指定: {{manual_override}}

## アクション

- [ ] 翻訳版を確認し、オリジナルの変更を反映
- [ ] PRを作成して更新
- [ ] 翻訳が不要な場合（軽微な変更等）は理由をコメントしてクローズ

---

⚠️ {{fallback_days}}日後に対応がない場合、AI翻訳によるドラフトPRが自動作成されます。
```

#### 変数一覧

| 変数 | 説明 |
| :--- | :--- |
| `{{slug}}` | 記事のslug |
| `{{lang}}` | 翻訳版の言語コード（en, zh, ko等） |
| `{{language_name}}` | 言語名（English, 中文, 한국어等） |
| `{{original_updated_at}}` | オリジナル記事の更新日 |
| `{{original_pr_number}}` | オリジナル更新のPR番号 |
| `{{translation_updated_at}}` | 翻訳版の最終更新日 |
| `{{priority_score}}` | 優先度スコア（0-100） |
| `{{priority_label}}` | 優先度ラベル（critical/high/medium/low） |
| `{{fallback_days}}` | AI翻訳フォールバックまでの日数 |

### 3.2 記事レビュー依頼 (`article-review.md`)

**TODO**: 定期更新チェックで自動作成される Issue のテンプレートを定義する。

### 3.3 新規記事提案 (`new-article.md`)

**TODO**: 新規記事を提案する際の Issue テンプレートを定義する。

### 3.4 バグ報告 (`bug-report.md`)

**TODO**: バグ報告用の Issue テンプレートを定義する。

---

## 4. GitHub Actions との連携

### 4.1 PRチェックボックスの読み取り

PR がマージされた際、GitHub Actions が PR の本文（body）を解析し、チェックボックスの状態を読み取る。

```yaml
# .github/workflows/translation.yml
name: Create Translation Issues

on:
  pull_request:
    types: [closed]
    branches: [main]
    paths:
      - 'articles/**/*.md'

jobs:
  create-translation-issue:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Parse translation priority from PR body
        id: priority
        env:
          PR_BODY: ${{ github.event.pull_request.body }}
        run: |
          if echo "$PR_BODY" | grep -q '\[x\].*🚨.*critical'; then
            echo "priority=critical" >> $GITHUB_OUTPUT
            echo "score=100" >> $GITHUB_OUTPUT
            echo "fallback_days=30" >> $GITHUB_OUTPUT
          elif echo "$PR_BODY" | grep -q '\[x\].*⬆️.*high'; then
            echo "priority=high" >> $GITHUB_OUTPUT
            echo "score=80" >> $GITHUB_OUTPUT
            echo "fallback_days=45" >> $GITHUB_OUTPUT
          elif echo "$PR_BODY" | grep -q '\[x\].*⬇️.*low'; then
            echo "priority=low" >> $GITHUB_OUTPUT
            echo "score=20" >> $GITHUB_OUTPUT
            echo "fallback_days=90" >> $GITHUB_OUTPUT
          elif echo "$PR_BODY" | grep -q '\[x\].*🚫.*none'; then
            echo "priority=none" >> $GITHUB_OUTPUT
            echo "skip=true" >> $GITHUB_OUTPUT
          else
            echo "priority=auto" >> $GITHUB_OUTPUT
          fi
      
      - name: Calculate priority score (if auto)
        if: steps.priority.outputs.priority == 'auto'
        id: calculate
        run: |
          # 学習パス含有数、被依存数、安定性から計算
          # 実装詳細は別途スクリプトで定義
          node scripts/calculate-translation-priority.js
      
      - name: Create translation issues
        if: steps.priority.outputs.skip != 'true'
        run: |
          # 各翻訳版に対してIssueを作成
          # 実装詳細は別途スクリプトで定義
          node scripts/create-translation-issues.js
```

### 4.2 ラベル自動付与

翻訳 Issue 作成時に、優先度に応じたラベルを自動付与する。

| 優先度 | 付与されるラベル |
| :--- | :--- |
| critical | `translation-needed`, `priority:critical`, `auto-generated` |
| high | `translation-needed`, `priority:high`, `auto-generated` |
| medium | `translation-needed`, `priority:medium`, `auto-generated` |
| low | `translation-needed`, `priority:low`, `auto-generated` |

---

## 5. 今後の課題

| 項目 | ステータス |
| :--- | :--- |
| 記事更新PR テンプレートの実装 | ⬜ 未着手（Phase 4.6） |
| 翻訳Issue テンプレートの実装 | ⬜ 未着手（Phase 4.6） |
| Engine変更PR テンプレートの定義 | ⬜ 未着手 |
| 記事レビューIssue テンプレートの定義 | ⬜ 未着手（Phase 4.6） |
| GitHub Actions の実装 | ⬜ 未着手（Phase 4.6） |
