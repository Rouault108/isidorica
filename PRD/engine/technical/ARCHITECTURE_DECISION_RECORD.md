# 技術選定書 (Architecture Decision Record)

**最終更新**: 2026-01-08
**バージョン**: 1.3.0
**定期レビュー**: 6ヶ月ごと（次回: 2026年7月）

---

## ADR番号インデックス

| ADR | 領域 | ステータス | 導入Phase |
|:---|:---|:---|:---|
| ADR-001 | Frontend Architecture | ✅ Accepted | Phase 4 |
| ADR-002 | Styling Strategy | ✅ Accepted | Phase 4 |
| ADR-003 | Search Architecture | ✅ Accepted | Phase 5 |
| ADR-004 | State Management | ✅ Accepted | Phase 4 |
| ADR-005 | Authentication & Persistence | ✅ Accepted | Phase 4 (Beta: None) |
| ADR-006 | Content Data Structure | ✅ Accepted | Phase 4 |
| ADR-007 | Quality Assurance | ✅ Accepted | Phase 4 |
| ADR-008 | Infrastructure | ✅ Accepted | Phase 4 |
| ADR-009 | Development Environment | ✅ Accepted | Phase 4 |
| ADR-010 | Monitoring & Analytics | ✅ Accepted | Phase 5 |
| ADR-011 | Cache Strategy | ✅ Accepted | Phase 4 |
| ADR-012 | PWA & Offline | ✅ Accepted | Phase 5〜 |
| ADR-013 | Image & Media Strategy | ✅ Accepted | Phase 4 |
| ADR-014 | Font Strategy | ✅ Accepted | Phase 4 |
| ADR-015 | Internationalization | ✅ Accepted | Phase 5〜 |
| ADR-016 | Error Pages | ✅ Accepted | Phase 5 |
| ADR-017 | Image Management | ✅ Accepted | Phase 4 |
| ADR-018 | Laboratory Runtime | ✅ Accepted | Phase 5〜 |
| ADR-019 | Discussion Platform | ✅ Accepted | Phase 5〜 |
| ADR-020 | Knowledge State Tracking | ✅ Accepted | Phase 5〜 |
| ADR-021 | Security | ✅ Accepted | Phase 4 |
| ADR-022 | Backup & Disaster Recovery | ✅ Accepted | Phase 5 |

> **ステータス凡例**: ✅ Accepted（採用）, 🔄 Proposed（提案中）, ❌ Deprecated（非推奨）, 🔀 Superseded（置換済み）

---

## 1. コンテキストと基本方針

Isidoricaは「100年続くインターネット時代の公共図書館」を目指す。
技術選定においては、以下の価値観を絶対的な基準とする。

1.  **Sustainability (持続可能性)**: 特定のフレームワークやベンダーに依存せず、Web標準技術（Web Components, Standard CSS, CommonMark）をベースにする。
2.  **Universal Access (公共性)**: あらゆるブラウザ、デバイス、能力（アクセシビリティ）を持つユーザーに対し、情報への到達を保証する。Firefox非互換などの排除は許容しない。
3.  **Performance (速度)**: 「知」へのアクセスにおける摩擦（ロード時間）をゼロにする。
4.  **Authoring Experience (編集体験)**: コンテンツ品質を高めるための執筆環境とフィードバックループを整備する。
5.  **Educational Effectiveness (教育効果)**: 学習理論に基づく機能（出典明示、メタ学習統合、知識状態追跡）を技術的に支援する。詳細は [INTEGRATED_DESIGN_GUIDELINES.md](../../shared/INTEGRATED_DESIGN_GUIDELINES.md) を参照。

---

## 2. 採用技術スタック (Decisions)

### 2.1 Frontend Architecture [ADR-001]
*   **Core**: **Astro (SSG)**
    *   **理由**: アイランドアーキテクチャによるJS削減。静的コンテンツ（文章）と動的コンポーネント（実験室）の共存に最適。
*   **Component**: **Lit (Web Components)**
    *   **理由**: ブラウザ標準技術。フレームワークの栄枯盛衰に左右されない永続的なコンポーネント資産。Shadow DOMによるスタイルカプセル化。
*   **Logic / A11y**: **Lion Web Components (`@lion/ui`)**
    *   **理由**: ING銀行が開発するホワイトレーベルWeb Components群。Litベースでネイティブに構築されており、アダプター不要で統合可能。金融機関レベルの厳格なアクセシビリティ（WAI-ARIA）とフォーム堅牢性を備える。
    *   **デザイン戦略**: Lionが提供するのは機能（Logic）とDOM構造のみであり、スタイルはほぼ無適用（White-label）。Isidorica独自のプレミアムデザインを自由に適用可能。 Zag.jsと同等のHeadless性を持ちつつ、Litとの親和性は遥かに高い。
    *   **リスク**: 高機能ゆえにクラス構造が深く、学習曲線が急峻。ドキュメントが実用的すぎて初学者に優しくない。
    *   **PoC計画**: Phase 4で最も複雑なコンポーネント（Combobox等）の実装検証を行い、開発者体験（DX）を確認する。
*   **Editor**: **Lit + ProseMirror**
    *   **理由**: Reactエコシステムへの依存（Tiptap等）を排除し、プロジェクト全体の技術スタックをWeb Componentsに統一するため。「100年続く」プラットフォームとして、フレームワークの寿命に左右されない、Framework-agnosticなコアエンジン（ProseMirror）をLitでラップして実装する。
    *   **コスト**: Tiptap等のラッパーが提供している便利機能（メニューバー等）を自作する必要があるため、初期実装コストは高い。
    *   **メリット**: 完全な制御権、Reactランタイムの排除、パフォーマンス向上、そしてLion Web Componentsを用いたアクセシビリティの統一が可能になる。

### 2.2 Styling Strategy [ADR-002]
*   **Engine**: **Vanilla CSS + CSS Variables**
    *   **理由**: 標準CSS機能（Nesting等）の活用。Shadow DOMを貫通できるCSS変数を基盤とする。
*   **Tokens**: **Open Props**
    *   **理由**: ゼロからデザインシステムを作るコストを削減し、統一されたデザイントークンを提供。

### 2.3 Search Architecture [ADR-003]
*   **Primary Engine**: **Pagefind**
    *   **理由**: 静的サイトに最適化されたRust製の検索エンジン。インデックスをチャンク分割してロードする仕組みにより、大規模サイトでも検索時の通信量を数百KBに抑えられる。
    *   **ホスティング**: Cloudflare Pages (Static Assets) として配信するため、サーバーコストはゼロであり、持続可能性が極めて高い。
*   **Query Tokenizer**: **Hybrid Strategy (Intl.Segmenter + TinySegmenter)**
    *   **方針**: 「機会の最大化」と「誰も置き去りにしない公共性」の両立。
    *   **Modern Browsers (Primary)**: **`Intl.Segmenter`**
        *   **理由**: ブラウザ標準の高精度な分かち書きを利用し、最高の検索体験を提供する。追加のJSロードはゼロ。
    *   **Legacy Browsers (Fallback)**: **`TinySegmenter`** (Dynamic Import)
        *   **理由**: `Intl.Segmenter` 非対応環境（古いOS/ブラウザ）でも検索機能を動作させるための安全装置。約25KBと軽量で、低帯域環境でも負担にならない。
        *   **精度補正**: ニュース記事で約95%の精度を持つが、辞書を持たないため最新語彙に弱い。これを補うため、検索UIに「自動分割されたクエリ」を表示し、ユーザーが手動でスペース修正できる透明性（Agency）を提供する。
    *   **共通**: クライアントサイドで完結するため、検索クエリはサーバーに送信されず、プライバシーは完全に保護される。

### 2.4 State Management [ADR-004]
*   **Synchronous**: **Unified Reactive Architecture (Standard Signals + Controllers)**
    *   **Core Data**: **Standard Signals Polyfill (`signal-polyfill`)**
        *   **役割**: 純粋なデータ（State）と計算グラフ（Computed）の管理。TC39標準仕様に準拠。
        *   **哲学**: 知識の依存関係（Aが変わればBが変わる）を表現する「Reactive Graph」モデルは、Isidoricaの概念モデルと一致する。
    *   **Binding**: **Reactive Controller & SignalWatcher**
        *   **役割**: ロジックの再利用とライフサイクル管理。Signalを内包し、コンポーネントへの接続（Binding）を担当する「Signals-backed Controller」パターンを標準とする。特定のフレームワーク（Preact実装など）には依存しない。
*   **Asynchronous**: **Typed Event Messaging (Custom Protocol)**
    *   **理由**: Comlink (RPCモデル) はSignalsアーキテクチャと整合しない一方、素の `postMessage` は型安全性やエラー処理の欠如というリスクがあるため、両者の欠点を補う厳密なプロトコルを定義する。
    *   **実装戦略**:
        *   **Schema**: Discriminated Union型による厳密なメッセージ定義をMain/Workerで共有し、完全な型安全性を確保する。
        *   **Pattern**: 基本はPush型（Signalへの流し込み）だが、必要に応じてRequest-Response型（Promise, Correlation ID）も扱えるハイブリッドなアダプタを実装する。
        *   **Safety**: エラー境界（Error Boundary）を設け、Worker内の例外を型付きのエラーイベントとしてメインスレッドに伝播させる仕組みを構築する。
*   **アーキテクチャ図**:
    ```
    ┌─────────────────────────────────────────────────────────────┐
    │                     　　　　　　　　　　　 Main Thread                         　　　　　　　　　　　 │
    ├─────────────────────────────────────────────────────────────┤
    │  ┌─────────────┐    　┌─────────────────┐    　　┌─────────────┐  　│
    │  │   　  Signal  　　　│────│ 　　　SignalWatcher   　　　│────│ 　　　Lit Element 　　│  　│
    │  │ 　　  (State)   　　│   　 │ 　　　(Controller)    　　　│    　　│　　　  (render)   　│    │
    │  └──────┬──────┘   　 └─────────────────┘    　　└─────────────┘    │
    │             │                                                   　　　　　　　　　　　　　　　　　 │
    │             │ Typed Event                                                                 │
    │             │ Messaging                                                                   │
    │             ▼                                                                             │
    └─────────────────────────────────────────────────────────────┘
                                                │
                                       postMessage (Typed)
                                                │
    ┌────────────────────────────▼────────────────────────────────┐
    │                                       Web Worker                                          │
    ├─────────────────────────────────────────────────────────────┤
    │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐              │
    │  │       Pyodide      │    │       QuickJS     │    │        webR        │             │
    │  │       (WASM)      │    │        (WASM)      │    │       (WASM)       │             │
    │  └─────────────┘    └─────────────┘    └─────────────┘              │
    └─────────────────────────────────────────────────────────────┘
    ```

### 2.5 Authentication & Persistence [ADR-005]
*   **User Settings (Anonymous)**: **localStorage (Local-First)**
    *   **理由**: アカウントなしでも設定（テーマ、言語）を保持するため。プライバシー優先。
*   **Authentication (Expert/Mod)**: **Cloudflare Access (Zero Trust)**
    *   **理由**: 管理画面へのアクセス制御はエッジレベルで行う。
    *   **リスク**: Cloudflareベンダー集中リスクについては[R-TECH-09](../../shared/RISK_ANALYSIS.md)参照。代替: Auth0, Okta等。
*   **General Contributor**: **Isidorica Guest Contribution (Account-less)**
    *   **Phase 1-2**: GitHubアカウント連携を必須とする（開発スピード優先）。
    *   **Phase 3+**: **Bot Proxy機構**を実装し、GitHubアカウントを持たないユーザーでも参加可能にする。
        *   **理由**: GitHubの操作難易度や英語UIが、一般知識人の参入障壁になることを防ぎ、真のユニバーサルな参加を実現するため。
*   **Bot Proxy Infrastructure (Phase 3+)**:
    *   **機構**: `isidorica-bot` サービスアカウントが匿名ユーザーに代わってGitHub操作を実行。
    *   **適用範囲**: 記事PR作成、Agora投稿、Issue作成。
    *   **共通対策**:
        *   IPレート制限（同一IPからの連続投稿を制限）
        *   CAPTCHA（hCaptcha - プライバシー重視）
        *   自動コンテンツフィルタ（スパム検出、禁止語、連投制限）
    *   **記事（執筆）向け**:
        *   `untrusted` ラベルでPR作成 → **Reviewerの事前承認後にマージ**
        *   理由: 永続コンテンツであり品質が重要。量が少なく事前レビュー可能。
    *   **Agora（議論）向け**:
        *   自動フィルタ通過後は**即時公開**（事前レビューなし）
        *   **事後モデレーション**: コミュニティフラグ + モデレーター対応
        *   理由: 量が多く即時性が重要。事前レビューはスケールしない。
    *   **透明性**: 投稿には「ゲスト投稿」と明記。IPはログ保持（法的対応用）。
*   **Sync**: **None (Initial Phase)**
    *   **理由**: 個人データはローカル完結（Privacy First）する。

### 2.6 Content Data Structure [ADR-006]
*   **Data Loading**: **Velite**
    *   **理由**: Content LayerをUIフレームワーク（Astro）から分離し、**Framework Agnostic** なJSONデータとして管理するため。これにより、将来的なフレームワーク移行（Next.jsやRust製SSG等）や、CLIツール等からのデータ再利用が容易になり、**The "Architecture of 100 Years"** の哲学（特定の技術への依存回避）に合致する。
    *   **Features**: Zodスキーマによる型安全性、高速なビルド、Framework非依存のJSON出力。
*   **Syntax**: **Standard Markdown + Generic Directives (`:::`)**
    *   **理由**: GFMに依存せず、Pandoc/CommonMark拡張案に準拠することで将来的なポータビリティを確保。
    *   **Format**: 注釈や埋め込みコンポーネントは `:::` 記法で統一。
*   **Processing**: **Unified (remark / rehype) + Custom Plugins**
    *   **理由**: 既存の汎用プラグインに頼らず、Isidorica専用のAST変換・検証プラグインを開発し、厳密なデータ構造を担保する。

### 2.7 Quality Assurance (QA) [ADR-007]
*   **Language**: **TypeScript** (Strict)
*   **Testing Pyramid**:
    *   **Unit**: **Vitest** (Logic, Transformer)
    *   **Property-Based**: **fast-check** (IFMパーサー、Transformer堅牢性)
        *   **理由**: 開発者が想定しないエッジケースを自動検出。複雑なDirective構文への耐性を検証。
    *   **Component**: **Storybook Interaction Testing** (UI, Events) - Testing Library代替
    *   **A11y (Accessibility)**: **Storybook + `@storybook/addon-a11y` (axe-core)** + **eslint-plugin-lit-a11y**
        *   **検証対象**: WCAG 2.2 AA準拠。キーボード操作、フォーカス管理、ARIAラベル、コントラスト比、スクリーンリーダー互換性。
        *   **理由**: 実ブラウザ上でWeb Componentsをレンダリングし、axe-coreによる自動検出と手動検証を組み合わせることで、Shadow DOM内部を含む包括的なアクセシビリティ検証を実現する。
    *   **E2E**: **Playwright** (Integration, Routing, Search)
*   **Security Testing**:
    *   **SCA (依存関係脆弱性)**: **pnpm audit** - high以上でPRブロック
    *   **Secrets検出**: **gitleaks** - pre-commit + CI実行
    *   **SAST (静的解析)**: **CodeQL** - PRごと + 週次スケジュール
        *   **理由**: 静的サイトでもTransformerやパーサーにはセキュリティリスクが存在。XSS、インジェクション等を検出。
        *   **DAST不採用理由**: 静的サイト（SSG）のため動的脆弱性検出の恩恵が限定的。アカウント機能導入時に再検討。
*   **Code Linting**:
    *   **Biome**: Formatter & General Linter (高速、ESLint/Prettier統合)
    *   **ESLint**: Accessibility (`eslint-plugin-lit-a11y`)
        *   **理由 (苦渋の選択)**: Biome は一般的なA11yルールを持つが、**Litのタグ付きテンプレートリテラル (`html\`...\``) を解析できない**ため、Lit固有のA11y検証にはESLintを併用せざるを得ない。BiomeがLitテンプレートを公式サポートした時点で移行を検討する。
*   **Content Quality**:
    *   **Structure**: **In-house Validation Library (Decoupled from Parser)**
        *   **方針**: `remark-lint` などの外部エコシステムには依存せず、Isidorica専用のバリデーションライブラリを内製する。これにより外部ライブラリの廃止リスクを完全に排除する。
        *   **設計**: AST解析（Parser）とはモジュールとして明確に分離し（Single Responsibility Principle）、かつ独自Directiveの検証や一般的なMarkdownルール（見出し順序など）も全て内包して管理する。
    *   **Writing (Japanese)**: **textlint** を採用。
        *   **選定理由**:
            *   200以上の既存ルールを即座に活用可能（`textlint-rule-preset-ja-technical-writing` 等）。
            *   カスタムルール追加が容易（`textlint-rule-*` として独自ルール作成可能）。
            *   VS Code拡張（`vscode-textlint`）が成熟しており、執筆者体験が良好。
            *   独自リンター作成は形態素解析統合やVS Code拡張開発のコストが大きく、エコシステム活用の方が合理的。
        *   **却下した選択肢**: 完全内製リンター（形態素解析、ルールエンジン、VS Code拡張をすべて自作）。開発・保守コストと比較してtextlint採用のメリットが上回るため却下。
        *   **ルール構成**:
            *   **表記ゆれ (Terminology)**: CIで自動修正 (`--fix`) し、強制的に統一する。
            *   **文体・構造**: Warning扱いとし、管理者がPRレビュー時に確認する運用から開始する。
            *   **Isidorica固有ルール**: ひらく語、外来語長音、読者呼びかけ禁止等をカスタムルールとして追加。
        *   **ブラウザ対応戦略 (Web執筆ツール用)**:
            *   **ハイブリッド方式**を採用。
            *   **リアルタイムチェック（ブラウザ）**: **限定的な即時フィードバック**を提供。
                *   **対象**: 禁止表現（正規表現で検出可能なもの）、明らかな問題（極端に長い段落、禁止ワード等）。
                *   **対象外**: 形態素解析を必要とするルール（文長計測、読点間隔、文体判定等）はブラウザでは実行しない。
                *   **理由**: ブラウザ側で中途半端なチェッカーを実装すると、textlintとの結果不一致がユーザー体験を損なう。
            *   **保存時チェック（サーバー）**: textlint API（Cloudflare Workers等）で全ルール適用。一貫した品質チェックを担保。
        *   **段階的導入計画**:
            *   **Phase 4**: CLI + VS Code拡張導入（ローカル執筆者向け）。CI/CDでのtextlint実行。
            *   **Phase 6以降**: textlint APIをCloudflare Workersに構築、Web執筆ツール統合。

> **詳細**: テスト戦略の全容は [TESTING_STRATEGY.md](./TESTING_STRATEGY.md) を参照。

### 2.8 Infrastructure (Hosting / CDN / Domain) [ADR-008]
*   **Hosting / CDN**: **Cloudflare Pages**
    *   **理由**:
        *   日本国内に東京・大阪のPoP（Point of Presence）を持ち、国内ユーザーへの配信が最速。「爆速の表示」という理念に合致。
        *   無料プランでもフルスペックのCDN機能が利用可能。帯域幅無制限（Fair Use Policy適用）。
        *   静的サイト（SSG）に最適化されており、Astroとの親和性が高い。
    *   **ビルド制限の回避**: 無料プランのビルド時間制限（月500分）を回避するため、**GitHub Actionsでビルドし、Cloudflare Pages Direct Uploadでデプロイ**する構成を採用。
*   **Domain Registrar**: **Cloudflare Registrar**
    *   **理由**: 原価（レジストリ手数料のみ）でドメインを取得可能。DNS管理もシームレスに統合され、運用コストを最小化できる。
    *   **ドメイン**: `isidorica.org`（予定）
*   **DNS**: **Cloudflare DNS**
    *   **理由**: Cloudflare Pagesと統合されており、設定が簡素。無料でDNSSECやDDoS保護も提供。

### 2.9 Development Environment [ADR-009]
*   **Package Manager**: **pnpm**
    *   **理由**: symlink構造による厳格な依存解決（phantom dependency防止）。高速かつディスク効率に優れる。Astro/Lit公式サポート。
    *   **必須設定** (`.npmrc`):
        ```
        shamefully-hoist=false
        strict-peer-dependencies=true
        ```
    *   **メリット**: Reactエコシステムの排除により、`shamefully-hoist=false` との互換性問題（Shadow DOM非対応パッケージ等）が激減する。Web Componentsネイティブな構成は、pnpmの厳格な依存解決と極めて相性が良い。
*   **Runtime**: **Node.js (LTS)**
    *   **理由**: 最も安定したJSランタイム。CI/CDでビルドするSSGプロジェクトであり、本番環境にランタイムは存在しない。
    *   **Bunへの移行可能性**: 将来的にBunが成熟した場合、テストが充実していれば移行は困難ではない。SSGのためランタイム変更が本番環境に直接影響しない点も移行ハードルを下げる。
*   **Version Manager**: **mise** (推奨) または `.nvmrc` + nvm
    *   **理由**: Node.js/pnpmのバージョンをプロジェクト単位で固定し、開発者間・CI環境間の差異を排除。
*   **Lockfile Policy**: `pnpm-lock.yaml` を必ずコミットする。
    *   **理由**: 再現可能なビルドを保証。CI/CDでの依存解決を高速化。
*   **依存関係更新**: **Renovate** (GitHub App)
    *   **理由**: 依存パッケージの更新PRを自動生成。セキュリティ脆弱性への迅速な対応を可能にする。
    *   **ポリシー**:
        *   **パッチ・マイナー更新（devDependencies）**: CI全テスト成功時に自動マージ。
        *   **メジャー更新（全て）**: 手動レビュー必須（破壊的変更の調査）。
        *   **dependencies（本番コード）**: 全て手動レビュー。

### 2.10 Monitoring & Analytics [ADR-010]
*   **アクセス解析**: **Cloudflare Web Analytics**
    *   **理由**: Cloudflare Pages統合済みで追加設定が最小限。Cookieを使用せずGDPR/CCPA準拠。無料。
    *   **取得データ**: PV、滞在時間、リファラー、Core Web Vitals（LCP, FID, CLS）。
    *   **プライバシー保証**: 個人識別情報は一切収集しない。IPアドレスも保存されない。ユーザー追跡なし。
    *   **制限**: 詳細なファネル分析やカスタムイベントトラッキングは不可。
    *   **代替オプション**: Cloudflare離脱時は**Plausible CE**（セルフホスト）または**Umami**への移行を検討。いずれもCookieなし、プライバシー重視。[R-TECH-09](../../shared/RISK_ANALYSIS.md)参照。
*   **検索アナリティクス**: **ゼロヒットフィードバック（コンセントベース）**
    *   **デフォルト**: Pagefindはクライアントサイド完結型であり、検索クエリはサーバーに送信されない（プライバシー保護）。
    *   **ゼロヒット時**: 検索結果が0件の場合のみ、ユーザーに「このクエリを改善のために送信しますか？」と確認し、同意された場合のみクエリを収集する。
    *   **データ内容**: クエリ文字列のみ（ユーザー識別子、IP、タイムスタンプは含まない）。
    *   **目的**: コンテンツ不足箇所の特定と検索インデックス改善。
*   **パフォーマンス監視（CI）**: **Lighthouse CI** (filesystem モード)
    *   **理由**: ビルド時にLighthouseを実行し、スコア劣化を自動検知。PRへのコメントで可視化。
    *   **プライバシー**: `--upload.target=filesystem` を使用し、Googleへのデータ送信は行わない。結果はCI環境内にのみ保存。
*   **エラー監視**: **クライアントサイドエラー報告UI（Beta期間）**
    *   **実装**: グローバルエラーハンドラを設置し、未捕捉エラー発生時にユーザーに報告ダイアログを表示。
    *   **収集データ（同意時のみ）**: エラーメッセージ、スタックトレース、ブラウザ/OS情報。ユーザー識別子はなし。
    *   **送信先**: GitHub Issue自動作成（テンプレート付き）。
    *   **理由**: 受動的な報告待ちではなく、構造化されたエラー情報を収集し、再現性を高める。SSGのためサーバーサイドエラーは発生しない。
    *   **将来的な拡張**: 正式版以降、ユーザー規模が拡大した段階でSentryを検討（要プライバシーポリシー更新）。

#### 将来的な拡張（Post-MVP）
*   **ファネル分析・イベントトラッキング**: 月間UV 10,000超過、またはLaboratory利用率分析が必要になった段階で、**Plausible**（Cloud または CE セルフホスト）の追加を検討する。セルフホスト版はCloudflare依存を軽減し、データ主権を完全に確保できる。

### 2.11 Cache Strategy [ADR-011]
*   **CDN**: **Cloudflare Pages**
    *   **理由**: 日本国内PoP（東京・大阪）による最速配信。無料プランで帯域幅無制限。詳細は[ADR-008](#28-infrastructure-hosting--cdn--domain-adr-008)参照。
    *   Cloudflareが自動的に最寄りのPoPからコンテンツを配信。開発者によるリージョン選択は不要。
    *   **代替オプション**: R-TECH-09に記載の移行候補（Vercel, Netlify, AWS CloudFront）はいずれも同等のCDN機能を提供。静的資産は可搬性が高い。
*   **Cache-Control ポリシー**:
    *   **HTML（記事ページ）**: `public, max-age=0, must-revalidate`
        *   常に最新を確認。Cloudflareエッジキャッシュは有効。
    *   **JS/CSS（ハッシュ付き）**: `public, max-age=31536000, immutable`
        *   Astro（Vite）がビルド時にファイル名にハッシュを付与。内容変更時はファイル名が変わるため、永久キャッシュでも安全。
    *   **画像/フォント（ハッシュ付き）**: `public, max-age=31536000, immutable`
        *   同上。
*   **Cloudflare固有の動作**:
    *   `max-age=0` はブラウザキャッシュを無効化するが、Cloudflareエッジでは**Edge Cache TTL**（デフォルト2時間、設定可能）でキャッシュが保持される。
    *   即時反映が必要な場合（緊急修正等）は、デプロイ時にCloudflare APIまたはDashboardで該当URLのキャッシュをパージする。
*   **キャッシュバスティング**: **フレームワーク管理（Astro/Vite）**
    *   **方式**: ビルド時にファイル内容のハッシュがファイル名に自動付与される（content-based hashing）。
    *   **意図**: カスタムソリューションを避け、フレームワークのベストプラクティスに従う。これにより保守コストがゼロになる。
*   **Service Worker**: **MVP段階では不採用**
    *   **理由**: キャッシュ戦略の設計・デバッグが複雑。CDNキャッシュで十分高速。更新が正しく反映されないトラブルが多い。
    *   **将来的な拡張**: オフライン対応が強く求められた段階で検討。

### 2.12 PWA & Offline [ADR-012]
*   **PWA (Progressive Web App)**: Phase 5（公開時）から**manifest.json**を導入。
    *   **スタンドアロンモード**: ホーム画面追加でフルスクリーン起動可能。ブラウザUIを非表示にし、アプリ風の体験を提供。
    *   **スタンドアロンUI**: `@media (display-mode: standalone)` で最小限の差分を適用（例: ボトムナビゲーション表示）。
    *   **manifest設計**:
        *   **アイコン**: 192x192, 512x512 (maskable icon対応)。
        *   **テーマカラー**: デザインシステム（Open Props）のプライマリカラーと統一。
        *   **インストール促進**: 自動促進バナーは表示しない（ユーザーの自発的な選択を尊重）。
*   **オフライン対応（将来）**: MVP段階では不採用。需要が顕在化した場合は以下の優先順位で検討：
    1.  **PDF/EPUBエクスポート**: ビルド時に自動生成（Pandoc等）。既存リーダーアプリで閲覧可能。低コストで「保存したい」ニーズに対応。
    2.  **PWA + Service Worker**: Web資産を再利用しつつオフライン対応。ただし複雑性が高い。

### 2.13 Image & Media Strategy [ADR-013]
*   **画像フォーマット**:

    | 用途 | ソースフォーマット | 出力フォーマット | 備考 |
    | :--- | :--- | :--- | :--- |
    | 写真・スクリーンショット | JPEG / PNG | **WebP** | Astro `<Image>` で自動変換 |
    | 図解・ダイアグラム | SVG | SVG | ベクター、拡大しても劣化なし |
    | アニメーション | WebP / MP4 | WebP / MP4 | GIFは非推奨（ファイルサイズ大） |
    | OGP画像 | ― | **PNG** | SNS互換性のため |

    *   **フォールバック**: Astroが `<picture>` タグでWebP非対応ブラウザ（iOS 13以前等）向けにJPEG/PNGを自動提供。
*   **画像最適化**: Astro `<Image>` コンポーネントに依存。ビルド時にリサイズ、フォーマット変換、`srcset` 生成、`loading="lazy"` 付与を自動実行。
*   **アイコン戦略**:
    *   **フレームワーク**: **Iconify** (`unplugin-icons`) でビルド時にSVG変換。ランタイム依存なし。使用したアイコンのみバンドル。
    *   **基本UIセット**: **Lucide**（ナビゲーション、操作系アイコン。デザイン一貫性を重視）。
    *   **技術ロゴセット**: **Devicon** or **Simple Icons**（React, TypeScript等のフレームワーク/言語ロゴ）。
*   **OGP画像生成**:
    *   **方式**: **Satori + Resvg** によるビルド時動的生成。
    *   **理由**: 高速（~50ms/枚）、軽量（npmパッケージのみ）、日本語フォント対応、Astro統合実績。Puppeteer（ヘッドレスブラウザ）はビルド時間・依存関係の観点で不採用。
    *   **デザイン**: 統一テンプレート（サイトロゴ + 記事タイトル）。詳細は実装時に決定。
    *   **フォーマット**: PNG（SNS互換性のため）。

### 2.14 Font Strategy [ADR-014]
*   **フォント選定**:

    | 用途 | フォント | 理由 |
    | :--- | :--- | :--- |
    | 本文・見出し（日本語） | **Noto Sans JP** | 多言語対応（Notoファミリー統一）、中立的、可読性◎ |
    | 本文・見出し（欧米語） | **Noto Sans** | Noto Sans JPと統一されたデザイン |
    | コードブロック | **JetBrains Mono** | プログラマーに広く認知、`lI1O0` の判読性◎、リガチャ対応 |

*   **ライセンス**: 全てOFL（Open Font License）。商用利用可、クレジット不要。
*   **フォント配信**:
    *   **方針**: **セルフホスト + サブセット化**。プライバシー保護と安定性・安全性を最優先。Google Fonts APIへの第三者リクエストを排除。
    *   **日本語**: **Noto Sans JP**
        *   **配信戦略**: **Google Fontsの分割配信（unicode-range）技術をセルフホストで再現**。
        *   **理由**:
            *   **パフォーマンス**: 1つの巨大なファイル（1.2MB超）をロードさせる従来方式は、First Paintを著しく遅延させる。Google Fonts方式（数百の小さなチャンクに分割）なら、ページに必要な文字のチャンクのみをダウンロードするため、初期表示が高速化される。
            *   **プライバシー**: ただし、Googleのサーバーには接続せず、これらの分割ファイルを事前にダウンロードして自社CDNから配信する。
        *   **管理**: ビルドツール（`unplugin-fonts`等）を使用し、Google Fontsのリソースをローカルにダウンローダーして配置。
        *   **JIS外対応**: 分割配信により全漢字をカバーできるため、手動サブセットのような「第2水準まで」という制限がなくなる。「豆腐（文字化け）」リスクを最小化。
    *   **欧米語・コード**: **Noto Sans / JetBrains Mono**（セルフホスト）。容量が小さいためそのまま配信。
*   **font-display戦略**: `swap`
    *   **理由**: FOIT（Flash of Invisible Text）を回避し、テキストを即座に表示。「爆速の表示」理念に合致。
    *   **動作**: フォールバックフォントで即座に表示し、Webフォント読込完了後に差し替え。
*   **プリロード**: ファーストビューで使用するフォント（Noto Sans JP Regular）をプリロード。
    *   **効果**: HTMLと同時にフォントダウンロードを開始し、FOUT（Flash of Unstyled Text）を大幅に軽減。2回目以降はブラウザキャッシュによりFOUTゼロ。
    *   **注意**: 本文用Regular（1ファイル）のみプリロード。Bold、コード用フォント等は遅延読み込みで十分。過剰なプリロードは帯域の無駄遣いとなる。
*   **フォールバック設定（MVP）**: `font-family: 'Noto Sans JP', system-ui, sans-serif;`
    *   特殊文字（アラビア語等）が出現した場合はOSのフォントで表示。MVP段階ではこれで十分。
*   **多言語対応時の方針**:
    *   対象言語のNotoフォントを**明示的に読み込む**（デザイン一貫性・表示確実性のため）。
    *   例: `font-family: 'Noto Sans JP', 'Noto Sans', 'Noto Sans SC', 'Noto Sans Arabic', system-ui, sans-serif;`
    *   **理由**: `system-ui` はOS依存で見た目が変わる。Notoファミリーなら全ユーザーで統一された読書体験を提供。「公共図書館」としての信頼性に合致。

### 2.15 Internationalization (i18n) [ADR-015]
*   **多言語対応方針**:
    *   **MVP**: 日本語のみ。ただし多言語化を見据えた設計（URL構造、hreflang対応）を最初から組み込む。
    *   **将来**: 言語切替方式を採用。同一概念の記事に対して複数言語版を用意し、ユーザーがページ内UIで切り替え可能。
    *   **SEO**: `hreflang` タグで言語版間のリンクを検索エンジンに明示。
*   **URLのロケール表記**: パスプレフィックス方式
    *   **形式**: `/ja/slug`（日本語）、`/en/slug`（英語）、`/zh/slug`（中国語）等
    *   **プレフィックスなしURL**:
        *   **MVP**: `/slug` → `/ja/slug` へ301リダイレクト。最初から正規URLを `/ja/...` に統一。
        *   **英語対応後**: `/slug` → `/en/slug` へ301リダイレクトに変更（英語をデフォルト化）。
    *   **理由**: 全言語にプレフィックスを付けることで言語が明確。プレフィックスなし=英語という業界慣習に合わせ、将来的に英語リダイレクトへ移行。
    *   **言語別分析**: URL構造から `/ja/...` と `/en/...` のPVをCloudflare Web Analyticsで個別計測可能。
*   **日付・数値フォーマット**: 言語ごとに固定（SSG時に決定）
    *   **実装**: ビルド時に `Intl.DateTimeFormat` / `Intl.NumberFormat` で言語別フォーマットを生成。
    *   **日本語**: 2025年12月12日 / 1,234,567
    *   **英語**: December 12, 2025 / 1,234,567
    *   **理由**: SSGと相性◎、ハイドレーション不要、同一言語版を見るユーザーは同じ表示。
*   **UI文言管理**: **TypeScript Module (Type-safe JSON)**
    *   **方式**: 言語ごとのTypeScriptモジュール（`src/i18n/messages/ja.ts`, `src/i18n/messages/en.ts`）で型安全なオブジェクトを定義。
    *   **理由**:
        *   依存ゼロ（サステナビリティ）。
        *   `as const` によりキー補完・typo検出が可能（型安全性）。
        *   SSGビルド時にインライン化され、ランタイムオーバーヘッドなし（パフォーマンス）。
    *   **Litコンポーネント連携**: `@lit/context` を通じてメッセージオブジェクトを提供。
    *   **将来的な拡張**: 複雑な文法規則を持つ言語（ポーランド語、アラビア語等）に対応する場合は、**Paraglide.js**（型安全、コンパイル時最適化、ICU MessageFormat対応）への移行を検討。

### 2.16 Error Pages [ADR-016]
*   **404ページ（Not Found）**:
    *   **発生場面**: URLの打ち間違い、記事の削除・移動、外部からのリンク切れ、言語版が存在しない場合。
    *   **設計方針**:
        *   ヘッダー・フッターを維持し、サイトの一部として見える
        *   「お探しのページが見つかりません」と明確に伝える
        *   次のアクションを提示（検索ボックス、ホームへ戻る、人気記事を見る）
        *   GitHub Issuesへのリンク切れ報告を促す
    *   **多言語対応**: MVP: 日本語のみ。将来: 言語別404ページ（`/ja/404`, `/en/404`）。
*   **500ページ（Internal Server Error）**:
    *   **発生頻度**: SSG（静的配信）のため**ほぼ発生しない**。ビルド時にエラーがあればデプロイ自体が失敗する。
    *   **設計方針**: シンプルなエラーメッセージと「ホームへ戻る」リンクのみ。念のため用意。
*   **アクセシビリティ**:
    *   **ページタイトル**: `<title>` にエラー状態を明示（例: 「ページが見つかりません - Isidorica」）。スクリーンリーダーが即座に状況を伝達。
    *   **ランドマーク**: `role="main"` 内にエラーメッセージを配置。
    *   **フォーカス管理**: ページロード時にエラーメッセージ（または検索ボックス）にフォーカスを移動し、ユーザーが迷子状態から即座に次のアクションを取れるようにする。
*   **実装**: `src/pages/404.astro`, `src/pages/500.astro`。Cloudflare Pagesが自動的にエラーレスポンスとして使用。

### 2.17 Image Management [ADR-017]
*   **ストレージ**: **Cloudflare R2**
    *   **理由**: Gitリポジトリの肥大化を防ぎ、CDN配信を最適化。Cloudflareエコシステム内で統一管理。
    *   **リスク**: Cloudflareベンダー集中リスクについては[R-TECH-09](../../shared/RISK_ANALYSIS.md)参照。代替: AWS S3, Backblaze B2等。
*   **2バケット方式**:
    *   `isidorica-originals`: 元画像（バックアップ、プライベート）
    *   `isidorica-assets`: 最適化済み画像（配信、パブリック）
*   **執筆者タイプ別フロー**:
    *   **直接Markdown編集**: 画像を `images/` に配置しPR作成 → CI/CDが自動でR2にアップロード・最適化・パス変換
    *   **Webエディタ使用**: 画像をドラッグ＆ドロップ → 即座にR2にアップロード、Markdownに自動挿入
    *   **共通**: 執筆者はR2やCDNのURLを意識する必要なし
*   **Gitとの分離理由**:
    *   リポジトリ肥大化防止（履歴にバイナリが蓄積しない）
    *   クローン高速化（新規開発者のオンボーディング改善）
*   **コスト**: R2無料枠10GB/月（約20,000記事分）。超過時$0.015/GB/月。
*   **ドメイン**: `assets.isidorica.org`（R2カスタムドメイン）
*   **詳細ドキュメント**: [IMAGE_MANAGEMENT.md](../IMAGE_MANAGEMENT.md)（執筆者ガイド、最適化設定、CI/CD実装）

### 2.18 Laboratory Runtime (実験室) [ADR-018]
*   **アーキテクチャ**: **Client-side WASM (WebAssembly)**
    *   **理由**: サーバーコストゼロ、スケーラビリティ無限大。ユーザーのブラウザで完結するため、プライバシー保護にも貢献。
*   **ランタイム**:

    | 言語 | ランタイム | 用途 | 選定理由 |
    | :--- | :--- | :--- | :--- |
    | **Python** | **Pyodide** | データサイエンス、ML | NumPy/Pandas/Matplotlib対応。事実上唯一の選択肢。 |
    | **JavaScript** | **QuickJS** | 軽量サンドボックス | DOM遮断、~500KB。ShadowRealm未標準化のため採用。 |
    | **R** | **webR** | 統計学、可視化 | R唯一のWASM実装。R Consortium維持。 |
    | **SQL** | **PGlite** | DB操作学習 | PostgreSQL互換。Window関数/CTE対応で実践的。 |

*   **制約と対策**:
    *   **CSP（Content Security Policy）**: `wasm-unsafe-eval` が必要。セキュリティとのトレードオフを認識。
    *   **メインスレッドブロック回避**: **Web Workers** でWASM実行を分離し、UIの応答性を維持。
    *   **初期ロード**: 遅延読み込み（必要になった時点でWASMをダウンロード）。
    *   **パフォーマンスリスク**: モバイル端末での遅延リスクについては[R-TECH-01](../../shared/RISK_ANALYSIS.md)参照。
*   **詳細ドキュメント**: [LABORATORY_DESIGN.md](../../engine/LABORATORY_DESIGN.md)

### 2.19 Discussion Platform (Agora) [ADR-019]
*   **バックエンド**: **GitHub Discussions API**
    *   **理由**: 既存のGitHubインフラを活用。認証、通知、モデレーション機能が無料。オープンソースプロジェクトとの親和性。
    *   **制限**: Phase 1-2はGitHubアカウント必須。Phase 3+でBot Proxyによる匿名投稿を実装（[ADR-005](#25-authentication--persistence-adr-005)参照）。
    *   **リスク**: GitHub API依存リスクについては[R-TECH-08](../../shared/RISK_ANALYSIS.md)参照。
*   **フロントエンド統合**: **GraphQL API + Client-side Hydration**
    *   **API選択**: GraphQL（GitHub's GraphQL API v4）。型安全、クエリ柔軟性、必要なデータのみ取得でパフォーマンス向上。
    *   **レンダリング**: Astro Island (Client-side)。SSGページ内にLitコンポーネントとして埋め込み、読み込み後にGraphQLでデータ取得。
    *   **キャッシュ**: ブラウザキャッシュ + SWR（Stale-While-Revalidate）パターンで、表示速度と鮮度を両立。
*   **構造化議論（Toulminモデル）**:
    *   Frontmatterまたはラベルで議論タイプ（主張、根拠、反論）を分類。
    *   将来的にはカスタムUIで構造化された議論を提示。
*   **ポライトネス支援**: 
    *   投稿前のトーンチェック（将来実装）。
    *   段階的なコミュニティ参加ガイド。
*   **詳細ドキュメント**: [DISCUSSIONS_DESIGN.md](../../library/DISCUSSIONS_DESIGN.md)

### 2.20 Knowledge State Tracking (知識状態追跡) [ADR-020]
*   **ローカル状態管理**: **OPFS + Custom Persistent Signal**
    *   **推奨ストレージ**: Origin Private File System（OPFS）
    *   **フォールバック**: IndexedDB → localStorage（古いブラウザ用）
    *   **データ構造**: `K = {concept_id: status}` 形式で学習済み概念を追跡。
    *   **ステータス値**: `unseen` | `in_progress` | `mastered`
    *   **理由**: Safari ITP対象外（7日削除ルール適用外）、容量制限緩和。プライバシー保護（データはユーザーのブラウザにのみ存在）。
    *   **リスク対策**: ストレージ制限は[R-TECH-06](../../shared/RISK_ANALYSIS.md)、スキーマ変更時のマイグレーションは[R-TECH-07](../../shared/RISK_ANALYSIS.md)を参照。
*   **前提概念の解決**:
    *   各記事のFrontmatterに `prerequisites: [concept_id, ...]` を定義。
    *   ユーザーの知識状態と照合し、未習得の前提があれば警告またはリンクを表示。
*   **将来的な拡張（正式版以降）**:
    *   OAuth認証によるクロスデバイス同期。
    *   学習パス推薦エンジン（前提と興味に基づく次の記事提案）。
*   **詳細ドキュメント**: [LEARNER_STATE_MODEL.md](../../shared/LEARNER_STATE_MODEL.md)

### 2.21 Security [ADR-021]
*   **依存パッケージの脆弱性スキャン**:
    *   **ツール**: `pnpm audit`（ローカル）, **GitHub Dependabot Security Alerts**（自動）
    *   **ポリシー**:
        *   **High/Critical**: 24時間以内に対応（パッチ適用または代替ライブラリへの移行）。
        *   **Medium/Low**: 次回のRenovate PRで対応。
    *   **CI統合**: `pnpm audit --audit-level=high` をCIパイプラインに組み込み、High以上でビルド失敗。
*   **Content Security Policy (CSP)**:
    *   **基本ポリシー**（注釈付き）:
        ```
        default-src 'self';                              # デフォルト: 同一オリジンのみ
        script-src 'self' 'wasm-unsafe-eval';            # JS: 同一オリジン + WASM実行用
        style-src 'self' 'unsafe-inline';                # CSS: Shadow DOMのAdopted Stylesheets用
        img-src 'self' https://assets.isidorica.org data:;  # 画像: CDN + data URI
        font-src 'self';                                 # フォント: セルフホストのみ
        connect-src 'self' https://api.github.com;       # API: Agora用GitHub API
        frame-ancestors 'none';                          # クリックジャッキング防止
        base-uri 'self';                                 # ベースURI固定
        form-action 'self';                              # フォーム送信先制限
        ```
    *   **Laboratory特有**: `wasm-unsafe-eval` はPyodide/QuickJS/webR/PGliteのWASM実行に必須。セキュリティトレードオフを認識。
    *   **unsafe-inline (style-src)**: Shadow DOM内のAdopted Stylesheetsが`unsafe-inline`を要求する場合がある。CSP Level 3の`strict-dynamic`導入を将来検討。
    *   **配信**: Cloudflare Pagesの`_headers`ファイルで設定。
*   **Subresource Integrity (SRI)**:
    *   **現状**: 全てのリソースをセルフホストするため、外部CDNへの依存がなくSRIは不要。
    *   **将来**: 外部スクリプト（Algolia等）を追加する場合は`integrity`属性を必須とする。
*   **その他のセキュリティヘッダー**:
    *   `X-Content-Type-Options: nosniff`
    *   `X-Frame-Options: DENY`
    *   `Referrer-Policy: strict-origin-when-cross-origin`
    *   `Permissions-Policy: geolocation=(), microphone=(), camera=()`
*   **コードレビュー視点**:
    *   XSS: ユーザー入力のサニタイズ（コメント機能対応時）
    *   Prototype Pollution: `Object.freeze`/`Object.seal`の適用
    *   ReDoS: 正規表現の複雑性チェック（textlintルール等）

### 2.22 Backup & Disaster Recovery [ADR-022]
*   **GitHubリポジトリ**:
    *   **ミラーリング**: GitHubの障害時に備え、**GitLab.com**への自動ミラーを設定。
    *   **方法**: GitHub Actionsで`git push --mirror`を毎日実行（またはGitLabのリポジトリミラーリング機能）。
    *   **対象**: `main`ブランチ、タグ、全履歴。
*   **Cloudflare R2画像**:
    *   **バケット構成**:
        *   `isidorica-originals`（プライベート）: 元画像のバックアップ。
        *   `isidorica-assets`（パブリック）: 最適化済み配信用。
    *   **地理的冗長性**: R2は単一リージョン。最重要データのcross-regionバックアップが必要な場合、以下を検討:
        *   Backblaze B2（主要リージョン）への定期同期。
        *   またはAWS S3（us-east-1等）へのクロスリージョンコピー。
    *   **MVP方針**: R2単一で運用開始。記事数が1,000件を超えた段階で冗長化を再検討。
*   **データ復旧手順**:
    *   **GitHub障害**: GitLabミラーからローカルクローン、新リポジトリにpush。
    *   **R2データ損失**: `isidorica-originals`から再最適化・再アップロード。
    *   **Cloudflare Pages障害**: GitHub Actionsでビルド済みアーティファクトを保持、代替ホスティング（Vercel/Netlify）に即時デプロイ可能。
*   **RPO/RTO目標**:
    *   **RPO (Recovery Point Objective)**: 24時間（毎日バックアップ）。
    *   **RTO (Recovery Time Objective)**: 4時間（手動復旧）。
*   **テスト計画**: 年次でミラーリングからの復旧訓練を実施。

---

## 3. 却下された選択肢 (Alternatives Considered)

| 領域 | 候補 | 却下理由 |
| :--- | :--- | :--- |
| **Search** | **Algolia / Meilisearch** | 審査落ちのリスク（Algolia）および高い運用コスト（月額4,500円〜）により、持続可能性と公共性に欠けるため。 |
| **Search** | **Kuromoji.js** | 辞書データ（数MB〜10MB）のロードが必須となり、低帯域環境でのアクセシビリティ（公共性）を著しく損なうため。 |
| **Search** | **Orama** | 日本語用辞書のサイズが大きく、初期ロード性能を損なうため。 |
| **Search** | **BudouX** | フレーズ単位の分割（改行用）であり、意味的な単語検索用トークナイザとしては精度が不十分なため。 |
| **State** | **Framework-specific Signals** | Preact/Solid/Vue等の独自実装は、そのフレームワークの寿命に依存するため。TC39標準仕様（将来のネイティブ実装）に準拠したPolyfillを採用する。 |
| **State** | **Comlink** | RPC（命令型）モデルが、Signals（反応型）中心のアーキテクチャと整合せず、インピーダンスミスマッチを起こすため。 |
| **State** | **RxJS** | バンドルサイズ過大。現状のユースケース（UI同期）に対してオーバーエンジニアリング。 |
| **Style** | **Tailwind CSS** | Shadow DOM内での適用が煩雑であり、Web Componentsとの親和性が低いため。 |
| **Content** | **Astro Content Collections** | 便利だがAstroフレームワークに密結合（ロックイン）しており、データ層の独立性を損なうため。Veliteを採用。 |
| **Content** | **MDX** | エディタ実装の難易度が高く、かつJSエコシステムへの依存度が強すぎるため。 |
| **Test** | **React Testing Library** | Shadow DOMを持つWeb Componentsのテストにおいて、Storybook（実ブラウザ）の方が信頼性が高いため。 |
| **QA** | **remark-lint** | 外部ライブラリ（特に個別のルールプラグイン）の寿命や仕様変更に依存するリスクを排除し、完全な制御権を持つため内製化を選択。 |
| **Offline** | **ネイティブアプリ / クロスプラットフォームアプリ** | 開発・メンテコストが2倍以上。App Store審査の不確実性。「Web標準」「持続可能性」「Universal Access」の理念に反する。URLによるDiscoveryが困難。 |

## Future Considerations (Post-MVP)
*   **Cross-Device Sync**: クロスデバイス同期が必要になった段階で、Clerk/Auth0等のマネージド認証サービスとDB同期の導入を検討する。

