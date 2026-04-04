# isidorica-engine

Isidoricaのアプリケーション基盤（フロントエンド、ビルドシステム、UIコンポーネント）。

> **関連リポジトリ**: コンテンツ（記事、画像）は [isidorica-library](https://github.com/isidorica/isidorica-library) で管理。

---

## クイックスタート

```bash
# クローン
git clone https://github.com/isidorica/isidorica-engine.git
cd isidorica-engine

# 依存関係インストール
pnpm install

# コンテンツ取得（開発用）
git clone https://github.com/isidorica/isidorica-library.git content

# 開発サーバー起動
pnpm run dev
```

---

## ディレクトリ構造

```bash
isidorica-engine/
├── src/
│   ├── ui/                 # 汎用UIコンポーネント (Stateless, Dumb)
│   │   ├── button/
│   │   ├── dialog/
│   │   └── card/
│   │
│   ├── features/           # 機能モジュール (Stateful, Domain Logic)
│   │   ├── laboratory/     # 実験室（WASMランタイム）
│   │   │   ├── components/ # Lit Components (Lion Web Components base)
│   │   │   ├── workers/    # Web Workers (WASM execution)
│   │   │   └── signals/    # Standard Signals + Controllers
│   │   ├── search/         # 検索機能 (Pagefind, client-side)
│   │   ├── agora/          # 議論プラットフォーム (GitHub Discussions API)
│   │   └── editor/         # 管理画面機能 (ProseMirror + Lit)
│   │
│   ├── layouts/            # ページレイアウト (Astro)
│   ├── pages/              # ルーティング定義 (Astro)
│   │
│   ├── lib/                # 汎用ユーティリティ
│   │   ├── markdown/       # Velite設定 + Unifiedプラグイン
│   │   │   ├── velite.config.ts
│   │   │   ├── schemas/    # Zodスキーマ（article, learning-path等）
│   │   │   └── plugins/    # remark/rehypeプラグイン
│   │   ├── signals/        # Standard Signals Polyfill + Adapters
│   │   └── storage/        # OPFS / IndexedDB wrapper (Persistent Signal)
│   │
│   ├── i18n/               # 国際化
│   │   └── messages/       # TypeScript Module (ja.ts, en.ts)
│   │
│   ├── styles/
│   │   ├── tokens.css      # Open Props & Custom Tokens
│   │   └── global.css
│   │
│   └── __tests__/          # 統合テスト（複数モジュールにまたがる）
│       └── integration/
│
├── tests/                  # E2Eテスト・テストデータ
│   ├── e2e/                # Playwright E2Eテスト
│   └── fixtures/           # テスト用フィクスチャ
│       ├── articles/
│       ├── topics/
│       └── learning-paths/
│
├── config/                 # 設定ファイル（Engine側管理）
│   ├── concept-relations.yml   # 概念間関係（不変、Engine側）
│   ├── learning-paths.yml      # 学習パス定義
│   └── disciplines.yml         # 学問分野定義
│
├── content/                # CI/CD で isidorica-library をクローンして配置
│   │                       # ⚠️ .gitignore対象（このリポジトリには含まれない）
│   ├── articles/
│   │   ├── ja/
│   │   └── en/
│   └── data/
│       ├── terms.yml       # 用語集
│       └── bibliography.json
│
├── public/                 # 静的アセット
│
├── .github/
│   └── workflows/
│       └── deploy.yml      # CI/CD設定
│
├── astro.config.mjs
├── velite.config.ts
├── package.json
└── pnpm-lock.yaml
```

> **テストファイルの配置規則**:
> - **Unit Test**: ソースと同じディレクトリにコロケーション（例: `slug.ts` → `slug.test.ts`）
> - **Component Stories**: 同様にコロケーション（例: `button.ts` → `button.stories.ts`）
> - **Integration/E2E**: `src/__tests__/` または `tests/` に分離

---

## 依存ルール

```
ui/       ← features/ ← pages/
  ↓
lib/（どこからでも参照可能）
```

- `ui/` は他のレイヤーに依存してはならない（Dumb Components）。
- `features/` は `ui/` と `lib/` に依存できる。
- `pages/` は全てのレイヤーを使用できる。

---

## 技術スタック

| カテゴリ | 技術 |
| :--- | :--- |
| **Framework** | Astro (SSG) |
| **UI Components** | Lit + Lion Web Components |
| **State Management** | Standard Signals Polyfill |
| **Content Processing** | Velite + Unified |
| **Styling** | Open Props + Vanilla CSS |
| **Search** | Pagefind (client-side) |

> **詳細**: [ARCHITECTURE_DECISION_RECORD.md](./docs/PRD/engine/technical/ARCHITECTURE_DECISION_RECORD.md) を参照。

---

## 関連ドキュメント

- [SYSTEM_ARCHITECTURE.md](./docs/PRD/engine/technical/SYSTEM_ARCHITECTURE.md) - システムアーキテクチャ
- [ARCHITECTURE_DECISION_RECORD.md](./docs/PRD/engine/technical/ARCHITECTURE_DECISION_RECORD.md) - 技術選定書
- [CONTRIBUTING.md](./CONTRIBUTING.md) - コントリビューションガイド
