# 非機能要件 (Non-Functional Requirements)

本ドキュメントでは、Isidoricaの非機能要件（パフォーマンス、セキュリティ、可用性等）を定義する。

**ステータス**: 確定
**最終更新**: 2025-12-15

---

## 1. ブラウザ互換性 (Browser Compatibility)

### 1.1 基本方針

Isidoricaは「公共知」として、経済的制約でデバイスを更新できないユーザーを排除しない。
**プログレッシブ・エンハンスメント**を採用し、基本コンテンツは全ブラウザで閲覧可能とする。

### 1.2 サポートレベル

| レベル | 対象ブラウザ | 利用可能な機能 |
| :--- | :--- | :--- |
| **フルサポート** | Chrome/Edge/Firefox/Safari（最新2バージョン） | 全機能（Laboratory含む） |
| **機能制限付き** | Safari 14+、Chrome 88+（Constructable Stylesheets対応） | 基本機能 + 一部Laboratory |
| **基本閲覧のみ** | それ以前のブラウザ | 記事閲覧のみ。Laboratoryは非表示または静的プレビュー |
| **非サポート** | IE11 | 2022年にMicrosoftがサポート終了 |

### 1.3 グレースフル・デグラデーション

| 機能 | モダンブラウザ | レガシーブラウザ |
| :--- | :--- | :--- |
| **記事閲覧** | ✅ 完全対応 | ✅ 完全対応 |
| **検索** | ✅ 完全対応 | ✅ 完全対応 |
| **目次（TOC）** | ✅ 完全対応 | ✅ 完全対応 |
| **Laboratory** | ✅ 完全対応 | ⚠️ 静的プレビュー（スクリーンショット + 説明テキスト） |
| **学習進捗保存** | ✅ LocalStorage | ✅ LocalStorage |

#### Laboratoryの静的プレビュー

レガシーブラウザ（Constructable Stylesheets非対応）では、Laboratoryは技術的に動作しない。
代わりに**静的プレビュー**を表示し、ブラウザの違いによる学習体験の格差を最小化する。

| 要素 | 内容 |
| :--- | :--- |
| **スクリーンショット** | Laboratoryの初期状態または代表的な状態の画像 |
| **説明テキスト** | インタラクションの結果や学習ポイントをテキストで説明 |
| **ブラウザ更新案内** | 「最新のブラウザで完全な体験ができます」という控えめな案内 |

> **詳細**: [LABORATORY_DESIGN.md](./LABORATORY_DESIGN.md) セクション4.5「レガシーブラウザ対応」を参照。

### 1.4 不採用の技術

| 技術 | 不採用の理由 |
| :--- | :--- |
| **View Transitions API** | Isidoricaの「静謐な美しさ」哲学に反する。学習中の注意散漫を避けるため、派手なアニメーションは不要 |
| **過度なアニメーション** | `prefers-reduced-motion` 対応の負担軽減。無ければ対応不要 |
| **CSP nonce/hash** | レガシーブラウザでは静的プレビューを表示するため、Litのフォールバックスタイルは不要 |
| **CSP Reporting** | プライバシー配慮（ユーザーの閲覧URLを収集しない）。SSGでCSP違反リスクは極めて低い |

---

## 2. セキュリティ (Security)

### 2.1 Content Security Policy (CSP)

#### 基本方針

**厳格なCSPを採用**し、問題が発生した場合にのみ最低限緩める方針とする。

#### CSP設定

```
default-src 'self';
script-src 'self';
style-src 'self';
img-src 'self' data:;
font-src 'self';
connect-src 'self';
worker-src 'self';
frame-src 'none';
object-src 'none';
base-uri 'self';
form-action 'self';
frame-ancestors 'none';
upgrade-insecure-requests;
```

#### 各ディレクティブの説明

| ディレクティブ | 設定 | 理由 |
| :--- | :--- | :--- |
| **default-src** | `'self'` | フォールバック。同一オリジンのみ許可 |
| **script-src** | `'self'` | 同一オリジンのスクリプトのみ。インラインスクリプトは禁止 |
| **style-src** | `'self'` | 同一オリジンのみ。フォントはセルフホスト |
| **img-src** | `'self' data:` | 同一オリジン + data URI（インラインSVG等） |
| **font-src** | `'self'` | 同一オリジンのみ。フォントはセルフホスト |
| **connect-src** | `'self'` | 同一オリジンのみ（Cloudflare Web Analyticsは自動統合） |
| **worker-src** | `'self'` | Web Worker（Three.js, MapLibre等のWebGLライブラリ用） |
| **frame-src** | `'none'` | iframeは禁止 |
| **object-src** | `'none'` | Flash等のプラグインは禁止 |
| **base-uri** | `'self'` | base要素のURLを同一オリジンに制限 |
| **form-action** | `'self'` | フォーム送信先を同一オリジンに制限 |
| **frame-ancestors** | `'none'` | 他サイトからのiframe埋め込みを禁止 |
| **upgrade-insecure-requests** | - | HTTPリクエストをHTTPSにアップグレード |

#### インラインスクリプト/スタイルの扱い

| 要素 | 対応 |
| :--- | :--- |
| **インラインスクリプト** | 使用禁止。外部ファイル化またはnonce/hash使用 |
| **インラインスタイル** | Lit Web Componentsが生成する場合はnonce使用を検討 |

#### Astroでの実装

```typescript
// astro.config.mjs
export default defineConfig({
  // Cloudflare Pagesのヘッダー設定で管理
  // または _headers ファイルで設定
});
```

```
# public/_headers
/*
  Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; worker-src 'self'; frame-src 'none'; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests;
```

#### 将来の検討事項

CSPに関する将来の検討事項（Laboratory WASM対応、Plausible追加等）は [PROJECT_ROADMAP.md](../shared/PROJECT_ROADMAP.md) を参照。

### 2.2 Subresource Integrity (SRI)

#### 概要

SRIは外部リソースが改ざんされていないことを検証する仕組み。

#### 対応方針

| リソース | 対応 | 備考 |
| :--- | :--- | :--- |
| **自サイトのスクリプト/CSS** | 不要 | 同一オリジン、Cloudflare Pagesで保護 |
| **フォント** | 不要 | セルフホスト。外部リクエストなし |
| **Plausible** | セルフホスト検討 | プライバシー向上のため |

*   **方針**: 外部CDNからのリソース読み込みは原則禁止。必要な場合はセルフホストする。
*   **フォント**: Google Fonts APIを使用せず、セルフホスト + サブセット化を採用。詳細は [ARCHITECTURE_DECISION_RECORD.md](./technical/ARCHITECTURE_DECISION_RECORD.md) を参照。

### 2.3 その他のセキュリティヘッダー

| ヘッダー | 設定 | 説明 |
| :--- | :--- | :--- |
| **Strict-Transport-Security** | `max-age=63072000; includeSubDomains` | HSTS。ブラウザにHTTPS接続を強制（2年間） |
| **X-Content-Type-Options** | `nosniff` | MIMEタイプスニッフィングを防止 |
| **X-Frame-Options** | `DENY` | クリックジャッキング防止（CSP frame-ancestorsと併用） |
| **Referrer-Policy** | `strict-origin-when-cross-origin` | リファラー情報の制御 |
| **Permissions-Policy** | 下記参照 | 使用しないブラウザAPIを無効化 |
| **Cross-Origin-Opener-Policy** | `same-origin` | Spectre等のサイドチャネル攻撃への耐性向上 |

#### Permissions-Policy 詳細

以下のAPIを明示的に無効化する。

| API | 無効化理由 |
| :--- | :--- |
| `geolocation` | 位置情報は使用しない |
| `microphone` | マイクは使用しない |
| `camera` | カメラは使用しない |
| `payment` | 決済機能は使用しない |
| `usb` | USB接続は使用しない |
| `magnetometer` | 磁気センサーは使用しない |

#### Cloudflare Pagesでの設定

```
# public/_headers
/*
  Strict-Transport-Security: max-age=63072000; includeSubDomains
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: geolocation=(), microphone=(), camera=(), payment=(), usb=(), magnetometer=()
  Cross-Origin-Opener-Policy: same-origin
```

#### 将来の検討事項

セキュリティヘッダーに関する将来の検討事項（HSTS preload登録等）は [PROJECT_ROADMAP.md](../shared/PROJECT_ROADMAP.md) を参照。

### 2.4 依存関係の脆弱性管理

#### 管理構成

| レイヤー | ツール | タイミング |
| :--- | :--- | :--- |
| **常時監視** | GitHub Security Alerts | 常時（脆弱性発見時にアラート） |
| **更新管理** | Renovate / Dependabot | 新バージョンリリース時（PRを自動作成） |
| **ビルド時チェック** | `pnpm audit` | CI実行時 |

#### GitHub Security Alerts

*   リポジトリ設定で有効化。
*   GitHub Advisory Databaseと連携し、依存関係の脆弱性を自動検出。
*   脆弱性が発見されるとアラートを表示。

#### Renovate / Dependabot

*   依存関係の自動更新PRを作成。
*   セキュリティアップデートを優先的に処理。

#### pnpm audit（CI組み込み）

```yaml
# .github/workflows/ci.yml
- name: Security audit
  run: pnpm audit --audit-level=high
```

*   **動作**: 高リスク（high）以上の脆弱性があればビルド失敗。
*   **目的**: 脆弱性を含むコードがデプロイされることを防止。

---

## 3. パフォーマンス (Performance)

### 3.1 目標値

| 指標 | 目標 | 備考 |
| :--- | :--- | :--- |
| **Lighthouse Performance** | 90+ | モバイル環境で測定 |
| **Lighthouse Accessibility** | 100 | 必須（下記注記参照） |
| **First Contentful Paint (FCP)** | < 1.8s | |
| **Largest Contentful Paint (LCP)** | < 2.5s | Core Web Vitals |
| **Cumulative Layout Shift (CLS)** | < 0.1 | Core Web Vitals |
| **Interaction to Next Paint (INP)** | < 200ms | Core Web Vitals（FIDの後継） |
| **Total Blocking Time (TBT)** | < 200ms | TTIの代替（Lighthouse 10でTTIは廃止） |
| **Time to First Byte (TTFB)** | < 100ms | Cloudflare Pagesのエッジキャッシュで達成 |

#### Lighthouse Accessibility についての注記

Lighthouseの自動チェックで検出できるのはアクセシビリティ問題の約30〜50%と言われている。
**100点はスタートライン**であり、以下の手動テストを併用すること。

- キーボード操作の実機確認
- スクリーンリーダー（VoiceOver, NVDA等）での確認
- 色見シミュレータでの確認

### 3.2 最適化戦略

#### 画像最適化

| 戦略 | 実装 |
| :--- | :--- |
| **フォーマット** | WebP/AVIF（ブラウザ対応に応じて） |
| **遅延読み込み** | `loading="lazy"` 属性 |
| **適切なサイズ** | `srcset` と `sizes` で viewport に合わせた画像配信 |
| **プレースホルダー** | LQIP（Low Quality Image Placeholder）またはドミナントカラー |

#### JavaScript最適化

| 戦略 | 実装 |
| :--- | :--- |
| **コード分割** | Astroのアイランドアーキテクチャで自動分割 |
| **Tree Shaking** | Rollupによる未使用コード削除 |
| **遅延読み込み** | Laboratoryは `client:visible` で viewport 進入時に読み込み |
| **ミニファイ** | ビルド時に自動実行 |

#### CSS最適化

| 戦略 | 実装 |
| :--- | :--- |
| **クリティカルCSS** | Astroが自動的にインライン化 |
| **未使用CSS削除** | ビルド時に自動実行 |
| **圧縮** | Gzip / Brotli（Cloudflareで自動適用） |

#### フォント最適化

| 戦略 | 実装 |
| :--- | :--- |
| **セルフホスト** | Google Fonts APIではなく、@fontsource を使用 |
| **最適化配信** | Google Fontsの分割配信（unicode-range）技術を利用し、必要な文字のみをロード。ただし完全セルフホスト化し外部リクエストは発生させない。 |
| **font-display** | `swap` を使用（FOIT回避） |
| **プリロード** | 日本語フォントをプリロード |

> **詳細**: [ARCHITECTURE_DECISION_RECORD.md](./technical/ARCHITECTURE_DECISION_RECORD.md) セクション2.4を参照。

### 3.3 測定方法

#### Labデータ（開発時）

| ツール | 用途 | タイミング | 詳細 |
| :--- | :--- | :--- | :--- |
| **Lighthouse CI** | CI/CD でのパフォーマンス計測 | PRごと | `.github/workflows/ci.yml` で設定 |
| **PageSpeed Insights** | 手動での詳細分析 | リリース前 | [pagespeed.web.dev](https://pagespeed.web.dev/) |

##### Lighthouse CI 設定概要

```yaml
# .github/workflows/ci.yml (抜粋)
- name: Lighthouse CI
  uses: treosh/lighthouse-ci-action@v10
  with:
    urls: |
      https://isidorica.org/
      https://isidorica.org/articles/sample
    budgetPath: ./lighthouse-budget.json
    uploadArtifacts: true
```

```json
// lighthouse-budget.json
{
  "ci": {
    "assert": {
      "assertions": {
        "categories:performance": ["error", {"minScore": 0.9}],
        "categories:accessibility": ["error", {"minScore": 1}],
        "first-contentful-paint": ["warn", {"maxNumericValue": 1800}],
        "largest-contentful-paint": ["warn", {"maxNumericValue": 2000}]
      }
    }
  }
}
```

#### Fieldデータ（運用時）

| ツール | 用途 | タイミング | 詳細 |
| :--- | :--- | :--- | :--- |
| **Cloudflare Web Analytics** | 本番環境の Core Web Vitals | 常時 | Cloudflareダッシュボードで自動有効（Cookieなし） |
| **Google Search Console** | CrUXデータ（実ユーザー） | 月次 | サイトのURLプロパティを登録後、Core Web Vitalsレポートで確認 |

##### Cloudflare Web Analytics

- **設定**: Cloudflare Pagesにデプロイすると自動的に有効化
- **ダッシュボード**: Cloudflareダッシュボード → Analytics & Logs → Web Analytics
- **取得データ**: PV、訪問者数、滞在時間、Core Web Vitals（LCP, INP, CLS）
- **プライバシー**: Cookieなし、IPアドレス非保存（GDPR/CCPA準拠）

##### Google Search Console

- **設定**: [search.google.com/search-console](https://search.google.com/search-console) でURLプロパティを登録
- **レポート**: エクスペリエンス → Core Web Vitals
- **データ源**: Chrome User Experience Report (CrUX) - 実際のChromeユーザーの匿名化されたデータ
- **プライバシー**: サイト側でデータ収集は行わない。GoogleがChromeブラウザから収集した匿名データを閲覧するのみ

#### 早期警告

Lighthouse CIで以下の閾値を超えた場合、警告を表示する。

| 指標 | 警告閾値 | 目標値 | 備考 |
| :--- | :--- | :--- | :--- |
| **LCP** | > 2.0s | < 2.5s | 0.5sのバッファ |
| **Performance** | < 95 | 90+ | 5点のバッファ |

---

## 4. 可用性 (Availability)

### 4.1 目標値

| 指標 | 目標 | 備考 |
| :--- | :--- | :--- |
| **アップタイム** | 99.9% | 年間ダウンタイム約8.76時間以内 |
| **計画メンテナンス** | 0時間 | SSGのため、デプロイ時のダウンタイムなし |
| **復旧時間目標 (RTO)** | < 5分 | ロールバックによる即時復旧 |
| **復旧時点目標 (RPO)** | 0 | 静的サイトのため、データ損失なし |

### 4.2 インフラ構成

#### Cloudflare Pages

| 特性 | 説明 |
| :--- | :--- |
| **グローバルCDN** | 世界200以上のデータセンターでエッジ配信 |
| **自動スケーリング** | トラフィック増加時も自動対応（設定不要） |
| **DDoS防御** | Cloudflareのエンタープライズグレード保護 |
| **SSL/TLS** | 自動的にHTTPS化 |

#### SSGの利点

| 利点 | 説明 |
| :--- | :--- |
| **サーバー障害リスクなし** | ビルド済みHTMLをCDNから配信 |
| **データベース依存なし** | 単一障害点がない |
| **デプロイ時ダウンタイムなし** | 新バージョンが完全に準備できてから切り替え |

### 4.3 障害対応

#### ロールバック手順

1. **検出**: Cloudflare Web Analyticsまたはユーザー報告で異常を検出
2. **判断**: 障害の影響範囲を確認
3. **ロールバック**: Cloudflare Pagesダッシュボードから以前のデプロイを選択し「Rollback」
4. **復旧確認**: サイトが正常に表示されることを確認
5. **原因調査**: ロールバック後に原因を調査し、修正版を再デプロイ

> **詳細**: [GOVERNANCE.md](../library/GOVERNANCE.md) セクション4.5参照

#### 障害レベル定義

| レベル | 定義 | 対応時間 |
| :--- | :--- | :--- |
| **Critical** | サイト全体がダウン | 5分以内にロールバック |
| **High** | 主要機能（検索、記事表示）が動作しない | 1時間以内に対応 |
| **Medium** | 一部機能（Laboratory等）が動作しない | 24時間以内に対応 |
| **Low** | 軽微な表示崩れ等 | 次回リリースで対応 |

### 4.4 監視

| 項目 | 方法 | 頻度 |
| :--- | :--- | :--- |
| **サイト死活監視** | Cloudflare Healthchecks または外部サービス（UptimeRobot等） | 1分間隔 |
| **パフォーマンス監視** | Cloudflare Web Analytics | 常時 |
| **エラー監視** | Cloudflare Analytics（4xx/5xxエラー率） | 常時 |

#### 障害通知

| 方法 | 対象 |
| :--- | :--- |
| **メール通知** | 管理者 |
| **Slack/Discord連携** | 運用チーム（将来） |

### 4.5 バックアップ

SSGアーキテクチャのため、従来のデータベースバックアップは不要。

| コンポーネント | バックアップ方法 |
| :--- | :--- |
| **記事・コード** | GitHub（全履歴を保存） |
| **画像** | Cloudflare R2（元画像を `isidorica-originals` に保存） |
| **静的サイト** | Cloudflare Pages（デプロイ履歴からロールバック可能） |

---

## 5. アクセシビリティ (Accessibility)

### 5.1 準拠基準

「公共知」としての性格上、可能な限り高いアクセシビリティを追求する。

| 基準 | レベル | 対応 |
| :--- | :--- | :--- |
| **WCAG 2.2 A** | 全項目 | 必須 |
| **WCAG 2.2 AA** | 全項目 | 必須 |
| **WCAG 2.2 AAA** | 達成可能な項目 | 推奨 |

#### AAA達成を推奨する項目

以下のAAA基準は、Isidoricaのコンテンツ特性上、達成可能であり推奨する。

| 基準 | 内容 | 備考 |
| :--- | :--- | :--- |
| **1.4.6 コントラスト（拡張）** | テキストのコントラスト比7:1以上 | 可能な範囲で |
| **2.4.9 リンクの目的（リンクのみ）** | リンクテキストだけで目的が分かる | 必須に近い |
| **2.4.10 セクション見出し** | コンテンツにセクション見出しを付ける | 既に実践 |
| **3.2.5 要求による変化** | ユーザーが要求しない限りコンテキストを変更しない | 実践 |

#### AAA達成が困難な項目（適用除外）

以下のAAA基準は、学術コンテンツの性質上、全面的な達成が困難であるため適用除外とする。

| 基準 | 内容 | 除外理由 |
| :--- | :--- | :--- |
| **3.1.5 読解レベル** | 中学2年生レベル以下の読解力で理解可能 | 高度な学術コンテンツは簡略化できない |
| **1.2.6 手話（収録済み）** | 音声コンテンツに手話通訳を提供 | 全動画に手話を付けることは現実的でない |

### 5.2 対応項目

#### 必須対応（AA）

| 項目 | 要件 | 実装 |
| :--- | :--- | :--- |
| **カラーコントラスト** | 4.5:1以上（通常テキスト）、3:1以上（大きなテキスト） | CSS変数で管理 |
| **キーボード操作** | 全機能がキーボードで操作可能 | フォーカス管理、tabindex |
| **フォーカス表示** | フォーカス状態が視認可能 | `outline: none` 単独使用禁止 |
| **スクリーンリーダー** | 適切なaria属性、セマンティックHTML | VoiceOver/NVDAでテスト |
| **モーション** | `prefers-reduced-motion` 対応 | アニメーション無効化/簡略化 |
| **画像代替テキスト** | 全ての意味のある画像にalt属性 | 装飾には空alt |
| **見出し構造** | 論理的な見出しレベル（h1→h2→h3） | ページ内にh1は1つ |

#### テスト方法

| 方法 | ツール | タイミング |
| :--- | :--- | :--- |
| **自動テスト** | Lighthouse Accessibility、axe-core | CI/CDでPRごと |
| **手動テスト（キーボード）** | 実機確認 | リリース前 |
| **手動テスト（スクリーンリーダー）** | VoiceOver (Mac/iOS)、NVDA (Windows) | リリース前 |
| **手動テスト（色覚）** | Chrome DevTools 色覚シミュレータ | リリース前 |

> **詳細ガイドライン**: [ACCESSIBILITY_GUIDELINES.md](./ACCESSIBILITY_GUIDELINES.md)（未作成）

## 6. 保守性 (Maintainability)

「100年続くサービス」を目指すため、長期的な保守性を重視する。

### 6.1 コード品質

#### 目標値

| 指標 | 目標 | 備考 |
| :--- | :--- | :--- |
| **TypeScriptカバレッジ** | 100% | `any` 型は禁止。`unknown` + 型ガードを使用 |
| **テストカバレッジ** | 80%以上 | ロジックのみ。UIコンポーネントはStorybookでカバー |
| **Lintエラー** | 0 | CIでブロック |
| **ビルド成功率** | 100% | `main` ブランチは常にビルド可能 |
| **循環参照** | 0 | Biome `noImportCycles` で検出 |

#### コーディング規約

| 項目 | 規約 | 理由 |
| :--- | :--- | :--- |
| **命名規則** | camelCase（変数/関数）、PascalCase（型/クラス/コンポーネント） | 一貫性 |
| **ファイル命名** | kebab-case.tsx | ファイルシステムの互換性 |
| **コメント** | 日本語で記述 | GEMINI.md参照 |
| **`any` 型** | 禁止 | `unknown` + 型ガードを使用 |
| **`useMemo`/`useCallback`/`React.memo`** | 禁止 | React Compilerが自動メモ化 |
| **`outline: none` 単独使用** | 禁止 | アクセシビリティ違反 |

> **詳細**: [CONTRIBUTING.md](./CONTRIBUTING.md) を参照

### 6.2 テスト戦略

#### テストピラミッド

```
        ┌───────────┐
        │   E2E     │  少量：重要なユーザーフロー
        ├───────────┤
        │ Component │  中量：UIコンポーネント
        ├───────────┤
        │   Unit    │  多量：ロジック、ユーティリティ
        └───────────┘
```

#### テスト種別

| 種別 | ツール | 対象 | タイミング |
| :--- | :--- | :--- | :--- |
| **Unit** | Vitest | ユーティリティ関数、hooks、Transformer | PRごと |
| **Component** | Storybook Interaction Testing | UIコンポーネント（UI, A11y, Events） | PRごと |
| **Visual Regression** | Storybook + Chromatic | UIコンポーネントの見た目 | PRごと |
| **E2E** | Playwright | 重要なユーザーフロー、ルーティング、検索 | リリース前 |
| **Accessibility** | axe-core、Lighthouse | 全ページ | PRごと |
| **Performance** | Lighthouse CI | 全ページ | PRごと |

#### テストカバレッジ対象

| 対象 | カバレッジ目標 | テスト方法 |
| :--- | :--- | :--- |
| **ユーティリティ関数** | 90%以上 | Unit (Vitest) |
| **カスタムhooks** | 90%以上 | Unit (Vitest) |
| **Transformer** | 90%以上 | Unit (Vitest) |
| **コンポーネント** | - | Component (Storybook) + Visual Regression |
| **ページ** | - | E2E (Playwright) |

### 6.3 ドキュメント基準

| 要件 | 内容 | 強制 |
| :--- | :--- | :--- |
| **README** | リポジトリルートに必須。セットアップ手順を含む | ✅ 必須 |
| **APIドキュメント** | 全ての公開APIにJSDoc | ✅ 必須 |
| **ADR** | 技術的な意思決定は記録 | ✅ 必須 |
| **CHANGELOG** | リリース毎に更新（Conventional Commits準拠） | ✅ 必須 |
| **Storybook** | 全UIコンポーネントにストーリー | ✅ 必須 |
| **コンポーネントJSDoc** | Props、使用例を記載 | ⚪ 推奨 |

### 6.4 依存関係管理

#### 更新方針

| 項目 | 方針 | 備考 |
| :--- | :--- | :--- |
| **パッチ/マイナー更新** | Renovateで自動マージ（テスト通過時） | 週次でまとめてマージ |
| **メジャーバージョンアップ** | 手動で検証後に適用 | Breaking Changesを確認 |
| **セキュリティアップデート** | 1週間以内に適用 | GitHub Security Alertsで検出 |

#### 新規パッケージ追加

| ステップ | 内容 |
| :--- | :--- |
| 1. **必要性検証** | 既存のユーティリティで代替できないか確認 |
| 2. **品質チェック** | メンテナンス状況、ダウンロード数、ライセンス確認 |
| 3. **サイズ確認** | バンドルサイズへの影響を測定 |
| 4. **承認** | PRで承認を得てから追加 |

#### 禁止事項

| 禁止 | 理由 |
| :--- | :--- |
| **承認なしのパッケージ追加** | セキュリティリスク、バンドルサイズ増加 |
| **`@types/*` を dependencies に配置** | devDependencies に配置すること |

### 6.5 CI/CD パイプライン

#### PRごとの自動チェック

| チェック | ツール | 失敗時の対応 |
| :--- | :--- | :--- |
| **Format & Lint** | Biome | PRブロック |
| **Accessibility Lint** | ESLint (`eslint-plugin-lit-a11y`) | PRブロック |
| **型チェック** | TypeScript | PRブロック |
| **Unit Test** | Vitest | PRブロック |
| **Component Test** | Storybook Interaction Testing | PRブロック |
| **ビルド** | Astro | PRブロック |
| **Accessibility** | Lighthouse CI | 警告（PRブロックしない） |
| **Performance** | Lighthouse CI | 警告（目標値未達時） |
| **セキュリティ** | pnpm audit | PRブロック（high以上） |

#### デプロイフロー

```
main へのマージ
    ↓
GitHub Actions
    ↓
Cloudflare Pages （自動デプロイ）
    ↓
プレビュー URL で確認
    ↓
Production へ自動昇格
```

> **詳細**: ワークフローファイルの設定例、ローカルでのCIチェック実行方法、ロールバック手順は [CONTRIBUTING.md](./CONTRIBUTING.md) セクション10を参照。

### 6.6 技術的負債管理

#### 負債の分類と対応

| レベル | 定義 | 対応期限 |
| :--- | :--- | :--- |
| **Critical** | セキュリティリスク、本番障害の可能性 | 1週間以内 |
| **High** | パフォーマンス低下、開発効率の著しい低下 | 1ヶ月以内 |
| **Medium** | コード品質の問題、リファクタリング対象 | 四半期以内 |
| **Low** | 改善の余地があるが緊急性なし | バックログに登録 |

#### 定期的な負債レビュー

| 頻度 | 内容 |
| :--- | :--- |
| **週次** | 新規追加された `// TODO` コメントの確認 |
| **月次** | 依存パッケージの更新状況確認 |
| **四半期** | 技術的負債の棚卸し、優先度見直し |

---

## 7. 国際化 (Internationalization)

### 7.1 現状

| 項目 | 状態 |
| :--- | :--- |
| **サポート言語** | 日本語のみ（MVP） |
| **文字エンコーディング** | UTF-8 |
| **言語検出** | URL構造から決定（`/ja/...`, `/en/...`） |

### 7.2 多言語化への準備

MVPは日本語のみだが、将来の多言語化を見据えて以下の設計を採用している。

| 項目 | 設計 |
| :--- | :--- |
| **URL構造** | パスプレフィックス方式（`/ja/slug`, `/en/slug`） |
| **デフォルト言語** | MVP: `/slug` → `/ja/slug` へ301リダイレクト。英語対応後は `/en/slug` へ変更 |
| **SEO** | `hreflang` タグで言語版間のリンクを検索エンジンに明示 |
| **フォント** | Notoファミリーで統一（多言語でデザイン一貫性） |
| **日付・数値** | `Intl.DateTimeFormat` / `Intl.NumberFormat` でSSG時に言語別フォーマット |

> **詳細**: [ARCHITECTURE_DECISION_RECORD.md](./technical/ARCHITECTURE_DECISION_RECORD.md) セクション2.15を参照。

### 7.3 翻訳ガバナンス

#### 基本方針

| 項目 | 方針 |
| :--- | :--- |
| **オリジナル言語** | 記事ごとにFrontmatterで宣言（`original_language: ja`） |
| **翻訳版の品質** | オリジナル版に基づく。独自の出典追加・削除は禁止 |
| **翻訳者の裁量** | 補足説明の追加、ページ番号の調整は許可 |
| **執筆言語の自由** | 任意の言語で新規執筆可能。最初に書かれた言語がオリジナル |

#### 機械翻訳の使用

| 項目 | 方針 |
| :--- | :--- |
| **使用可否** | 許可（禁止も推奨もしない） |
| **レビュー** | 機械翻訳を使用した場合も、レビューを経て公開する |
| **開示義務** | 機械翻訳の使用有無を開示する義務はない（判別困難なため） |

#### 翻訳版のレビュー要件

| 方法 | 説明 |
| :--- | :--- |
| **ネイティブレビュワー** | その言語のネイティブスピーカーがレビュー |
| **言語堪能者** | ネイティブでなくても十分な言語力を持つ者がレビュー |
| **AI判定** | AIが翻訳品質をチェック（補助、最終責任は人間） |

> **詳細**: [GOVERNANCE.md](../library/GOVERNANCE.md) セクション5-6を参照。

### 7.4 将来の検討事項

国際化に関する将来の検討事項（英語対応、RTL言語、言語別404ページ等）は [PROJECT_ROADMAP.md](../shared/PROJECT_ROADMAP.md) を参照。

---

## 8. スケーラビリティ (Scalability)

SSG（静的サイト生成）アーキテクチャに基づくスケーラビリティ特性を定義する。

### 8.1 現状のアーキテクチャ特性

#### トラフィックスケーラビリティ

| 項目 | 特性 | 備考 |
| :--- | :--- | :--- |
| **配信能力** | 無制限 | Cloudflare CDNが自動スケール |
| **同時接続数** | 無制限 | 静的ファイル配信のため制約なし |
| **地理的分散** | 世界200以上のPoP | エッジキャッシュによる低レイテンシ |
| **DDoS耐性** | 高 | Cloudflareのエンタープライズグレード保護 |

SSGの最大の利点は、**トラフィック増加に対してサーバーリソースを増やす必要がない**こと。
Cloudflare Pagesの無料プランでも帯域幅無制限（Fair Use Policy適用）。

#### コンテンツスケーラビリティ

| 項目 | 特性 | 備考 |
| :--- | :--- | :--- |
| **記事数上限** | なし（ビルド時間に依存） | ファイルシステムの制約のみ |
| **ビルド方式** | 全記事を毎回ビルド | Astro標準動作 |
| **増分ビルド** | 未対応 | Astroでの正式対応を待つ |

#### 検索スケーラビリティ

| 項目 | 特性 | 備考 |
| :--- | :--- | :--- |
| **検索エンジン** | Algolia DocSearch | サーバーサイド検索。クライアント負荷なし |
| **フォールバック** | Meilisearch（セルフホスト） | Algolia審査落ち時の代替 |
| **インデックスサイズ** | Algolia側で管理 | クライアントにダウンロード不要 |
| **検索速度** | 高速（サーバー処理） | 日本語形態素解析対応 |
| **スケーラビリティ** | Algoliaが自動スケール | 記事数増加に対応 |

#### ストレージスケーラビリティ

| 項目 | 特性 | 備考 |
| :--- | :--- | :--- |
| **静的ファイル（HTML/CSS/JS）** | 無制限 | Cloudflare Pagesの制約なし |
| **画像** | Cloudflare R2で管理 | Gitリポジトリには含めない |
| **Gitリポジトリ** | 軽量を維持 | 画像URL参照のみを含む |

#### 画像管理アーキテクチャ

画像はGitリポジトリに含めず、**Cloudflare R2**で一元管理する。

##### 2バケット方式

| バケット | 用途 | アクセス |
| :--- | :--- | :--- |
| `isidorica-originals` | 元画像（バックアップ） | プライベート |
| `isidorica-assets` | 最適化済み画像（配信） | パブリック |

##### 執筆者タイプ別フロー

| 執筆者タイプ | 操作 | システム処理 |
| :--- | :--- | :--- |
| **直接Markdown編集** | 画像を `images/` に配置、PRを作成 | CI/CDが自動でR2にアップロード・最適化・パス変換 |
| **Webエディタ使用** | 画像をドラッグ＆ドロップ | 即座にR2にアップロード、Markdownに自動挿入 |

**共通**: 執筆者はR2やCDNのURLを意識する必要なし。

##### ストレージ見積もり

| 記事数 | 最適化後画像サイズ | 備考 |
| :--- | :--- | :--- |
| 1,000 | 約500MB | R2無料枠内 |
| 20,000 | 約10GB | R2無料枠上限 |

> **詳細**: 執筆者向けガイド、最適化設定、CI/CD実装は [IMAGE_MANAGEMENT.md](./IMAGE_MANAGEMENT.md) を参照。

### 8.2 スケーラビリティの閾値と監視

#### ビルド時間

| 状態 | ビルド時間 | 記事数目安 | 対応 |
| :--- | :--- | :--- | :--- |
| **正常** | < 3分 | 〜1,000記事 | 対応不要 |
| **注意** | 3〜10分 | 1,000〜5,000記事 | 監視継続、最適化検討 |
| **警告** | 10〜30分 | 5,000〜20,000記事 | 増分ビルド導入を検討 |
| **危険** | > 30分 | 20,000記事以上 | アーキテクチャ転換を検討 |

> **注**: 画像をR2で管理することで、ビルド時間への画像最適化の影響を削減。

#### Gitリポジトリサイズ

画像をGitに含めないため、リポジトリサイズは小さく維持される。

| 状態 | サイズ | 対応 |
| :--- | :--- | :--- |
| **正常** | < 100MB | 対応不要（画像なしの想定） |
| **注意** | 100MB〜500MB | 不要ファイルの確認 |
| **警告** | > 500MB | 画像がGitに混入していないか確認 |

### 8.3 スケーラビリティ向上策

#### 短期的対策（現アーキテクチャ維持）

| 対策 | 効果 | 実装コスト |
| :--- | :--- | :--- |
| **画像の外部化** | Gitサイズ削減、ビルド時間短縮 | 低 |
| **キャッシュ最適化** | 再ビルド時間短縮 | 低 |
| **並列ビルド** | ビルド時間短縮 | 中 |
| **Astro増分ビルド** | 変更記事のみ再ビルド | リリース待ち（Astro 5.0再設計中） |

#### 長期的対策（アーキテクチャ転換）

| 対策 | 適用条件 | 実装コスト |
| :--- | :--- | :--- |
| **データベース導入** | 記事数10万件以上、ビルド時間30分超 | 高 |
| **ハイブリッドSSG/SSR** | 動的コンテンツ増加時 | 高 |
| **分散ビルド** | 複数リポジトリでの並列ビルド | 高 |

> **詳細**: アーキテクチャ転換の具体的な検討事項は [PROJECT_ROADMAP.md](../shared/PROJECT_ROADMAP.md) を参照。

### 8.4 現時点での結論

| 項目 | 結論 |
| :--- | :--- |
| **MVP〜1,000記事** | 現アーキテクチャで問題なし |
| **監視対象** | ビルド時間、Gitサイズ |
| **転換の判断基準** | ビルド時間30分超、またはDXの著しい低下 |

---

## 9. 関連ドキュメント

| ドキュメント | 説明 |
| :--- | :--- |
| [GOVERNANCE.md](../library/GOVERNANCE.md) | コンテンツガバナンス（ロールバック戦略含む） |
| [ARCHITECTURE_DECISION_RECORD.md](./technical/ARCHITECTURE_DECISION_RECORD.md) | 技術選定 |
| [LEGAL_REQUIREMENTS.md](../library/LEGAL_REQUIREMENTS.md) | 法的要件（プライバシー等） |
| [KPI_FRAMEWORK.md](../shared/KPI_FRAMEWORK.md) | 成功指標（パフォーマンス目標との関連） |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | コントリビューターガイド |
