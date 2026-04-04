# コントリビューターガイド (Contributing Guide)

本ドキュメントでは、`isidorica-engine` への貢献方法を説明する。

**ステータス**: ドラフト

**注意**: 本ドキュメントは `isidorica-engine` リポジトリのルートに配置予定。現在はPRDの一部としてドラフトを作成している。

---

## 1. 概要

`isidorica-engine` はIsidoricaのシステム（UI、ビルドロジック）を管理するリポジトリである。

| 項目 | 内容 |
| :--- | :--- |
| **技術スタック** | Astro, Lit, TypeScript |
| **対象** | 開発者 |
| **コンテンツ** | UIコンポーネント、スタイル、ビルドロジック |

---

## 2. 開発環境のセットアップ

### 2.1 前提条件

| ツール | バージョン | 備考 |
| :--- | :--- | :--- |
| **Node.js** | 20.x (LTS) | fnmで管理推奨 |
| **pnpm** | 9.x | パッケージマネージャー |
| **Git** | 最新版 | - |

### 2.2 リポジトリのクローン

Engine と Library は**完全に独立したリポジトリ**として管理されている。

#### Engine 開発者（UI・ビルドロジック開発）

```bash
# isidorica-engine をクローン
git clone https://github.com/isidorica/isidorica-engine.git
cd isidorica-engine

# 依存関係のインストール
pnpm install

# コンテンツなしで開発する場合（UI開発等）
pnpm dev

# コンテンツを含めてプレビューしたい場合
git clone https://github.com/isidorica/isidorica-library.git content
pnpm dev
```

*   `content/` は `.gitignore` に含まれており、Engine リポジトリには含まれない。
*   コンテンツを必要としない UI 開発であれば、Library のクローンは不要。

### 2.3 エディタ設定

**推奨エディタは定めない。** 好みのエディタ（Vim, Emacs, VS Code, Cursor, その他）を自由に使用できる。

---

## 3. 開発ワークフロー

### 3.1 ブランチ戦略

**TODO**: `isidorica-engine` のブランチ戦略を定義する。

| 候補 | 説明 |
| :--- | :--- |
| **GitHub Flow** | シンプル（main + feature branches） |
| **Git-flow** | develop, release, hotfix を含む |

### 3.2 コミットメッセージ

**TODO**: コミットメッセージの規約を定義する（Conventional Commits等）。

### 3.3 PRの作成

1. ブランチを作成
2. 変更をコミット
3. PRを作成（タイトルで作業内容を明確に説明）
4. レビューを待つ
5. マージ

---

## 4. コードスタイル

### 4.1 Linter / Formatter

| ツール | 用途 |
| :--- | :--- |
| **Biome** | Linter + Formatter |
| **ESLint (jsx-a11y)** | アクセシビリティチェック |

### 4.2 実行コマンド

```bash
# Lint
pnpm lint

# Format
pnpm format

# 型チェック
pnpm typecheck
```

---

## 5. セキュリティ制約（CSP）

### 5.1 インライン禁止

本プロジェクトでは**厳格なContent Security Policy（CSP）** を採用している。以下のインライン記述は**本番環境で動作しない**。

| 禁止される記述 | 例 | 代替方法 |
| :--- | :--- | :--- |
| **インラインスクリプト** | `<script>alert('hello')</script>` | 外部ファイル化 (`.js`) |
| **イベントハンドラ属性** | `<button onclick="...">` | `addEventListener()` を使用 |
| **インラインスタイル** | `<div style="color: red">` | 外部CSS (`.css`) またはクラス使用 |
| **styleタグ** | `<style>...</style>` | 外部CSS (`.css`) |

### 5.2 開発時の注意

*   **開発環境では動作しても本番では動作しない**ことがある。必ずビルド後に確認すること。
*   Lit Web Componentsが生成するスタイルについては、将来的にnonce/hashを導入予定。
*   詳細は [NON_FUNCTIONAL_REQUIREMENTS.md](./NON_FUNCTIONAL_REQUIREMENTS.md) を参照。

### 5.3 正しい書き方

#### ❌ 禁止: インラインイベントハンドラ

```html
<button onclick="handleClick()">クリック</button>
```

#### ✅ 推奨: addEventListener

```typescript
// component.ts
button.addEventListener('click', handleClick);
```

#### ❌ 禁止: インラインスタイル

```html
<div style="color: red; font-size: 16px;">テキスト</div>
```

#### ✅ 推奨: CSSクラス

```html
<div class="error-text">テキスト</div>
```

```css
/* styles.css */
.error-text {
  color: red;
  font-size: 16px;
}
```

---

## 6. テスト

### 6.1 テストフレームワーク

| ツール | 用途 |
| :--- | :--- |
| **Vitest** | ユニットテスト |
| **Storybook** | コンポーネントカタログ |

### 6.2 実行コマンド

```bash
# テスト実行
pnpm test

# テスト（ウォッチモード）
pnpm test:watch

# Storybook起動
pnpm storybook
```

---

## 7. デバッグ

### 7.1 ブラウザデバッグ

開発サーバー起動後、ブラウザの開発者ツールでデバッグ可能。

```bash
pnpm dev
# http://localhost:4321 でアクセス
```

### 7.2 VS Code / Cursor でのデバッグ

**TODO**: `.vscode/launch.json` を提供する。

以下のデバッグ設定を予定：

| 設定名 | 用途 |
| :--- | :--- |
| **Astro: Dev Server** | 開発サーバーのデバッグ |
| **Vitest: Debug Tests** | テストのデバッグ実行 |
| **Storybook: Debug** | Storybookのデバッグ |

### 7.3 コマンドラインでのデバッグ

```bash
# Node.js デバッグモードで実行
node --inspect-brk node_modules/.bin/astro dev

# Vitest デバッグモードで実行
node --inspect-brk node_modules/.bin/vitest run
```

---

## 8. ドキュメント

### 8.1 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [SYSTEM_ARCHITECTURE.md](./technical/SYSTEM_ARCHITECTURE.md) | システムアーキテクチャ |
| [ARCHITECTURE_DECISION_RECORD.md](./technical/ARCHITECTURE_DECISION_RECORD.md) | 技術選定 |

---

## 9. 記事作成CLI (Scaffolding Tool)

新規記事ファイルの作成を支援するCLIツール（`pnpm new-article`）を提供する。

*   **分野選択**: 対話形式で分野を選択し、適切なディレクトリに自動配置。
*   **Slug重複チェック**: 既存Slugとの重複を事前チェック。
*   **Frontmatterテンプレート**: 必須フィールドを含むテンプレートを自動生成。

執筆者は「分野」を選択するだけで、引用形式に基づく適切なディレクトリにファイルが作成される。これにより、ディレクトリ構造を意識する必要がなくなる。

> **詳細**: CLIツールの仕様、対話フロー、設計原則については [CLI_TOOLS_SPEC.md](./technical/CLI_TOOLS_SPEC.md) を参照。

### GitHub Web UIからの記事作成について

| 用途 | GitHub Web UI |
| :--- | :--- |
| **typo修正、軽微な編集** | ✅ 使用可能 |
| **新規記事作成** | ⚠️ **非推奨** |

**非推奨の理由**:
*   Frontmatterの必須フィールドを手動で入力する必要があり、ミスを誘発する。
*   CLIツールが提供する重複チェックやテンプレート生成が利用できない。

新規記事を作成する場合は、必ずCLIツール（`pnpm new-article`）か執筆者支援ツールを使用すること。

---

## 10. フォント管理

本プロジェクトでは、プライバシー保護のためフォントを**セルフホスト**し、Web Vitals最適化のため**サブセット化**を行う。

### 9.1 ツール

| ツール | 用途 |
| :--- | :--- |
| **@fontsource/noto-sans-jp** | フォントソース（npmパッケージ） |
| **pyftsubset** | サブセット生成（fonttools） |
| **Renovate / Dependabot** | 依存関係の更新監視 |

### 9.2 サブセット生成

```bash
# pyftsubsetのインストール（Python環境）
pip install fonttools brotli

# サブセット生成（常用漢字 + ひらがな + カタカナ + ASCII）
pyftsubset NotoSansJP-Regular.ttf \
  --unicodes="U+0020-007E,U+3040-309F,U+30A0-30FF,U+4E00-9FFF" \
  --flavor=woff2 \
  --output-file=public/fonts/NotoSansJP-Regular-subset.woff2

# Boldも同様に生成
pyftsubset NotoSansJP-Bold.ttf \
  --unicodes="U+0020-007E,U+3040-309F,U+30A0-30FF,U+4E00-9FFF" \
  --flavor=woff2 \
  --output-file=public/fonts/NotoSansJP-Bold-subset.woff2
```

*   **Unicode範囲**:
    *   `U+0020-007E`: ASCII（英数字、記号）
    *   `U+3040-309F`: ひらがな
    *   `U+30A0-30FF`: カタカナ
    *   `U+4E00-9FFF`: CJK統合漢字（常用漢字含む）
*   **出力形式**: WOFF2（最新の圧縮形式）
*   **出力先**: `public/fonts/`

### 9.3 フォント更新ワークフロー

```
Renovate/Dependabot: @fontsource/noto-sans-jp の更新PRを作成
    ↓
レビュワー: PRをレビュー
    ↓
マージ後: pnpm run build:fonts でサブセットを再生成
    ↓
生成されたフォントをコミット
```

### 9.4 ビルドスクリプト（将来実装）

```json
// package.json
{
  "scripts": {
    "build:fonts": "node scripts/build-fonts.js"
  }
}
```

*   **詳細**: [ARCHITECTURE_DECISION_RECORD.md](./technical/ARCHITECTURE_DECISION_RECORD.md) のFont Strategyを参照。

---

## 11. CI/CD パイプライン

### 11.1 概要

GitHub ActionsでCI/CDパイプラインを構築し、Cloudflare Pagesにデプロイする。

```
PR作成/更新
    ↓
GitHub Actions (CI)
    ├── Format & Lint (Biome)
    ├── Accessibility Lint (ESLint)
    ├── 型チェック (TypeScript)
    ├── Unit Test (Vitest)
    ├── Component Test (Storybook)
    ├── ビルド (Astro)
    ├── Lighthouse CI
    └── セキュリティ監査 (pnpm audit)
    ↓
すべてPass → PRマージ可能
    ↓
main へのマージ
    ↓
GitHub Actions (CD)
    ↓
Cloudflare Pages へデプロイ
```

### 11.2 ワークフローファイル

| ファイル | 用途 |
| :--- | :--- |
| `.github/workflows/ci.yml` | PR時のCIチェック |
| `.github/workflows/deploy.yml` | mainへのマージ時のデプロイ（または Cloudflare Pages 自動連携） |

### 11.3 CI ワークフロー詳細

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint    # Biome
      - run: pnpm lint:a11y  # ESLint (eslint-plugin-lit-a11y)

  typecheck:
    name: Type Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck

  test:
    name: Unit & Component Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm test           # Vitest
      - run: pnpm test:storybook  # Storybook Interaction Testing

  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm build

  lighthouse:
    name: Lighthouse CI
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      - name: Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          configPath: ./lighthouserc.json
          uploadArtifacts: true

  security:
    name: Security Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - run: pnpm audit --audit-level=high
```

### 11.4 デプロイフロー

#### Cloudflare Pages との連携

Cloudflare Pagesの無料プランはビルド時間制限（月500分）があるため、**GitHub Actionsでビルドし、Direct Uploadでデプロイ**する。

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      - name: Deploy to Cloudflare Pages
        uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: isidorica
          directory: dist
```

### 11.5 ローカルでのCIチェック実行

PRを作成する前に、ローカルでCIチェックを実行することを推奨する。

```bash
# 全チェックを順次実行
pnpm lint
pnpm lint:a11y
pnpm typecheck
pnpm test
pnpm build
```

### 11.6 ロールバック手順

デプロイ後に問題が発生した場合のロールバック手順。

1. **Cloudflare Pagesダッシュボード**にアクセス
2. プロジェクト → **Deployments** を開く
3. 正常に動作していた以前のデプロイを選択
4. **「Rollback to this deployment」** をクリック
5. 即座に以前のバージョンに切り替わる

> **注意**: ロールバックは一時的な対応。必ず原因を調査し、修正版を再デプロイすること。

---

## 12. TODO

本ドキュメントは現在ドラフト段階である。以下の項目は今後整備予定。

| 項目 | 状況 | 備考 |
| :--- | :--- | :--- |
| **ブランチ戦略の定義** | 未着手 | GitHub Flow or Git-flow の選定 |
| **`.vscode/launch.json` の提供** | 未着手 | Astro/Vitest/Storybookのデバッグ設定 |
| **コミットメッセージ規約** | 未着手 | Conventional Commits等の検討 |
| **Issue/PRテンプレート** | 未着手 | `.github/ISSUE_TEMPLATE/`, `.github/PULL_REQUEST_TEMPLATE.md` |
