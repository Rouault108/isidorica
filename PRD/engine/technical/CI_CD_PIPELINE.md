# CI/CD パイプライン仕様 - Engine (CI/CD Pipeline - Engine)

本ドキュメントでは、`isidorica-engine` リポジトリの CI/CD パイプライン仕様を定義する。

**ステータス**: フェーズ4実装予定
**最終更新**: 2025-12-22

---

## 1. 概要

`isidorica-engine` は Isidorica の UI、ビルドロジック、スタイルを管理するリポジトリである。
本パイプラインは、**コード品質の保証**と**安全なデプロイ**を自動化する。

> **注意**: コンテンツ（記事）に関するバリデーションやアーカイブ処理は `isidorica-library` リポジトリで管理される。[Library CI/CD](../../library/technical/CI_CD_PIPELINE.md) を参照。

### 1.1 ワークフロー一覧

| ワークフロー名 | ファイル名 | トリガー | 目的 |
| :--- | :--- | :--- | :--- |
| **Pull Request Checks** | `ci.yml` | PR作成・更新 | マージ前の品質チェック（Lint, Test, Build） |
| **Deploy** | `deploy.yml` | mainへのプッシュ | 本番デプロイ |
| **Dependency Update** | Renovate (GitHub App) | 依存関係更新時 | パッケージ更新PR自動作成 |

### 1.2 アーキテクチャ

```
┌─────────────────── Pull Request ───────────────────┐
│                                                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ Lint &   │  │ Type     │  │ Unit Test        │  │
│  │ Format   │  │ Check    │  │ (Vitest)         │  │
│  └──────────┘  └──────────┘  └──────────────────┘  │
│       ↓             ↓               ↓              │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ A11y     │  │ Build    │  │ Lighthouse CI    │  │
│  │ Lint     │  │          │  │                  │  │
│  └──────────┘  └──────────┘  └──────────────────┘  │
│                      ↓                             │
│              All Pass? → Merge可能                  │
└────────────────────────────────────────────────────┘
                       ↓
┌─────────────────── main マージ ────────────────────┐
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │ Build (Astro) + Deploy (Cloudflare Pages)    │ │
│  └──────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘
```

---

## 2. Pull Request Checks (Quality Gate)

マージを許可するための必須チェック。エラーがある場合はマージをブロックする。

> **参照**: [CONTRIBUTING.md](../CONTRIBUTING.md) セクション10

### 2.1 Static Analysis (静的解析)

| チェック | ツール | 失敗時の動作 |
| :--- | :--- | :--- |
| **Format & Lint** | Biome | PRブロック |
| **Accessibility Lint** | ESLint (`eslint-plugin-lit-a11y`) | PRブロック |
| **Type Check** | TypeScript (`tsc --noEmit`) | PRブロック |
| **Spell Check** | Typos (crate-ci/typos) | 警告 |

> **参照**: [ARCHITECTURE_DECISION_RECORD.md](./ARCHITECTURE_DECISION_RECORD.md) セクション2.5

### 2.2 Tests

| テストタイプ | ツール | 対象 | 実行タイミング |
| :--- | :--- | :--- | :--- |
| **Unit Test** | Vitest | ユーティリティ関数、hooks、Transformer | PRごと |
| **Component Test** | Storybook Interaction Testing | UIコンポーネント | PRごと |
| **Visual Regression** | Storybook + Chromatic | UIコンポーネントの見た目 | PRごと |
| **Accessibility Test** | axe-core | 全ページ | PRごと |
| **E2E Smoke Test** | Playwright | 主要ページ（トップ、記事詳細、検索） | リリース前 |

> **参照**: [NON_FUNCTIONAL_REQUIREMENTS.md](../NON_FUNCTIONAL_REQUIREMENTS.md) セクション6.1

#### カバレッジ目標

| 対象 | 目標 | テストタイプ |
| :--- | :--- | :--- |
| ユーティリティ関数 | 90%以上 | Unit (Vitest) |
| カスタムhooks | 90%以上 | Unit (Vitest) |
| Transformer | 90%以上 | Unit (Vitest) |
| UIコンポーネント | - | Component (Storybook) |
| ページ | - | E2E (Playwright) |

### 2.3 Performance & Security

| チェック | ツール | 失敗条件 |
| :--- | :--- | :--- |
| **Lighthouse CI** | treosh/lighthouse-ci-action | スコア閾値未達 |
| **Security Audit** | `pnpm audit` | high以上の脆弱性 |

### 2.4 Build Check

| チェック | 説明 | 失敗時の動作 |
| :--- | :--- | :--- |
| **Astro Build** | 静的サイト生成が成功するか | PRブロック |

### 2.5 Pre-commit Hook (ローカル)

PR作成前にローカルで実行されるチェック。`husky` + `lint-staged` で設定。

| チェック | ツール |
| :--- | :--- |
| **Format** | Biome (`biome format --write`) |
| **Lint** | Biome (`biome lint`) |
| **Type Check** | TypeScript (`tsc --noEmit`) |

---

## 3. Deploy (Post-Merge)

`main` ブランチへのマージ後に実行される処理。

### 3.1 Production Deploy

| ステップ | 説明 |
| :--- | :--- |
| **1. Checkout** | リポジトリをクローン |
| **2. Library取得** | `isidorica-library` リポジトリを `content/` にクローン |
| **3. Build** | Astro による静的サイト生成 (`pnpm build`) |
| **4. Publish** | Cloudflare Pages Direct Upload (`wrangler pages publish`) |

> **参照**: [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md) セクション1.2

#### Direct Upload を採用する理由

| 理由 | 説明 |
| :--- | :--- |
| **ビルド時間制限回避** | Cloudflare Pages無料プランのビルド時間制限（月500分）を回避 |
| **柔軟性** | GitHub Actionsで任意の前処理を実行可能 |

### 3.2 Library変更時のデプロイトリガー

`isidorica-library` リポジトリが更新された場合、Webhookを通じて `isidorica-engine` 側のデプロイをトリガーする。

> **参照**: [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md) セクション1.3.4

---

## 4. Rollback (ロールバック)

> **参照**: [CONTRIBUTING.md](../CONTRIBUTING.md) セクション10.6

### 4.1 即時ロールバック（Cloudflare Pages）

| ステップ | 内容 |
| :--- | :--- |
| 1 | Cloudflare Pagesダッシュボードにアクセス |
| 2 | プロジェクト → Deployments を開く |
| 3 | 正常に動作していた以前のデプロイを選択 |
| 4 | 「Rollback to this deployment」をクリック |

### 4.2 正式ロールバック（Git revert）

```bash
git revert <commit-hash>
git push origin main
# → 自動デプロイがトリガーされる
```

---

## 5. ツールスタック

| カテゴリ | ツール | 用途 |
| :--- | :--- | :--- |
| **CI Platform** | GitHub Actions | ワークフロー実行基盤 |
| **Hosting** | Cloudflare Pages | サイトホスティング |
| **Linter** | Biome | Format & Lint (TypeScript/JavaScript) |
| **Linter** | ESLint (`eslint-plugin-lit-a11y`) | Accessibility Lint |
| **Linter** | Typos | スペルチェック |
| **Testing** | Vitest | Unit Test |
| **Testing** | Storybook | Component Catalog & Test |
| **Testing** | Chromatic | Visual Regression Test |
| **Testing** | axe-core | Accessibility Test |
| **Testing** | Playwright | E2E Test |
| **Performance** | Lighthouse CI | パフォーマンス測定 |
| **Deploy** | wrangler | Cloudflare Pages Direct Upload |
| **Dependency** | Renovate | 依存関係自動更新 |
| **Pre-commit** | husky + lint-staged | ローカルフック |

---

## 6. ワークフローファイル一覧

| ファイル | 用途 | 実装フェーズ |
| :--- | :--- | :--- |
| `.github/workflows/ci.yml` | PR時のCIチェック | Phase 4 |
| `.github/workflows/deploy.yml` | mainマージ時のデプロイ | Phase 4 |

---

## 7. 品質基準

> **参照**: [NON_FUNCTIONAL_REQUIREMENTS.md](../NON_FUNCTIONAL_REQUIREMENTS.md) セクション6

| 指標 | 目標 | 確認方法 |
| :--- | :--- | :--- |
| **Lintエラー** | 0 | CIでブロック |
| **ビルド成功率** | 100% | `main` ブランチは常にビルド可能 |
| **テストカバレッジ（ユーティリティ）** | 90%以上 | Vitest |
| **Lighthouse Performance** | 90以上 | Lighthouse CI |
| **Lighthouse Accessibility** | 100 | Lighthouse CI |

---

## 8. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [Library CI/CD](../../library/technical/CI_CD_PIPELINE.md) | コンテンツバリデーション、アーカイブ保存 |
| [CONTRIBUTING.md](../CONTRIBUTING.md) | 開発者向けガイド |
| [NON_FUNCTIONAL_REQUIREMENTS.md](../NON_FUNCTIONAL_REQUIREMENTS.md) | 品質基準、テスト戦略 |
| [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md) | システム全体アーキテクチャ |
| [ARCHITECTURE_DECISION_RECORD.md](./ARCHITECTURE_DECISION_RECORD.md) | 技術選定 |

---

## 9. スケジュールワークフロー (Scheduled Workflows)

定期的に実行される自動化ワークフロー。

### 9.1 定期レビューリマインダー

PRDドキュメント等、定期的な見直しが必要な文書のレビューを促すIssueを自動作成する。

| 項目 | 内容 |
| :--- | :--- |
| **ファイル** | `.github/workflows/scheduled-review-reminder.yml` |
| **トリガー** | `schedule: cron '0 9 1 * *'`（毎月1日 9:00 UTC） |
| **実装フェーズ** | Phase 5 |

#### ワークフロー仕様

```yaml
name: Document Review Reminder

on:
  schedule:
    - cron: '0 9 1 * *'  # 毎月1日 18:00 JST
  workflow_dispatch:  # 手動実行も可能

jobs:
  check-review-dates:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Check documents for upcoming reviews
        id: check
        run: |
          # PRDドキュメントのヘッダーから「定期レビュー」を抽出
          # 30日以内にレビュー期限が到来するものを検出
          python3 scripts/check-review-dates.py \
            --docs-dir docs/PRD \
            --days-ahead 30 \
            --output review-report.json
          
      - name: Create Issues for due reviews
        if: steps.check.outputs.has_due_reviews == 'true'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const report = JSON.parse(fs.readFileSync('review-report.json', 'utf8'));
            
            for (const doc of report.due_reviews) {
              // 既存のOpenなIssueをチェック（重複防止）
              const existingIssues = await github.rest.issues.listForRepo({
                owner: context.repo.owner,
                repo: context.repo.repo,
                labels: 'review-reminder',
                state: 'open'
              });
              
              const alreadyExists = existingIssues.data.some(
                issue => issue.title.includes(doc.file)
              );
              
              if (!alreadyExists) {
                await github.rest.issues.create({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  title: `📋 定期レビュー: ${doc.file}`,
                  body: `## 定期レビューのお知らせ\n\n` +
                        `**対象ファイル**: \`${doc.path}\`\n` +
                        `**レビュー期限**: ${doc.review_date}\n` +
                        `**前回更新**: ${doc.last_updated}\n\n` +
                        `### チェックリスト\n` +
                        `- [ ] 技術選定の妥当性を再評価\n` +
                        `- [ ] 依存ライブラリの状態を確認（Deprecated、EOL等）\n` +
                        `- [ ] 新しい選択肢の検討\n` +
                        `- [ ] 「定期レビュー」日付を更新\n\n` +
                        `> このIssueは自動生成されました。`,
                  labels: ['review-reminder', 'documentation', 'priority:low']
                });
              }
            }
```

#### 検出対象

文書のヘッダーに以下の形式で記載された日付を解析：

```markdown
**定期レビュー**: 6ヶ月ごと（次回: 2026年7月）
```

#### 対象文書

> **参照**: レビュー対象文書の一覧は [GOVERNANCE.md §定期レビュー対象文書](../../library/GOVERNANCE.md) を参照。

### 9.2 専門家レビュー追跡ワークフロー

コンテンツ記事のPRがマージされた際に、専門家によるレビューが行われたかを自動追跡する。

> **参照**: [GOVERNANCE.md §専門家レジストリ](../../library/GOVERNANCE.md) 専門家の定義と認定プロセス
> **参照**: [KPI_FRAMEWORK.md §5.5](../../shared/KPI_FRAMEWORK.md) 専門家レビュー率

| 項目 | 内容 |
| :--- | :--- |
| **ファイル** | `.github/workflows/track-expert-review.yml` |
| **トリガー** | `pull_request` (closed, merged)（isidorica-library） |
| **実装フェーズ** | Phase 5 |

#### ワークフロー仕様

```yaml
name: Track Expert Review

on:
  pull_request:
    types: [closed]
    paths:
      - 'content/**/*.md'  # 記事ファイルのみ対象

jobs:
  track-expert-review:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Get PR reviewers
        id: reviewers
        uses: actions/github-script@v7
        with:
          script: |
            const reviews = await github.rest.pulls.listReviews({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.payload.pull_request.number
            });
            
            // 承認したレビュワーのみ抽出
            const approvers = reviews.data
              .filter(r => r.state === 'APPROVED')
              .map(r => r.user.login);
            
            return [...new Set(approvers)];
      
      - name: Check expert registry
        id: check-experts
        run: |
          # 専門家レジストリを読み込み
          EXPERTS=$(yq '.experts[].github' config/expert-reviewers.yml)
          APPROVERS='${{ steps.reviewers.outputs.result }}'
          
          # 承認者に専門家が含まれるか確認
          HAS_EXPERT=false
          EXPERT_NAMES=""
          
          for approver in $(echo $APPROVERS | jq -r '.[]'); do
            if echo "$EXPERTS" | grep -q "^$approver$"; then
              HAS_EXPERT=true
              EXPERT_NAMES="$EXPERT_NAMES $approver"
            fi
          done
          
          echo "has_expert=$HAS_EXPERT" >> $GITHUB_OUTPUT
          echo "expert_names=$EXPERT_NAMES" >> $GITHUB_OUTPUT
      
      - name: Record expert review
        if: steps.check-experts.outputs.has_expert == 'true'
        run: |
          # 変更されたファイルパスを取得
          FILES=$(git diff --name-only HEAD~1 HEAD | grep '^content/')
          
          # メトリクスログに記録（月次集計用）
          for file in $FILES; do
            echo "{\"date\": \"$(date -u +%Y-%m-%d)\", \"file\": \"$file\", \"experts\": \"${{ steps.check-experts.outputs.expert_names }}\", \"pr\": ${{ github.event.pull_request.number }}}" >> metrics/expert-reviews.jsonl
          done
          
          # 変更をコミット
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add metrics/expert-reviews.jsonl
          git commit -m "chore: record expert review for PR #${{ github.event.pull_request.number }}" || true
          git push
```

#### 集計方法

月次で `metrics/expert-reviews.jsonl` を集計し、KPI（専門家レビュー率）を算出する。

```bash
# 専門家レビュー済み記事数
jq -s '[.[].file] | unique | length' metrics/expert-reviews.jsonl

# 公開記事総数
find content -name '*.md' | wc -l

# 専門家レビュー率 = 専門家レビュー済み記事数 / 公開記事総数
```

#### 注意事項

| 項目 | 内容 |
| :--- | :--- |
| **同一記事の複数PR** | 最初の専門家レビューのみ記録（重複排除は集計時） |
| **専門家追加後の遡及** | 過去のレビューは遡及適用しない（追加時点以降のみ） |
| **分野マッチング** | 現時点では分野に関係なくカウント（将来的に分野別集計も検討） |

---

## 10. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [Library CI/CD](../../library/technical/CI_CD_PIPELINE.md) | コンテンツバリデーション、アーカイブ保存 |
| [CONTRIBUTING.md](../CONTRIBUTING.md) | 開発者向けガイド |
| [NON_FUNCTIONAL_REQUIREMENTS.md](../NON_FUNCTIONAL_REQUIREMENTS.md) | 品質基準、テスト戦略 |
| [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md) | システム全体アーキテクチャ |
| [ARCHITECTURE_DECISION_RECORD.md](./ARCHITECTURE_DECISION_RECORD.md) | 技術選定 |
| [GOVERNANCE.md](../../library/GOVERNANCE.md) | 定期レビュー対象文書のポリシー |
