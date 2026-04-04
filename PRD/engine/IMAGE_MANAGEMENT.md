# 画像管理 (Image Management)

本ドキュメントでは、Isidoricaにおける画像の管理方法を定義する。

**ステータス**: ドラフト
**最終更新**: 2025-12-15

---

## 1. 概要

Isidoricaでは、**Gitリポジトリの肥大化を防ぎ**、**執筆者の負担を最小化**するため、画像を **Cloudflare R2** で管理する。

### 基本方針

| 原則 | 説明 |
| :--- | :--- |
| **Gitに画像を含めない** | リポジトリの軽量化、クローン高速化 |
| **執筆者は技術を意識しない** | R2やCDNのURLを知る必要なし |
| **自動最適化** | CI/CDで画像を最適化（WebP/AVIF） |
| **元画像の保全** | 将来の再最適化に備えて元画像を保存 |

---

## 2. アーキテクチャ

### 2.1 2バケット方式

| バケット | 用途 | アクセス |
| :--- | :--- | :--- |
| `isidorica-originals` | 元画像（バックアップ） | プライベート |
| `isidorica-assets` | 最適化済み画像（配信） | パブリック |

### 2.2 配信ドメイン

| ドメイン | 用途 |
| :--- | :--- |
| `assets.isidorica.org` | 画像配信（R2カスタムドメイン） |

---

## 3. 執筆者向けガイド

### 3.1 執筆者タイプ

| タイプ | 編集方法 | 技術レベル |
| :--- | :--- | :--- |
| **A. 直接Markdown編集** | Git + エディタ（VSCode等） | 高（開発者） |
| **B. Webエディタ使用** | ブラウザ上のエディタ | 低〜中（一般執筆者） |

### 3.2 タイプA: 直接Markdown編集

#### 操作方法

1. 画像を記事と同じディレクトリの `images/` フォルダに配置
2. Markdownで相対パスで参照

```markdown
![囚人のジレンマの図解](./images/prisoner-dilemma.png)
```

3. PRを作成

#### システムが自動処理すること

- PR内の画像ファイルを検出
- `isidorica-originals` に元画像をアップロード
- 自動最適化（WebP生成、リサイズ）
- `isidorica-assets` に最適化済み画像をアップロード
- Markdown内のパスをR2 URLに自動変換

```markdown
<!-- 変換前（執筆者が書く） -->
![囚人のジレンマの図解](./images/prisoner-dilemma.png)

<!-- 変換後（システムが自動変換） -->
![囚人のジレンマの図解](https://assets.isidorica.org/images/prisoner-dilemma.webp)
```

#### ディレクトリ構造例

```
articles/
├── prisoner-dilemma.md
└── images/
    ├── prisoner-dilemma.png
    └── payoff-matrix.png
```

### 3.3 タイプB: Webエディタ使用

#### 操作方法

1. Webエディタで記事を編集
2. 画像をドラッグ＆ドロップ（またはクリップボードから貼り付け）
3. 自動的にMarkdownに画像が挿入される
4. 記事を保存

#### システムが自動処理すること

- 画像を即座に `isidorica-originals` にアップロード
- 自動最適化
- `isidorica-assets` にアップロード
- Markdownに画像URLを自動挿入
- 保存時にGitHub PRを自動作成

---

## 4. 画像最適化

### 4.1 最適化設定

| 項目 | 設定 | 理由 |
| :--- | :--- | :--- |
| **フォーマット** | WebP（主）、AVIF（対応ブラウザ向け） | 高圧縮率、広いブラウザサポート |
| **最大幅** | 1200px | 記事幅に合わせて（Retinaディスプレイ考慮） |
| **品質** | 80% | 視覚的に劣化なし、ファイルサイズ最適化 |

### 4.2 最適化ツール

| ツール | 用途 |
| :--- | :--- |
| **Sharp** | Node.js画像処理（リサイズ、フォーマット変換） |
| **Squoosh** | 代替（GUI/CLI） |

### 4.3 レスポンシブ画像（将来対応）

将来的には複数サイズを生成し、`srcset` で配信することを検討。

```html
<img
  src="https://assets.isidorica.org/images/xxx-800w.webp"
  srcset="
    https://assets.isidorica.org/images/xxx-400w.webp 400w,
    https://assets.isidorica.org/images/xxx-800w.webp 800w,
    https://assets.isidorica.org/images/xxx-1200w.webp 1200w
  "
  sizes="(max-width: 600px) 400px, 800px"
  alt="..."
/>
```

---

## 5. ストレージ見積もり

### 5.1 容量計算

| 記事数 | 画像数（5枚/記事） | 最適化後サイズ（100KB/枚） | 合計 |
| :--- | :--- | :--- | :--- |
| 1,000 | 5,000 | 100KB | **500MB** |
| 5,000 | 25,000 | 100KB | **2.5GB** |
| 20,000 | 100,000 | 100KB | **10GB** |

### 5.2 コスト

| 項目 | 内容 |
| :--- | :--- |
| **R2無料枠** | 10GB/月のストレージ |
| **超過時** | $0.015/GB/月 |
| **エグレス** | 無料（Cloudflare CDN経由） |

---

## 6. CI/CD 実装

### 6.1 GitHub Actions ワークフロー

```yaml
# .github/workflows/image-upload.yml
name: Image Upload

on:
  pull_request:
    paths:
      - 'articles/**/images/**'

jobs:
  upload:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Find new images
        id: images
        run: |
          # PRで追加された画像ファイルを検出
          
      - name: Optimize images
        run: |
          # Sharp等で最適化
          
      - name: Upload to R2
        run: |
          # wrangler r2 object put で元画像と最適化画像をアップロード
          
      - name: Update Markdown paths
        run: |
          # Markdown内のパスをR2 URLに変換
```

### 6.2 使用ツール

| ツール | 用途 |
| :--- | :--- |
| **wrangler** | Cloudflare R2へのアップロード |
| **Sharp** | 画像最適化 |

---

## 7. 管理者向け操作

### 7.1 手動アップロード（MVP期間）

```bash
# wrangler CLIでアップロード
wrangler r2 object put isidorica-originals/path/to/image.png --file ./image.png
```

### 7.2 画像の削除

画像を削除する場合：

1. 使用中の記事がないことを確認
2. `isidorica-assets` から削除
3. `isidorica-originals` から削除（バックアップ不要な場合）

---

## 8. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [ARCHITECTURE_DECISION_RECORD.md](./technical/ARCHITECTURE_DECISION_RECORD.md) | 技術選定（セクション2.17） |
| [NON_FUNCTIONAL_REQUIREMENTS.md](./NON_FUNCTIONAL_REQUIREMENTS.md) | スケーラビリティ要件（セクション8.1） |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | 開発者向けガイド |
