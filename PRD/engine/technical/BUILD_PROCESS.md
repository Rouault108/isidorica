# ビルドプロセス仕様 (Build Process Specification)

本ドキュメントでは、Isidorica のビルドプロセスの全体像と実装詳細を定義する。

**ステータス**: フェーズ4実装予定
**最終更新**: 2025-12-22

---

## 1. 概要

Isidorica は **SSG（静的サイト生成）** アーキテクチャを採用している。ビルド時に全記事を静的HTMLに変換し、Cloudflare Pages経由で配信する。

### 1.1 ビルドの責務

| 責務 | 説明 |
| :--- | :--- |
| **コンテンツ変換** | Markdown → HTML への変換 |
| **メタデータ生成** | hreflang、Canonical、OGP等のSEOタグ生成 |
| **アセット最適化** | CSS/JS のミニファイ、画像最適化 |
| **検索インデックス** | Pagefind / Algolia 用インデックス生成 |
| **静的ファイル出力** | `dist/` ディレクトリへの出力 |

### 1.2 ビルドの入出力

```
┌─────────────────────────────────────────────────────────────┐
│                         入力                                 │
├─────────────────────────────────────────────────────────────┤
│  isidorica-library/                                         │
│  ├── articles/{lang}/{discipline}/{slug}.md                 │
│  ├── sources/bibliography.json                              │
│  └── config/                                                │
│      ├── topics.yml                                         │
│      ├── learning-paths.yml                                 │
│      ├── concept-relations.yml                              │
│      └── permalinks.json                                    │
│                                                             │
│  isidorica-engine/                                          │
│  ├── src/                                                   │
│  │   ├── components/                                        │
│  │   ├── layouts/                                           │
│  │   ├── pages/                                             │
│  │   └── lib/                                               │
│  └── astro.config.mjs                                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      ビルドプロセス                          │
│                      (pnpm build)                           │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                         出力                                 │
├─────────────────────────────────────────────────────────────┤
│  dist/                                                      │
│  ├── ja/                                                    │
│  │   ├── game-theory/index.html                             │
│  │   └── paths/index.html                                   │
│  ├── en/                                                    │
│  │   └── game-theory/index.html                             │
│  ├── _astro/                                                │
│  │   ├── [hash].css                                         │
│  │   └── [hash].js                                          │
│  └── _headers                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. ビルドパイプライン

ビルドは以下の段階で実行される。

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: 事前処理 (Pre-processing)                         │
│  ├─ Libraryリポジトリのクローン                              │
│  ├─ 設定ファイルの読み込み                                   │
│  └─ インデックス構築（Slug、Topics、hreflang用）             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 2: コンテンツ処理 (Content Processing)               │
│  ├─ Frontmatter解析                                         │
│  ├─ Markdown → AST 変換（IFMパーサー）                       │
│  ├─ Transformer処理（略語展開、ルビ付与、参照解決等）         │
│  └─ AST → HTML 変換（Renderer）                             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 3: ページ生成 (Page Generation)                      │
│  ├─ レイアウト適用                                           │
│  ├─ SEOタグ生成（hreflang、Canonical、OGP）                  │
│  ├─ コンポーネントレンダリング（Lit Web Components）         │
│  └─ 静的HTML出力                                             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Phase 4: 後処理 (Post-processing)                          │
│  ├─ CSS/JS ミニファイ・バンドル                              │
│  ├─ 検索インデックス生成                                     │
│  ├─ OGP画像生成（Satori + Resvg）                            │
│  └─ ハッシュ付きファイル名生成（キャッシュバスティング）      │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 環境変数 (Environment Variables)

ビルド時に参照される環境変数一覧。

| 変数名 | 説明 | 例 |
| :--- | :--- | :--- |
| `ISIDORICA_ENV` | ビルド環境 (`production`, `preview`, `development`) | `production` |
| `SITE_URL` | サイトのルートURL（Canonical生成用） | `https://isidorica.org` |
| `R2_ASSETS_HOST` | 画像配信ドメイン。`production` 時に使用 | `assets.isidorica.org` |

---

## 4. Phase 1: 事前処理

### 4.1 Libraryリポジトリのクローン

| 項目 | 内容 |
| :--- | :--- |
| **トリガー** | `isidorica-engine` のビルド開始時 |
| **処理** | `isidorica-library` を `content/` ディレクトリにクローン |
| **参照** | [CI_CD_PIPELINE.md](./CI_CD_PIPELINE.md) セクション3.1 |

### 4.2 設定ファイルの読み込み

以下の設定ファイルをメモリに読み込む。

| ファイル | 用途 |
| :--- | :--- |
| `config/topics.yml` | Topics のマスター定義、alias |
| `config/learning-paths.yml` | 学習パスの定義 |
| `config/concept-relations.yml` | 概念間の関係 |
| `config/permalinks.json` | パーマリンクハッシュ履歴 |
| `sources/bibliography.json` | 出典データ |

### 4.3 アセットの準備

Laboratoryや画像の静的アセットを準備する。

| アセット | 処理 |
| :--- | :--- |
| **Laboratory WASM** | Pyodide, QuickJS 等のWASMバイナリを `public/assets/wasm/` にコピー |
| **ローカル画像** | `preview` / `development` 時のみ、`articles/**/images/` を `public/images/` にコピー |

> **Note（フォント）**: フォントサブセット（`pyftsubset` で生成）は**ビルド毎に生成しない**。開発時に事前生成し、リポジトリにコミットしておく。詳細は [CONTRIBUTING.md](../CONTRIBUTING.md) セクション9を参照。

### 4.4 インデックス構築

ビルド効率を確保するため、**事前にインデックスを構築**する。

#### Slugインデックス

同一Slugを持つ言語版を高速に取得するためのインデックス。

```typescript
// O(n) で構築、各記事ビルド時は O(1) で参照
const slugIndex = new Map<string, { lang: string; path: string }[]>();

for (const article of allArticles) {
  const { slug, lang } = article;
  if (!slugIndex.has(slug)) {
    slugIndex.set(slug, []);
  }
  slugIndex.get(slug).push({ lang, path: article.path });
}
```

**用途**:
- hreflang タグの生成（同一Slugの他言語版を取得）
- 翻訳版の存在確認

#### Topicsインデックス

```typescript
const topicsIndex = new Map<string, string[]>(); // topic-id → [slug1, slug2, ...]
```

**用途**:
- Related Articles の計算
- 未使用トピックの検出

> **参照**: [URL_AND_TAXONOMY_DESIGN.md](../../library/technical/URL_AND_TAXONOMY_DESIGN.md) セクション1.7.1

---

## 5. Phase 2: コンテンツ処理

### 5.1 Markdownパイプライン

| ステップ | 処理 | ツール |
| :--- | :--- | :--- |
| **1. Frontmatter解析** | YAMLメタデータの抽出・検証 | Zod |
| **2. Lexer** | Markdown → トークン列 | IFMパーサー（独自実装） |
| **3. Parser** | トークン列 → AST | IFMパーサー |
| **4. Transformer** | AST変換（下記参照） | IFMパーサー |
| **5. Renderer** | AST → HTML | IFMパーサー |

> **参照**: [IFM_PARSER_SPEC.md](./IFM_PARSER_SPEC.md) セクション2

### 5.2 Transformer処理

AST変換で実行される処理一覧。

| 処理 | 説明 | 参照 |
| :--- | :--- | :--- |
| **略語展開** | `topics` に基づき略語を `<abbr>` に変換 | IFM_PARSER_SPEC セクション4.6 |
| **自動ルビ付与** | 形態素解析（Sudachi）で漢字にルビ | IFM_PARSER_SPEC セクション5 |
| **見出しID生成** | 各見出しに一意のIDを自動付与 | IFM_PARSER_SPEC セクション4.3 |
| **リンク分類** | 内部/外部リンクの自動判定、属性付与 | IFM_PARSER_SPEC セクション4.1 |
| **数式レンダリング** | KaTeX互換エンジンでHTML生成 | IFM_PARSER_SPEC セクション7 |
| **出典参照解決** | `[:source-id]` を bibliography.json から解決 | MARKDOWN_SPEC セクション11 |
| **参照番号採番** | 図・表・数式の番号を自動採番 | IFM_PARSER_SPEC セクション3.6.4 |
| **画像パス解決** | 環境に応じて画像パスを変換（下記参照） | IMAGE_MANAGEMENT セクション3.2 |
| **日付・数値フォーマット** | `Intl.DateTimeFormat` / `Intl.NumberFormat` で言語別フォーマット生成 | ADR セクション2.9 |

#### 画像パス解決ロジック

`ISIDORICA_ENV` に応じて画像パスを書き換える。

| 環境 | 変換ルール | 例 |
| :--- | :--- | :--- |
| `production` | R2 URLに置換 | `./images/foo.png` → `https://assets.isidorica.org/images/foo.webp` |
| `preview` / `dev` | ローカルパスに置換 | `./images/foo.png` → `/images/foo.png` |

> **Note**: `production` 時は、事前に画像が最適化されR2にアップロード済みであることを前提とする（Post-merge処理）。 `preview` 時は Phase 1 でコピーしたローカル画像を参照する。

---

## 6. Phase 3: ページ生成

### 6.1 SEOタグ生成

| タグ | 生成方法 | 参照 |
| :--- | :--- | :--- |
| **hreflang** | Slugインデックスから同一Slugの言語版を取得し生成 | URL_AND_TAXONOMY_DESIGN セクション1.7.1 |
| **Canonical** | 通常ページは自己参照、アーカイブは最新版を参照 | URL_AND_TAXONOMY_DESIGN セクション1.7.2 |
| **OGP (Open Graph)** | Frontmatter + Satori で動的生成 | ADR セクション2.7 |
| **robots** | アーカイブページは `noindex` | URL_AND_TAXONOMY_DESIGN セクション1.7.3 |

### 6.2 コンポーネントレンダリング

| コンポーネント | レンダリング方式 |
| :--- | :--- |
| **静的コンポーネント** | ビルド時にHTMLとして出力 |
| **アイランドコンポーネント** | `client:visible` 等でクライアントサイドハイドレーション |
| **Laboratory** | 静的プレビュー + クライアントサイドWASM |

> **参照**: [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md) セクション2

---

## 7. Phase 4: 後処理

### 7.1 アセット最適化

| 最適化 | 説明 | ツール |
| :--- | :--- | :--- |
| **CSS ミニファイ** | 未使用CSS削除、圧縮 | Astro（Vite） |
| **JS ミニファイ** | Tree Shaking、バンドル | Astro（Vite/Rollup） |
| **ハッシュ付きファイル名** | `[hash].css` 形式で永久キャッシュ対応 | Astro |

### 7.2 検索インデックス生成

| 方式 | 説明 |
| :--- | :--- |
| **Pagefind** | ビルド後に静的インデックス生成 |
| **Algolia** | ビルド後にAPI経由でインデックス同期 |

> **参照**: プロジェクトの検索エンジン選定に応じて切り替え

### 7.3 OGP画像生成

| 項目 | 内容 |
| :--- | :--- |
| **ツール** | Satori + Resvg |
| **速度** | 約50ms/枚 |
| **対応** | 日本語フォント対応 |

> **参照**: [ARCHITECTURE_DECISION_RECORD.md](./ARCHITECTURE_DECISION_RECORD.md) セクション2.7

### 7.4 HTTPヘッダー設定ファイル生成

Cloudflare Pages用の `_headers` ファイルを生成する。

| 設定 | 内容 |
| :--- | :--- |
| **CSP (Content-Security-Policy)** | スクリプト・スタイルのソース制限 |
| **キャッシュ制御** | ハッシュ付きアセットは `immutable`、HTMLは `no-cache` |
| **Permissions-Policy** | 不要なブラウザ機能の無効化 |

> **参照**: [NON_FUNCTIONAL_REQUIREMENTS.md](../NON_FUNCTIONAL_REQUIREMENTS.md) セクション2（セキュリティ）

> **Note（ビルド後検証）**: Lighthouse CIによるパフォーマンス・アクセシビリティ検証はビルド後にCIが実行する。詳細は [CI_CD_PIPELINE.md](./CI_CD_PIPELINE.md) セクション2.3を参照。

### 7.5 将来機能（未実装）

| 機能 | 説明 | 参照 |
| :--- | :--- | :--- |
| **PDF/EPUBエクスポート** | 記事をオフライン閲覧用にビルド時に自動生成（Pandoc等） | ADR セクション2.5 |
| **RSS/Atomフィード** | ディスカッションのフィード配信 | DISCUSSIONS_DESIGN セクション5 |

---

## 8. パフォーマンス最適化

### 8.1 ビルド時間の目標

| 状態 | ビルド時間 | 記事数目安 | 対応 |
| :--- | :--- | :--- | :--- |
| **正常** | < 3分 | 〜1,000記事 | 対応不要 |
| **注意** | 3〜10分 | 1,000〜5,000記事 | 監視継続、最適化検討 |
| **警告** | 10〜30分 | 5,000〜20,000記事 | 増分ビルド導入を検討 |
| **危険** | > 30分 | 20,000記事以上 | アーキテクチャ転換を検討 |

> **参照**: [NON_FUNCTIONAL_REQUIREMENTS.md](../NON_FUNCTIONAL_REQUIREMENTS.md) セクション8.2

### 8.2 最適化手法

| 手法 | 効果 | 実装コスト |
| :--- | :--- | :--- |
| **事前インデックス構築** | hreflang生成を O(n²) → O(n) に削減 | 低 |
| **画像の外部化（R2）** | ビルド時の画像処理を削減 | 低 |
| **キャッシュ活用** | 変更のない記事の再処理をスキップ | 中 |
| **並列処理** | 記事の並列ビルド | 中 |
| **増分ビルド** | 変更記事のみ再ビルド（Astro 5.0 対応待ち） | 高 |

### 8.3 実装上の注意

| 処理 | 悪い実装 | 良い実装 |
| :--- | :--- | :--- |
| **hreflang生成** | 各記事で全記事を検索 O(n²) | 事前にSlugインデックス構築 O(n) |
| **Related Articles** | 各記事で全記事と比較 O(n²) | 事前にTopicsインデックス構築 O(n) |
| **出典参照解決** | 各参照で bibliography.json を読み込み | 起動時に1回読み込み、メモリに保持 |

---

## 9. エラーハンドリング

### 9.1 ビルド時エラー

| エラータイプ | 原因 | 対応 |
| :--- | :--- | :--- |
| **Frontmatter検証エラー** | 必須フィールド欠落、型不一致 | ビルド失敗、エラーメッセージ表示 |
| **数式パースエラー** | 不正なLaTeX構文 | ビルド失敗、行番号付きエラー表示 |
| **出典参照エラー** | 存在しない出典ID | ビルド失敗、該当記事を表示 |
| **画像参照エラー** | 存在しない画像パス | 警告（ビルドは続行）、プレースホルダー表示 |

### 9.2 エラー対応フロー

```
ビルドエラー発生
    ↓
GitHub Actions ログにエラー出力
    ↓
PR の場合 → PRにコメントでエラー通知
main の場合 → デプロイ停止、前バージョンを維持
    ↓
開発者が修正コミット
```

---

## 10. ローカル開発

### 10.1 開発サーバー

```bash
pnpm dev
```

| 項目 | 内容 |
| :--- | :--- |
| **HMR** | ファイル変更時に自動リロード |
| **ポート** | `localhost:4321`（デフォルト） |
| **Library参照** | ローカルの `content/` または symlink |

### 10.2 本番ビルドのテスト

```bash
pnpm build
pnpm preview
```

---

## 11. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [CI_CD_PIPELINE.md](./CI_CD_PIPELINE.md) | CIでのビルド・デプロイフロー |
| [IFM_PARSER_SPEC.md](./IFM_PARSER_SPEC.md) | Markdownパーサーの詳細仕様 |
| [SYSTEM_ARCHITECTURE.md](./SYSTEM_ARCHITECTURE.md) | システム全体アーキテクチャ |
| [NON_FUNCTIONAL_REQUIREMENTS.md](../NON_FUNCTIONAL_REQUIREMENTS.md) | ビルド時間の閾値、スケーラビリティ |
| [URL_AND_TAXONOMY_DESIGN.md](../../library/technical/URL_AND_TAXONOMY_DESIGN.md) | hreflang、Canonical の仕様 |
| [ARCHITECTURE_DECISION_RECORD.md](./ARCHITECTURE_DECISION_RECORD.md) | 技術選定の根拠 |
| [IMAGE_MANAGEMENT.md](../IMAGE_MANAGEMENT.md) | 画像管理ワークフロー、パス解決 |
| [LABORATORY_DESIGN.md](../LABORATORY_DESIGN.md) | Laboratory機能の設計、WASMアセット |
